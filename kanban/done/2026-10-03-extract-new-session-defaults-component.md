# Task Tree

- `Inventory defaults state, model options, revision admission, and persistence`()
- `Specify typed ports and full default-configuration rendering`()
- `Review field baselines and accepted-write lifetime`()
- `Extract spec, ViewModel, and complete controls and menus`()
- `Integrate global adapters and Settings New Session page`()
- `Validate revisions, merging, queue drain, and shared failure reporting`()
- `Record acceptance and archive component task`

# Details

- 实施/集中验证完成；本组件 14 项 JVM 测试通过，真实字段 CAS、独立队列
  排空及 Settings/Title 接线验证见主任务。用户已验收，内层代码随批次
  提交为 `721ac0fe`。

## Scope and source map

- 属于 [五组件批次](2026-10-03-extract-session-configuration-and-browsing-components.md)，
  用户已审阅并授权实现，当前 executable。
- 目标 `app/component/new-session-defaults/{spec,impl/viewmodel,impl/view}`。
- 源 NewSessionSettingsViewModel/State、`RpcNewSessionSettings.kt`、
  SettingsPopup 的 NewSessionConfigurationContent/四菜单。
  [原队列与 revision 准入，L12 起](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcNewSessionSettings.kt#L12)。
- 只编辑持久化的新建默认值；当前 Draft、已存在 Session 和自动标题生成
  配置是不同所有者。Title child 在同页独立展示，不合并进本组件。

## Proposed spec and ports

- 迁入原公共 State/ViewModel，保留唯一 FQCN；domain 值
  `KodexNewSessionSettings`、ModelInfo 等继续复用原 canonical 类型。
- factory 接受 typed dependencies/ownerScope；deps 只给 defaults/models
  只读来源、四个 field-specific admission、共享 failure/report/ack。
- admission 捕获 expected field 与 requested value，返回接受而非保存回执；
  adapter 使用原 defaults 队列及 BackendSettings CAS，不能把 Global store/
  全量 BackendSettings 暴露给 spec，不能复制默认 model 字符串。
- VM 持有 local revision 与 modelOptions 投影、关闭状态；四个命令接收
  expectedRevision。模型目录变化只更新选项，不无故使 defaults revision 失效。
- spec 写明每个状态怎样渲染、reasoning/tier/mode 选项规则，以及
  cancellation/ordinary errors/closed admission 与异步失败归属。

## Behavior and lifecycle

- 目录顺序加当前 default model 去重；缺失 model 仍可显示/选择；
  reasoning 固定已知列表，tier/mode 用现有 enum，不改成 Runtime 能力过滤。
- 已接受编辑冻结字段 baseline/payload，只改该字段，保留 concurrent
  title/auth/context/MCP 等值；无乐观保存成功状态。
- 原默认值 editor 具有独立应用级 SettingsUpdateQueue，不与 Global editor
  原队列混用；close 停止 child 观察，但**继续排空已接受写入**。
- `SettingsOperationFailure` 继续来自已有共享 authority；成功/失败按现行
  adapter 规则报告/清除，不新增 defaults-local save-success/error store。
- hide 仅关闭 menus；renderer mount 不重新创建 child/初始化持久化默认值，
  关闭子组件不关闭 models/global/backend。
- 不为统一 Session editor API 改成 cancel-on-close，也不新增跨 editor
  的全局 admission order 保证。

## Checks and handoff

### Integration handoff (component lane, delivered)

- Frozen creation seam in `io.github.stream29.kodex.app.settings`:
  `createNewSessionDefaultsViewModel(dependencies: NewSessionDefaultsDependencies, ownerScope: CoroutineScope): NewSessionSettingsViewModel`.
  Moved State/ViewModel retain `io.github.stream29.kodex.app.settings.contract`.
- `NewSessionDefaultsDependencies` borrows `defaults: StateFlow<KodexNewSessionSettings>`,
  `models: StateFlow<List<ModelInfo>>`, `operationFailure: StateFlow<Boolean>`.
  Four typed methods return `NewSessionDefaultsAdmission` (`Accepted`/`Rejected`):
  `admitModel(expected: OpenAiModelId, requested: OpenAiModelId): NewSessionDefaultsAdmission`,
  `admitReasoningEffort(expected: ReasoningEffort, requested: ReasoningEffort): NewSessionDefaultsAdmission`,
  `admitServiceTier(expected: ServiceTier, requested: ServiceTier): NewSessionDefaultsAdmission`,
  `admitRequestUserInputMode(expected: RequestUserInputMode, requested: RequestUserInputMode): NewSessionDefaultsAdmission`.
  `reportFailure(failure: Throwable): Unit`, `dismissFailure(): Unit` use shared authority.
- Main binds these ports to the original independent application-scope defaults queue,
  freezes field baseline/payload synchronously, reports later outcome there, and closes/drains
  queue **after** child close. Component closes only observations; never cancels accepted writes.
- Old four Unit commands retained. New `operationFailure`/`dismissOperationFailure()`;
  state adds defaulted `active: Boolean = true`, false on close without resetting defaults.
- `DefaultNewSessionDefaultsViewModelFactory.create(dependencies: NewSessionDefaultsDependencies, ownerScope: CoroutineScope): NewSessionSettingsViewModel`
  implements the spec factory.
- Renderer exports in `io.github.stream29.kodex.cli.newsessiondefaults`:

  ```kotlin
  @Composable fun rememberNewSessionDefaultsDropdowns(): NewSessionDefaultsDropdowns
  @Composable fun BoxScope.NewSessionDefaultsComponent(
      viewModel: NewSessionSettingsViewModel, showOperationFailure: Boolean = true,
  ): Unit
  @Composable fun NewSessionDefaultsPanel(
      viewModel: NewSessionSettingsViewModel, dropdowns: NewSessionDefaultsDropdowns,
      showOperationFailure: Boolean = true,
  ): Unit
  @Composable fun BoxScope.NewSessionDefaultsDropdownMenus(
      viewModel: NewSessionSettingsViewModel, dropdowns: NewSessionDefaultsDropdowns,
  ): Unit
  ```

- For SettingsPopup, create menus once with the helper; put Panel inside the page viewport,
  then the existing independent Title panel. Render DropdownMenus directly in TuiPopupHost,
  after persistent content. Set `showOperationFailure=false` when the host renders shared
  failure once. Hiding dismisses only renderer menu state, not the child or accepted writes.
- Each typed port freezes expected field/requested value synchronously and submits into the
  **original defaults-specific** application queue. Rejected admission is a no-op, not a fabricated
  error. Later CAS false is not persistence; later ordinary failure/success uses the existing
  shared report/clear rules. Preserve unrelated BackendSettings fields during field CAS.
- Main replaces RpcNewSessionSettings's projection/commands with this factory and a narrow adapter.
  The adapter/parent retains queue ownership: close child first, then queue.close() to reject
  new admission and drain frozen accepted payloads in applicationScope. Do not bind that worker
  to the child ownerScope, mix it into Global editor's queue, or inject global/backend storage.
- Main deletes old State/ViewModel declarations before compile, and updates old test doubles
  with `operationFailure` and `dismissOperationFailure()`. Four command signatures remain Unit
  with expectedRevision; catalog-only changes do not stale a command. There is no additional
  local draft, save-success/error store, default-model literal, or title authority.
- Register `:app-component-new-session-defaults-{spec,impl-viewmodel,impl-view}` against
  `app/component/new-session-defaults/{spec,impl/viewmodel,impl/view}`. Parent contract depends
  on spec; RPC adapter on VM; Settings renderer on view. No app-contract-settings/app-view-settings
  dependency. Staged debt remains in borrowed app-shared-settings-contract's existing mixed
  closure (including utils-shell-client-impl); this lane did not expand/migrate that shared root.
- Delivered canonical-state constructor tests, fake typed-port VM tests and full Mosaic renderer
  tests: all fields/menus, baseline capture, no optimistic state, catalog/revision, rejection,
  ordinary failure/cancellation, close/owner cancellation, fake app-queue drain, keyboard/Escape,
  disabled close, shared banner acknowledgement/suppression and independent-sibling layout.
  Real defaults CAS/drain, Settings+Title integration and old suites remain main-owned.
- Static whitespace/import/dependency review completed; central compilation/regression status
  is recorded in the parent batch. This lane performed no Gradle, IDE, commit, push or branch operation.
- Authoritative contract:
  [Defaults spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/NewSessionSettingsViewModel.kt#L49).
  Implementation:
  [VM/factory](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/NewSessionSettingsViewModel.kt#L9),
  [complete renderer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/newsessiondefaults/NewSessionDefaultsComponent.kt#L36).

- fake deps：revision stale/no-op、catalog-only change、不在目录的 model、
  四字段、关闭、普通失败/取消、admission rejection、接受后排空。
- 保留 RpcSettingsTest 的 defaults revision、accepted-after-close、
  visible failure 及无关字段 CAS merge；设置文件编码保持原样。
- 完整 renderer 测试覆盖四字段、菜单选项/selection、keyboard/Escape、
  disabled/closed、与独立 Title child 的同页共存。
- 整合负责人处理 RpcNewSessionSettings 的 adapter 化、父 contract、
  SettingsPopup、队列/source lifetime 和既有 suite；组件线只改新三项目。
- 实际 shared-settings 依赖闭包若仍混合，记录债务，不借本批迁移整个 shared 根。
