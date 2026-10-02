# Task Tree

- `Review MCP Settings ports, credentials, import, and OAuth ownership`()
- `Define list, dialog, draft, import, and interaction spec`()
- `Extract dependency-only ViewModel and complete renderer`()
- `Hand backend settings and OAuth adapters to the batch integrator`()
- `Validate conflicts, accepted-write drain, auth cleanup, and rendering`()
- `Record batch acceptance and commit`()

# Details

## Centralized integration result

- Main integrator 已接入 Global Settings stable child、Settings-lifetime URL handler、
  exact baseline adapter 与原写队列；旧镜像 dialog/import authority 和旧 DTO 声明已移除。
- 本组件 ViewModel/parser 27 项、View 24 项 JVM 测试通过。真实 RPC 回归证明
  accepted import/edit 在隐藏及关闭后排空，与 Hook 共用队列，不因组件 release
  失去 captured candidates/baseline；同值 Replace 不强制 reconnect。
- 浏览器旧 effect 失败不取消新的同名登录，hide 不取消 OAuth，close 才清理；
  原 RpcOAuthTest 通过。组件及宿主批次合计 392 项通过，详见主任务。
- 新测试 coroutine opt-in 已补齐，state 为只读 wrapper，shared failure 不重复渲染；
  Native/JS/CLI/真实网络集成未运行，本批已验收并提交为 `7310a445`，进入 done。

## Integration handoff checkpoint (worker implementation complete)

- Shared visual-helper requirements are listed in the Hook subtask checkpoint.
  Keep their existing Settings package in `app-view-components`.
- MCP spec copies `McpServerSettingsState` / `McpServerSettingsStatus` in their
  existing `io.github.stream29.kodex.app.settings.contract` package; integrator
  removes the old declarations before compilation.
- Adapter must detach accepted import candidates/baseline from mutable active
  preview before enqueueing. The pre-migration `RpcMcpSettings.applyImport` read the active
  preview at execution and checks identity during CAS; merely clearing that preview
  on hide/close would invalidate already accepted writes. Component contracts
  require bound import handles whose accepted payload survives release.
- Spec/VM/view APIs and component tests are complete in the worker's assigned paths:
  - Package `io.github.stream29.kodex.app.mcpsettings`.
  - `createMcpSettingsViewModel(dependencies: McpSettingsDependencies,
    ownerScope: CoroutineScope): McpSettingsViewModel`, also
    `DefaultMcpSettingsViewModelFactory.create(...)`.
  - `McpSettingsDependencies`: `servers: StateFlow<List<McpServerSettingsState>>`,
    `operationFailure: StateFlow<Boolean>`, `captureEditor(serverName: String?):
    McpEditHandle?`, `captureServer(serverName: String): McpServerHandle?`,
    suspend `readImport(): McpImportHandle`, suspend
    `startLogin(serverName: String, interactionScope: CoroutineScope):
    McpSettingsLogin?`, suspend `reconnect(serverName: String)`,
    suspend `logout(serverName: String)`, `reportFailure(Throwable)`, `dismissFailure()`.
  - `McpEditHandle`: `originalName: String?`, `initialDraft: McpServerDraft?`,
    `save(draft: McpServerDraft): McpWriteAdmission`, `release()`.
    Add has null originalName/initialDraft. Edit initialDraft uses Keep only.
  - `McpServerHandle`: `serverName: String`, `delete(): McpWriteAdmission`,
    `setEnabled(enabled: Boolean): McpWriteAdmission`, `release()`.
  - `McpImportHandle`: `preview: McpImportPreview`, `filter(filter: String):
    McpImportPreview`, `apply(decisions: Map<String, McpImportDecision>):
    McpWriteAdmission`, `release()`. Accepted payload outlives release.
  - `McpSettingsLogin`: `serverName: String`, `authorizationUrl: String`,
    suspend `awaitCompletion()`, exact-attempt `cancel()`; reuse existing
    `startRpcOAuth` with the passed interaction scope, and keep listener/protocol
    cleanup in that adapter. No browser opening in adapter.
  - Admission is `Accepted` / `Rejected(message: String)` only; unknown admission
    errors are reported once by VM. Application queue owns asynchronous errors.
  - Renderer package remains `io.github.stream29.kodex.cli.settings`:
    `McpSettingsPanel(viewModel)` / `BoxScope.McpSettingsDialogs(viewModel)`
    or full `BoxScope.McpSettingsComponent(viewModel)`.
    Install `McpSettingsEffects(viewModel, openUrl: suspend (String) -> Boolean)`
    outside the page branch, for the entire Settings lifetime. Alternatively consume
    `viewModel.effects` once: `OpenAuthorizationUrl(serverName, url, cancel)`.
    Browser failure MUST invoke that effect's captured `cancel()`, not current-name
    lookup. The callback is bound to the exact job/attempt.
  - Page switch calls `hidePage()` (does not cancel OAuth); close calls `close()`.
    Host must remove its old parallel detail/editor/delete/import state.
- State/status declarations were copied with unchanged field/package shape;
  remove old declarations before building. Parser/validation moved to VM.
- No builds, IDE/resource manipulation, commits, push or ref changes performed.

### Testing state and remaining integration gates

- Fake-port VM suites cover sanitized observation, draft validation/queue admission,
  captured editor baseline/Keep markers/rename, known rejection, latest/missing
  details, delete/enable capture, reconnect/logout, replacement/hide/close, explicit
  import Loading/Preview/Failed/Retry, default New/Conflict/Unsupported decisions,
  filter-retained decisions, select/clear/no-selection/apply, rejected apply,
  cancellation and noncooperative late read, application failure/reopen/acknowledgement,
  exact old-effect cancellation versus newer same-name OAuth, hide versus close,
  late preparation cleanup, protocol preparation error/cancellation and reentrant
  close during admission.
- Parser suite preserves quoted/escaped/empty arguments, cwd text, Keep/Replace/Remove,
  OAuth optional normalization/scopes and exact displayed validation messages.
- Relocated `McpSettingsContentTest` / `McpSettingsDialogsTest` retain original
  rendering/dropdown coverage; default-decision business assertions moved to VM.
  New `McpSettingsComponentTest` uses real VM/fake ports, all six runtime statuses,
  all seven authentication branches, empty/failure, editor validation/text/transport,
  sanitized Keep/OAuth, delete Cancel-first focus, import Loading/Failed/Retry/
  supported/unsupported/empty/rejection, and late platform-opener failure after
  a newer login while the MCP page is hidden.
- Static checks completed: new-file whitespace, scoped relocation, outer-document
  diff and renderer business-authority/dependency scans. Views contain widget
  buffers/focus/dropdown/scroll only; parsers and decision state are VM-owned.
- Tests were authored, **not compiled or executed by this worker**. Integrator
  owns all Gradle and RPC CAS/OAuth/write-drain tests; native/JS/CLI remain unvalidated.
  Queue-drain, same-value Replace and real credential/identity coordination require
  host regression tests, not just fake-port admission tests.
- 主 Session 已完成集中验证和用户验收，本任务已归档到 done。
  Worker 未创建临时文件或占用额外构建资源。

## Scope and source map

- 属于 [四组件双线批次](2026-10-02-extract-pending-tool-and-resource-components.md)，
  设置资源线的复杂组件，沿 Hook 的宿主 child/queue 接线模式实施。
- 用户已验收并授权提交/推送；组件实现和集中验证完成，当前 done，提交 `7310a445`。
- 目标：`app/component/mcp-settings/{spec,impl/viewmodel,impl/view}`。
- 从 GlobalSettingsViewModel 移入 McpServerSettingsState/status 和组件命令；
  从 SettingsPopup 移入详情/编辑/删除/导入的交互 state。
- 整体迁移 McpSettingsContent、McpSettingsDialogs，以及纯展示映射；
  RpcMcpSettings 的 raw configuration baseline/CAS 和 RpcGlobalEditor 的
  application queue/OAuth 桥接仍作为宿主 adapter，不让 spec 依赖具体 RPC。
- 覆盖 Add/Edit/Rename、Delete、Enable/Disable、Login/Cancel/Logout、
  Reconnect、Codex Import 加载/过滤/选择/冲突决策/提交。
  不改后端 manager/service、协议、credential 格式或连接策略。

## Proposed spec and dependencies

- `McpSettingsDependencies` 提供 sanitized servers 只读源、捕获编辑 capability、
  接受保存/删除/启停意图、reconnect、绑定名称的认证交互、
  导入读取/过滤/应用 capability、既有操作 failure 观察/清除。
- raw 配置与凭据留 adapter 内；编辑只返回原 typed McpServerDraft 的
  Keep/Replace/Remove marker 与输入值，不提供真实凭据回填或整份 BackendSettings。
- 提议用 typed bound editor/import handle 固定 baseline/previewId；
  opaque 私有快照不要求序列化，不当成新 RPC DTO。接受写入时把执行所需值
  交给应用队列，之后 UI dispose 不能使已接受意图的资源失效。
- spec 只依赖 MCP spec、必要 RPC models/基础库；不依赖 app settings contract、
  GlobalRpc、RpcMcpSettings、store、HTTP client 或 URL opener 实现。
- state 分清 backend runtime 观察与 frontend 交互：服务器列表/状态、
  当前详情、编辑草稿/校验、删除确认、导入加载/预览/决策/错误。
  配置/连接不是原子快照，不把 save/login 返回当作 Healthy/Authorized。
- 所有命令明确 target、准入、取消/失败、关闭和效果时序。
  已知校验/冲突走现有返回路径；未知错误沿应用 failure 机制，
  公开异常与 cancellation 写明 `@throws`，不展示远端敏感详情。

## Editor and details semantics

- 详情按准确 serverName 从最新 sanitized 列表定位；只展示允许的 URL/command/args、
  header/environment 名称及认证摘要，不显示真实 secret 或 raw 配置 toString。
- 编辑打开时捕获完整 baseline，但 renderer 只接触 typed draft；
  更新/改名以原值字段冲突规则提交，OAuth identity 保留/清理沿后端既有行为。
- 验证错误留草稿；确认写入队列与等待持久化不是同一状态，不自造保存回执。
  启停/删除/reconnect/login 不隐式改变其他配置或复制 client 所有权。
- stdio cwd 继续是文本字段，不强制换成 Working Directory 浏览器；
  参数/header/environment/secret 编辑语义和现有 parser 行为不顺带重设计。

## OAuth and Codex import

- OAuth 复用现有 RPC authentication adapter：exact attempt、实际 redirect URI、
  loopback/callback 及取消清理留该 adapter。renderer 只执行声明过的 URL 打开 effect；
  URL 打开失败通知对应交互，不按“当前最新登录”取消另一次 attempt。
- 授权临时 URL 不持久化或广播到全应用共享 effect；组件 disposer 取消它拥有的
  短期登录，不能注销共享服务器、关闭 backend client 或关闭所有认证交互。
- 保留现有“换 Settings 页不取消已开始 OAuth、关闭 Settings 才清理”的区别；
  对话框隐藏不等于登录 owner 结束。认证 effect 仍有一个 Settings 生命周期的
  handler，不因 MCP 页未渲染而丢失 URL 或延迟到重新进入才打开。
- 导入仅显式点击时读取；组件表达 Loading/Preview/失败，而非自动轮询。
  新项默认 Import、冲突默认 Replace、不支持不可选；过滤/决策绑定准确 previewId。
- 原完整配置在 adapter 临时保存，renderer 只看脱敏预览。
  应用只改选中可支持项，遵循 BackendSettings CAS 字段冲突规则。
- RPC 当前 Replace 仅是配置值合并，同值 CAS 不额外 invalidate/reconnect；
  不恢复旧 manager Replace 的专属副作用，不导入 OAuth token 或持续同步 Codex。
- 换页/dismiss 清理未接受的 preview；已接受 apply 的 baseline/候选值须活到
  queue 排空。请求替换与迟到读取/认证结果不能重开旧弹窗或污染新 preview。

## Checks

- fake ports：列表空/运行/认证分支、草稿验证、Keep/Replace/Remove、改名/删除、
  stale editor、启停/reconnect、失败与重复操作、close/换页。
- 导入：显式加载、错误/重试、过滤和默认选择、Unsupported、
  preview 替换/过期、字段冲突、单次合并、已接受 apply 后切页/关闭。
- OAuth：浏览器打开失败、exact attempt 取消、关闭与迟到回调；
  保留 RpcOAuthTest，不使用真实网络登录证明单元测试通过。
- 移入 Content/Dialogs 测试，补全 ViewModel 交互与所有 renderer branch；
  Settings 不再拥有平行 dialog/draft authority。
- 保留 RpcSettingsTest 中 rename/同值 Replace/跨关闭排空、真实 backend
  CAS 和本地 failure 观察。扫描 state、错误和日志的 secret 泄露风险；
  前端观察取消不关闭共享 MCP service/client。
