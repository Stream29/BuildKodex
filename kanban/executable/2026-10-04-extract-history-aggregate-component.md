# Task Tree

- `Audit parallel models and compiler-only compatibility layers`()
- `Discard the rejected HistoryAggregate and duplicate item model implementation`()
- `Move the original AgentHistoryViewModel family into the History component`()
- `Migrate all consumers and remove the old History projects`()
- `Remove the parallel New Session component model and adapter`()
- `Validate the hard migration against the original UI and downstream tests`()
- **`Track remaining validation risk after the authorized commit`()**

# Details

## Confirmed scope

- 用户确认这是一次性硬迁移，不是 staged compatibility migration。
- 展示对象是 Agent 的完整历史，跨 index/work/unstable 等 timeline；公开模型保持
  `AgentHistoryViewModel`，不重新定义 Aggregate、Timeline 或 Session History。
- 本任务文件名保持稳定，以免打断既有看板链接；名称不代表代码中的领域概念。
- 本轮同时修正同一未提交批次里 New Session 的旧/新 ViewModel 双轨。
- 用户已授权提交并推送当前硬迁移切片；用户其他任务和共享 IDE 保持不动。

## Hard-migration rule

- 保留原类型身份、可观察行为与 KDoc，直接移动真实声明和实现。
- 删除被替代的生产声明及 Gradle 项目，全部消费者切到新物理路径。
- 不添加旧模型到新模型的 adapter、facade、resolver、typealias 或转发实现。
- 与真实 RPC/storage 边界的读取和命令依赖可以保留；不把一个组件的状态再映射为
  另一个平行组件状态，只为让旧宿主继续编译。
- 原测试随实际实现迁移；被否决的平行模型测试不能充当完成证据。

## History target

```text
app/component/history/
  spec/
  impl/viewmodel/
  impl/view/
```

- `spec` 接管原 `app/contract/history`：`AgentHistoryViewModel`、窗口、流式状态、
  原 History Item ViewModel/状态族。`AgentHistorySource` 作为已存在的读取依赖迁入 spec，
  保留 FQCN，不额外增加依赖 adapter 或 factory facade。
- `impl/viewmodel` 接管原 `app/viewmodel/history`：稀疏读取、有界分页、generation/
  cache nonce 失效、懒加载 item、Work Group 子项创建/释放及 turn duration。
- `impl/view` 接管原 `app/view/history`：原完整 UI、paging/focus、流式输出、待处理工具、
  展开/折叠、准确 generation/index/item context-menu 目标。
- 旧三个项目不留 forwarding build；Gradle 坐标改为 `app-component-history-*`。

## Item-layer ablation

审计确认之前九个独立新 item 项目并未硬迁移原 item 类型，而是另造状态与投影，再由
宿主转换回原类型。其 `Legacy*` renderer、重复 header/target/projection、child union
包装及 staged adapter 都不保留。

本轮将真实九类 item 的契约、实现和渲染作为 History 组件的内部职责一起接管：

- 保留原具体 `Message/Reasoning/Tool/Patch/PlanUpdate/RequestUserInput/
  SuggestSubagentTask/ContextCompaction/WorkGroupHistoryItemViewModel`。
- Work Group 的 `Expanded.children` 直接保存真实
  `WorkGroupChildHistoryItemViewModel`；没有第二套 wrapper union。
- item 懒加载、失败、折叠释放和 storage metadata 继续使用原已验证状态机。
- 不为了保留项目数量增加跨模块 loading-context 服务、公开内部生命周期 hook、
  新 resolver 或未使用 factory。后续进一步物理细分必须直接移动这些真实类型，
  而不是恢复本次被否决的平行类型。

## New Session correction

```text
app/component/new-session/
  spec/
  impl/viewmodel/
  impl/view/
```

- 直接迁移原 `app.session.contract.NewSessionViewModel` 与真实
  `DefaultNewSessionViewModelFactory` / `RpcNewSessionViewModel`。
- 删除 `app.newsession.contract.*` 平行 API、creation/effect/receipt 状态复制及
  `NewSessionComponentAdapter`。
- 共同 `SessionViewModel` 改为普通 interface，以允许真实子接口位于独立 spec；
  不增加第二套 Session 基类或 typealias。
- 页面 renderer 接收同一个 registry child；tab 切换不关闭它。materialize 的返回值仍是
  真实 `PersistedSessionViewModel`，由根 registry 替换准确 draft。
- 公用 runtime status controls 保留在宿主，通过普通 composable visual slot 提供给
  页面，不产生第二套设置状态或 business dependency。
- Draft 的 Composer submit/steer 不能伪造 Persisted/Accepted 成功；真正创建仍走
  draft owner 的 materialize，误调用 Agent 命令端口明确失败。

## Compatibility and remaining boundaries

- 原 newest-first、有界窗口、tail-first 初始化、paging anchor 和 follow-latest 保持。
- 原 Reasoning/Compaction 摘要、Tool/Patch 展开和 shell 状态渲染保持。
- context menu 捕获真实 item、generation、storage index，仍用 `contains` 校验。
- History Index Check out 仍只滚动，不新增 revert/fork/backend mutation。
- New Session 仍串行化编辑和物化，不新增 receipt/replay/rollback 协议。
- 原 History spec 中自定义 LazyList/scroll presentation 类型仍保留；这是真实旧边界，
  本次不以创建平行 VM 来“修复”它。若要移到纯 renderer-local presentation，应另行
  直接修改这一唯一契约及其全部消费者。
- 未迁移的 `agent-state-contract` 依赖如实保留，不声称已经完成全仓 spec-only 图。

## Composer correction in the same working batch

- 保留原 `ComposerState.text`，删除 `draft`/`text` compatibility getter 双命名。
- 删除只有单分支的 Submit/Steer receipt 类型，依赖操作真实完成后正常返回 `Unit`。
- 删除宿主的第二套 `submitComposer` mutex/算法及 result，所有 draft submit 使用唯一
  Composer child；外部 prepared-content submit 不消费 draft，因此保留。
- 删除重复的 `updateDraft`/cursor fallback，保留唯一真实 `update(text, cursorOffset)`；
  Agent 与 New Session 使用同一份真实输入 widget。
- 删除仅更改 identity 而不更换固定依赖的 `replaceOwner`；真实 owner 替换关闭旧 child
  并创建新 child，测试验证旧操作迟到完成不能污染它。
- submit/steer 从固定 runtime 的当前值选择，而非等 renderer 投影同步；新增立即改变
  running、观察尚未更新时仍正确走 steer 的回归。
- New Session 误调 Agent submit 的失败回归和 renderer unmount 不关 registry child 的
  生命周期回归已加入验证矩阵。

## Validation gates

- 唯一生产 `AgentHistoryViewModel` 和 `NewSessionViewModel`；无 Aggregate、旧/新 adapter。
- 无旧 History/New Session implementation Gradle 坐标或生产根目录。
- conventional plugins、spec 不依赖本组件 impl、生产不存在无意义的转发项目。
- 原 History spec/ViewModel/View 测试与 New Session ViewModel/UI 回归。
- Application、Agent、Session、RPC 下游编译/测试，以及 Integration 测试源码编译。
- 原 UI 关键输出、context-menu、稀疏加载、关闭/取消/失效、准确 draft 替换回归。
- 内外层 `diff --check`、活动文档链接、旧命名/旧坐标扫描。
- Native、JS、真实 CLI/网络、IDE sync 未运行时明确记录，不能以 JVM 验证代替。

## Current checkpoint

- 原 History 三项目与原 New Session controller 已物理迁移，消费者已切换。
- 平行 Aggregate/item/New Session 实现与 adapter 已从生产工作树移除。
- 最终定向 JVM 矩阵通过：308 项测试、0 failures、0 skipped；包括原 History
  86 项、Composer 15 项、New Session controller 5 项、Tab Bar 5 项和宿主 197 项。
  无源码任务不计入测试数。
- Application、Session、RPC 主/测试源码以及 Integration 测试源码编译通过；
  未运行真实 Integration 网络测试。
- 三个核心 ViewModel 各仅有一个生产声明；被否决类型、旧坐标/根目录扫描和
  内外层 `diff --check` 通过。
- 用户授权提交当前结构修正；内层提交为 `821b6627`。Native、JS、真实 CLI 与
  IDE sync 尚未运行；下述压力用例风险仍未闭合，任务保留在 executable，不归档 done。

### Open validation risk

- 原 1000-item History 有界窗口/UI 连续往返用例在本轮复跑中两次出现 5 秒超时；
  不能把随后成功解释为根因已修复。
- fixture 现在等待新窗口真正被 renderer 测量、load state Ready 和准确可见 row
  identity，再开始下一次滚动；没有延长超时、直接请求 VM 页或跳过该用例。
- 已增加失败时 window/load/layout 诊断。诊断版连续三次强制 History renderer suite
  重跑通过，最终矩阵也通过；偶发超时的根因尚未确定。
- 当前没有因此修改原 paging 算法；这一风险保持未闭合，不宣称整批已无条件通过验收。
