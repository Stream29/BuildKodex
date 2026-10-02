# Task Tree

- `Review Hook Settings ports and application-owned writes`()
- `Define component spec for list, editor, detail, and confirmation`()
- `Extract ViewModel and renderer without local storage ownership`()
- `Hand Settings and queue adapters to the batch integrator`()
- `Validate local persistence, stale drafts, rendering, and disposal`()
- `Record batch acceptance and commit`()

# Details

## Centralized integration result

- Main integrator 已完成 Settings stable child、共享样式、RPC Hook adapter 与队列接线。
  captured original/updated 值在 hide/close 后继续排空；renderer 不保存业务草稿。
- 本组件 ViewModel 12 项、View 9 项 JVM 测试通过；宿主队列、本地存储、
  通知执行器及关闭回归通过，批次合计 392 项，详见主任务。
- 新测试的 coroutine test opt-in 已补齐，state 对外为只读 wrapper；
  宿主渲染 shared failure 时 panel 不重复显示 banner。
- Native/JS/CLI 尚未验证；本批已验收并提交为 `7310a445`，进入 done。

## Integration handoff checkpoint (worker implementation complete)

- Settings-resource worker owns only the two component trees, four relocated renderer
  files and dedicated tests; hosts/build registration remain integrator-owned.
- Framework-only helpers required from `app-view-components`, retaining package
  `io.github.stream29.kodex.cli.settings`: `SettingsActionButton`,
  `SettingsPrimaryButton`, `SettingsDangerButton`, `SettingsContentButton`,
  `SettingsErrorText`, and colors `SettingsHomeBackground`,
  `SettingsSectionHeaderBackground`, `SettingsForeground`, `SettingsDialogBackground`,
  `SettingsHeaderBackground`, `SettingsActionBackground`.
- MCP additionally needs `SettingsDropdownField`, `SettingsCheckboxItem`,
  `PopupMenuBackground`. Existing signatures suffice. Component views will depend
  on `app-view-components`, never `app-view-settings`.
- Baseline Hook editor trims only the name on save; command text is preserved.
- Spec/VM/view APIs and component tests are complete in the worker's assigned paths:
  - Package `io.github.stream29.kodex.app.hooksettings`.
  - `createHookSettingsViewModel(dependencies: HookSettingsDependencies,
    ownerScope: CoroutineScope): HookSettingsViewModel`, also
    `DefaultHookSettingsViewModelFactory.create(...)`.
  - `HookSettingsDependencies`: `hooks: StateFlow<List<NotificationHook>>`,
    `operationFailure: StateFlow<Boolean>`, `captureEditor(name: String?):
    HookEditHandle?`, `delete(original: NotificationHook): HookWriteAdmission`,
    `reportFailure(Throwable)`, `dismissFailure()`.
  - `HookEditHandle`: `original: NotificationHook?`,
    `save(updated: NotificationHook): HookWriteAdmission`, `release()`.
    Null original is Add; nonnull original is exact opening baseline.
  - Admission is `Accepted` or `Rejected(message: String)`; Accepted is queue
    admission only. Adapter copies captured original/update before returning.
    Queue conflicts/failures use existing application failure state.
  - Public renderer package remains `io.github.stream29.kodex.cli.settings`:
    `HookSettingsPanel(viewModel)` in the Settings viewport,
    `BoxScope.HookSettingsDialogs(viewModel)` in its popup host, or full
    `BoxScope.HookSettingsComponent(viewModel)`.
  - All dialogs/drafts are `viewModel.state`; no host request mirror.
    Page switching calls `hidePage()`; Settings lifetime calls `close()`.
- Six new build scripts use existing conventions, but registration/dependency
  consumers are integrator-owned. No Gradle/IDE operations performed.

### Testing state and remaining integration gates

- Fake-port suite: ordered/empty observation, Add/Edit/Rename, captured baseline,
  name trim/command preservation, name/command/types validation, admission rejection,
  delete captured original, duplicate confirmation, missing details, replacement
  token, hide/close/owner cancellation, generic failure/reopen/acknowledgement,
  unchanged cancellation propagation, and reentrant close during admission.
- Relocated `HookSettingsContentTest` covers original list/details/editor rendering.
  New `HookSettingsComponentTest` consumes the real VM and fake ports, covering
  empty/failure, validation/input synchronization, keyboard Save, delete Cancel-first
  focus and confirm, Escape and stale-token replacement.
- Static checks completed: whitespace checks for every new component source/build
  file, scoped relocated-path checks and outer-document diff checks. Renderer scan
  found no parallel `mutableStateOf` business state, draft validation/parser or
  Hook construction; spec/VM imports contain no framework/store/host dependency.
- Tests were authored, **not compiled or executed by this worker**. Integrator owns
  all Gradle, host/RPC queue persistence tests and platform smoke. Worker created no
  temporary files/resources and did not commit/push/switch or edit host integrations.
- Worker 交接后由主 Session 完成集中验证和用户验收，本任务已归档到 done。

## Scope and source map

- 属于 [四组件双线批次](2026-10-02-extract-pending-tool-and-resource-components.md)，
  设置资源线先完成本组件，随后处理 MCP。
- 用户已验收并授权提交/推送；设置线 Session 458 完成独立组件，当前 done。
- 目标：`app/component/hook-settings/{spec,impl/viewmodel,impl/view}`。
- 原 GlobalSettingsViewModel 的 hooks/编辑命令归入组件 child；
  SettingsPopup 中详情名称、编辑/删除请求，以及 HookSettingsContent、
  HookSettingsDialogs 的列表/详情/编辑/删除渲染整体迁入。
- `RpcGlobalEditor` 的当前 Hook 草稿 baseline、保存/删除逻辑和失败队列
  拆成宿主依赖 adapter；CliFrontendSettingsStore 仍为应用拥有。
- 使用原 `rpc/spec/models` 的 NotificationHook/NotificationHookType；
  这不是旧后端控制型 Hook，不迁移 Hook executor 或修改通知协议。

## Proposed contract

- `HookSettingsDependencies` 提供只读 hooks、捕获编辑目标、接受保存/删除意图
  和既有 failure 观察/清除能力；具体 store 与 SettingsUpdateQueue 不进 spec。
- 编辑打开时捕获准确原值；提议由 typed bound edit handle 携带 baseline，
  不以最新同名条目重建旧意图。组件拥有该 handle 的草稿，不持有整份 frontend settings。
- ViewModel/state 表达列表与 None configured、详情目标、Add/Edit 草稿、
  类型选择、校验消息和删除确认。焦点/scroll 不进 spec。
- name/command 非空、类型非空且只含现有四类型；沿 baseline 编辑器在保存时
  trim name，保留 command 原字符串，不做改名合并、命令去重或新增类型。
- 编辑保留列表位置，新建追加、删除只删匹配的原值；同名冲突和过期 baseline
  沿现存存储/adapter 规则。详情按名字读取当前条目；条目消失不可继续操作旧详情。
- 端口明确“接受到应用队列”而非“写盘成功”；具体 admission 返回形式在 API
  审阅时固定。组件不能据此发布虚假的 persisted 状态或吞掉异步错误。
- KDoc 覆盖字段编辑、详情与编辑切换、确认/取消、换页清理、
  关闭后的命令和 `@throws`；校验失败与存储未知异常分别描述。

## Ownership and rendering

- Global Settings 发布稳定 `hookSettings` child；SettingsPopup 只接入渲染与
  页切换 command，不保存另一份编辑/删除 request。
- 页面隐藏清理短期对话框/未提交草稿；关闭 Settings 释放 child 观察并拒绝新意图。
  已接受写入的 captured original/updated 值由应用级 queue 继续持有到排空。
- Hook 命令执行仍在原 CLI 通知执行器；进入列表、查看详情、提交配置都不试跑命令。
- 保留列表名称/类型摘要、command-free 详情、编辑中的命令输入、类型选择、
  删除确认及现有键盘语义。
  不借迁移增加测试执行、enable、timeout 或环境字段。

## Checks

- fake dependency 测试所有编辑/取消/确认路径、过期原值、同名冲突、
  重复确认、换页与关闭；渲染所有 branch，不只测试对话框文本存在。
- 移入 HookSettingsContent 测试，补充 editor/details/delete 输入与焦点回归。
- RPC Settings 现有前端 Hook 写入和 close 后排空测试保留并扩展；
  本地文件旧 codec、顺序和 NotificationHook 序列化不变。
- failure 状态跨关闭/重开保留，取消不记为业务失败；
  组件 dispose 不取消应用通知订阅、执行器或已接受本地写入。
