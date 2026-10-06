# Task Tree

- `Trace Settings navigation, children, effects, writes, and disposal`()
- `Specify original contract and real implementation ownership`()
- `Receive review of Settings root hard-cutover and lifetime gates`()
- `Receive implementation authorization`()
- `Move the original Settings root and renderer`()
- `Cut over RPC helper consumers and remove old entries`()
- `Validate Settings, Login, queue, and root rendering`()
- `Return evidence for whole-batch acceptance`()
- `Receive scoped acceptance with the parent batch`()

# Details

## Scope and actual owners

- [Parent plan](2026-10-06-hard-migrate-frontend-ownership-roots.md)；用户已授权执行。
- [SettingsViewModel](../../Kodex/app/component/settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/SettingsViewModel.kt)
  拥有 selectedPage、准确 Session target 和 global/session/newSession children；
  [GlobalSettingsViewModel](../../Kodex/app/component/settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/GlobalSettingsViewModel.kt)
  只组合现有编辑/认证/用量组件，不把它们的 mutable state 镜像到根。
- [实际根实现](../../Kodex/app/component/settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/SettingsViewModel.kt)
  直接接管，原 `createSettingsViewModel(initialPage, global, session, newSession)`
  保留原角色；typed child 参数已是依赖端口，不新增 SettingsController。
- [RpcGlobalEditor](../../Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcGlobalEditor.kt)
  是真实 GlobalSettings 实现；由协调者迁入 `app/impl/rpc`，没有第二个默认
  GlobalSettings VM 加 façade 才算独立组件。

## Project and declaration mapping

- `app/contract/settings` → `app/component/settings/spec`；
  SettingsPage、SettingsViewModelArguments/Factory、GlobalSettingsEffect/VM 原 FQCN 保留。
- 原 SettingsViewModelImpl → `app/component/settings/impl/viewmodel`；
  原 SettingsPopup 和测试 → `app/component/settings/impl/view`。
- SettingsUpdateQueue 是 application scope worker，真实消费者为 RpcGlobalEditor 和 RpcNewSessionSettings；
  McpSettingsProjection 是真实 RPC 状态转换。二者原 FQCN 移到 `app/impl/rpc`，
  删除 RPC→Settings VM 的 helper 依赖，不给这两段代码分别造 provider/spec 项目。
- 未被实际装配引用的旧 Settings Koin marker 在核对生成来源后删除；
  组件 convention/其他 child 的真实装配不改。旧三项目不以 forwarding build 保留。
- 消融自查：根 VM 的 selectPage 实际执行 hidePage/refresh/reset dismissal，
  close 执行三 child 清理，所以保留原实现。其参数已能说明依赖，spec 不另添
  SettingsRootDependencies/Controller；queue 是既有 writer worker，不升级成新服务。

## Normative contract and preserved behavior

- child identity 为整个 Settings owner 稳定，Current Session 捕获打开时 target；
  切页不重新选当前 tab，不读全局 selectedAgent 重新绑定。
- 同页/closed 切页 no-op；离开 CurrentSession、MCP、Hooks、Context、General、
  NewSession/OpenAI 分别调用原 hidePage；不得概括成关闭所有 child。
- 初始 OpenAI 或进入 OpenAI 触发原单次 usage refresh；不让 constructor 和
  renderer 再各触发一次。离开 OpenAI 清未确认 logout，reset 消费期间原 dismissal 不生效。
- Settings close 清理三组 child；RPC observation/queue 的真正 owner 按原宿主结束，
  popup close 不变成 backend shutdown。规范列出关闭顺序及异常可观察行为。
- 队列 Accepted 仅为准入，payload/baseline 已冻结；已接受写入能在 popup close 后排空。
  不把 worker 改到 popup scope，不增加“receipt 已持久化”保证或跨关闭重试。
- 一个宿主失败出口；各 child 的 failure authority 不再复制到 Settings 根。
  Login 是原独立短期 child，OpenLogin effect 仍由 Application 创建并捕获 source。
- MCP URL effect 的 consumer 挂在原 retained Settings owner：
  Settings→Login→return 不重建/取消；真正 owner close 或整个 renderer 卸载的行为区分。
  不添加全局 URL manager 或 raw credential StateFlow。
- spec 声明真实 child 依赖、全部 page/overlay render 分支、hide/close/cancel/
  write admission 和实际 `@throws`；renderer 仅持 scroll、dropdown/focus/geometry。

## File ownership and integration

- Settings 工作线只写新 spec/VM/View 及专属测试。
  原共享文件删除、RpcGlobalEditor/RpcSettings/RpcMcpSettings 与 root effect host、
  SettingsLoginLifetimeTest 和宿主 Gradle 由协调者处理。
- Root renderer → Settings View；Settings View → 已有 child View；
  Settings spec → Session spec 与 child spec，不依赖 root Application impl/View。
- SettingsOperationFailureBanner、navigation width/page viewport 是既有 renderer 行为，
  从旧实现完整迁移；重复 visual helper 只在确有多个消费者时保留一份 components 实现。

## Verification gates

- 完整迁移原 SettingsViewModelTest/WorkingDirectoryOwnershipTest 与十套 renderer suites；
  旧真实 root `SettingsLoginLifetimeTest` 不因文件移位被删除。
- 实际点击 General/OpenAI/MCP/Hooks/CurrentSession/NewSession，
  检查单次 refresh、草稿关闭、stable child 与准确 target。
- 用 gated prepare/openURL 验证 Login 往返保留，owner close 只取消捕获 attempt；
  同帧字段编辑、renderer 卸载与 queued persistence 分开断言。
- 场景保留 authentication/source 文案、unknown usage、失败 banner、
  窄 popup、滚动/焦点以及 dropdown/menu layering；不能只测新 ViewModel。
- 共享 queue/MCP helper 的测试归真实消费者项目，原有 private-file/wire 断言不改变。
- 提交 handoff 必须列出每个原文件新位置、唯一声明、RPC 构建边删除、
  host callback 与未验证平台；实际构建由协调者统一执行。

## Integration handoff (lane checkpoint)

- **Lane implementation ready; central validation was pending at handoff.** Early handoff was published before implementation;
  Settings-owned implementation and static parity audit are now complete. Baseline remains
  `refactor/spec` at `034b778c637ee1749368765d307cc53dfe8acd58`.
- Settings line writes only `Kodex/app/component/settings/{spec,impl/viewmodel,impl/view}`
  and this section. Main owns old-source/build deletion, project registration, consumer
  retargeting, RPC, retained-root effects and centralized validation.
- Paths below are relative to `Kodex/`; each `src/` suffix and package stays unchanged:
  - `app/contract/settings/src/` → `app/component/settings/spec/src/`:
    `SettingsViewModel.kt` and `GlobalSettingsViewModel.kt`, both in
    `io.github.stream29.kodex.app.settings.contract`.
  - `app/viewmodel/settings/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/SettingsViewModel.kt`
    → `app/component/settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/SettingsViewModel.kt`.
  - `SettingsViewModelTest.kt` and `WorkingDirectoryOwnershipTest.kt` retain their
    `commonTest/kotlin/io/github/stream29/kodex/app/settings/` suffix in the new VM.
    The two queue-only cases in the former belong in main's RPC tests, not this VM.
  - `app/view/settings/src/` → `app/component/settings/impl/view/src/`:
    full `mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt`.
    All ten original suites retain `mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/`:
    AuthenticationSettingsTest, CodexAccountUsageSettingsTest, GlobalSettingsGroupingTest,
    SessionRenameAdapterTest, SettingsButtonStyleTest, SettingsDropdownFieldTest,
    SettingsOperationFailureBannerTest, SettingsPageTest, SettingsPageViewportTest,
    SettingsPathFieldTest (each `.kt`).
- Unchanged public factory:
  `io.github.stream29.kodex.app.settings.createSettingsViewModel(initialPage: SettingsPage,
  global: GlobalSettingsViewModel, session: SessionSettingsViewModel,
  newSession: NewSessionSettingsViewModel): SettingsViewModel`.
  `SettingsViewModelArguments(target: SessionViewModel, initialPage: SettingsPage)` and
  `SettingsViewModelFactory.create(arguments): SettingsViewModel` retain exact signatures.
  No new dependencies/controller façade or second Global VM is introduced.
- Unchanged renderer:
  `io.github.stream29.kodex.cli.settings.BoxScope.SettingsPopup(viewModel: SettingsViewModel,
  onDismissRequest: () -> Unit, onOpenLogin: () -> Unit)`. Main must preserve exact popup
  dismissal and Login callbacks; URL handling remains under the retained Settings owner
  in `SessionTreeCliScreen`, never duplicated in this renderer.
- New coordinates: `:app-component-settings-spec`,
  `:app-component-settings-impl-viewmodel`, `:app-component-settings-impl-view`.
  Spec uses `kodex.kmp-cli` and `:app-spec-session`; VM uses `kodex.kmp-viewmodel`;
  View uses `kodex.kmp-view`. VM and new renderer RPC fixtures use `:app-test-support-rpc`
  with its main-owned retarget to `:app-impl-rpc`; no VM main dependency on RPC.
  Renderer fixture tests also use `:app-settings-impl-filesystem`. No external-URL
  implementation dependency remains in Settings View.
- Main must move `SettingsUpdateQueue.kt` and `McpSettingsProjection.kt` from the old VM
  into `app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/`,
  preserving FQCNs and tests. Queue consumers include **both RpcGlobalEditor and
  RpcNewSessionSettings**; projection consumer is RpcMcpSettings.
  Remove the old RPC→Settings VM helper edge; RPC uses Settings spec instead.
- Production wiring checked in original `Application.kt`: the factory captures
  `arguments.target`, builds the exact Session source, passes the actual RpcGlobalEditor
  and RpcNewSessionSettings, and directly invokes `createSettingsViewModel`.
  SettingsViewModelModule is only a scan marker; no source consumer or annotated root
  implementation/factory was found. It was not copied; conventions/DI stay unchanged.
- Original declarations remain temporarily untouched by this line. They must be
  deleted by main at integration, not shipped as a second surface.
- No Gradle, IDE, build, platform/UI execution, commits, pushes or resource operations
  are performed by this line. Root SettingsLoginLifetimeTest and helper suites remain
  main-owned validation gates.
- Static audit against the baseline:
  - Both spec files have identical non-KDoc declarations/signatures after stripping KDoc
    and blank lines. Added normative KDoc covers dependencies, stable children, all pages
    and overlays, admission, hide/refresh order, cancellation, close order and exceptions.
  - Root VM and full SettingsPopup are byte-identical to the originals.
    Ten renderer suites, six directory cases and four retained RPC/root cases are
    byte-identical; the two queue cases were separated, not weakened.
  - Main's `app/impl/rpc/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/SettingsUpdateQueueTest.kt`
    now contains the two original queue case names. This line did not edit it.
  - Owned source/build files contain no old project coordinates, queue/projection
    declaration, Koin marker, URL consumer, controller or dependencies façade.
    Whitespace checks passed. Main's concurrent old-build/helper moves are not this line's edits.
- New meaningful regression sources, not executed here:
  - `SettingsViewModelTest.kt`: initial OpenAI refresh once, stable original children,
    closed navigation no-op; real child disposal order, finally cleanup and last exception
    propagation (including an earlier cancellation).
  - `impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopupInputTest.kt`:
    actual pointer press/release through the full renderer and original root with actual
    RpcGlobalEditor/Session/defaults children and real in-memory RPC fixture. Navigates every
    page, checks captured target/stable children, single refresh per entry/same-page no-op,
    and actual Close→child disposal. Frontend fixture uses the renderer interaction dispatcher.
    It does not manually install/invoke navigation callbacks or duplicate URL handling.
- Twenty owned source/build files contain thirty test declarations (three added cases);
  these are inventory counts, not passed-test evidence. JVM/native compilation, execution,
  narrow-screen UI comparison, retained-root Login/URL lifetime and centralized host validation
  remain main-owned and unverified by this line.

## Coordinator final evidence

- 旧 Settings 三项目、Koin marker 和原声明已删除；queue/projection 原 FQCN
  由 RPC 真实消费方接管，不保留转发项目或第二个 Global VM。
- 实际通过：Settings VM 12 项、View 18 项，原 Root View 的 Login/MCP lifetime、
  original popup/recording 与 RPC queue 回归亦通过；整体明细见
  [主任务最终矩阵](2026-10-06-hard-migrate-frontend-ownership-roots.md#final-validation-and-remaining-limits)。
- 新完整 popup 输入用例的 fixture 修正为真正的 rendering coroutine scope，
  不将 TestMosaic 当作 CoroutineScope，不取消借用的外层 scope。
  SettingsPopup 本身与原实现相同；renderer 不另建导航或业务草稿 authority。
- spec/VM/View 的 Linux x64 主源码编译通过；追加真实无账号 CLI
  [局部验收](2026-10-06-hard-migrate-frontend-ownership-roots.md#actual-cli-behavior-acceptance)
  验证七页导航、重命名、目录选择、输入设置、Login Ready/Cancel 和持久化；
  live OAuth/MCP 浏览器执行与其他平台仍未验。
  短期独立工作线 handoff 的 pending 状态由此处中央证据补全。
  用户已随整批验收，按 [验收范围](2026-10-06-hard-migrate-frontend-ownership-roots.md#scoped-acceptance) 归档。
