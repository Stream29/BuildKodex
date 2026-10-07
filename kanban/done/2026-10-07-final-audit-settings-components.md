REVIEW READY

# Task Tree

- `Verify the pinned source baseline`()
- `Trace all thirty-eight assigned projects`()
- `Compare current tests and original ownership history`()
- `Publish the independently evidenced report`()

# Details

## Baseline, method and verdict

- Independent read-only source reacceptance; Session 523, 2026-10-07.
- Inner branch `refactor/spec`; HEAD `6b7129fa0e719dde8f88a0034b9d572679984a03`.
  Opening, checkpoint and final checks returned this HEAD and empty inner `git status --short`.
  Outer pre-existing changes were not treated as this audit's work.
- Only this outer child was written, with `apply_patch`. No source changes, other reports,
  builds/tests, IDE, network/browser, credentials, services, temporary files, commits,
  pushes or branch changes. Existing build products were not accepted as fresh evidence.
- Loaded AGENTS, buildkodex-change, kanban/checklist/programmatic-planning workflows,
  ask-user, document/IDE/Gradle guidance; Draft, parent executable, relevant extraction,
  repair and cutover records; frontend/spec-impl/state, Settings/Auth/Usage/RPC, hooks,
  MCP, external-URL and resource-lifecycle checklists.
- [Parent evidence standard](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-final-reaccept-spec-impl-repository.md#L53):
  **R** reviewed static critical path, **D** explicit exception/debt, **B1** confirmed
  current gate/contract failure, **B2** concrete suspicion with an unrun experiment,
  **U** unfinished evidence. None means a new platform/runtime test pass.
- **38/38 static project records: R 33, D 2, B1 2, B2 1, U 0.**
  Pending experiments/platform evidence are separately **U**, not silently approved.
  B1 rows share ONE structural finding; B2 includes cross-line adapter/queue evidence,
  not three independently reproduced UI bugs.
- **Not an unconditional final pass.** Physical declarations/stores are substantially
  cut over; real component contracts are used by production. Current strict
  “no obsolete queue or compatibility” acceptance fails on retained Session Settings
  compatibility residue. Two production-boundary suspicions remain unrun.
- The retained Settings MCP effect owner and Hook/MCP latest-draft transforms are
  present and tested in source. This audit does **not** certify real OAuth, browser
  launch, provider consumption, or native filesystem behavior.
- Checkpoint was published as `IN PROGRESS` after production tracing, before tests/history
  grading. This version is ready for the main reviewer to read and deduplicate.

## Inventory and build closure

- Twelve component triplets plus two persistence projects, from tracked `build.gradle.kts`.
  Each table entry names its actual project, not an inferred package.
- [Registration](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L34)
  maps directory slashes to project-name hyphens and walks actual build files, excluding
  `build/`. A table project therefore maps directly to its named directory.
- All 38 build files were read. Specs use `kodex.kmp-cli`; VM projects use
  `kodex.kmp-viewmodel`, API their own spec; view projects API their spec and place
  renderer dependencies in `mosaicMain`. Renderer tests depend on actual VM implementations.
  No owned spec has a direct project edge to an implementation or Mosaic renderer.
- [CLI convention](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-cli.gradle.kts#L1),
  [VM convention](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-viewmodel.gradle.kts#L1),
  [view convention](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-view.gradle.kts#L1),
  [host and Mosaic source-set targets](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt#L22):
  JVM, linuxX64, linuxArm64, macosArm64, mingwX64. This is declared target closure,
  not resolved dependencies, compilation or all-platform execution.
- The parent's global RPC dependency-substitution caveat still applies; this report
  does not claim direct project-edge inspection proves the whole resolved graph.
- Filesystem uses serialization/Kaml and coroutine filesystem; persistence API exposes
  actual RPC models, flow and Path. Login adds coroutine child-scope implementation
  and renderer external-URL ports. Session Settings adds real rename/working-directory
  children and logging, not another Settings source.
- D1 records over-broad Settings spec and unused renderer main edges. Do not remove
  real read-through binding/resource owners merely because they are adapters.

### Per-project coverage

Each evidence key below resolves to an exact current file URI/line. The linked numbered
chain supplies FQCN, factory, dependency port, real host/RPC/backend, callbacks, test
limitations, history and minimal ablation. Spec-only values are not required to implement
an invented interface.

| # | Actual project | Grade | Current evidence and disposition |
| ---: | --- | --- | --- |
| 1 | `:app-component-settings-spec` | D | [S-spec], [S-global]; chain 1; D1 broad edges, honest original disposal exception policy |
| 2 | `:app-component-settings-impl-viewmodel` | B2 | [S-vm], [S-test]; chain 1; integrates actual ports with B2-Q/B2-D below, not a second source |
| 3 | `:app-component-settings-impl-view` | D | [S-view], [S-input]; chain 1; real UI composition, D1 unused VM main edge |
| 4 | `:app-component-account-usage-spec` | R | [A-spec], [A-value]; chain 2; actual wire/state types, refresh ≠ consume |
| 5 | `:app-component-account-usage-impl-viewmodel` | R | [A-vm], [A-test]; chain 2; borrowed source/local refresh waits |
| 6 | `:app-component-account-usage-impl-view` | R | [A-view], [A-input]; chain 2; genuine refresh/reset input, fake supplier ports |
| 7 | `:app-component-application-preferences-spec` | R | [P-spec], [P-value]; chain 3; actual canonical key/width values |
| 8 | `:app-component-application-preferences-impl-viewmodel` | R | [P-vm], [P-test]; chain 3; transient widths versus queued key |
| 9 | `:app-component-application-preferences-impl-view` | R | [P-view], [P-input]; chain 3; real mouse/key callbacks |
| 10 | `:app-component-authentication-settings-spec` | R | [H-spec], [H-value]; chain 4; summary-only, command-start source semantics |
| 11 | `:app-component-authentication-settings-impl-viewmodel` | R | [H-vm], [H-test]; chain 4; exact confirmation, no credential service |
| 12 | `:app-component-authentication-settings-impl-view` | R | [H-view], [H-input]; chain 4; source/logout/login callbacks, safe summary |
| 13 | `:app-component-context-source-settings-spec` | R | [C-spec]; chain 5; real path policy and exact dialog token |
| 14 | `:app-component-context-source-settings-impl-viewmodel` | R | [C-vm], [C-test]; chain 5; validation/reentrant identity preserved |
| 15 | `:app-component-context-source-settings-impl-view` | R | [C-view], [C-input]; chain 5; real Add/type/submit plus direct-model scenarios |
| 16 | `:app-component-hook-settings-spec` | R | [K-spec]; chain 6; latest-field transform and truthful conflict no-op contract |
| 17 | `:app-component-hook-settings-impl-viewmodel` | R | [K-vm], [K-test]; chain 6; sole invalid editor draft, captured admission |
| 18 | `:app-component-hook-settings-impl-view` | R | [K-view], [K-dialog], [K-input]; chain 6; real same-frame Name/Command burst |
| 19 | `:app-component-mcp-settings-spec` | R | [M-spec], [M-value]; chain 7; safe summary, private handles and captured effect |
| 20 | `:app-component-mcp-settings-impl-viewmodel` | R | [M-vm], [M-test]; chain 7; exact token/job, no secret/OAuth backend owner |
| 21 | `:app-component-mcp-settings-impl-view` | R | [M-view], [M-dialog], [M-input]; chain 7; real same-frame Name/URL burst; root effect mount |
| 22 | `:app-component-new-session-defaults-spec` | R | [N-spec], [N-value]; chain 8; four canonical fields, no title duplication |
| 23 | `:app-component-new-session-defaults-impl-viewmodel` | R | [N-vm], [N-test]; chain 8; read-through revision and frozen field admission |
| 24 | `:app-component-new-session-defaults-impl-view` | R | [N-view], [N-input]; chain 8; four actual menu inputs, menus renderer-local |
| 25 | `:app-component-openai-login-spec` | R | [L-spec], [L-deps], [L-attempt]; chain 9; attempt port is UI capability, not credential owner |
| 26 | `:app-component-openai-login-impl-viewmodel` | R | [L-vm], [L-test]; chain 9; one implementation, typed production construction |
| 27 | `:app-component-openai-login-impl-view` | R | [L-view], [L-render]; chain 9; source-safe copy/effect callbacks; input/runtime U |
| 28 | `:app-component-session-settings-spec` | B1 | [T-spec]; chain 10; B1-C explicit compatibility API remains |
| 29 | `:app-component-session-settings-impl-viewmodel` | B1 | [T-vm], [T-test]; chain 10; B1-C buffered producer/test-only old seam remains |
| 30 | `:app-component-session-settings-impl-view` | R | [T-view], [T-input]; chain 10; exact authoritative children, no compatibility collector |
| 31 | `:app-component-session-title-settings-spec` | R | [I-spec], [I-value]; chain 11; actual title value/default, no new generator interface |
| 32 | `:app-component-session-title-settings-impl-viewmodel` | R | [I-vm], [I-test]; chain 11; UI calls/subscribes only |
| 33 | `:app-component-session-title-settings-impl-view` | R | [I-view], [I-input]; chain 11; checkbox/menus actual input, no title generation |
| 34 | `:app-component-usage-reset-spec` | R | [U-spec], [U-value]; chain 12; nonblank concrete credit/exact confirmation |
| 35 | `:app-component-usage-reset-impl-viewmodel` | R | [U-vm], [U-test]; chain 12; no auto-consume/retry or frontend account owner |
| 36 | `:app-component-usage-reset-impl-view` | R | [U-view], [U-input]; chain 12; real choice/confirm/result with fake supplier |
| 37 | `:app-settings-spec-persistence` | R | [F-back-spec], [F-front-spec]; chain 13; two actual interfaces implemented/used |
| 38 | `:app-settings-impl-filesystem` | R | [F-back], [F-front], [F-interface-test]; chain 13; real factories/codecs/atomic files, no combined-store shim |

## Common production chain — actual host, not test composition

- [CLI main](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L31)
  calls `io.github.stream29.kodex.cli.app.withKodexApplication`, then actual
  `io.github.stream29.kodex.cli.app.SessionTreeCliScreen(application.viewModel, ...)`.
- [Application composition][Host] opens backend and in-memory RPC, opens the frontend
  store at lines 86–87 and `RpcGlobalSettings`. Lines 106–122 implement
  `SettingsViewModelFactory.create(SettingsViewModelArguments)`:
  **the captured target**, not selected tab at callback time, chooses
  `RpcSessionSettingsSource(target.view, frontendScope)` or
  `RpcDraftSettingsSource(target.draft, frontendScope)`.
- This factory calls actual `createSettingsViewModel`, `RpcGlobalEditor`,
  typed `createSessionSettingsViewModel(SessionSettingsDependencies(...), frontendScope)`
  and `RpcNewSessionSettings`. These are not alternative owners selected only by tests.
- [Application popup lifecycle][PopupOwner] creates/install Settings under ownership checks
  (209–223); Login requires `popup === returnTo` (253–267), creates a separate child and
  retains that exact Settings object. Dismiss rejects a foreign/replaced expected object
  (306–318). Login dismissal closes Login only and returns the same Settings;
  tab/application removal closes Login **and retained Settings** (469–480).
- [Root renderer][RootEffect] obtains effect owner from either current Settings or
  `Login.returnTo`, keys by the **Settings popup object**, and mounts precisely one
  `McpSettingsEffects(owner.viewModel.global.mcpSettings, openMcpUrl)` at 123–130.
  It is outside the page/panel branch. The [actual popup branch][RootPopup] supplies exact
  VMs, `dismissPopup(open)` and `openLoginPopup(open)`. No active-tab lookup in dismissal.
- [RPC global source][GlobalSource] initializes Get then separately collects real settings,
  models, authentication, usage and MCP flows (90–117). `projectState` is a
  [read-through projection][ReadThrough]: `value` selects current upstream `value`;
  collection conflates a field. It is **not another editable store**.
- [RpcGlobalEditor][Editor] implements `GlobalSettingsViewModel`. Child creation at
  40–95 uses each declared dependency interface; queued writes share one application-scope
  queue, not per-renderer command copies. Reset and account usage use the same borrowed
  usage source; refresh and consume stay distinct calls.
- [Backend binding][BackendRpc] implements `io.github.stream29.kodex.rpc.contract.GlobalRpc`:
  settings/CAS (26–29), auth/usage Get/Flow (33–36), refresh/consume (37–39),
  OAuth/remove (40–45), MCP operations (46–52). UI subscriptions/calls end here.
  Backend auth/title ownership internals are **the other lane's responsibility**.
- [Backend services][BackendServices] opens the real backend store (52), creates backend
  auth/services, supplies current stored title value to runtime (72), and installs the
  global service. [BackendGlobalState][BackendSettingsBinding] takes
  `BackendSettingsStore`, exposes its actual flow (107), and calls its CAS with validation
  and MCP commit coordination (111–116); MCP updates use that same store (125–135).
- Backend/frontend settings are independent files and scopes. Auth/usage streams are
  not an atomic account lock; this UI adds no confirmation-time account binding.

## Component-by-component chains

### Exact VM factory/implementation identities

These are the fully qualified names of the shortened symbols used in the chains.
Each factory creates the one implementation in its linked current source, not a test-only
replacement. Login and the Settings root use the actual host-bound factory lambdas noted
below rather than an invented `Default*Factory`.

| Component | Actual factory FQCN / binding | Actual implementation FQCN |
| --- | --- | --- |
| Settings | `io.github.stream29.kodex.app.settings.contract.SettingsViewModelFactory` implemented in [Host]; calls `io.github.stream29.kodex.app.settings.createSettingsViewModel` [S-vm] | `io.github.stream29.kodex.app.settings.SettingsViewModelImpl`; global `io.github.stream29.kodex.cli.rpc.RpcGlobalEditor` |
| Account usage | `io.github.stream29.kodex.app.accountusage.DefaultAccountUsageViewModelFactory` [A-vm] | `io.github.stream29.kodex.app.accountusage.DefaultAccountUsageViewModel` |
| Preferences | `io.github.stream29.kodex.app.applicationpreferences.DefaultApplicationPreferencesViewModelFactory` [P-vm] | `io.github.stream29.kodex.app.applicationpreferences.DefaultApplicationPreferencesViewModel` |
| Authentication | `io.github.stream29.kodex.app.authenticationsettings.DefaultAuthenticationSettingsViewModelFactory` [H-vm] | `io.github.stream29.kodex.app.authenticationsettings.DefaultAuthenticationSettingsViewModel` |
| Context sources | `io.github.stream29.kodex.app.contextsourcesettings.DefaultContextSourceSettingsViewModelFactory` [C-vm] | `io.github.stream29.kodex.app.contextsourcesettings.DefaultContextSourceSettingsViewModel` |
| Hooks | `io.github.stream29.kodex.app.hooksettings.DefaultHookSettingsViewModelFactory` [K-vm] | `io.github.stream29.kodex.app.hooksettings.DefaultHookSettingsViewModel` |
| MCP | `io.github.stream29.kodex.app.mcpsettings.DefaultMcpSettingsViewModelFactory` [M-vm] | `io.github.stream29.kodex.app.mcpsettings.DefaultMcpSettingsViewModel` |
| New-session defaults | `io.github.stream29.kodex.app.settings.DefaultNewSessionDefaultsViewModelFactory` [N-vm] | `io.github.stream29.kodex.app.settings.DefaultNewSessionSettingsViewModel` |
| OpenAI Login | `io.github.stream29.kodex.app.settings.contract.OpenAiLoginViewModelFactory` implemented in [Host]; calls typed `io.github.stream29.kodex.app.settings.createOpenAiLoginViewModel` [L-vm] | `io.github.stream29.kodex.app.settings.OpenAiLoginViewModelImpl` |
| Session Settings | `io.github.stream29.kodex.app.settings.DefaultSessionSettingsViewModelFactory` [T-vm] | `io.github.stream29.kodex.app.settings.SessionSettingsViewModelImpl` |
| Session title | `io.github.stream29.kodex.app.sessiontitlesettings.DefaultSessionTitleSettingsViewModelFactory` [I-vm] | `io.github.stream29.kodex.app.sessiontitlesettings.DefaultSessionTitleSettingsViewModel` |
| Usage reset | `io.github.stream29.kodex.app.usagereset.DefaultUsageResetViewModelFactory` [U-vm] | `io.github.stream29.kodex.app.usagereset.DefaultUsageResetViewModel` |

### 1. Settings root

- FQCN `io.github.stream29.kodex.app.settings.contract.SettingsViewModel` [S-spec] (46);
  `SettingsViewModelArguments` (96), `SettingsViewModelFactory` (111);
  `GlobalSettingsViewModel` [S-global] (42). Actual root
  `io.github.stream29.kodex.app.settings.SettingsViewModelImpl` [S-vm] (15)
  implements the former via `createSettingsViewModel` (8); `RpcGlobalEditor` implements
  the latter. Stable `global/session/newSession` are injected real child owners.
- Host is the common production chain. `SettingsPopup` [S-view] collects selected page
  and shared failure; page buttons call `selectPage(candidate)` (74–78), child panels
  render stable handles (84–105), Hook/MCP overlays stay at popup level (115–130).
  Its global effect collector opens Login only (36–39); **it does not collect MCP URLs**.
- Local scroll/menu handles are legitimate visual state. Root navigation (25–36) hides
  the leaving child/dialog, dismisses reset away from OpenAI, and refreshes only entering
  OpenAI; same/closed navigation is a no-op. Initial OpenAI refresh is one request (24).
- Close (38–43) marks closed, then nested finally attempts global/session/defaults;
  last disposal exception wins. Spec @throws accurately allows construction/navigation/
  cleanup failures and cancellation. Global spec expressly documents its existing
  sequential disposal exception limitation (76–91); do **not** rewrite it as ideal
  best-effort cleanup or disk rollback.
- Tests: [S-test] (19 initial refresh/identity; 48 real delegated children plus exception
  spies; 84 independent sources; 109 navigation; 161 exact captured target; 180 separate
  hook/backend writes), [S-cwd-test] (17–117 exact picker replacement/consumption).
  These exercise real RPC/source/store fixtures but navigation is mostly direct-model.
- [S-input] (30–100) mounts full SettingsPopup with actual root/RpcGlobalEditor/session/
  defaults, clicks rendered pages and Close, and checks stable identities/refresh.
  Its fixture constructs the root directly (43–50) and overrides refresh (40–42);
  **not proof of production Application installation or supplier refresh**.
- Original ten renderer suites remain byte-identical to `9272f26f^`, including
  authentication/usage snapshots, grouping/sidebar input, page/viewport/dropdown/path/
  button/failure styles and SessionRenameAdapter. The latter uses direct model commands,
  despite its renderer-module location. Actual [S-original-input] (51) does mouse resize.
- Root VM and SettingsPopup are byte-identical to their pre-cutover implementation files.
  Root tests changed to add actual lifecycle cases; original two queue assertions moved,
  not disappeared, to [QueueTest] (14, 28), with new closed-admission order case (39).
- Minimal ablation: D1 prune unused edges; keep this root and nested disposal semantics.
  Do not replace stable children with a mirror or rebind to latest tab. B2-Q/B2-D
  are integrated-port gates; the root itself is not a duplicate draft owner.

### 2. Account usage

- FQCN `io.github.stream29.kodex.app.accountusage.AccountUsageDependencies` [A-spec] (14),
  `AccountUsageViewModel` (77), `AccountUsageViewModelFactory` (103).
  `DefaultAccountUsageViewModelFactory` [A-vm] (13) → `createAccountUsageViewModel` (19)
  → private `DefaultAccountUsageViewModel` (24), implementing the actual interface.
- `RpcGlobalEditor` (80–89) supplies upstream usage/failure, RPC refresh, and one-way
  `usageReset.show()`. `SettingsAccountUsageState` [A-value] is the actual Serializable
  FQCN under `io.github.stream29.kodex.app.settings.contract`, not a copied transport DTO.
  `snapshotOrNull` retains only the source-provided fallback; no account-isolation algorithm
  is recreated here.
- VM projects display state, never credential/cache ownership; no constructor refresh.
  Each active explicit refresh launches one local wait (48); requestReset (61) delivers
  intent, never consumes. Close (74) stops local observation/waits, not the shared source.
  Dependency @throws preserves cancellation and command/intent failures; Unit refresh
  is not promised to synchronously become Available.
- `io.github.stream29.kodex.cli.accountusage.AccountUsageComponent` [A-view] (30)
  routes callbacks at 32/71/73. Unavailable/loading/failed/redeeming and unknown versus
  zero counts render separately; count-only usage cannot auto-select a reset credit.
- [A-test] verifies repeated refresh and owner close with fake ports; [A-input] (162–182)
  sends actual Enter refresh twice and Tab/reset once to the real VM/renderer.
  Provider account isolation/HTTP and reset redemption are not established by those fakes.
- History: extraction `05a472e9`, source/RPC shape retained through root cutover.
  Minimal ablation: none in owned files; removing local job would lose lifetime isolation.
  Do not merge refresh with consumption to eliminate a “layer.”

### 3. Application preferences

- FQCN `io.github.stream29.kodex.app.applicationpreferences.ApplicationPreferencesDependencies`
  [P-spec] (24), VM (68), Factory (92). `DefaultApplicationPreferencesViewModelFactory`
  [P-vm] (15) → create (21) → private Default implementation (25).
- Actual canonical types [P-value] retain `io.github.stream29.kodex.cli.settings`:
  `SidebarSettings` (minimum/default widths), `SidebarContent`, `NewLineKey`, `SubmitKey`.
  They moved from the shared consolidated declaration, not into parallel values.
- [Preferences binding][ConfigPorts] (136–149) implements the dependency interface:
  widths are frontend-local; each setter uses the **current other width**.
  Newline writes use the actual frontend store through application-scope admission.
  `setSubmitKey` maps to canonical newline (VM 72), no second persisted submit field.
- VM clamps display widths only; negative active setters require and propagate documented
  IllegalArgumentException; synchronous cancellation propagates, ordinary errors report.
  Close detaches local observation; page hide does not undo accepted key writes.
- `io.github.stream29.kodex.cli.settings.ApplicationPreferencesComponent` [P-view] (29),
  actual width callbacks 53–54/94–98 and menu selections 80/85. Renderer menu/focus and
  +/- display snapshot are legitimate buffers, not file ownership.
- [P-test] fake queue drain is not disk proof. [P-input] (43, 88) drives actual resize and
  canonical key menus with mouse/keyboard. [RpcTests] (339) checks widths don't create
  either settings file; (284) checks unrelated global fields survive.
  [P-value-test] (11–39) verifies invariants, canonical pairs/descriptors/wire names.
- History: extraction `05a472e9`, actual value move `206266c8`. Minimal ablation: none;
  leave transient width state and canonical-key conversion, not an invented store port.

### 4. Authentication settings

- FQCN `io.github.stream29.kodex.app.authenticationsettings.AuthenticationSettingsDependencies`
  [H-spec] (15), VM (88), Factory (130).
  `DefaultAuthenticationSettingsViewModelFactory` [H-vm] (16) → create (24) →
  private Default implementation (29).
- [H-value] is the real Serializable `SettingsAuthenticationState` FQCN used by RPC;
  `KodexAuthSource` retains its original `cli.settings` FQCN and codex/kodex wire names.
  No access token, private auth store or source lookup logic enters the component.
- `RpcGlobalEditor` (59–79) supplies selected-source read-through, sanitized summary,
  field-baseline source CAS, `rpc.removeAuthentication(source)` and buffered Login intent.
  Host captures Login destination **when creating Login**, not at dialog construction.
- Reference-identity `AuthenticationLogoutConfirmation` [H-spec] (61) intentionally does
  not freeze source at open. VM confirm (85–108) validates exact handle and captures
  dependency selected source **at command start**, publishes SigningOut synchronously,
  then calls remove. This is original/current documented semantics, not a bug to “fix”
  by binding an account earlier. Source/login commands and disabled renderer controls
  need not have identical admission rules.
- `io.github.stream29.kodex.cli.authenticationsettings.AuthenticationSettingsComponent`
  [H-view] (37) delegates source/requestLogin/logout, exact expected confirmation
  134–138. Text is source-safe, summary only; no old Reload/private-auth direct calls.
- [H-test] (65, 190) covers command-start source/close using fake removal;
  [H-input] (86, 193) mixes direct dialog setup with genuine confirmation/source/menu/
  mouse Sign-in callbacks. [H-value-test] (10) protects descriptor/wire shape.
  These do not verify actual credentials removed or a real login completed.
- History: extraction `05a472e9`; values/backend naming cutover `206266c8`.
  Minimal ablation: none in component; preserve exact confirmation and shared failure.
  BackendAuth lane owns real selected credential source/commit semantics.

### 5. Context sources

- FQCN `io.github.stream29.kodex.app.contextsourcesettings.ContextSourcePathPolicy`
  [C-spec] (26), `ContextSourceSettingsDependencies` (45), VM (110), Factory (144).
  `DefaultContextSourceSettingsViewModelFactory` [C-vm] (14) → create (20) →
  private Default implementation (24).
- [ConfigPorts] `FrontendContextPathPolicy` (39–54) does frontend environment/path
  normalization, not file creation. `RpcContextSourceSettingsDependencies` (56–106)
  implements the actual interface and queues copied field baselines/payloads.
  Stored original custom path remains enable/remove identity.
- VM (67–102) owns the one editable/invalid String draft; policy/validation checks exact
  token again after callbacks, handles reentrant replacement, trims/normalizes at owner,
  and copies list payload before admission. Cancellation stays cancellation; policy/
  admission failure is reported, not mislabelled invalid input.
- `io.github.stream29.kodex.cli.settings.ContextSourceSettingsComponent` [C-view] (29)
  calls add/toggle/remove at 64/74/78/93, owns cursor buffer keyed by token (106),
  sends String changes/save to VM (129/134). It does not normalize or author the list.
- [C-test] (120/138) covers reentrant token protection with fake path policy.
  [C-input] (65) genuine Add/type/Tab+Enter submit; later replacement/invalid-save cases
  include direct VM calls, not all UI input. No real home/path filesystem runtime here.
- History: extraction `05a472e9`; latest production ports moved intact with RPC root.
  Minimal ablation: none; keep policy and token guards, don't turn a widget cursor
  buffer into a second business draft.

### 6. Hook settings

- FQCN `io.github.stream29.kodex.app.hooksettings.HookEditHandle` [K-spec] (30),
  `HookSettingsDependencies` (42), `HookEditorDraft` (67), VM (118), Factory (161).
  `DefaultHookSettingsViewModelFactory` [K-vm] (13) → create (18) →
  private Default implementation (23), sole invalid/editing draft authority.
- [Hook binding][HookPort] implements the declared interface. Capture holds original
  exact hook at open; accepted save closure captures validated hook and baseline before
  releasing dialog handle. The closure edits actual frontend store (24–36).
  Stale/same-name Add or changed baseline is an unchanged no-op, including stale delete
  (42–47); original insertion order and command data survive.
- Current spec 8–27 describes that real no-op behavior; prior audit's “conflict is reported”
  overpromise is no longer present. Accepted means queue admission, not file success.
  Hook Settings never runs a command; Application notification consumer remains separate.
- VM `updateDraft(token, transform)` (83–87) runs transform against **latest exact-token
  draft**. If transform throws, assignment never happens, draft/validation unchanged.
  Save/delete catches ordinary capture/admission failures once, preserves cancellation;
  hide closes only modal resources, accepted application writes can drain.
- `io.github.stream29.kodex.cli.settings.HookSettingsComponent` [K-view] delegates
  dialogs at 61–68. [K-dialog] Name 53–55, types 60–62, command 65–66 all send
  `current.copy(field = ...)` transforms, not a captured whole draft.
  `rememberHookInput` (186–199) is cursor/display synchronization, not source authority.
- [K-input] (23–63) genuinely queues Name key, five Tabs, Command key before a snapshot,
  witnesses both callbacks in the same composition and preserves both fields.
  (105) drives keyboard Save. It uses real VM but fake storage ports.
  [K-test] tests transform exception/latest token; a fake duplicate-rejection scenario
  is **not** proof of actual adapter conflict behavior.
- [RpcTests] (82–146) instead uses actual Hook adapter/store/queue, admits stale actions,
  releases handle, updates original store, drains and checks real no-op/valid Add outcome.
  It uses direct dependency methods, not a renderer click.
- History: extraction `7310a445`, repaired contract/field-burst at `a1adf2f2`.
  Minimal ablation: none for transforms or valid queue; B2-Q is the shared queue's
  terminal-cancellation admission gate, not evidence to restore a renderer-local draft.

### 7. MCP settings

- FQCN `io.github.stream29.kodex.app.mcpsettings.McpSettingsDependencies` [M-spec] (88),
  VM (225), Factory (298), reference-identity handles at 30/40/56, Login attempt (71).
  `McpEditorDraft` (135) preserves invalid and inactive transport fields.
  `DefaultMcpSettingsViewModelFactory` [M-vm] (20) → create (25) →
  private Default implementation (30).
- [Mcp binding][McpPort] implements actual deps, captures private baseline/draft with
  secret `Keep`, queued edit/delete/enable actions and real reconnect/logout/OAuth calls.
  [RpcMcpSettings][McpBinding] owns source operations, not renderer state:
  save (46–66) checks original/destination baselines, handles rename as new identity;
  import (85–132) freezes private candidate/baseline/selected decisions for queued apply,
  independent of modal handle release. Equal config avoids gratuitous reconnect.
- `McpServerSettingsState` [M-value] is safe display projection of actual runtime summaries.
  VM does not resolve existing tokens. [M-parser] moved real parsers to VM owner:
  inactive fields survive editor switching, only selected transport is validated;
  `<keep>` versus replacement, quoted args and duplicate secret names retain explicit rules.
  Validation is not a second OAuth protocol.
- VM latest-token transform (100–104) handles field bursts. OAuth (167–193) owns one
  interaction job per name; effect's cancel closure captures **that job**, not latest
  name lookup. Late preparation cleanup, hide versus close and effect rendezvous differ:
  hide (283) releases dialog/import only; close (305) cancels interaction owner/effects.
- `io.github.stream29.kodex.cli.settings.McpSettingsComponent` [M-view] delegates exact
  token callbacks. [M-dialog] uses latest transforms for name/URL/command/transport and
  OAuth fields (63–126/156); token-keyed visual input (464–477) is legitimate.
  Details show secret names/markers, not stored values; import choices are projections.
- `McpSettingsEffects` [M-view] (114–135) cancels captured operation on opener false,
  failure or collector cancellation. Removing this consumer can cancel that opener,
  but does not itself close the entire Settings VM. Actual consumer is retained root
  [RootEffect], not this short-lived page or SettingsPopup.
- [M-input] (24–62) real same-composition Server name/URL input burst; (135) actual
  transport/edit keyboard paths. Import cases (206+) contain direct VM commands;
  effects (261+) use fake opener. [M-test] (316/344/361) exact job/hide/late preparation;
  parser cases (563+) use actual parser, not supplier credentials.
- [RpcTests] (384) accepted real MCP VM edit survives close through actual backend CAS;
  (417) import survives hide/close, but candidate RPC is overridden; (451) actual MCP
  then Hook queue order; (591) safe summary/equal import direct adapter tests.
- [Retained-owner tests][RootTests] exercise actual root renderer: real MCP Login button,
  auth Sign-in and Escape-return, opener gating, same-name replacement, unmount/remount,
  tab-close disposal (62/110/158). **BoundSettingsRoot** (209–254) substitutes the shown
  Settings/MCP dependency owner while delegating Application; tab/page setup partly uses
  direct commands, OAuth uses fake attempts/openers. This is strong UI lifetime source
  evidence, not an unmocked production MCP authorization chain.
- History: extraction `7310a445`; concrete page-owned consumer and stale whole-draft
  issues repaired at `a1adf2f2`, preserved by byte-identical root renderer tests across
  `9272f26f`. Minimal ablation: do not move consumer back into SettingsPopup or remove
  captured-job closure. U-OAuth remains.

### 8. New-session defaults

- FQCN `io.github.stream29.kodex.app.settings.contract.NewSessionDefaultsDependencies`
  [N-spec] (49), `NewSessionDefaultsViewModelFactory` (91),
  `NewSessionSettingsViewModel` (107). Actual `DefaultNewSessionDefaultsViewModelFactory`
  [N-vm] (9) → `createNewSessionDefaultsViewModel` (16) →
  private `DefaultNewSessionSettingsViewModel` (21).
- `io.github.stream29.kodex.cli.settings.KodexNewSessionSettings` [N-value] is the real
  Serializable four-field value, no fifth duplicated title field. Its original descriptor,
  defaults and question mode survive move.
- [Defaults binding][DefaultsPort] `RpcNewSessionSettings` wraps actual child/resources,
  owns independent application-scope write queue, and closes both child and dependency
  admission. Private `RpcNewSessionDefaultsDependencies` (33) implements real interface,
  field-baseline CAS (43–58) merges other backend fields and reports shared failure.
  It is not an empty forwarding wrapper; remove it only if resource ownership is preserved.
- VM observes borrowed current defaults/models; revision covers only defaults.
  It refreshes synchronously before admission (50–61), so an unscheduled observer cannot
  authorize stale callbacks. Frozen typed field payload and caller-facing Accepted do
  not promise immediate persistence. Child close stops observation/admission, not host's
  already accepted queue; whole app cancellation does not promise a disk flush.
- `io.github.stream29.kodex.cli.newsessiondefaults.NewSessionDefaultsComponent` [N-view]
  (36) passes revision+typed option at 87/92/97/102; dropdown hide on disposal
  (54/83) is visual state only. Closed read-only display is permitted.
- [N-input] (51, 88, 160) genuinely drives four menus, keys/Escape, unmount/close.
  [N-test] (105) fake queued drain isn't persistence; [RpcTests] (268/358) stale callbacks
  reject and real accepted defaults CAS survives close. [N-value-test] (17/31) verifies
  original descriptor/RPC golden/missing question behavior.
- History: extraction `721ac0fe`, canonical value move `206266c8`.
  Minimal ablation: none in child; B2-Q needs real host queue experiment, not deletion
  of local read-through revision or addition of a defaults store.

### 9. OpenAI Login

- FQCN `io.github.stream29.kodex.app.settings.contract.OpenAiLoginViewModel` [L-spec]
  (102), Factory (167), actual `OpenAiLoginDependencies` [L-deps] (14).
  `io.github.stream29.kodex.cli.auth.KodexAuthLoginAttempt` [L-attempt] (12) is the same
  public capability FQCN moved into this spec; authorizationUrl/awaitCompletion/cancel
  do not expose token or source store.
- Actual `io.github.stream29.kodex.app.settings.OpenAiLoginViewModelImpl` [L-vm] (22)
  implements spec. Typed create at 153–157 is used by [Host] (123–130) with
  `OpenAiLoginViewModelFactory`. Lambda overload (146–150) adapts to **the same port and
  implementation**, used in original tests; no second login algorithm/credential source.
- Host freezes `OAuthTarget.OpenAi(settings.settings.value.authSource)` at child creation,
  starts only on explicit user action. [Frontend OAuth binding][OAuthPort] starts local
  listener before RPC preparation, relays one callback to captured backend attempt,
  bounds pending-callback and cleanup waits and cancels exact delivered attempt.
  Backend accepted credential exchange/commit is not frontend authority.
- VM suppresses duplicate start (37), uses positive monotonic local effect id (137),
  cancels late returned/replaced handle (45), matches completion to active identity (62).
  Retry-browser resends the same effect, not another OAuth prepare. Cancellation/close
  terminates local work/handle, cannot revoke committed credentials. Completed copy is
  source-neutral, not “Kodex private store used.”
- `io.github.stream29.kodex.cli.settings.OpenAiLoginPopup` [L-view] (38) delegates
  Start/Retry/Cancel, uses actual external-URL utility, active exact effect id (54–60),
  reports Started/Failed to VM, dismisses exact parent and disposes Login child.
  Browser Started is not authorization completion.
- [L-test] typed-port cases (22/57/83/113/138/159/188) use actual VM with fake handles
  for lazy start/cancellation/late results/retry. [L-original-test] (16/46) checks actual
  VM using adapting lambda overload. [L-render] (34, 69) is a **fake VM render/effect
  contract**, its start/retry/cancel commands no-op; not genuine input acceptance.
  [L-popup-test] (19) is real VM Ready snapshot only.
- [OAuthTests] (28) crosses actual RPC to a captured destination with fixture provider;
  (47) contains actual loopback tests; (81/109/125/143/158/181) fake listener/decorated
  RPC tests preparation, exact cancellation, lost reply and deadlines; (207) directly
  calls VM browser-failure/retry, not actual popup keyboard. None was run here.
- History: extraction `fc9f1936`; neutral completed copy `a1adf2f2`; attempt declaration
  moved/backend-only auth cutover `206266c8`. Old LocalKodexLogin/store tests were retired
  with that backend ownership change, not accepted as current UI auth proof.
- Minimal ablation: unused `RpcGlobalEditor.createLogin` (104–111) can be removed after
  checking consumers; production uses Application factory. Lambda adaptation could be
  consolidated with typed test construction without changing lifetime/algorithm.
  Real Login button→attempt→browser→completion end-to-end remains **U**, not fake-green.

### 10. Session Settings

- FQCN `io.github.stream29.kodex.app.settings.contract.SessionSettingsDataSource`
  [T-spec] (58), `SessionSettingsDependencies` (183), Factory (191), VM (212).
  `DefaultSessionSettingsViewModelFactory` [T-vm] (17) → typed create (24) →
  `SessionSettingsViewModelImpl` (40), sole parent configuration/child owner.
- Production [Host] uses typed dependencies and exact captured real/draft source.
  [Persisted source][SessionPort] `RpcSessionSettingsSource` binds `view.current()`,
  changes local revision only for relevant values, merges changed fields against current
  configuration and preserves runtime-owned fields (70–98). Close releases only binding.
  [Draft source][DraftPort] is a distinct legitimate draft binding, not backend store;
  **B2-D** below questions its full-configuration transform before observer convergence.
- VM queue at 50 is legitimate **cancel-on-close FIFO** for frozen revision/name/cwd/config.
  Worker finally closes the model (78–80), unlike B2-Q shared queue.
  Close cancels source/retries/children, publishes Unavailable; never drains pending edits.
  Accept/select true means handle consumed or admission, not persistence. Browser creation
  failure/cancellation, source failure and validation require clauses have actual @throws.
- `SessionSettingsRename`/`SessionWorkingDirectoryPicker` are **reference identities**;
  expected revision is data, not replacement identity. Rename keeps revision-only admission
  even when config noneditable. VM owns actual rename child; cwd selection consumes exact
  handle before queued attempt, stale callback cannot consume replacement.
- `io.github.stream29.kodex.cli.sessionsettings.SessionSettingsComponent` [T-view] (43)
  routes four menus revision+typed option (72/79/125/130/135/140). Overlays (105+)
  collect authoritative handles, key by object, render `.viewModel` rename (146) and
  `.selection` working directory (155), use captured expected handles.
  Removal calls hidePage (59/115), not close/store cancellation; no compatibility collector.
- [T-test] (35 onward) fake source proves FIFO typed changes; (237) cancellation/retry;
  (298–310) **tests only obsolete effect buffer drainage after close**. [T-input]
  (52/93) actual four menu inputs; (116) actual rename typing/Shift-Enter/Enter;
  (146) actual browser filter/Escape; replacement cases partly direct VM calls.
  Test fixture still uses compatibility creation overload (246), not typed production seam.
- [RpcTests] (226/241) real session source binding expiry/CAS retry close; (561) unrelated
  backend instruction field does not change UI revision and survives rename; (576) draft
  rename stays local before materialization. No competing draft configuration/paused
  collector test covers B2-D.
- History: extraction `721ac0fe` explicitly retained compatibility. This is not newly
  introduced double rename ownership. Current stricter user gate changes its acceptance:
  **B1-C** covers unused stream/buffer/getter/old factory, not valid queue or current view.
  Minimal ablation and original retention evidence are below.

### 11. Session title settings

- FQCN `io.github.stream29.kodex.app.sessiontitlesettings.SessionTitleSettingsDependencies`
  [I-spec] (25), VM (76), Factory (95).
  `DefaultSessionTitleSettingsViewModelFactory` [I-vm] (14) → create (20) →
  private Default implementation (24).
- Actual `io.github.stream29.kodex.cli.settings.SessionTitleSettings` [I-value] (16)
  retains nullable model/default reasoning. Actual
  `io.github.stream29.kodex.cli.sessiontitle.DefaultSessionTitleModel` lives in spec
  [I-default] (6), not a UI-local duplicate default/generator.
- [ConfigPorts] `RpcSessionTitleSettingsDependencies` (109–133) implements actual interface,
  injects original default and merges one title field at a time through shared queued CAS.
  [BackendServices] supplies `store.settings.value.sessionTitle` to actual runtime.
  Frontend neither invokes generator nor writes generated session names.
- VM (56–69) uses current field baseline, closed commands no-op; ordinary errors report,
  cancellation propagates as documented. Enabling/title defaults is independent from
  New Session's four field defaults.
- `io.github.stream29.kodex.cli.settings.SessionTitleSettingsComponent` [I-view] (31)
  delegates checkbox (57) and model/effort choices (85/90). Fixed choices display stored
  custom effort; renderer does not normalize or generate title.
- [I-input] (40/61/82) actual checkbox/disabled-no-dispatch and menu input;
  [I-test] (15/53) default injection/commands with fake ports.
  [RpcTests] (310) actual queued title write persists after page close;
  [I-value-test] (15/29) original descriptor/null-model/default round trips.
- History: extraction `05a472e9`, actual value/default owner move `206266c8`.
  Minimal ablation: none in owned files. Backend title reviewer owns server generator/source;
  no conclusion about live title generation is imported into this UI verdict.

### 12. Usage reset

- FQCN `io.github.stream29.kodex.app.usagereset.contract.UsageResetDependencies`
  [U-spec] (15), Factory (54), VM (86).
  `DefaultUsageResetViewModelFactory` [U-vm] (21) → create (33) →
  private Default implementation (38).
- `UsageResetState` [U-value] is original local UI state, **not a new wire protocol**.
  Some legacy-shaped nullable option/selectedCredit result fields remain values;
  no null/blank auto-credit command path is reachable. There is no frontend account/
  prepare/attempt/replay store.
- [Editor] (52–58) supplies real shared usage, `rpc.consumeUsageReset(creditId)` and
  `rpc.refreshAccountUsage()`. [BackendRpc] dispatches backend admission; selected account
  is checked at command-start, not frozen by frontend confirmation, as explicit spec states.
- VM show/select reads source current value, requires concrete unambiguous ID,
  excludes blank/duplicate IDs. Confirm (82–118) matches **same confirmation object**,
  synchronously marks operation active/Consuming, consumes once; four outcomes definitive.
  Ordinary unknown failure triggers one refresh, never auto-reconsume; known result isn't
  erased by later refresh failure. Retry rebuilds choices and requires another confirmation.
- Continuations ensure current child/job active (91/97 etc); close (155) hides/cancels
  local waits, no backend rollback. Dependency @throws admits remote accepted outcome
  can be unknown after cancellation/error; this is a real guarantee, not exactly-once policy.
- `io.github.stream29.kodex.cli.usagereset.UsageResetDialog` [U-view] (39) branches/key
  by actual state, choice ID (83), captured confirmation `onConfirm(state)` (97),
  no commands during Consuming (101), Retry (116), explicit unknown expiry (145).
- [U-input] (300+) drives actual choice/confirm/result Enter/Tab through real VM with
  fake supplier; (390+) drives failure/retry/fresh choice, no reconsume. Some early branch
  tests only record callbacks/fake states. [U-test] (165/237/261/488/589) exact object,
  typed outcomes, unknown refresh, noncooperative late completion and caller-vs-owner
  cancellation. These do not run supplier redemption.
- [RpcTests] (624/641/667) cover no-details/exact confirmation/unknown refresh through
  a helper whose consume/usage ports are overridden (690); do not call those unmocked
  production backend consumption. UI only calls/subscribes; server reviewer owns auth.
- History: extraction `05a472e9`; actual backend account admission contract retained.
  Minimal ablation: none in active behavior. Do not remove reference identity, explicit
  confirmation, unknown-result refresh, or replace with an auto-picking credit RPC.

### 13. Settings persistence — real interfaces, implementations and files

- FQCN `io.github.stream29.kodex.cli.settings.BackendSettingsStore` [F-back-spec] (29)
  and `CliFrontendSettingsStore` [F-front-spec] (25) are actual public interfaces.
  Private implementation `FileSystemBackendSettingsStore` [F-back] (20–24)
  **implements** the former; `FileSystemCliFrontendSettingsStore` [F-front] (15–19)
  **implements** the latter. No marker interface beside the real concrete store.
- Unique public factories `openBackendSettings` [F-back] (95–100) and
  `openCliFrontendSettings` [F-front] (58–63) return their respective interface after
  reload. App uses frontend factory [Host] (86); BackendServices uses backend factory
  (52), BackendGlobalState/BackendMcpConfigurationStore use its actual flow/CAS.
  No current production/test call to retired combined `KodexSettingsStore`;
  frozen versioned migration models are historical input, not a second active owner.
- Each store owns a process-local instance mutex; no promise of cross-instance/process
  transactional locking. `reload` only reads/publishes. `update` reads latest file,
  invokes transform once under lock, atomically writes even equal values and then publishes.
  Backend CAS reads/publishes actual current, rejects stale expectation without write,
  equal succeeds without write/validation, validates real change once under lock and
  writes/publishes proposal. Business validation/MCP effects stay backend's responsibility.
- [File codecs][F-codecs] (30–79) use private snake-case file DTOs and injected defaults;
  absent field ≠ explicit null, known invalid values fail, unknown fields are ignored on
  read/dropped on normal rewrite. RPC canonical camel-case DTO serializers aren't replaced.
  Compiled shell/default values come from backend environment, no Codex fallback loading.
- [Atomic persistence][F-io] (15–63) wraps decode failures except cancellation, creates
  private unique temporary alongside destination, writes, ensureActive, atomicMove,
  bounded NonCancellable cleanup. Primary failure is retained with suppressed cleanup.
  Cancellation/error after replacement can mean disk committed while flow remains old.
  Neither interface nor factory promises rollback or application queue drainage.
- @throws accurately covers invalid YAML/known fields, IOException, cancellation and
  original transform/validation throwable. CAS publication of actual read before failed
  validation differs from update's previous flow retention; documented/tested, not
  artificially normalized. Cleanup-only failure may propagate after persistence.
- [F-interface-test] (17–35) statically types actual factory references/returned stores
  and StateFlows to interfaces; (43–54) fault-injectable concrete classes implement/use
  those same interfaces. No test passes by exercising legacy combined facade instead.
- [F-cas-test] original stale/equal/sparse/one-winner/concurrency/corrupt/write-cancel
  cases plus current validation skip/exact locked values (100), original throwable (127),
  lost completion (148). [F-store-test] covers actual both files/defaults/sparse/null/full
  credential payload/order; current-file/concurrency/transform/write/cancel cleanup cases
  (294–436) use actual factories with faulting filesystem, not mock stores.
- [F-golden-test] ports valid old codec/default assertions onto actual split stores:
  no legacy/Codex fallback (21), ultra→max (41), missing question defaults (55),
  side-specific canonicalization (74), independent file golden versus RPC shape (101).
  Historical automatic fallback/migration obligations were not resurrected.
- [F-permissions-test] (13) tests real temporary/destination POSIX permissions for both
  files with fault observation. It skips when POSIX view unsupported; source is not Windows
  ACL proof. No test suite was run here; native atomic move/permissions/error behavior **U**.
- History: real codec and I/O helper are byte-identical to `206266c8^`; real stores renamed/
  implemented interfaces in `206266c8`, tests retained and expanded, not green marker tests.
  Minimal ablation: none for mutex, reload-before-return, private DTO codec or temp cleanup.
  They have necessary behavior; do not add empty store wrappers or merge the files again.

## Findings, history and minimal experiments

### B1-C — current strict compatibility/obsolete-buffer gate is not closed

- **Confirmed structural residue, not observed double rename behavior.**
  [T-spec] still exports compatibility picker `.viewModel` getter (135–136),
  `SessionSettingsEffect.RenameSession` (154–174), and `effects` (216–222).
  [T-vm] still exports explicitly named compatibility factory (29–38), owns extra
  buffered effect channel (51/58), produces notification on requestRename (161) and
  closes it on disposal. Real renderer [T-view] uses direct authoritative children only.
- Tracked source search found the active effect's consumer only in [T-test] (305).
  Constructor checks in
  [SessionSettingsContractTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/spec/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/contract/SessionSettingsContractTest.kt#L22)
  protect old DTO invariants, not current production behavior.
  Numerous legitimate component/RPC/root test fixtures still create via old overload.
- History is explicit: [original retention authorization](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-03-extract-session-settings-component.md#L71)
  deliberately kept factory/effects; lines 98–99 prohibited effect-driven second child.
  Thus this was a **D-style historical exception**, not an introduced migration regression.
  Current user final requirement forbids obsolete queues/compatibility, so it cannot be
  reported R or silently grandfathered. Coordinator may retain D only with an explicit
  current acceptance exception, not by inheriting the old done verdict.
- Impact: extra public API/buffer/test maintenance remains without a production duty;
  fails full hard-cutover gate. There is **no evidence** of duplicate renderer draft/source
  ownership: current rename and cwd handles are legitimate and source-owned.
- Minimal ablation: remove compatibility effect API, channel/producer/close and buffer-only
  test/invariant assertions; switch legitimate callers/tests to typed
  `SessionSettingsDependencies`; replace old picker getter calls with `.selection.picker`.
  Retain actual rename/working-directory handles, revision rules, closed checks,
  cancel-on-close FIFO and all genuine UI input/lifetime tests.
- Unrun gate **U-C**: targeted declaration/call-site search and current component/RPC/UI
  suites after that minimal removal; no behavior rewrite or alternate child creation.

### B2-D — draft source can use an observed revision with a full stale configuration

- Real source contract [T-spec] (63–66): only changed fields replace latest; unrelated/
  runtime-owned values survive. Real production [Host] selects [DraftPort] for virtual tab.
- [DraftPort] (156–175) compares revision from its own observer-backed flow, then invokes
  `draft.edit { it.withConfiguration(configuration) }`; full helper [SessionPort]
  (119–123) replaces model/cwd/effort/tier/questions. Source mutex doesn't serialize
  all other writers with revision publication.
- [Actual draft](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionDraft.kt#L67)
  separately locks latest settings and exposes `edit`. Until source collector publishes,
  revision may still match an old UI configuration. A concurrent different-field update
  can therefore be overwritten by the full transform. This is a **static interleaving
  suspicion**, not a reproduced terminal user input/data-loss result.
- History: `RpcSessionSettings.kt` move in `9272f26f` has zero content changes;
  this port behavior predates that physical cutover. [RpcTests] (576) only proves draft
  rename locality, not competing config preservation; fake source tests can't clear it.
- Smallest unrun experiment **U-D**: actual RpcSessionDraft + RpcDraftSettingsSource and
  real Session Settings VM; gate/pause source observation, change tier through the other
  actual draft editor, then submit an old-revision model choice before observation catches
  up. Check either stale rejection or model merge preserving latest tier and runtime fields.
  Repeat same-field conflict and close during wait. Don't fake a successful source CAS.
- Minimal remedy **only if reproduced/confirmed**: compute changed fields against captured
  source baseline, validate those fields against latest draft under its actual edit boundary,
  and merge; or provide truly atomic source revision/edit. Keep single draft owner; no
  optimistic copy or extra queue/service. Coordinate with frontend/RPC lane.

### B2-Q — application write queue admission can outlive a terminal worker

- [Queue] (27–37) rethrows command `CancellationException`, terminating its worker;
  no finally closes/rejects its channel. `submit` (39–47) returns Unit, only checks
  `closed`; [Editor] (98–101) checks its separate popup owner, calls submit and returns
  true. [DefaultsPort] (43–57) likewise reports Accepted after submit.
- If an **individual admitted operation** throws CancellationException while application/
  popup/source owners remain alive, worker can be dead but later input still returns
  admission and queues work without a consumer. `close(onDrained)` invokes callback
  on any worker completion (59–68), not specifically successful drainage.
- This does **not** assert that cancelling whole Application must preserve work or that
  caller cancellation is always command-local: actual scopes normally cooperate.
  That distinction and real reachability are why the finding remains B2.
  Actual global CAS/read subscription failure is a relevant boundary to inject, not an
  invented renderer owner; Session Settings FIFO has its own finally-close and is not
  the same defect.
- Tests [QueueTest] cover normal drain, ordinary failure continuation, closed admission;
  [RpcTests] cover ordinary settings failure/shared banner and accepted close-drain
  lifetimes, not a terminal operation-cancellation followed by new live-owner admission.
- History: original queue behavior moved into RPC with `9272f26f`; not proven to be a
  new migration regression. Valid app-scope close-drain queue must remain.
- Smallest unrun experiment **U-Q**: real RpcGlobalEditor (then defaults variant), actual
  shared queue and decorated real RPC; first queued CAS throws CancellationException
  with parent still active, drain scheduler, then use real Hook/key/default command.
  Assert either admission rejects with meaningful source failure or write actually reaches
  store; ensure no false “drained” success. Include parent cancellation separately.
- Minimal correction if warranted: make terminal worker state reject actual admission,
  surface/adapt its receipt, close pending work honestly; do not suppress cancellation
  indiscriminately or move queue into short-lived renderer. Owned root row B2 records this
  integration risk; source root cause belongs frontend/RPC lane for deduplication.

### D1 — over-broad Settings edges and bounded original cleanup

- [Settings spec build](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/spec/build.gradle.kts#L19)
  still exports working-directory/path-picker/RPC models/MCP/OpenAI/io/serialization
  edges and applies serialization plugin despite its two current root declaration files
  importing child/session contracts and flow, not defining Serializable values.
  Some are redundant transitively; source-reading isn't resolved-variant proof.
- [Settings view build](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/impl/view/build.gradle.kts#L21)
  includes session-rename viewmodel in **mosaicMain** although current SettingsPopup
  doesn't construct rename child; original adapter tests still need actual test implementation.
- These are inherited cutover edge debt, not demonstrated behavior failure or framework
  leakage into owned spec. Minimal ablation: move test-only VM edge to mosaicTest and
  remove unused root spec API/plugin edges one at a time, validating actual compile/API
  closure centrally. Do not introduce empty interfaces to make a paired diagram.
- Existing root/global cleanup exception semantics are explicit in [S-spec]/[S-global]
  and current actual-close test [S-test] (48). Their guarantee is bounded and was preserved;
  do not change “last disposal exception” into an idealized policy in this audit.

### Cross-line documentation signal — coordinator/backend deduplication

- [GlobalRpc KDoc](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/GlobalRpc.kt#L21)
  still describes a review contract without client/server implementation. Actual [BackendRpc]
  implements it and [Host] uses it. This is stale documentation, not missing UI integration.
  Outside the 38 project grades; backend/coordinator should reconcile it once, not add a
  parallel auth/title implementation or count another Settings B1.

## History verification and evidence limits

- Compared tracked Git objects, not modified files. Current SettingsPopup/root VM,
  WorkingDirectoryOwnershipTest, ten original Settings renderer suites and
  SettingsLoginLifetimeTest are **byte-identical** to matching files at `9272f26f^`
  under old paths. New SettingsPopupInputTest and root lifecycle assertions are additions.
- The two original queue tests were moved to actual queue's RPC test project.
  Current queue suite retains ordinary drain/failure tests and adds closed order;
  terminal cancellation remains untested. No removed-test-green claim.
- Codec/I/O helper exact byte comparison against `206266c8^` is equal.
  `BackendSettingsCasTest` retains original bodies plus validation/lost-completion cases
  and model import; `SplitSettingsStoreTest` retains original cases plus cleanup-only failure.
  POSIX permissions suite was rewritten for two actual stores/temporary observation;
  old combined-store test wasn't a current interface conformance test.
- Original consolidated settings values and AuthLoginAttempt came from
  `206266c8^:app/shared/settings/contract/.../KodexGlobalSettings.kt`,
  `app/shared/auth/contract/.../KodexAuthStore.kt`; DefaultSessionTitleModel from
  `app/shared/session-title/.../SessionTitleGenerator.kt`.
  Current originals' FQCN/descriptors/defaults protected by current value/golden tests;
  not all extracted files could be byte-compared by basename because declarations were
  consolidated previously. Don't misreport that as absent history.
- Extraction commits: `fc9f1936` Login; `721ac0fe` Session/defaults; `7310a445` Hook/MCP;
  `05a472e9` config/auth/usage/preferences/title/reset. Repair `a1adf2f2`, pure models/
  real store/auth/title owner move `206266c8`, frontend root move `9272f26f`.
  Historical test commands/results in done records are context, **not fresh acceptance**.
- Historical records consulted:
  [prior independent Settings audit](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-05-audit-settings-account-components.md#L1),
  [Settings lifetime repair](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-05-repair-settings-lifetimes.md#L1),
  [root cutover](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-06-cutover-settings-popup-root.md#L1),
  [models/persistence cutover](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-06-cutover-settings-models-and-persistence.md#L1),
  [auth/backend naming](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-06-cutover-settings-auth-and-backend-naming.md#L1).

## Pending evidence — U, not an implied green

| Gate | Current evidence | Still unverified |
| --- | --- | --- |
| U-C | Confirmed compatibility stream/old seam with no real renderer consumer | Minimal removal/typed callers and targeted test execution; not authorized here |
| U-D | Real draft full-config/revision interleaving suspicion | Paused-collector competing actual draft writers, same-field/close experiments |
| U-Q | Real shared queue terminal-cancellation admission suspicion | Live parent/worker death/next real write and truthful drain receipt |
| U-Login-input | Source-correct actual popup callbacks; fake render contract/Ready snapshot | Genuine Start/Retry/Cancel/completion input against actual Login VM/port chain |
| U-OAuth | Root lifetime/UI tests plus fake listener/fixture RPC source | Actual selected-source OpenAI/MCP browser/listener/provider commit/cancel/return runtime |
| U-Account | UI calls refresh/consume with concrete ID; actual backend dispatcher located | Supplier account isolation/credit consumption and lost-result behavior, backend lane |
| U-Platform | Declared common/Mosaic targets, real persistence code/test source | Fresh JVM/native builds/tests, POSIX/Windows ACL and atomic/error behavior |
| U-Graph | All 38 build files/direct edges/conventions inspected | Resolved variant/substitution/compiler closure, coordinator only |

- No owned project was skipped in static inventory; runtime U entries qualify R and
  cannot be collapsed into a repository “all passed.” BackendAuth/title source-owner
  reacceptance remains the designated backend reviewer’s decision.
- Main reviewer can now independently validate B1-C, decide current D exception policy,
  and deduplicate B2-D/B2-Q with frontend/RPC evidence. This child doesn't authorize fixes,
  experiments or advancing the parent.

## Exact current evidence index

All links are local source/task URIs. Line anchors point to declarations or the discussed
test/operation entry; further line ranges are given in each chain.

[Host]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L55
[PopupOwner]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L209
[RootEffect]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L121
[RootPopup]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L989
[RootTests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SettingsLoginLifetimeTest.kt#L62
[GlobalSource]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSettings.kt#L48
[ReadThrough]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/ReadThroughState.kt#L13
[Editor]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcGlobalEditor.kt#L24
[ConfigPorts]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcConfigurationSettingsDependencies.kt#L25
[HookPort]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcHookSettingsDependencies.kt#L10
[McpPort]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcMcpSettingsDependencies.kt#L12
[McpBinding]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcMcpSettings.kt#L27
[DefaultsPort]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcNewSessionSettings.kt#L16
[SessionPort]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionSettings.kt#L22
[DraftPort]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionSettings.kt#L126
[OAuthPort]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcOAuth.kt#L34
[Queue]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/SettingsUpdateQueue.kt#L15
[QueueTest]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/SettingsUpdateQueueTest.kt#L14
[RpcTests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/RpcSettingsTest.kt#L82
[OAuthTests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/RpcOAuthTest.kt#L28
[BackendServices]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L52
[BackendRpc]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalRpc.kt#L18
[BackendSettingsBinding]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalState.kt#L36
[S-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/SettingsViewModel.kt#L46
[S-global]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/GlobalSettingsViewModel.kt#L42
[S-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/SettingsViewModel.kt#L8
[S-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt#L23
[S-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/SettingsViewModelTest.kt#L19
[S-cwd-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/WorkingDirectoryOwnershipTest.kt#L17
[S-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopupInputTest.kt#L30
[S-original-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/GlobalSettingsGroupingTest.kt#L51
[A-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/accountusage/AccountUsageViewModel.kt#L14
[A-value]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/SettingsAccountUsageState.kt#L12
[A-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/accountusage/AccountUsageViewModel.kt#L13
[A-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/accountusage/AccountUsageComponent.kt#L30
[A-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/accountusage/AccountUsageViewModelTest.kt#L99
[A-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/accountusage/AccountUsageComponentTest.kt#L162
[P-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/application-preferences/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/applicationpreferences/ApplicationPreferencesViewModel.kt#L24
[P-value]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/application-preferences/spec/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/ApplicationPreferencesValues.kt#L14
[P-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/application-preferences/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/applicationpreferences/ApplicationPreferencesViewModel.kt#L15
[P-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/application-preferences/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/ApplicationPreferencesComponent.kt#L29
[P-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/application-preferences/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/applicationpreferences/ApplicationPreferencesViewModelTest.kt#L74
[P-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/application-preferences/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/ApplicationPreferencesComponentTest.kt#L43
[P-value-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/application-preferences/spec/src/commonTest/kotlin/io/github/stream29/kodex/cli/settings/ApplicationPreferencesValuesTest.kt#L11
[H-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/authenticationsettings/AuthenticationSettingsViewModel.kt#L15
[H-value]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/SettingsAuthenticationState.kt#L13
[H-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/authenticationsettings/AuthenticationSettingsViewModel.kt#L16
[H-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/authenticationsettings/AuthenticationSettingsComponent.kt#L37
[H-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/authenticationsettings/AuthenticationSettingsViewModelTest.kt#L65
[H-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/authenticationsettings/AuthenticationSettingsComponentTest.kt#L86
[H-value-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/spec/src/commonTest/kotlin/io/github/stream29/kodex/cli/settings/KodexAuthSourceTest.kt#L10
[C-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/context-source-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/contextsourcesettings/ContextSourceSettingsViewModel.kt#L26
[C-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/context-source-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/contextsourcesettings/ContextSourceSettingsViewModel.kt#L14
[C-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/context-source-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/ContextSourceSettingsComponent.kt#L29
[C-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/context-source-settings/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/contextsourcesettings/ContextSourceSettingsViewModelTest.kt#L120
[C-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/context-source-settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/ContextSourceSettingsComponentTest.kt#L65
[K-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/hooksettings/HookSettingsViewModel.kt#L30
[K-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/hooksettings/HookSettingsViewModel.kt#L13
[K-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/HookSettingsComponent.kt#L19
[K-dialog]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/HookSettingsDialogs.kt#L30
[K-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/hooksettings/HookSettingsViewModelTest.kt#L18
[K-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/HookSettingsComponentTest.kt#L23
[M-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/mcpsettings/McpSettingsViewModel.kt#L88
[M-value]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/McpServerSettingsState.kt#L1
[M-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/mcpsettings/McpSettingsViewModel.kt#L20
[M-parser]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/mcpsettings/McpEditor.kt#L13
[M-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/McpSettingsComponent.kt#L23
[M-dialog]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/McpSettingsDialogs.kt#L40
[M-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/mcpsettings/McpSettingsViewModelTest.kt#L316
[M-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/McpSettingsComponentTest.kt#L24
[N-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/NewSessionSettingsViewModel.kt#L49
[N-value]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/spec/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/KodexNewSessionSettings.kt#L11
[N-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/NewSessionSettingsViewModel.kt#L9
[N-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/newsessiondefaults/NewSessionDefaultsComponent.kt#L36
[N-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/NewSessionDefaultsViewModelTest.kt#L105
[N-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/newsessiondefaults/NewSessionDefaultsComponentTest.kt#L51
[N-value-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/spec/src/commonTest/kotlin/io/github/stream29/kodex/cli/settings/KodexNewSessionSettingsTest.kt#L17
[L-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/OpenAiLoginViewModel.kt#L102
[L-deps]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/OpenAiLoginDependencies.kt#L14
[L-attempt]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/spec/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/KodexAuthLoginAttempt.kt#L12
[L-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/OpenAiLoginViewModel.kt#L22
[L-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/OpenAiLoginPopup.kt#L38
[L-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/OpenAiLoginDependencyTest.kt#L22
[L-original-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/OpenAiLoginViewModelTest.kt#L16
[L-render]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/OpenAiLoginRenderContractTest.kt#L34
[L-popup-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/OpenAiLoginPopupTest.kt#L19
[T-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/SessionSettingsViewModel.kt#L58
[T-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/SessionSettingsViewModel.kt#L17
[T-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessionsettings/SessionSettingsComponent.kt#L43
[T-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/SessionSettingsComponentViewModelTest.kt#L35
[T-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessionsettings/SessionSettingsComponentTest.kt#L52
[I-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-title-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/sessiontitlesettings/SessionTitleSettingsViewModel.kt#L25
[I-value]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-title-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/SessionTitleSettings.kt#L16
[I-default]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-title-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/DefaultSessionTitleModel.kt#L6
[I-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-title-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/sessiontitlesettings/SessionTitleSettingsViewModel.kt#L14
[I-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-title-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SessionTitleSettingsComponent.kt#L31
[I-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-title-settings/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/sessiontitlesettings/SessionTitleSettingsViewModelTest.kt#L15
[I-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-title-settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/SessionTitleSettingsComponentTest.kt#L40
[I-value-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-title-settings/spec/src/commonTest/kotlin/io/github/stream29/kodex/cli/settings/SessionTitleSettingsTest.kt#L15
[U-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/usagereset/contract/UsageResetViewModel.kt#L15
[U-value]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/UsageResetState.kt#L20
[U-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/usagereset/UsageResetViewModel.kt#L21
[U-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/usagereset/UsageResetDialog.kt#L39
[U-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/usagereset/UsageResetViewModelTest.kt#L165
[U-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/usagereset/UsageResetDialogTest.kt#L300
[F-back-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/spec/persistence/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/BackendSettingsStore.kt#L29
[F-front-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/spec/persistence/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/CliFrontendSettingsStore.kt#L25
[F-back]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/FileSystemBackendSettingsStore.kt#L20
[F-front]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/FileSystemCliFrontendSettingsStore.kt#L15
[F-codecs]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/SplitSettingsFiles.kt#L30
[F-io]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/SplitSettingsPersistence.kt#L15
[F-interface-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/cli/settings/SettingsStoreInterfaceTest.kt#L17
[F-cas-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/cli/settings/BackendSettingsCasTest.kt#L31
[F-store-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/cli/settings/SplitSettingsStoreTest.kt#L52
[F-golden-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/cli/settings/SettingsCompatibilityEvidenceTest.kt#L21
[F-permissions-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/impl/filesystem/src/jvmTest/kotlin/io/github/stream29/kodex/cli/settings/KodexSettingsPermissionsTest.kt#L13
