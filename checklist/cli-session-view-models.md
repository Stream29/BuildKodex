# CLI Session 与 Agent ViewModel 边界

## 所有权层级

- 保留一个应用级 `CodexCliAppViewModel`，并为每个已打开的真实 Session 建立一个稳定的 `CodexSessionViewModel`。
- 每个 Session ViewModel 包含常驻 root Agent ViewModel、轻量 Agent topology、selected Agent 和按需 materialize 的 Agent ViewModel registry。
- 使用 root `sessionIndex` 标识 Session ViewModel，使用 `(sessionIndex, agentId)` 标识其中的 Agent ViewModel；Agent storage id 不能单独充当跨 Session 地址。
- Root Agent ViewModel 在创建 Session ViewModel 时立即 materialize；subagent ViewModel 只在被选择、展开或其他 UI 明确请求时按需 materialize。
- Agent ViewModel 通过 parent address、direct-child slots 和所属 Session ViewModel 的 registry 表达递归树关系；构造父节点不得递归构造全部后代。
- Application、Session 和 Agent ViewModel 的 state slice 与 history source 必须遵守[CLI ViewModel状态与懒History](cli-view-model-state.md)。
- 将虚拟 `NewSession` 建模为应用级唯一的 `NewSessionViewModel`，不得伪造真实 Session 或 Agent identity、storage、lease、runtime。
- 将终端布局、焦点、hover、popup anchor、Agent tree expansion 和 history viewport 保留为 frontend-local 状态。

## Application-global ViewModel

- 应用级 ViewModel 只持有全局设置、模型目录、持久化 Session 摘要、opened Session ViewModel registry、当前 Session/NewSession target、全局 overlay 和退出状态。
- 当前单终端 frontend 的 active Session target 属于应用级 ViewModel；切换 target 只切换 Session ViewModel 引用，不销毁未关闭的 Session ViewModel。
- Session browser、global settings、Session 创建/打开/关闭/fork/import 和 application shutdown 由应用级 ViewModel 编排。
- 保留一个应用级命令串行化边界处理跨 Session 生命周期、全局设置提交和 shutdown flush；Agent response job 继续由现有执行层并发运行。
- 全局通知只报告设置、目录、认证、Session catalog 和应用退出等跨 Session 结果；Session 或 Agent 操作结果不得覆盖其他 Session 的状态。
- 应用级 overlay 只负责独占显示与路由；Session/Agent draft 必须保存在对应 Session 或 Agent ViewModel，并携带明确 owner identity。

## Per-session ViewModel

- 每个 Session ViewModel 持有 session index、生命周期 handle、root Agent ViewModel、轻量 Agent topology、materialized Agent ViewModel registry 和 selected Agent identity。
- Session root title、aggregate running、Agent tree、Session 级通知、打开/关闭状态和 Agent selection 属于 Session ViewModel。
- Session ViewModel 必须从轻量 topology 构造 Agent 树，不得为了显示树节点而读取所有 Agent history 或 materialize 全部 Agent ViewModel。
- Session ViewModel 负责按 Agent address `getOrPut` Agent ViewModel，并保证同一 Agent 在一个 Session 中只有一个共享实例。
- Session ViewModel 拥有 Agent ViewModel，但不得把子 Agent 的 mutable state 扁平复制进 Session state。
- Session ViewModel 的 root Agent ViewModel 始终存在；subagent slot 可以只包含轻量摘要而没有已 materialize 的 ViewModel。
- Session ViewModel 的 active Agent 投影由 selected identity 从 registry 解析；选择未 materialize Agent 时先创建对应 Agent ViewModel，再原子发布 selection。
- Session ViewModel 关闭时停止接收 UI 事件、flush 全部已 materialize Agent ViewModel，并关闭其 UI job 和订阅；执行资源仍交给 Session manager 释放。

## Per-agent ViewModel

- 每个 Agent ViewModel 只投影一个 Agent 的 settings、history、stream、activity、token count、plan 和 runtime state。
- Composer、composer revision、配置草稿、checkout 确认、Agent 通知和 request-user-input answer draft 按 Agent ViewModel 隔离。
- Request-user-input identity 使用 Agent address 与 call id 的组合；auto-resolution job 按 Agent ViewModel 独立持有和取消。
- Agent ViewModel 必须暴露 parent address 和 direct children 的轻量 slot；child slot 可以没有已 materialize 的 ViewModel。
- Direct children 使用显式 `Unloaded`、`Loading`、`Loaded` 或等价状态；展开一个节点只允许 materialize 它的 direct children，不递归 materialize descendants。
- 首版可以将已 materialize 的非 root Agent ViewModel 缓存到 Session 关闭，以保留 composer 和 dirty draft；不得因此提前 materialize 未访问节点。
- Agent ViewModel 的事件和异步 completion 必须捕获显式 Agent address 与 revision，不能在执行时重新解析应用级 active Session 或 Session 级 selected Agent。
- Agent ViewModel 关闭时只取消自身 UI job、timer 和订阅；Agent runtime、storage 和 coordinator 的资源关闭仍由 Session manager 执行。

## 轻量拓扑与详细投影

- 使用 coordinator 发布的 Agent identity、parent、task name、status 和 activity version 构建 Session ViewModel 的轻量树索引；展示树结构不得要求读取完整 Agent history。
- 未 materialize 的 Agent 只保留树摘要和执行层本来就需要的 transient state，不构造或缓存完整 conversation projection。
- 将当前 `SessionSnapshot` 明确为 Agent 级详细投影并改用 Agent 命名；Session ViewModel state 与 Agent 详细 state 必须使用不同模型。
- Materialize Agent ViewModel 时绑定既有 AgentSession storage，只读取轻量状态与有限 history tail，并合并执行层仍在进行的 stream/activity；不得构造一次完整 history 快照。
- 后台 Agent 的 status 和 activity version 继续更新所属 Session ViewModel 的轻量 topology；只有已 materialize Agent ViewModel 接收按需 history source、stream 和详细 UI 投影。
- Session manager 操作必须显式接收 Session/Agent identity 或稳定 handle；只移除隐式 selected routing，不改变现有 runtime 链和 response 执行语义。
- `SessionSummary` 不保存 application-level `selected`；selection 属于应用级 active Session target，Session root title 和 aggregate running 由 Session ViewModel 向 catalog 投影。

## NewSession ViewModel

- NewSession ViewModel 只持有 new-session defaults draft、revision、dirty/materialization 状态和独立 composer。
- `CodexGlobalSettings.newSession` 继续是 defaults 的唯一持久化真源；NewSession ViewModel 不保存第二份持久化 authority。
- 首条有效内容发布真实 Session 后，应用级 ViewModel 创建对应 Session ViewModel；Session ViewModel 创建常驻 root Agent ViewModel，并按捕获的 composer revision 清理 NewSession composer。
- 本规划不支持同时存在多个未物化 NewSession draft；该能力需要独立产品语义和持久化决策。

## 生命周期与验证

- Startup 只加载 Session 摘要和 NewSession ViewModel，不为未打开的持久化 Session 建立 Session ViewModel、Agent ViewModel 或 runtime。
- Open Session 创建或复用 Session ViewModel，并且只 materialize root Agent ViewModel；既有 subagent 不得触发完整 UI 投影。
- 展开或选择 subagent 时由所属 Session ViewModel materialize 所需路径或 direct children，并复用 registry 中已有的 Agent ViewModel。
- 切换 active Session 只更换应用级引用；每个 Session ViewModel 保留自己的 selected Agent、Agent registry 和 Agent UI drafts。
- Close Session 先 flush 对应 Session ViewModel，再释放 manager 资源、移除 application registry entry 并关闭已 materialize Agent ViewModel 子树。
- Shutdown 停止接收新命令，按确定顺序 flush NewSession、全部 Session ViewModel 和其中已 materialize Agent ViewModel，然后释放 application resources。
- 验证打开含大量 completed subagent 的 Session 只创建一个 Session ViewModel 和 root Agent ViewModel，不读取所有 subagent history。
- 验证按需展开只 materialize direct children，选择深层 Agent 只加载所需路径且 parent/children 关系正确。
- 验证两个 Session ViewModel 的 selected Agent、Session 通知和 Agent registry 相互隔离。
- 验证多个 Agent ViewModel 的 composer、设置草稿、pending input、通知及 auto-resolution 相互隔离。
- 验证未 materialize 的后台 Agent 继续发布轻量运行状态，materialize 后能按视口读取 committed history 并取得当前 transient stream。
- 验证全局设置和模型目录更新可被全部 Session/Agent ViewModel 观察，但不会改写已有 Agent 的持久化 settings。
- 验证多个 frontend 共享 application、Session 和 Agent ViewModel 状态，同时各自保留独立 Agent tree expansion、history viewport、焦点和 layout 状态。
