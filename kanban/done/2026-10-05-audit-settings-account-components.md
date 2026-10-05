# Task Tree

- `Trace settings and account component specs through real hosts and renderers`() // complete
- `Review states, dependencies, callbacks, tests and ablation candidates`() // complete
- `Publish evidence and unreviewed coverage`() // complete

# Details

- Parent: [Spec/impl reacceptance](../executable/2026-10-05-reaccept-spec-impl-refactor.md).
- Own review scope: `app/component/{account-usage,application-preferences,
  authentication-settings,context-source-settings,hook-settings,mcp-settings,
  new-session-defaults,openai-login,session-settings,session-title-settings,usage-reset}`.
- Existing host/RPC/shared files may be read to validate wiring, not modified or separately migrated.
- Report only in this file; source, other tasks and shared resources are read-only.
- Apply parent acceptance rules and pinned baseline; no independent fixes or builds.
- Status: **REVIEW READY** for coordinator review; NOT blanket reaccepted.

## Review report

### 1. Baseline, method and result

- Reviewed on 2026-10-05, Session 501, at inner branch `refactor/spec`,
  HEAD **`821b66275aae2f0a9541fa2e9e03023d875c30b5`**.
  Repeated `rev-parse` / `status --short`: pinned HEAD, clean inner worktree.
- Loaded AGENTS, change/checklist/kanban/planning/ask-user/document/IDE-collaboration
  skills; parent and owned task; frontend/spec boundaries; RPC settings,
  authentication and account-usage checklists; relevant archived extraction records.
- Source, tracked-reference searches, build files and Git history only.
  No Gradle/test execution, IDE, source edits, credentials, network/browser,
  commit/push/branch or shared-resource actions. Only this outer report changed.
- All **11 component specs have real implementations and production wiring**.
  No wholly unused Shell-like component spec, old parallel GlobalSettingsState,
  renderer-created replacement business child or duplicate wire DTO found here.
- Three **B1** findings block unconditional clean reacceptance: Hook's promised
  conflict reporting mismatches its adapter; MCP's effect consumer does not cover
  retained Settings lifetime; Login's completion text falsely asserts private-source use.
- One **B2** root risk spans Hook/MCP: whole-draft callbacks copy composition snapshots
  and can overwrite another field before recomposition. Actual input loss was NOT reproduced.
- Inherited behavior is distinguished from migration-introduced promises.
  This report authorizes no repairs, source switches, new abstraction or migration.

### 2. Full classification matrix

- `R`: key static path traced without a blocker; not runtime acceptance.
- `D`: documented staged debt, evidence in §6. `B1`: confirmed contract/render/ownership
  mismatch, not necessarily a newly introduced functional regression.
- `B2`: concrete risk requiring §5's experiment. `U`: coverage not judged.
  `runtime` means §8, not an unread whole component. An em dash is no finding,
  not test proof. Component commits are extraction commits; current links use pinned HEAD.

| Component | R | D | B1 | B2 | U | Extraction commit |
| --- | --- | --- | --- | --- | --- | --- |
| account-usage | static path | — | — | — | runtime | `05a472e9` |
| application-preferences | static path | D-settings | — | — | runtime | `05a472e9` |
| authentication-settings | static path | D-settings | — | — | runtime | `05a472e9` |
| context-source-settings | static path | — | — | — | runtime | `05a472e9` |
| hook-settings | — | — | H1 | HM2 | runtime; burst callbacks | `7310a445` |
| mcp-settings | — | — | M1 | HM2 | runtime; popup transition | `7310a445` |
| new-session-defaults | static path | D-settings | — | — | runtime | `721ac0fe` |
| openai-login | — | D-auth | L1 | — | runtime; real button sequence | `fc9f1936` |
| session-settings | static path | D-session | — | — | runtime; draft concurrency | `721ac0fe` |
| session-title-settings | static path | D-settings | — | — | runtime | `05a472e9` |
| usage-reset | static path | — | — | — | runtime | `05a472e9` |

### 3. Shared production chains

- **H-global** — `io.github.stream29.kodex.cli.rpc.RpcGlobalEditor` implements
  `GlobalSettingsViewModel`. Actual stable children/factories:
  [40–95](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcGlobalEditor.kt#L40).
  Context/title/preferences receive typed adapters; Auth/Usage/Reset receive actual
  RPC ports; Hook/MCP receive captured-write adapters, not placeholder ports.
  [Global contract:20–58](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/contract/settings/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/GlobalSettingsViewModel.kt#L20)
  exposes stable children/shared failure/effects, not another child's mutable business state.
- **H-app** — actual composition:
  [Application.kt:106–130](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L106).
  Captured persisted-session/draft source → real Session child, RpcGlobalEditor,
  real Defaults child; bound zero-argument Login factory fixes source at creation.
  This does not rely on the unused `RpcGlobalEditor.createLogin` convenience method.
- **H-root** — Application captures the exact Settings target at
  [212–225](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L212);
  [openLogin:256–270 / dismissPopup:309–321](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L256)
  validates identity and restores the same retained Settings child. Root replacement
  closes retained children (472–482), not a renderer-local duplicate route.
- **H-settings** — real page/child lifecycle:
  [SettingsViewModel.kt:24–43](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/settings/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/SettingsViewModel.kt#L24).
  Initial/enter-OpenAI Usage refresh; same-page no-op; relevant child hide on leaving;
  Reset dismissal off OpenAI; finally closes global/session/defaults. No per-page reconstruction.
- **V-settings** — production
  [SettingsPopup.kt:30–54](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt#L30)
  owns renderer dropdown handles and global Login/MCP effect handlers.
  [73–135](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt#L73)
  mounts General→Preferences, Context→Context Sources, OpenAI→Auth+Usage+Reset,
  MCP/Hook→their panels/dialogs, CurrentSession→Session, NewSession→Defaults+Title.
  Shared failure renders once; child panels suppress redundant banners.
- **V-root** — real terminal root's exclusive branches:
  [SessionTreeCliScreen.kt:972–1006](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L972).
  Dismiss/openLogin callbacks capture the exact popup `open`.
  [CLI Main.kt:31–42](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L31)
  starts that Application/root renderer; this establishes use, not runtime verification.
- **P-write** — existing application-scope queue:
  [RpcGlobalEditor:29–32,98–102,113–126](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcGlobalEditor.kt#L29).
  Typed field baselines:
  [RpcConfigurationSettingsDependencies:25–35](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcConfigurationSettingsDependencies.kt#L25).
  [RpcSettings:31–44](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSettings.kt#L31)
  paces CAS retries only while the captured field still matches, excluding unknown-error replay.
- **P-read** — [ReadThroughState:13–32](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/ReadThroughState.kt#L13)
  reads current source `.value` and selects/conflates; no second mutable source mirror.
  [RpcSettings:48–79](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSettings.kt#L48)
  keeps persisted settings, models, safe summaries and application-local widths distinct.

### 4. Per-component spec → actual factory/dependencies → host → renderer/callbacks

#### account-usage

- FQCN `io.github.stream29.kodex.app.accountusage.AccountUsageViewModel`;
  [spec:14,77,103](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/accountusage/AccountUsageViewModel.kt#L14).
  [DefaultAccountUsageViewModelFactory / createAccountUsageViewModel:13–24](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/accountusage/AccountUsageViewModel.kt#L13)
  constructs the spec implementation. H-global 80–89 supplies safe usage, actual
  RPC refresh, shared failure and `usageReset.show()` intent.
- [AccountUsageComponent:30–79](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/accountusage/AccountUsageComponent.kt#L30)
  at V-settings 101 sends buttons to that child. Null detail/count-zero branches
  remain distinct (84–101); no frontend fallback/account/reset-credit authority.
- VM 30–45,74–77 stops child observations only; explicit refresh is not silently
  deduplicated (48–58). Reset sends an intent, never chooses/consumes a credit.

#### application-preferences

- FQCN `io.github.stream29.kodex.app.applicationpreferences.ApplicationPreferencesViewModel`;
  [spec:24,68,92](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/application-preferences/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/applicationpreferences/ApplicationPreferencesViewModel.kt#L24).
  [DefaultApplicationPreferencesViewModelFactory / createApplicationPreferencesViewModel:15–23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/application-preferences/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/applicationpreferences/ApplicationPreferencesViewModel.kt#L15)
  is H-global 48–51. Real
  [RpcApplicationPreferencesDependencies:136–149](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcConfigurationSettingsDependencies.kt#L136)
  uses the other CURRENT width; only canonical `newLineKey` is queued to local persistence.
- [Component/Panel/DropdownMenus:29–99](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/application-preferences/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/ApplicationPreferencesComponent.kt#L29)
  supplies V-settings 89–92,121; actual +/- and both key menus call this VM.
  Submit/newline selections map to one value, not independent parallel settings.
- VM 44–72 derives/clamps, not optimistic persistence mirrors. No-op hidePage is
  legitimate without business draft; renderer closes menus. Widths are transient
  application state, not missing persistence. D-settings remains.

#### authentication-settings

- FQCN `io.github.stream29.kodex.app.authenticationsettings.AuthenticationSettingsViewModel`;
  [spec:15,88,130](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/authenticationsettings/AuthenticationSettingsViewModel.kt#L15).
  [DefaultAuthenticationSettingsViewModelFactory / createAuthenticationSettingsViewModel:16–27](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/authenticationsettings/AuthenticationSettingsViewModel.kt#L16)
  receives H-global 59–79. Source selection dismisses Reset then queues typed CAS;
  remove targets source captured at confirmation command admission, not dialog opening.
  Login intent is buffered; H-app fixes destination at child creation.
- [Component/Panel/Overlays:37–59,119–170](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/authenticationsettings/AuthenticationSettingsComponent.kt#L37)
  at V-settings 97–100,124 uses exact logout confirmation; initial focus is Cancel.
  Live source display can change before confirmation; this is intentional baseline behavior.
- VM 85–108 rejects forged/replaced handles and synchronously marks SigningOut.
  Hide/close stops child work, not shared client. Source/Login during signout is
  existing allowed behavior, not an invented lock requirement. D-settings remains.

#### context-source-settings

- FQCN `io.github.stream29.kodex.app.contextsourcesettings.ContextSourceSettingsViewModel`;
  [spec:45,110,144](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/context-source-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/contextsourcesettings/ContextSourceSettingsViewModel.kt#L45).
  [DefaultContextSourceSettingsViewModelFactory / createContextSourceSettingsViewModel:14–22](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/context-source-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/contextsourcesettings/ContextSourceSettingsViewModel.kt#L14)
  is H-global 40–43; real
  [RpcContextSourceSettingsDependencies:56–106](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcConfigurationSettingsDependencies.kt#L56)
  freezes original/list writes and injects existing frontend-home policy (39–54).
  Renderer neither reads files nor substitutes a backend-environment policy.
- [Component/Dialogs:29–34,39–83,99–135](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/context-source-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/ContextSourceSettingsComponent.kt#L29)
  at V-settings 93–95,122 sends toggle/add/remove/save to the actual child.
  Text business draft is VM-owned; token-keyed widget state synchronizes to it.
- VM 67–102 rechecks token after reentrant validation. Invalid/rejected admission
  retains draft; accepted hides without persisted-success claim. Single-string
  widget buffer is not HM2's cross-field copy risk.

#### hook-settings

- FQCN `io.github.stream29.kodex.app.hooksettings.HookSettingsViewModel`;
  [spec:39,114](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/hooksettings/HookSettingsViewModel.kt#L39).
  [DefaultHookSettingsViewModelFactory / createHookSettingsViewModel:13–21](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/hooksettings/HookSettingsViewModel.kt#L13)
  reaches H-global 93–95 and actual
  [RpcHookSettingsDependencies:17–47](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcHookSettingsDependencies.kt#L17).
  It captures original, freezes queue intent, never executes shell commands, and
  admitted values no longer depend on the released editor.
- [Component/Panel/Dialogs:19–70](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/HookSettingsComponent.kt#L19)
  at V-settings 104,135 passes exact dialog tokens. Business draft moved to VM;
  no unused staged child under an old renderer-local save implementation.
- VM 89–121 retains validation/rejection and clears on accepted admission,
  correctly not claiming persistence. H1 is eventual conflict reporting;
  HM2 is snapshot construction, not missing factory use.

#### mcp-settings

- FQCN `io.github.stream29.kodex.app.mcpsettings.McpSettingsViewModel`;
  [spec:88,223,289](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/mcpsettings/McpSettingsViewModel.kt#L88).
  [DefaultMcpSettingsViewModelFactory / createMcpSettingsViewModel:20–28](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/mcpsettings/McpSettingsViewModel.kt#L20)
  is H-global 90–92; real
  [RpcMcpSettingsDependencies:21–79](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcMcpSettingsDependencies.kt#L21)
  captures private config, substitutes `<keep>`, binds exact server OAuth target,
  explicitly reads import and delegates actual backend commands.
- [RpcMcpSettings:40–66,85–132](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcMcpSettings.kt#L40)
  compares captured private baselines/selected import values and merges intended
  changes. Released previews don't invalidate accepted payloads. Sanitized runtime
  flow is projection, not another draft authority.
- [Component/Panel/Dialogs/Effects:23–132](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/McpSettingsComponent.kt#L23)
  feeds V-settings 43,103,134. List/editor/import decisions reach real VM;
  opening failure cancels exact job, not name lookup. M1/HM2 remain.
- OAuth survives hidePage; close stops child waits/reads, never admitted writes.
  Filtering preserves hidden-row import decisions; parser/admission remain actual.

#### new-session-defaults

- FQCN `io.github.stream29.kodex.app.settings.contract.NewSessionSettingsViewModel`;
  [spec:49,91,107](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/NewSessionSettingsViewModel.kt#L49).
  [DefaultNewSessionDefaultsViewModelFactory / createNewSessionDefaultsViewModel:9–19](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/NewSessionSettingsViewModel.kt#L9)
  is actually constructed via
  [RpcNewSessionSettings:16–69](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcNewSessionSettings.kt#L16)
  at H-app 120. Delegation is to the real child; wrapper owns typed writes/queue disposal.
- [Component/Panel/DropdownMenus:36–103](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/newsessiondefaults/NewSessionDefaultsComponent.kt#L36)
  at V-settings 107,128 carries rendered revisions for four menus. Title is separate,
  not the fifth defaults field. New draft initialization reads source at H-app 134–139.
- VM refreshes upstream before stale-revision check (45–62), no optimistic persistence
  mirror. Adapter close stops admission but drains accepted field-specific queue
  writes, unlike Session. D-settings remains; wrapper is not semantically empty.

#### openai-login

- FQCN `io.github.stream29.kodex.app.settings.contract.OpenAiLoginViewModel`;
  [spec:97–100,167](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/OpenAiLoginViewModel.kt#L97);
  [typed Dependencies:7–29](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/OpenAiLoginDependencies.kt#L7).
  `OpenAiLoginViewModelImpl` implements the spec; typed
  [createOpenAiLoginViewModel:153–157](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/OpenAiLoginViewModel.kt#L153)
  is used by H-app's bound `OpenAiLoginViewModelFactory`. H-root owns exact returnTo.
- [OpenAiLoginPopup](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/OpenAiLoginPopup.kt#L25)
  is V-root 1003. Start/retry/cancel are real callbacks; effect results carry active
  IDs, retry reopens same pending URL, disposal closes child. L1 is Completed text.
- VM 37–108 owns lazy preparation, sequence, handles, late-result guards and cancel.
  [RpcOAuth:34–112](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcOAuth.kt#L34)
  owns listener/redirect target, sanitized errors, wait timeout and bounded cleanup.
  Launch success ≠ protocol completion; dependency completion ≠ source switch.
  Caller/owner cancellation and destination remain the real typed path. D-auth remains.

#### session-settings

- FQCN `io.github.stream29.kodex.app.settings.contract.SessionSettingsViewModel`;
  [spec:58–91,183–222](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/SessionSettingsViewModel.kt#L58).
  [DefaultSessionSettingsViewModelFactory / typed createSessionSettingsViewModel:17–27](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/SessionSettingsViewModel.kt#L17)
  constructs `SessionSettingsViewModelImpl`; H-app injects exact
  `SessionSettingsDependencies(source, models, directoryPicker)`.
- [RpcSessionSettingsSource:21–97 / RpcDraftSettingsSource:125–177](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionSettings.kt#L21)
  preserve captured binding/revision, persisted changed-field CAS and original draft
  edit path. `git diff 721ac0fe^ 721ac0fe -- <this path>` is empty; draft concurrency
  wasn't independently rewritten by extraction.
- [Component/Panel/Overlays:43–160](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessionsettings/SessionSettingsComponent.kt#L43)
  at V-settings 105,131 carries revision in menus, renders exact owned Rename and
  WorkingDirectory children, keys callbacks by those handles. Removal hides, not closes.
- VM 94–122 freezes revision/configuration/cwd into local FIFO; 125–174 validates
  directory/Rename identity. Close 181–190 cancels local CAS waits/releases source;
  DOES NOT drain like Defaults. D-session is retained compatibility surface,
  not a second production Rename draft or target.

#### session-title-settings

- FQCN `io.github.stream29.kodex.app.sessiontitlesettings.SessionTitleSettingsViewModel`;
  [spec:25,76,95](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-title-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/sessiontitlesettings/SessionTitleSettingsViewModel.kt#L25).
  [DefaultSessionTitleSettingsViewModelFactory / createSessionTitleSettingsViewModel:14–22](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-title-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/sessiontitlesettings/SessionTitleSettingsViewModel.kt#L14)
  is H-global 44–47;
  [RpcSessionTitleSettingsDependencies:109–133](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcConfigurationSettingsDependencies.kt#L109)
  injects canonical DefaultSessionTitleModel/catalog and separate field baselines.
- [Component/Panel/DropdownMenus:31–91](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-title-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SessionTitleSettingsComponent.kt#L31)
  at V-settings 108–110,129 routes toggle/model/effort to this child. Unsupported
  current model remains displayed; null uses injected default without persisting
  another effective-model authority.
- No-op hidePage has no business draft to discard; renderer dismisses menu handles.
  Disabled UI is not an invented ban on programmatic commands. P-write/shared
  failure semantics remain. D-settings remains.

#### usage-reset

- FQCN `io.github.stream29.kodex.app.usagereset.contract.UsageResetViewModel`;
  [spec:15,54,86](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/usagereset/contract/UsageResetViewModel.kt#L15).
  [DefaultUsageResetViewModelFactory / createUsageResetViewModel:21–36](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/usagereset/UsageResetViewModel.kt#L21)
  is H-global 52–58: actual `rpc.consumeUsageReset(creditId)` / refresh, same safe summary.
- [DialogHost/Dialog:39–139](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/usagereset/UsageResetDialog.kt#L39)
  at V-settings 125 submits exact rendered confirmation reference (`onConfirm(state)`,
  92–98), not latest ID or equality-copy.
- VM 54–117 requires detailed/nonblank credit, marks active synchronously, consumes
  once, preserves actual outcome, refreshes once on unknown result and never auto-retries.
  Generation guards 128–159 reject late completion after dismiss/close. Confirming
  state retains original package identity; no duplicate wire credit protocol introduced.

### 5. Findings, provenance and minimum discriminating checks

#### H1 — B1: Hook promises eventual conflict reporting the real adapter does not implement

- [Admission KDoc:8–13](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/hooksettings/HookSettingsViewModel.kt#L8)
  promises eventual conflicts/unknown failures in application failure flag.
  Handle KDoc 21–25 also preserves existing stale/same-name rules.
  These promises do not consistently describe the actual stale conflict behavior.
- [Real adapter:27–36,42–47](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcHookSettingsDependencies.kt#L27)
  returns unchanged settings for duplicate Add or stale original edit/delete,
  without reporting conflict; queue admission returns Accepted.
  [CliFrontendSettingsStore:34–39](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/shared/settings/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/CliFrontendSettingsStore.kt#L34)
  transforms, writes and publishes unchanged settings normally.
  P-write 98–102 then clears failure; Hook VM 102–120 already hid draft/confirmation.
  This is confirmed reporting-contract mismatch, NOT a claim that Accepted promises persistence.
- History: `git show 7310a445^:app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcGlobalEditor.kt`,
  lines 153–165, has identical silent stale no-op. `7310a445` introduced contradictory
  guarantee, not a new stale-target overwrite.
- Future authorized test: real local store/adapter with Hook `x`; Add another `x`,
  drain queue. Observe closed dialog, unchanged persisted value, no failure.
  Repeat edit/delete after replacing captured original. Start with failure=true
  to check unconditional acknowledgement after no-op. No shell execution needed.
- Minimal reconciliation: decide whether preserved stale-no-op rules or new reporting
  guarantee is intended. If preserving baseline, narrow contradictory KDoc and add
  real-adapter coverage; if reporting mandated, change only conflict outcome/reporting.
  Do not silently invent/change storage behavior to satisfy an ideal API.

#### M1 — B1: MCP handler lifetime ends while its retained Settings owner remains alive

- [Spec:201–205,215–220](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/mcpsettings/McpSettingsViewModel.kt#L201)
  and [frontend checklist:26](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/frontend-application-boundary.md#L26)
  require Settings-lifetime single handler; hide retains OAuth.
  V-settings correctly places handler outside page switch, but inside root popup switch.
- H-root 256–270 retains Settings in Login.returnTo; 309–321 restores that same,
  still-open child. V-root 977–1006 removes SettingsPopup during Login.
  Hence its MCP consumer is absent while Settings owner remains alive.
  This mount/owner mismatch is confirmed statically.
- [MCP VM:167–192](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/mcpsettings/McpSettingsViewModel.kt#L167)
  sends through rendezvous effect channel (41–43) before awaiting completion.
  Preparation finishing during Login cannot deliver/open until Settings returns.
  An opener suspended at removal is cancelled by
  [McpSettingsEffects:117–125](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/McpSettingsComponent.kt#L117),
  cancelling exact MCP work despite retained owner.
  Actual frequency, timeout and user-visible failure are unverified; NOT a claim
  that every ordinary immediate browser launch fails.
- History: `7310a445` introduced component/effect contract; exclusive Login and retained
  Settings host existed already. Integration lifetime inconsistency, not new backend
  OAuth target/protocol defect.
- Future discriminating test: gate fake MCP preparation, drive actual root
  Settings→OpenAI→Login rendering, release preparation with Login still open and
  observe effect delivery. Separately gate opener and check removal cancels exact job.
  No browser/network/listener needed.
- Minimum ablation: mount existing single handler with actual Settings owner,
  including retained returnTo, without duplicate consumers/global URL authority.
  Alternatively explicitly agree on hidden-parent authentication suspension policy.
  User decision required; no generic service abstraction is needed or proposed.

#### L1 — B1: Completed Login renderer asserts source selection completion does not imply

- [Renderer:126–135](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/OpenAiLoginPopup.kt#L126)
  says “Sign-in complete. Kodex is now using its private credentials.”
  H-app 123–130 fixes `OAuthTarget.OpenAi(selectedAuthSource)` at creation, including
  Codex; completion does not switch source.
  [KodexAuthSource:90–99](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/shared/settings/contract/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/KodexGlobalSettings.kt#L90)
  explicitly distinguishes Codex storage and Kodex private auth.yml.
- [Attempt contract:27–36](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/shared/auth/contract/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/KodexAuthStore.kt#L27)
  says awaitCompletion does not imply source selection. A successful Codex-target
  login therefore gets incorrect renderer text. Actual destination capture remains used.
- History: `git show fc9f1936^:app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/OpenAiLoginPopup.kt`,
  lines 113–122, contains same wording. Inherited renderer misinformation, not
  refactor-introduced credential/source persistence regression. Still not truthful
  render semantics at reacceptance baseline.
- Minimum ablation: neutral “Sign-in complete.” No source-query service, DTO, forced
  Kodex switch or new VM source authority required. Later test binds Codex-target
  fake completion and rejects private-source assertion.

#### HM2 — B2: Hook/MCP whole-draft callbacks copy a stale composition snapshot

- Hook [dialog:38–66](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/HookSettingsDialogs.kt#L38)
  captures editor.draft and sends `draft.copy(name/types/command=...)`.
  MCP [dialog:46–131,148–154](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/McpSettingsDialogs.kt#L46)
  does same across all fields/transport/OAuth.
  Child updateDraft replaces entire supplied value:
  [Hook:83–86](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/hooksettings/HookSettingsViewModel.kt#L83),
  [MCP:100–104](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/mcpsettings/McpSettingsViewModel.kt#L100).
- Concrete path: two different field callbacks before recomposition; second whole
  copy carries old first field, replacing just-updated VM value. Same token guard
  does not detect stale draft. Actual scheduling allowing input loss NOT reproduced:
  B2, not a confirmed ordinary typing failure.
- [TextInput:173–198](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TextInput.kt#L173)
  updates callbacks from latest composition, not latest VM value after every command.
  Token-keyed widget synchronization exists (Hook 185–200; MCP 462–476).
  Cursor/undo buffers are legitimate renderer state, NOT parallel business authority.
  Whole multi-field copy is the suspected fault.
- History: pre-`7310a445` Hook used separate widget buffers read at Save; VM draft
  migration introduced these snapshot copies. Reason to test, not undo child ownership.
- Future test: pause recomposition; dispatch actual name edit then command/type/
  transport edit; resume and Save through renderer; assert both changes survive.
  Snapshot waits between inputs do not distinguish risk.
  If reproduced, minimally apply field changes against current token-bound draft,
  without introducing another draft authority. No blanket rename/API recommendation.

### 6. Documented debt and minimum duplicate-surface ablation

- **D-settings** — Preferences/Auth/Title/Defaults consume mixed
  `app-shared-settings-contract`; its
  [build:8–15](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/shared/settings/contract/build.gradle.kts#L8)
  still depends on `utils-shell-client-impl`, and
  [declarations](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/shared/settings/contract/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/KodexGlobalSettings.kt#L12)
  include shell types/later in-memory store. Explicit allowed debt:
  [archived batch:208–213](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-02-extract-settings-configuration-and-account-components.md#L208).
  Real ports are used; unchanged mixed root isn't independently failed migration.
  Do not claim transitive pure spec graph; future shared splitting remains frozen.
- **D-auth** — Login consumes actual `io.github.stream29.kodex.cli.auth.KodexAuthLoginAttempt`,
  not duplicated protocol, but its legacy module contains
  [InMemoryKodexAuthStore:39–57](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/shared/auth/contract/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/KodexAuthStore.kt#L39).
  Explicit recorded exception/bound factory:
  [archived Login task:32–47](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-02-extract-openai-login-component.md#L32).
- **D-session** — production-unused compatibility notification:
  [spec:154–173,216–222](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/SessionSettingsViewModel.kt#L154)
  still declares effects; impl 51,58,161,185 produces/buffers it.
  Renderer uses authoritative `rename`, never effects to construct another child.
  Browser compatibility getter (spec 135–136) and old factory overload (impl 29–38)
  remain mainly for tests/compatibility. H-app uses typed dependencies.
  [Explicit retention:71–78,98–99](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-03-extract-session-settings-component.md#L71).
  Later minimum ablation, after public compatibility review: remove notification
  channel/producer/drain-only test, redundant getter/overload if no external callers.
  Do not remove owned Rename child or relabel authorized retention a new bypass.
- Additional unused convenience construction seam:
  [RpcGlobalEditor.createLogin:104–111](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcGlobalEditor.kt#L104).
  No tracked caller found; production uses H-app factory. Predates extraction;
  later deletion candidate, not proof actual Login contract is unused.
- Preferences/Title no-op hidePage represents absence of business draft; Defaults
  wrapper actually owns queue resources; source projections, sanitized runtime
  state, menu handles and cursor buffers weren't falsely called duplicate authority.
- Read all 33 component build files. Specs have no Mosaic/terminal/direct I/O
  dependency. VM implementations use own specs; production views use specs/shared
  UI (Session also uses real owned child topics). Renderer-test VM dependencies
  are not production reverse edges. Scoped direct check, not whole-repository purity.

### 7. Test-source evidence and limitations

- **No tests run here.** Prior green counts are historical execution evidence only.
  Real VM with fake dependency, manually called VM and actual renderer-driven
  callback establish different slices; none alone proves full host→RPC integration.
- Compared old/current hosts/tests and extraction diffs at `fc9f1936`, `7310a445`,
  `721ac0fe`, `05a472e9`. Old API removal was checked in code, not inferred from counts.

| Component | Meaningful evidence read | Missing or inadequate evidence |
| --- | --- | --- |
| account-usage | [ComponentTest:162–233](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/accountusage/AccountUsageComponentTest.kt#L162): actual keyboard→real VM/fake ports, reset intent, null-count vs zero, failure ack. [VM:99–133,185–217](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/accountusage/AccountUsageViewModelTest.kt#L99): refresh/owner cancel. | Not root Usage→Reset→real supplier flow. |
| application-preferences | [ComponentTest:43–64,88–119](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/application-preferences/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/ApplicationPreferencesComponentTest.kt#L43): actual keys/mouse menus. [RpcSettingsTest:232–245](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/RpcSettingsTest.kt#L232): widths not persisted. | UI fake ports and RPC manual VM calls are separate slices. |
| authentication-settings | [ComponentTest:86–115,193–224](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/authenticationsettings/AuthenticationSettingsComponentTest.kt#L86): live source then actual confirm, source/Login intent menus. [VM:65–136](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/authenticationsettings/AuthenticationSettingsViewModelTest.kt#L65): identity/admission source. | No root Login completion/source-text assertion; manual source/Login-during-signout isn't renderer proof. |
| context-source-settings | [ComponentTest:46–114](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/context-source-settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/ContextSourceSettingsComponentTest.kt#L46): real add/type/save/remove slice; VM suite covers reentrant validation/cancel. | Later validation/rejection display tests call VM manually; not real-adapter rejected save through UI. |
| hook-settings | [ComponentTest:60–79](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/HookSettingsComponentTest.kt#L60): real type/Save, waits between fields. [VM:74–111](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/hooksettings/HookSettingsViewModelTest.kt#L74): original capture/injected Rejected. | Fake records/accepts pairs (247–279), not H1 duplicate/stale real adapter. Waits avoid HM2. RPC successful after-close (381–392) doesn't prove conflict reporting. |
| mcp-settings | [ComponentTest:92–119,163–194,218–252](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/McpSettingsComponentTest.kt#L92): real field typing; manual import commands; fake opener effect. [RpcSettingsTest:483–514](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/RpcSettingsTest.kt#L483): captured import/save. | OAuth hide test keeps Effects mounted while manually calling hidePage (244); not actual Login root removal. Retry/toggle/apply include manual VM; no HM2 full burst save. |
| new-session-defaults | [ComponentTest:51–109,160–198](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/newsessiondefaults/NewSessionDefaultsComponentTest.kt#L51): actual menu mouse/key, unmount hides without close. [VM:105–147](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/NewSessionDefaultsViewModelTest.kt#L105): fake queue drains; RPC 251–275 separately real after-close CAS. | No whole-root UI→persistence→new-draft initialization test. Fake drain alone is not RPC proof. |
| openai-login | [RenderContractTest:24–95](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/OpenAiLoginRenderContractTest.kt#L24): branch snapshots, exact active effects, actual removal. VM/dependency tests cover preparation/retry/stale/cancelled late attempt. | Fake start/retry/cancel are no-op (114–116); Completed asserts only “Sign-in complete”, misses L1. [PopupTest:19–45](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/OpenAiLoginPopupTest.kt#L19) is real VM Ready snapshot, not button interaction. |
| session-settings | [ComponentTest:121–161,187–215](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessionsettings/SessionSettingsComponentTest.kt#L121): real Rename/Browse keys/text; replacement partly manual. [ComponentViewModelTest:82–125,298–310](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/SessionSettingsComponentViewModelTest.kt#L82): frozen revision/handle, compat drain. | Drain test doesn't show production effects use. RPC/source suites manually drive VM; no concurrent draft renderer test or platform picker I/O run. |
| session-title-settings | [ComponentTest:63–103](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-title-settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/SessionTitleSettingsComponentTest.kt#L63): actual model/effort menus; earlier disabled/custom/catalog cases. [RpcSettingsTest:177–231](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/RpcSettingsTest.kt#L177): fields/admitted close. | Separate UI/RPC slices, not full root persistence/title-generation acceptance. |
| usage-reset | [DialogTest:300–356,395–447](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/usagereset/UsageResetDialogTest.kt#L300): real VM/key consumption/failure needs new choice. [VM:165–325,488–589](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/usagereset/UsageResetViewModelTest.kt#L165): identity/outcome/late noncooperative/caller-not-owner. | [RpcSettingsTest:517–579](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/RpcSettingsTest.kt#L517) uses resetForRpc and overridden operations; exact-ID/no-replay slice, not real supplier redemption through root renderer. |

- Application VM tests cover Login restoration/disposal. No tracked application-view
  test reference found driving SettingsPopup/openLoginPopup/McpSettingsEffects through
  the actual root transition. Missing integration evidence is separate from confirmed M1.
- DTO use is real, not just Gradle wiring:
  [GlobalRpc:3–5,83–102](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/GlobalRpc.kt#L3)
  imports/returns/streams auth+usage; refresh/consume/remove at 111,126,193;
  [BackendGlobalRpc:33–45](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalRpc.kt#L33)
  supplies actual backend values/commands. At `05a472e9`, original DTOs moved from
  GlobalSettingsViewModel preserving package/serializable names/fields; tracked
  declaration searches found a single definition, not parallel wire models.
- [AuthenticationValuesTest:16–49](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonTest/kotlin/io/github/stream29/kodex/rpc/contract/AuthenticationValuesTest.kt#L16)
  asserts fixed JSON/null fields/plans/reasons/source.
  [AccountUsageValuesTest:56–95](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonTest/kotlin/io/github/stream29/kodex/rpc/contract/AccountUsageValuesTest.kt#L56)
  asserts nullable detail/count wire branches.
  No credentials/listener/browser URL/new duplicate credit protocol added to summaries.
  Meaningful wire evidence, not newly executed tests.

### 8. Unreviewed paths and asynchronous handoff

- **U-runtime, all 11:** no fresh tests/builds, Native/JS/Desktop/CLI run, IDE sync,
  physical-terminal focus/layout acceptance, real OAuth/provider/browser/credential
  storage, listener or network acceptance.
- **U-integration:** full root-renderer→real adapter/store/RPC isn't established by
  isolated tests. Concrete follow-ups: M1 retained-popup transition, HM2 burst,
  H1 duplicate/stale adapter, L1 source-bound completion/real buttons.
  Only discriminating experiments proposed; no blanket build/migration campaign.
- **U-concurrency depth:** application queue vs service shutdown, cross-process local
  file mutation, corrupt/partial writes and concurrent draft configuration updates
  not exhaustively proved. Draft adapter traced/history-compared, not rewritten.
- Backend safe summaries/RPC consumer followed enough to establish data/wire use;
  supplier freshness/redemption internals, process transports and other components
  belong to other lanes, not independently reaccepted here.
- Coordinator gate: independently recheck H1/M1/L1 at pinned HEAD; decide preserved
  behavior vs new guarantees before repair authorization; later run authorized
  discriminating checks and settle B2. No synthetic service layer, generic renames,
  extra state authority, source switch or commit implied.
- **REVIEW READY:** this owned file is the asynchronous handoff. New migration
  remains frozen; archived test-green does not override findings. No source fixes made.
- Final report check: all 107 file-URI links resolve and line anchors are in range;
  whitespace check emitted no diagnostics. Final inner HEAD still matches the pinned
  baseline and inner worktree remains clean. This outer child is untracked; no Git
  staging, parent-task edits or other-file changes were made by this audit.
