# Task Tree

- `Inventory exact-target editing, revision checks, child dialogs, and baseline tests`()
- `Specify the complete component and typed dependency ports`()
- `Review cancellation, rename, directory selection, and rendering contracts`()
- `Extract spec, dependency-only ViewModel, and complete renderer`()
- `Integrate Session and Draft adapters with stable parent children`()
- `Validate target replacement, stale callbacks, and queue cancellation`()
- `Record acceptance and archive component task`

# Details

- 实施/集中验证完成；本组件 25 项 JVM 测试通过，宿主准确 target、CAS、
  Rename/CWD 与关闭验证见主任务的 Central validation checkpoint。用户已验收，
  内层代码随批次提交为 `721ac0fe`。

## Scope and source map

- 属于 [五组件批次](2026-10-03-extract-session-configuration-and-browsing-components.md)，
  用户已审阅并授权实现，当前 executable。
- 目标 `app/component/session-settings/{spec,impl/viewmodel,impl/view}`。
- 源：`app/contract/settings/.../SessionSettingsViewModel.kt`、
  `app/viewmodel/settings/.../SessionSettingsViewModel.kt`、
  `RpcSessionSettings.kt`、SettingsPopup 的 Session content/四菜单/rename 接线。
- 完整覆盖 MaterializedSession 与 NewSessionDraft；不是只移动接口文件。
  宿主现仅保留 [稳定 child 的配置入口](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt#L60)。

## Proposed spec and ports

- 单一迁入 Configuration、Snapshot、TargetKind、DataState、ViewModel state、
  directory selection handle、必要 effect；保留原公共包名。
- factory 接受 `SessionSettingsDependencies` 与明确 ownerScope。
  deps 提供准确 source 的只读 snapshot、models、目标写入能力、Rename 与
  Working Directory factory/port、既有 failure reporter；不暴露 GlobalRpc/
  Session registry 或任意“当前 selected Agent”查询。
- source 仍可作为实际借用的 infrastructure port，规范 acquire/close 所有权。
  spec 说明 revision 检查、false 准入拒绝、异常/取消、队列和 exact handle；
  不能把 `selectWorkingDirectory` 返回 true 描述成 CAS 已持久化成功。
- Settings 父级发布稳定 child；renderer 不再保存 rename 业务目标作为唯一 authority。
  Rename 复用已迁移组件，目标/revision 在请求时捕获；不复制名称草稿/校验。
- Working Directory child 继续拥有 browser；准确 selection handle 被消费一次，
  替换或关闭释放旧 child，不建立第二份 picker state。

## Behavior and ownership

- source 绑定一个准确 binding/Draft；Agent 重开导致旧 source Unavailable，
  不重定向新 binding。snapshot revision 非 backend 全局事务号。
- modelOptions 按目录顺序加当前 model 去重；catalog 变化不修改配置。
- 配置要求 writable；Rename 基线仅检查准确 revision 和非空 trim 名称，
  不擅自增加 editable 全局限制。目录弹窗要求准确可编辑 revision。
- 在准入时冻结 configuration/revision；adapter 按实际变更字段合并最新
  settings，保留 runtime-owned/无关字段，不写陈旧整份 KodexAgentSettings。
- **RPC 公共 factory 当前 close 会取消本地 pending CAS/retry，并释放 source**；
  与 defaults 的应用队列排空不同。后端已接受操作不保证回滚。
  内部未取消排空变体若仍有实际消费者，列出消费者并显式建模；否则移除私有遗留，
  不能不加说明地改变公开 factory 的关闭语义。
- Unavailable 渲染明确提示；Available 显示名称/cwd/四字段，editable 控制
  配置与 Browse。菜单保持固定 reasoning 列表、ServiceTier enum 和 Mode enum。
- KDoc 逐一覆盖 require、CancellationException、source failure、closed、
  effect/child 准入、父级与 renderer disposal，禁止承诺成功写盘或后端 rollback。

## Checks and integration handoff

### Integration handoff (component lane, delivered)

- Frozen creation seam in `io.github.stream29.kodex.app.settings`:
  `createSessionSettingsViewModel(dependencies: SessionSettingsDependencies, ownerScope: CoroutineScope): SessionSettingsViewModel`.
  Spec types remain `io.github.stream29.kodex.app.settings.contract`.
- `SessionSettingsDependencies(source: SessionSettingsDataSource, models: StateFlow<List<ModelInfo>>, createDirectoryPicker: (Path) -> DirectoryPickerViewModel? = { null }, reportUnhandledError: ((Throwable, Path) -> Unit)? = null)`.
  Component owns/closes source; borrows models; pending local queue cancels, never drains.
- Retained legacy factory overload:
  `createSessionSettingsViewModel(source: SessionSettingsDataSource, models: StateFlow<List<ModelInfo>>, ownerScope: CoroutineScope, createDirectoryPicker: (Path) -> DirectoryPickerViewModel? = { null }, reportUnhandledError: ((Throwable, Path) -> Unit)? = null): SessionSettingsViewModel`.
- New authoritative `rename: StateFlow<SessionSettingsRename?>`, exact
  `dismissRename(expected: SessionSettingsRename): Boolean`, `hidePage(): Unit`.
  Existing `effects` is compatibility notification only: remove old host effect-driven
  child construction. Render the owned handle; no second rename draft/target.
- `DefaultSessionSettingsViewModelFactory.create(dependencies: SessionSettingsDependencies, ownerScope: CoroutineScope): SessionSettingsViewModel`
  implements the spec factory. Both creation overloads have the same cancellation policy.
- Renderer exports in `io.github.stream29.kodex.cli.sessionsettings`:

  ```kotlin
  @Composable fun rememberSessionSettingsDropdowns(): SessionSettingsDropdowns
  @Composable fun BoxScope.SessionSettingsComponent(viewModel: SessionSettingsViewModel): Unit
  @Composable fun SessionSettingsPanel(
      viewModel: SessionSettingsViewModel, dropdowns: SessionSettingsDropdowns,
  ): Unit
  @Composable fun BoxScope.SessionSettingsOverlays(
      viewModel: SessionSettingsViewModel, dropdowns: SessionSettingsDropdowns,
  ): Unit
  ```

- For SettingsPopup, create menus once with the helper, put Panel inside the page viewport
  and Overlays directly in TuiPopupHost, after persistent content. Render both only for the
  current page. Each renderer removal invokes `hidePage()`; parent navigation should also
  call it when leaving CurrentSession (including frontend-independent navigation).
- Overlays renders the exact `SessionSettingsRename(expectedRevision: Long, viewModel: SessionRenameViewModel)`
  with Labeled presentation and the exact existing `SessionWorkingDirectoryPicker` child.
  Do not keep `renameRequest`, construct a renderer-local rename child, or collect compatibility
  effects to open one. Both dismiss/completion callbacks capture the exact owned handle.
- Main keeps exact RpcSessionSettingsSource/RpcDraftSettingsSource adapters and their changed-field
  merge/CAS rules. Supply a fresh owned browser, or null. Bind the existing reporter; its cwd is
  frozen at admission. Rename remains revision-only, trims, and queues asynchronously.
  `selectWorkingDirectory(...) == true` still means consumed handle, not persistence.
- All existing public commands are retained: four field updates, directory request/select/dismiss,
  requestRename/renameSession, state/directoryPicker/effects and close. Main adds the three new
  interface members (`rename`, `dismissRename`, `hidePage`) to old test doubles.
- Delete the original public declaration file and old VM file before compiling. The unreferenced
  internal `cancelEditsOnClose=false` variant and `UnavailableSessionSettingsDataSource` are not
  retained; repository source search found no external consumers. Replace the old
  SessionRenameAdapter/its renderer-local-child tests with exact VM-owned handle coverage.
- Register `:app-component-session-settings-{spec,impl-viewmodel,impl-view}` against
  `app/component/session-settings/{spec,impl/viewmodel,impl/view}`. Parent contract depends on
  spec; adapters on VM; host renderer on view. None depends on app-contract-settings or app-view-settings.
- Delivered spec constructor tests, dependency-only VM tests and full Mosaic renderer tests:
  both target kinds, catalog order/revision, all fields/menus, disabled controls, exact children,
  trim/stale rename, directory consumption, page hiding, failure/cancellation and source release.
  Real binding/CAS/paced retry tests and shared host tests remain main-owned.
- Static whitespace/import/dependency review completed; central compilation/regression status
  is recorded in the parent batch. This lane performed no Gradle, IDE, commit, push or branch operation.
- Authoritative contract:
  [Session spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/SessionSettingsViewModel.kt#L183).
  Implementation:
  [VM/factories](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/SessionSettingsViewModel.kt#L17),
  [complete renderer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessionsettings/SessionSettingsComponent.kt#L43).

- fake sources：两个 target kind、不可用/不可编辑、目录更换、迟到 revision、
  Rename trim、模型缺失/重复、配置合并、普通失败/取消、close/source release。
- 保留 `WorkingDirectoryOwnershipTest`、SettingsViewModelTest 和
  RpcSettingsTest 的 real CAS、binding replacement、paced retry cancellation。
- 完整 renderer 测试覆盖 identity区、四菜单、disabled、rename/cwd child、
  replacement callback、离页/关闭；Settings 宿主不再镜像 component state。
- RPC source/Draft source 与 Application factory、父 Settings contract、
  SettingsPopup 和所有旧测试迁移由整合负责人处理；组件线只拥有新树。
- 不把当前 Session 参数、new-session defaults、title config 合成同一业务对象。
  Native/JS/CLI 等实际验证结果由主批次集中记录。
