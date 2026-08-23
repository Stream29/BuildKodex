# Task Tree

- 优化 History 一级 item 的测量与内容加载热路径
  - 将结构测量与 history payload 读取解耦
    - `WorkGroup` 首轮测量保持一行且不读取 child payload
    - `Reasoning`、普通 `Tool`、`Patch` 在折叠态或单项态直接使用一行骨架
    - Message、展开项和确实依赖内容的 item 保留异步读取与真实文本测量
  - 移除 History payload 读取的全局并发限制
    - 删除 `Semaphore(8)` 及其排队逻辑
    - 保留 item 状态稳定性，避免重组重复启动读取任务
    - 保留读取任务的生命周期取消和错误 fallback
  - 降低可见区重建时的 Compose 测量成本
    - 确保一行骨架不进入完整 event render 和长文本布局
    - 保持已加载 item 状态与 LazyColumn key 稳定
    - 检查展开、折叠、session 切换和 marker 更新不会重复解析稳定 payload
  - 补充热路径回归与性能验证
    - 验证大量 WorkGroup、最新未折叠 foldable item 和 singleton foldable item
    - 验证长 Message 不阻塞一行 item 的首次显示
    - 记录可见区重建、首轮测量、payload 读取和文本布局耗时
    - 构建 release binary 并使用真实长历史 session 端到端验证
  - 用户确认完整计划后再进入 executable

# Details

- 状态：`planning`。本任务只规划 History item 测量和读取并发优化，不改变 folding、turn time marker 或 History timeline 的语义。
- 当前问题：
  - 已折叠的 `WorkGroup` 本身已经可以只渲染一行，但最新 open foldable run 和 singleton foldable item 仍按普通 entry 读取完整 event。
  - `StoredHistoryContent` 当前在有效内容显示前调用 `model.read(item)`，导致本来已知是一行的 item 进入完整 render 和文本测量路径。
  - `Semaphore(8)` 会让这些不必要的 payload 读取排队，放大可见区重建时的逐项出现。
- 目标测量语义：
  - `WorkGroup`、折叠态或单项态的 `Reasoning` / 普通 `Tool` / `Patch` 首轮无需 payload 即可确定一行。
  - 加载完成前允许显示空行；读取失败显示红色 `Error`。
  - Message 和展开后的 item 才进入内容相关的换行与真实高度测量。
  - 读取与测量分离：payload 异步更新显示内容，不得成为一行骨架完成首轮测量的前置条件。
- 并发语义：
  - 不再设置固定的全局读取并发上限。
  - LazyColumn 的可见区、缓存区和 item 自身状态是自然边界。
  - 取消、去重和稳定 key 仍必须保留，防止重组产生重复读取或旧 generation 写回。
- 主要实现位置：
  - `Kodex/app/view/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt`
  - `Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt`
  - `Kodex/app/contract/history/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/AgentHistoryViewModel.kt`
- 验收重点：
  - 一行 item 的首轮布局不等待完整 event 读取。
  - 长历史切换 session 或 marker 更新时，不出现按 item 从下往上的明显重建。
  - 已加载稳定 item 不因普通重组重复解析。
  - 长 Message 的读取和布局不会阻塞其他一行 item。
  - release binary 在真实长历史 session 中行为正确，且性能数据能区分 VM 投影、payload 读取和 Compose 测量成本。
