# Task Tree

- [done] 修复 History 无上滚时丢失最新跟随
  - [done] 复现 viewport 几何变化导致的跟随丢失
  - [done] 确认布局位置被误用为用户跟随意图
  - [done] 明确退出与恢复跟随的输入语义
  - [done] 明确尺寸、换行、内容与焦点变化语义
  - [done] 明确 frontend-local、per-Agent 状态生命周期
  - [done] 确认本任务不增加暂停或未读 UI
  - [done] 建立 frontend-local Agent History UI 状态
    - [done] 显式持有 `LazyListState` 和 follow-latest 意图
    - [done] 按 Session 与 Agent 组合 identity 隔离状态
    - [done] 切换 Agent 或 Tab 时保留对应状态
    - [done] Session 关闭或 Agent 消失时清理对应状态
  - [done] 接入无竞态的显式滚动状态转换
    - [done] 同步观察已提交的 pointer 与 paging interaction
    - [done] 仅在向旧历史实际消费行数后暂停跟随
    - [done] 用户滚到最新边缘后恢复跟随
    - [done] focus relocation 与 programmatic 定位不暂停跟随
  - [done] 替换基于瞬时布局位置的跟随推断
    - [done] 内容变化前只按显式 follow-latest 请求最新边缘
    - [done] follow-latest 开启时纠正尺寸、换行和 item 高度变化
    - [done] follow-latest 关闭时继续用 stable key 保持阅读锚点
    - [done] 布局被动到达最新边缘时自动恢复跟随
  - [done] 补充状态机与 History 集成回归测试
    - [done] 覆盖 viewport 高度缩小与宽度重排
    - [done] 覆盖 pointer、PageUp、PageDown 和零消费边界
    - [done] 覆盖 focus relocation 与 programmatic 定位
    - [done] 覆盖 streaming、window append 和暂停阅读锚点
    - [done] 覆盖 Agent、Tab 隔离与状态清理
  - [done] 更新 History 交互决策并运行定向验证
  - [done] 与用户确认完整计划
  - [done] 获得用户授权进入 executable

# Details

- 状态：`done`。显式 follow-latest 意图、按 Session 与 Agent 隔离的 UI 状态、同步滚动转换和回归测试均已完成。
- 修复前 `AgentHistoryView` 没有独立的 follow-latest 用户意图，而是由 `requestLatestIfAtLatest()` 在内容变化前读取 `LazyListState.canScrollForward`。
- `canScrollForward` 是最近一次布局产生的几何状态。`LazyListState` 在普通重测量时恢复 stable-key anchor，viewport 变矮或内容重新换行后可以在没有任何滚动输入的情况下离开最新边缘：`Kodex/app/cli/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/LazyListState.kt:136`、`Kodex/app/cli/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/LazyColumn.kt:287`。
- 调研阶段用临时 Linux X64 回归测试确认：viewport 从三行缩到两行后，History 保留旧锚点并将 `canScrollForward` 变为 `true`；随后追加最新内容仍不恢复跟随。临时测试文件已删除。
- 界面存在多条无上滚的几何变化路径：hover 展开侧栏会动画缩窄内容宽度；Composer 换行或运行中出现 `Submit to steer` 会减少 History 高度；request-user-input、pending steer 和终端 resize 也会改变 History viewport。
- 已显式保存 frontend-local 的 follow-latest 用户意图，不再从瞬时 `canScrollForward` 反推用户意图。通用 `LazyListState` 和 `LazyColumn` 继续只提供滚动与布局能力，不引入 History 语义。
- follow-latest 开启时，viewport 尺寸、宽度换行、item 高度和 tail 内容变化继续定位最新边缘；follow-latest 关闭时，stable key 继续保持阅读锚点。
- 已补齐几何变化、内容增长、输入状态转换、Agent 与 Tab 隔离以及状态清理的回归测试。
- 定向验证通过：`JAVA_HOME=/home/stream/.jdks/openjdk-26.0.2 ./gradlew --no-configuration-cache --console=plain :app-cli-components:linuxX64Test :app-cli-history:linuxX64Test :app-cli-application:linuxX64Test`。

## 已确认行为

- 新 History UI 状态初始启用 follow-latest。
- 只有 pointer 滚轮或 PageUp 等 paging 输入向旧历史实际消费了行数，才关闭 follow-latest；边界上的零消费输入不关闭。
- 用户通过向下滚轮或 PageDown 回到最新边缘后恢复 follow-latest。
- Focus relocation 和 programmatic 定位不改变 follow-latest 意图。follow-latest 开启时，由它们造成的 viewport 偏移立即纠正到最新边缘。
- follow-latest 开启时，viewport 尺寸、宽度换行、item 高度、streaming tail 和 committed window 变化都保持最新边缘。
- follow-latest 关闭时，上述变化不把 viewport 拉到最新边缘，继续由 stable key 保持阅读位置。
- follow-latest 关闭期间，如果布局变大或内容缩短而被动到达最新边缘，则自动恢复 follow-latest。布局位置只允许恢复意图，不允许关闭意图。
- 本任务不增加暂停标记、未读标记或未读计数。

## 状态所有权

- 显式 `AgentHistoryUiState` 属于 Mosaic frontend，不进入共享 `AgentHistoryViewModel`。
- 每份状态持有一个 Agent 的 `LazyListState`、follow-latest 意图和滚动 interaction source。
- Frontend registry 使用 `(sessionIndex, agentId)` 作为 identity，避免不同 Session 的 Agent identity 冲突。
- 切换 Agent 或 Tab 复用 registry 中的状态；关闭 Session 时移除其全部状态，Agent 从当前 Session 拓扑消失时移除对应状态。
- `AgentHistoryViewModel` 继续只发布 committed window、streaming tail 和 history 数据命令；多个 frontend 共享它时不会互相改变滚动意图。

## 接线方案

- 为 `MutableScrollInteractionSource` 增加可选的同步 committed-interaction observer，并继续发布原有 `SharedFlow`；History 在输入处理返回前更新 follow-latest，避免布局纠正与用户上滚竞争。
- Pointer 与 Keyboard interaction 根据 reverse-layout 的 consumed delta 更新意图：负值且非零时暂停；正值到达最新边缘时恢复。
- FocusRelocation 与 Programmatic interaction 不进入暂停转换；既有 paging focus effect 继续订阅同一 interaction flow。
- `AgentHistoryView` 的 window、request-response 和 streaming content 回调改为读取显式 follow-latest，不再调用基于 `canScrollForward` 的 `requestLatestIfAtLatest()`。
- 使用 `snapshotFlow` 协调 follow-latest 与 `LazyListState.canScrollForward`：启用跟随但布局离开最新边缘时请求 logical start；暂停跟随但布局已经到达最新边缘时恢复跟随。
- 同步 interaction observer 先完成用户意图转换，布局协调器再处理最终状态；测试必须证明一次有效上滚不会被同一帧的尾部纠正撤销。
- 不修改 Mosaic fork，不把 History、Session、unread 或 follow-latest 语义放入通用 `LazyListState` 或 `LazyColumn`。

## 验证范围

- Components 测试覆盖同步 observer、原有 flow 发布、实际消费与零消费语义。
- History 状态机测试覆盖全部 input source、delta 方向、初始状态、暂停和恢复。
- History 渲染测试覆盖高度从大变小、宽度变窄导致换行、streaming 连续增长、committed append 和 paused stable-key anchor。
- Focus 测试证明 follow-latest 开启时 relocation 不会留下旧 viewport；暂停后 relocation 不会自行恢复，除非最终布局确实到达最新边缘。
- Application 测试覆盖两个 Agent 和两个 Tab 的状态隔离、切换恢复、Session 关闭及 Agent 消失后的清理。
- 定向验证至少运行 `app-cli-components`、`app-cli-history` 和 `app-cli-application` 的 Linux X64 tests；JVM 目标只在既有 Mosaic JVM/JDK22 编译阻断解除后运行。
- 实施时同步更新 `checklist/tui-interaction-components.md`，记录“布局位置只能恢复、不能关闭跟随意图”以及 focus relocation 不暂停跟随的决策。
