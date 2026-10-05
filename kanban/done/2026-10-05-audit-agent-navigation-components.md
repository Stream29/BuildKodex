# Task Tree

- `Trace Agent and navigation component specs through real hosts and renderers`() — complete
- `Review states, dependencies, callbacks, tests and ablation candidates`() — complete
- `Publish evidence and unreviewed coverage`() — complete; **REVIEW READY**

# Details

- Parent: [Spec/impl reacceptance](../executable/2026-10-05-reaccept-spec-impl-refactor.md).
- Own review scope: `app/component/{composer,history,history-index,new-session,path-picker,
  request-user-input,runtime-configuration,session-catalog,session-delete,session-rename,
  session-tab-bar,suggest-subagent-task,working-directory}`.
- Existing host/RPC/shared files may be read to validate wiring, not modified or separately migrated.
- Report only in this file; source, other tasks and shared resources are read-only.
- Apply parent acceptance rules and pinned baseline; no independent fixes or builds.
- Status: **REVIEW READY** — Session 502; read-only static reacceptance completed.

## Review report

### Baseline, method and verdict

- Inner HEAD verified as `821b66275aae2f0a9541fa2e9e03023d875c30b5`; inner
  branch `refactor/spec` and tracked worktree clean at audit entry and final
  handoff recheck. All code links below refer to this baseline.
- Read project AGENTS, change/checklist/kanban/planning skills, applicable harness
  skills, parent task, prior extraction tasks, and the frontend/module-boundary,
  RPC architecture/timeline/session, CLI Session/state, path-picker, coroutine
  lifecycle and Agent mutation-serialization checklists.
- Used source reads, tracked-reference searches, rename-aware Git history,
  parent-commit comparisons, and test-source inspection. No Gradle, IDE, browser,
  network, PTY, process/resource experiment, source edit, commit or branch operation.
- **Not accepted unconditionally.** Four confirmed cutover blockers below:
  F1/F2 Composer production behavior; F3 delete compatibility alias; F4 catalog
  forwarding shell. These are not four unused-spec components: all thirteen
  components have real production consumers.
- F5/F6 are concrete lifecycle risks requiring experiments, not new runtime
  failures proved by this audit. Root index/registry races are separated below.
- `R` = reviewed static path, not test execution/platform acceptance; `D` =
  recorded retained debt; `B1` = confirmed blocker; `B2` = suspected blocker;
  `U` = unreviewed. A row can contain both reviewed wiring and a blocker/debt.
- No component remains `U` for the requested source/history/wiring review.
  Runtime reproduction and full-platform acceptance remain **U** for every row.

### Original-to-current migration provenance

- Original RPC frontend baseline is `c1f9fdfe`: real Agent presentation, Session
  drafts/views, prepared-content submission and RPC settings/storage sources.
  The old API was inspected rather than inferred from task completion records.
- `fc9f1936` extracts application interaction components: path-picker,
  session-catalog, delete and rename. F3/F4 originate here.
- `7310a445` extracts pending-tool and settings components: request-user-input,
  suggest-subagent-task and working-directory; real RPC pending events remain
  sources, not newly invented frontend Agent identities.
- `721ac0fe` extracts runtime-configuration/history-index and completes
  catalog/settings interaction ownership. Index read handles replace actual
  renderer read work; they are not unused marker interfaces.
- `821b6627` hard-migrates composer, history aggregate, new-session and tab bar.
  F1/F2/F5 originate in this commit. History aggregate implementation is a
  rename with essentially its original algorithm (factory KDoc adjustment);
  restored NewSession implementation retains the old real draft/materialization
  algorithm. Restoration is evidence of identity, **not proof of correctness**.
- History's original item implementation lineage includes `9fe4cd126`
  (Message's lazy loading job, 2026-08-24), subsequent grouping/bounded-window
  work, then RPC adaptation at `c1f9fdfe`. F6 predates this cutover.
- Compared original `app/contract/{agent,session,path-picker,session-catalog,
  application}` declarations, `app/viewmodel/{agent,history,new-session,
  path-picker,session-catalog,application}` implementations and original
  `app/view/{agent,history,history-index,path-picker,session-catalog,application}`
  renderers against their moved/replaced declarations.
- Current package identities are retained where appropriate: Agent contracts
  are `io.github.stream29.kodex.app.agent.contract`; Session draft contract is
  `...app.session.contract`; History aggregate/items are `...app.history.contract`
  / `.item`; original source port remains `...cli.history.AgentHistorySource`.
  RuntimeConfiguration uses the original `...app.runtimeconfiguration` package
  (without `.contract`). The other contracts use `...app.{pathpicker,sessioncatalog,
  sessiondelete,sessionrename,sessiontabbar,workingdirectory}.contract`.
- The nine previously rejected fake History item projects are absent from this
  baseline. Do **not** request their resurrection or separate fake factories.
  Reasoning and compaction are legitimate immutable concrete spec models.

### Shared production spine: actual Application → Session → Agent → RPC

- [Application assembly][application] lines 55–119:
  `withKodexApplication` → RPC services → `RpcSessionViews` →
  `DefaultPersistedSessionViewModelRegistry` / `DefaultNewSessionViewModelFactory`
  → `ApplicationViewModelImpl`; catalog uses **DefaultSessionCatalogViewModel**
  plus **RpcSessionCatalogDependencies**, not the forwarding `RpcSessionCatalog`.
  Settings resolve the real persisted view or real draft; directory-picker
  factory is supplied once. Defaults become genuine NewSession arguments.
- [Session registry/projection][sessions] lines 15–49, 52–127:
  persisted tab owns its view/root-Agent projection, not backend Session
  destruction. Root Agent is replaced with the binding's presentation.
  Settings/name availability projection reads current RPC data; writes use
  `view.current()` and current settings at lines 101–107. It is not a second
  independently writable backend settings authority.
- [RPC Session owner][rpc-views] lines 45–90, 142–209:
  shared Session views; exact binding, lifetime job/keepalive, recovery;
  real Composer factory at 173–182; Ready/failed/missing terminal handling.
  Replaced binding closes children and clears projection at 190–193.
- [RPC binding][binding] lines 54–74, 86–157:
  actual six timeline caches/storage, observed runtime/settings/state,
  global latest index, remote commands and invalidation. RPC/cache nonce belongs
  here; component IDs must not substitute for it.
- [Agent presentation][presentation] lines 62–127:
  `createAgentHistoryViewModel(AgentHistorySource(...))`;
  `createHistoryIndexViewModel` with real timelines, external-write state and
  bound history scroll; pending-input/suggestion factories observe **actual**
  ToolPending events and complete original stable tool-call events through RPC.
- [Agent VM][agent] lines 24–39, 46–54, 64–91:
  stable children including bound RuntimeConfiguration; token/state derivations;
  presentation-failure notification; live CAS settings edits preserving other
  fields. `submit(content)` is the legitimate prepared-content command, **not**
  a parallel Composer draft/submission authority.
- [Production screen][agent-screen] and [root navigation screen][tree] render
  those actual children. Panels borrow owner lifetimes; visual status slots,
  cursor/focus/scroll, menus and captured popup handles are not business VMs.

### Exhaustive component R/D/B1/B2/U table

Each chain names the real contract, factory/implementation, source/dependencies,
production host, renderer action and inspected test. Links include line anchors;
line ranges in prose describe the relevant portions of those files.

| Component / origin | Classification | Exact production contract → implementation → host/renderer action | Test chain and meaningful smallest ablation |
| --- | --- | --- | --- |
| **composer** / `821b6627` | R wiring; **B1 F1/F2**; nominal-owner caveat F5 | [ComposerViewModel spec][composer-spec] 292–372 → `createComposerViewModel` / `ComposerViewModelImpl` [26–34, 42–283][composer-impl] → [createRpcComposerViewModel][rpc-composer] 32–82 using exact binding running/steer/append/resume and pending-input source → [RpcSessionViews][rpc-views] 173–182 → Agent child → [AgentRuntimeScreen][agent-screen] 176–185 → [ComposerView][composer-view] 57–120 / [ComposerInput][composer-input] 28–78 → update / submit(captured revision). Draft is sole owned mutable state; current source determines steer versus append. | [VM tests][composer-vm-test] 49–215 use fake dependencies for failure/revision/close/replacement; [view tests][composer-ui-test] 26–185 assert real snapshots and keyboard newline/Enter, but fake RenderComposerModel and default auxiliary visibility. [RPC tests][rpc-test] 293–319 submit **directly**, covering running steer only. Ablate suppression/no-op reporter separately from asynchronous command exception boundary; actual host+RPC Enter and injected failures are missing. |
| **history** / `821b6627`, original aggregate | R aggregate + all nine items; **D** retained Compose/LazyList boundary; **B2 F6**; known timeout open | [AgentHistoryViewModel/window spec][history-spec] 21–154 + [original source port][history-source] 20–25 → [factory/aggregate][history-impl] 242–391, 394–855 → RPC observed storage/latest/display/nonce [presentation][presentation] 62–66 → [AgentRuntimeScreen][agent-screen] history → [AgentHistoryView][history-view] 88–242, 245–299, 328–521 → actual viewport edge effects, captured entry menu, expand/collapse and latest-scroll intents. Publish guarded against stale generation at 774–805. | [models tests][history-model-test] cover grouping, sparse identity, paging/invalidation, checkout, compaction; [entry UI tests][history-entry-test] 81/123/162/205 send mouse events into real rows. [bounded-window test][history-window-test] 170–222 checks rendered snapshots/readiness; viewport scroll can be programmatic but does not directly call pager requests. [anchor test][history-anchor-test] direct requestOlder is mechanics only. Ablate release of evicted lazy loaders, preserving source/window/visual code. No attribution of known 1000-item timeout to F6. |
| **history-index** / `721ac0fe` | R; D upstream mixed Agent contract | [HistoryIndexViewModel/read contract][index-spec] 66–185 → [factory/impl][index-impl] 48–279, exact read-handle release 161–215 → real source timelines/external-write/current nonce in [presentation][presentation] 67–82 → [root sidebar][tree] exact selected Agent and child → [HistoryIndexComponent][index-view] 81–83, 98–217, 342–374 → acquired row/timestamp handles, hover/menu revision guard, `checkOut` 370 → captured history scroll. No storage-copy shadow VM. | [VM index tests][index-vm-test] cover noncooperative read, nonce/release; [renderer tests][index-ui-test] mouse 119/125, wheel 144–153, keyboard Enter 277/328 reach fake child callback. Storage regression suite retains real index semantics. Smallest ablation: remove read-handle owner/revision guard in a fixture; delayed completion after replacement must fail. This guards real IO, not needless wrappers. Production binding supplies global latest index while spec describes index latest; sparse scan remains original, not a proved new error. |
| **new-session** / restored original at `821b6627` | R component; root queued-index **B2** separately | [NewSessionViewModel, arguments/factory][new-spec] 18–68 → [DefaultNewSessionViewModelFactory/RpcNewSessionViewModel][new-impl] 14–61 → genuine [RpcSessionDraft][draft] settings/composer/allocation/retry 27–111 → [Application][application] 97–98 / [materialize owner][app-vm] 165–184 → [root captured draft][tree] 543–569 → [NewSessionScreen][new-view] 31–98 borrowed model/status slot → ComposerInput external submit callback → parent materialization. No staged adapter/receipt model remains. | [real RPC draft tests][new-vm-test] directly materialize and test allocation/closed/edit guards; [host-screen tests][new-ui-test] 18–207 use real keyboard/paste/undo/layout and borrowed-lifetime assertions. Enter fixture callback reads revision/unmounts; it does not traverse actual queued Application allocation. Ablate screen submit callback: Enter must reach exact original draft and root replacement once; late slot mutation requires additional root fixture. |
| **path-picker** / `fc9f1936` | R | [DirectoryPickerViewModel/factory][path-spec] 21–106 + [typed dependency port][path-deps] 13–49 → [impl][path-impl] 27–200, factory 208–232 → [SystemDirectoryPickerBrowser][path-browser] 16–89 → [Application factory][application] 99 and owning WorkingDirectory/Settings child → [DirectoryPickerPopup][path-view] 52–86, 115–188 → edit/load/parent/confirm; selected-path effect checked against current exact model. Latest navigation request and confirm validation are real separate work, not backend receipts. | [dependency tests][path-test] 17–101 use typed fake browser and cancellation; real filesystem/browser tests also inspected. [UI test][path-ui-test] Enter 58, mouse 105, filter/Enter 202/207, Escape 253/258. Smallest ablation: ignore latest-request token; late old directory result must not replace current. Preserve local filesystem port (approved same-process frontend RPC architecture). Linux symlink/native behavior not freshly verified. |
| **request-user-input** / `7310a445` | R; retained accepted-work semantics explicitly in spec | [RequestUserInputViewModel/deps/factory][input-spec] 131–234 → [create/impl][input-impl] 33–49, submit 107–170 → actual pending event / complete original call / resume [presentation][presentation] 83–94 → [AgentRuntimeScreen][agent-screen] 145–153 → [RequestUserInputPanel][input-view] → option/edit/submit(captured callId, revision). Accepted work uses owner-scope async UNDISPATCHED, not caller scope; close cancels observation, not already accepted completion. | [VM tests][input-vm-test] include deferred replacement, cancellation and closed owner; [options UI test][input-ui-test] real keyboard/options/typing from 166 into fake model; [RPC test][rpc-test] 663 onward completes secret input through child **directly**. Smallest ablation: submit newly current event instead of captured original; delayed fixture must reject cross-call completion. Retain admitted callback semantics; replacement-clearing behavior is documented original behavior, not established new bug. |
| **runtime-configuration** / `721ac0fe` | R | [state/deps/VM/factory][runtime-spec] 33–142 → [factory/impl][runtime-impl] 21–103 → [createBoundRuntimeConfigurationViewModel][rpc-runtime] 14–33, exact AgentSettings CAS target + read-through projection/models → Agent/NewSession child → [AgentRuntimeStatusBar][status-bar] 67 and [root menus][tree] 966–969 → [RuntimeConfiguration renderer][runtime-view] triggers 77–95, leaf tuple 188, mode 198–205. State is combined derivation; model/effort menu navigation does not mutate settings. | [VM tests][runtime-vm-test] 100–197 atomic tuple/caller cancellation/close/stale target; [UI tests][runtime-ui-test] real mouse then right/right/down/Enter 65–73, mode 81, Escape 102, replacement; [RPC test][rpc-test] 264–289 binds actual Agent but invokes command directly. Smallest ablation: write tuple on intermediate navigation; UI Escape must leave source unchanged. No speculative fallback correction; preserve catalog order/custom fallback. |
| **session-catalog** / `fc9f1936`, ownership `721ac0fe` | R true path; **B1 F4** test-only old shell | [catalog VM/deps/interactions/serialized entry][catalog-spec] → [DefaultSessionCatalogViewModel][catalog-impl] 26–145 → [RpcSessionCatalogDependencies][rpc-catalog] 22–40 → [Application direct construction][application] 102–105 / [exact popup interactions][app-vm] 187–210 → [SessionCatalogPopup][catalog-view] refresh 74, row 94–120, captured menu 140–170, delete child 177–181 → actual open/fork/archive/delete/root navigation. Operations serial/cancellation-linked; failed reload restores prior rows; no duplicate Session DTO/wire shape. | [interaction tests][catalog-vm-test] 21–359 captured rows, failure, stale/queued close, delete/reload; [popup UI tests][catalog-ui-test] real pointer/menu/Down+Enter 88–90 and false-delete/replacement. One direct stale-delete call is a negative admission test, not UI evidence. Smallest ablation F4 removes only old forwarding constructor; production unchanged; all tests construct true impl+real dependencies. Preserve original source port and serialized timestamps/running flags. |
| **session-delete** / `fc9f1936` | R implementation; **B1 F3** host compatibility alias | [SessionDeleteViewModel/deps][delete-spec] 10–66 → [create/DefaultSessionDeleteViewModel][delete-impl] 6–28 → root [deleteSessionFromPopup/delete owned tabs][app-vm] 238–253, 403–424, or catalog exact child+reload → [SessionDeletePopup][delete-view] 28–59 → captured Int/delete Boolean; completion callback gated by active lifetime. Root still spells field type through [compatibility alias][popups] 39–40, 65–66. | [VM tests][delete-vm-test] Boolean/failure/cancel/close; [UI tests][delete-ui-test] Tab/Enter 76–77 and replacement 91–126. Smallest ablation removes alias and types root field with actual spec; no runtime behavior should change. False result intentionally does not dismiss catalog child. |
| **session-rename** / `fc9f1936`, Settings ownership `721ac0fe` | R; root target adapter legitimate | [SessionRenameViewModel/deps][rename-spec] 16–69 → [factory/Default impl][rename-impl] 9–35 → root [ApplicationSessionRenameAdapter][app-vm] 457–463 (real target ownership), or [Settings child/source revision][settings-vm] 149–174 → [SessionRenamePopup][rename-view] 42–118 → local cursor+draft echo, nonblank plain Enter → UNDISPATCHED captured trimmed rename → exact active callback. Settings uses actual source `tryRename`, not a receipt. | [VM tests][rename-vm-test] trim/one call/failure/cancel/close; [editor UI tests][rename-editor-test] plain Enter; [popup UI tests][rename-ui-test] modified Enter 79 then actual Enter 84 and replacement. Smallest ablation drops Settings source revision check; renamed/replaced-source deferred fixture must fail. Do not delete target adapter: it identifies the owning popup for close-on-target-removal. |
| **session-tab-bar** / `821b6627`, original root rendering | R pure presentation/callback spec; **B2 F5** host identity retention; root select-index B2 separately | [immutable SessionTabBarState/callbacks][tabs-spec] 9–79 → no business VM needed → [root projection/admission][tree] 115–138, 382–430 → [SessionTabBar renderer][tabs-view] 57–177; labels/order/running/truncation and geometry stay renderer-local; primary/secondary actions check current snapshot → exact identity mapped to exact Session handle → root commands. Close uses handle; select converts to index before queued root operation. | [renderer tests][tabs-ui-test] 24–65 snapshots/bounds, 68–114 actual mouse actions, 116–155 replacement. The removed-identity test clicks **after** removed row has disappeared; it is not a retained old callback or queued production-root race reproduction. Menu fixture directly calls recorded close; not actual root menu lifecycle acceptance. Smallest ablation F5 prunes only departed identity keys; current labels/actions remain identical and closed handles become releasable. |
| **suggest-subagent-task** / `7310a445` | R; documented nontransactional/accepted-work limits | [SuggestSubagentTaskViewModel/deps/factory][suggest-spec] 69–201 → [create/impl][suggest-impl] 34–187 → [actual pending + batch create/complete/resume][presentation] 95–127 → [AgentRuntimeScreen][agent-screen] 155–165 / [root config menus][tree] 959–963 → [SuggestSubagentTaskPanel][suggest-view] 67–145, 177–230 → captured call/revision accept/reject; only config leaf commits tuple. [suggestion cwd adapter][suggest-cwd] merges into latest config only for same call/not submitting. Rejecting feedback is owned child state, not copy in renderer. | [VM feedback tests][suggest-vm-test] deferred owner/stale/cancel/partial create; [panel UI tests][suggest-ui-test] wheel 97, real mouse 114–116, feedback typing 128/Enter 133; [config UI tests][suggest-config-test] actual menus. Fixtures are fake ports, not real RPC batch allocation/navigation. Smallest ablation drops call/revision guard, then delayed old Accept must not act on replacement suggestion. Preserve original event, untrimmed rejection feedback, unchecked Long IDs and documented failure editing state; no new transaction/idempotency protocol inferred. |
| **working-directory** / `7310a445` | R; root ownership adapter legitimate | [WorkingDirectoryViewModel/deps/factory][cwd-spec] 18–87 → [create/Default impl][cwd-impl] 16–39 → owned DirectoryPicker + root exact `target::updateCwd` [app-vm][app-vm] 272–306 / adapter 465–470, or [Settings captured revision][settings-vm] 125–146 → [WorkingDirectoryPopup][cwd-view] 21–49 → borrowed picker (`closeOnDispose=false`), selected path, UNDISPATCHED bound write → child close → lifetime-guarded host callback. No copied picker state or artificial backend Session ID. | [VM tests][cwd-vm-test] 30–93 handoff/failure/consumed dependency/close after accepted wait; [UI tests][cwd-ui-test] actual Enter 79, Esc 90/108/112, replacement 120 onward; ownership/revision tests at Settings/Application roots. Smallest ablation removes exact popup handle/revision guard; late path selection must not write new target. Preserve accepted old callback work and renderer lifetime guard; ordinary dependency failure leaves picker usable. |

### History: all nine real item branches, not fake migration receipts

All item links below are current **true spec** declarations. Implementations are
the original `...cli.history.*Impl` private/internal classes created by
[AgentHistoryViewModel][history-impl] 263–348 and
[HistoryItemLoadContext][history-context] 44–100, 111–172. They read the original
source/storage and share aggregate generation. Production rendering is the
[AgentHistoryView][history-view] dispatch, not tests calling synthetic snapshots.

| Item spec / factory implementation | Actual renderer and action | Evidence/classification and minimal ablation |
| --- | --- | --- |
| [MessageHistoryItemViewModel][item-message] 10–55 → [Message Impl][message-impl] 13–53 | stored row 441–450; original steerable event, elapsed/final-turn marker | R, F6 B2 for lazy loader eviction. Models timestamp/final duration; CleanEvent UI displays real event. Remove marker derivation only: final assistant row should visibly lose marker, not all event content. |
| [ReasoningHistoryItemViewModel][item-reasoning] 11–19; concrete spec data constructed from descriptor | 428–433 `"Think"` header, elapsed; grouped child dispatch 372–380 | R. Legitimate immutable summary, no impl class needed. Models grouping/elapsed + row renderer comparison; remove summary branch and grouped history visibly loses Think row. |
| [ContextCompactionHistoryItemViewModel][item-compaction] 9–17; concrete descriptor summary | 434–439 `"Context compacted"` header | R. Models compaction test 718 onward plus real dispatch. Remove branch and actual compaction header disappears. No artificial expansion/loading VM demanded. |
| [ToolHistoryItemViewModel][item-tool] 26 onward → [Tool Impl][tool-impl] 14–105 | 483–496 collapsed header/expand/collapse; original command/tool event; current shell presentation is derived | R. Models expansion/sealing; CleanEvent UI real mouse actions. Ablate captured expansion binding: clicking collapsed tool must no longer expose details. Release cancels initial loader and detail. |
| [PatchHistoryItemViewModel][item-patch] 9–90 → [Patch Impl][patch-impl] 13–92 | 499–516 patch summary/status, expand/collapse of original event | R. Header/model tests plus CleanEvent renderer actions; ablate only expand intent, preserve patch rendering. Release cancels initial loader/detail. |
| [PlanUpdateHistoryItemViewModel][item-plan] 9–31 → [Plan Impl][plan-impl] 12–45 | 473–480 original event → PlanUpdateView | R + F6 B2. Models/source/header and actual event renderer, no isolated newly invented plan VM. Ablate loader release, inspect dormant jobs after eviction rather than interpreting UI blankness as a new receipt bug. |
| [RequestUserInputHistoryItemViewModel][item-input] 12–38 → [Request Impl][input-history-impl] 12–47 | 453–460 stable event → CleanEventView; answered/secret presentation from original event | R + F6 B2. Historical row is distinct from **active pending** RequestUserInput child, not parallel authority. CleanEvent UI covers stored output; remove stable branch and completed answer history disappears. |
| [SuggestSubagentTaskHistoryItemViewModel][item-suggest] 9–26 → [Suggest Impl][suggest-history-impl] 12–39 | 463–470 stable suggestion `renderSuggestion`; original task/result IDs | R + F6 B2. Models missing-result test 94 onward, SuggestSubagentTaskHistoryViewTest actual output. Separate pending action panel is legitimate. Remove stable renderer branch and accepted/rejected history disappears. |
| [WorkGroupHistoryItemViewModel][item-group] 13–89 → [WorkGroup Impl][group-impl] 17–105, aggregate grouping 295–348 | 328–385 virtual expandable work group and original child rows; branch 519 correctly excludes it from stored content switch | R. Models sparse child identity/expand/sealing and entry interaction UI. Ablate child identity reuse/generation guard, then old menu must fail admission. Initial lazy job is canceled by collapse/release; only original foldable Reasoning/Tool/Patch are admitted children. |

- All nine declarations are consumed by implementation/renderer exhaustiveness,
  actual descriptor construction or original source serialization/presentation.
  No plain Gradle spec dependency with an old parallel History production model
  was found.
- Original `AgentHistorySource` is a genuine storage/display/nonce input port;
  keeping its original package is not a compatibility wrapper.
- Captured `(generation, storageIndex, exact item, anchor)` callbacks at renderer
  388–418 are intentional. Root checks the selected Agent and actual child
  admission before menu/fork/revert; RPC presentation validates cache nonce,
  editability and range at 185–192. Keep these handles and checks.

### Confirmed blockers

#### F1 — B1: production Composer suppresses its only status/failure output

- Origin: `821b6627`; old AgentRuntimeScreen rendered `ComposerInput` with
  `submitHint=submitToSteerHint(...)` and its old submission callback.
- Current [AgentRuntimeScreen][agent-screen] 82–95 still computes the nonblank
  running hint, reserves/subtracts `submitHintRows`, then passes
  **`showAuxiliary=false`** to ComposerView at 176–185. PendingSteerPreview is
  retained separately at 167–169; it does not display the submit hint or error.
- [ComposerView][composer-view] 69–105 applies that flag to **both** pending
  preview and `composerStatusLine`; 126–140 is the only view branch for
  `"Submit to steer"`, submitting and `"Unable to submit: ..."` failure.
  Thus production loses the previous running draft hint while retaining an
  unexplained layout row.
- [Composer impl][composer-impl] 176–184 catches ordinary append/steer errors,
  sets Failed, invokes its failure port and returns Failed.
  [RPC factory][rpc-composer] 79 installs a **no-op failure reporter**.
  ComposerView's actual Enter callback 116–120 ignores submit's returned result.
  Both available failure outputs are therefore discarded on the real screen.
- This is not hypothetical unused spec: exactly the real implementation state
  is consumed, but its semantic output is suppressed by the production host.
- Existing standalone [Composer UI tests][composer-ui-test] 26–99 exercise
  default `showAuxiliary=true`, including a fake Error state. They cannot detect
  this host regression. RPC steer test uses direct model submit.
- Smallest diagnostic ablation: separate duplicate pending-preview suppression
  from status rendering (or temporarily render status in the real host), and
  connect a recording actual failure boundary instead of no-op. Compare original
  and current **AgentRuntimeScreen** snapshots for nonempty running draft; send
  real Enter through production-bound Composer with injected append failure.
  Expect one visible/reportable failure and unchanged draft/revision, no duplicate
  steer preview. No diagnostic modification or execution performed here.

#### F2 — B1: production Composer resume bypasses the original caught command boundary

- Origin: `821b6627`. Parent `RpcAgentPresentation.submitComposer` cleared the
  admitted draft then called presentation `resume()`; resume/cancel used the
  still-existing [operate][presentation] 195–205:
  ordinary Throwable → `mutableFailure`, cancellation rethrown.
- New [RpcComposer][rpc-composer] 67–73 launches `binding.stop()` /
  `binding.resume()` directly in the **binding owner scope**, without that catch.
  Composer impl calling `resume` cannot catch the later child failure.
- The idle Enter → Composer submit → resume path is production-active.
  Composer's cancel port has the same latent error policy, but current production
  Stop still uses [RuntimeStatusBar][status-bar] 52 → AgentViewModel.cancel →
  [presentation.stop][agent] 91, the original caught boundary. Do **not** count
  Stop-button behavior as a demonstrated regression of this cutover.
- [RpcSessionViews][rpc-views] 147–152 uses `Job(owner)`, not a supervisor for
  the binding's children. The uncaught command failure cancels that binding job
  and completes `failed`. Coordinate then removes binding/presentation/closes
  Composer at 190–193; a normal non-session-invalidating error enters
  `SessionViewStatus.Failed` and `awaitCancellation()` at 202–205.
- [RpcSessionBinding][binding] 54–74 only gives special invalidation treatment
  to SessionNotActive/SessionNotFound; ordinary RPC/model/transport errors remain
  throwable. Backend [runtime resume][backend-runtime] 104–120 is a real
  throwing command path, not an infallible setter.
- Previously those ordinary command errors entered the Agent's notification
  path [46–54][agent] while the view stayed usable. New Composer-triggered errors
  bypass it and can remove the entire root-Agent projection. This is a
  confirmed exception/ownership-policy difference; frequency not measured.
- No current test drives idle Composer **UI Enter** into a failing real binding
  resume or verifies Ready/rootAgent survives that failure. Isolated Composer
  cancel-port failure handling also lacks real binding validation.
- Smallest diagnostic ablation: use the existing caught presentation command
  boundary (or the same error policy) for these two ports, without restoring
  the old submission VM/algorithm. Inject ordinary resume failure (and a
  separate port-level stop failure, not an alleged current UI Stop regression);
  one Agent failure should be observable, Session remains Ready, accepted draft
  behavior unchanged. SessionNotActive recovery/real cancellation remain distinct.

#### F3 — B1 structural cutover: delete root still exposes a compatibility alias

- Origin: `fc9f1936`, replacing the original root delete interface with an alias.
- [ApplicationPopupState][popups] 39–40 declares
  `DeleteSession.viewModel: DeleteSessionPopupViewModel`; 65–66 explicitly says
  **Compatibility name** and aliases it to `SessionDeleteViewModel`.
- Reference audit finds only that host field and alias declaration. It adds no
  target/lifecycle behavior or wire model. The actual component is correctly
  implemented/rendered; this is a hard-cutover violation, **not a semantic
  delete failure or unused delete spec**.
- Smallest ablation: type the field directly as the true SessionDeleteViewModel
  and remove only the alias. Root/catalog delete result and renderer tests
  should remain identical. No parallel delete implementation is required.
- Contrast: RenameSessionPopupViewModel / WorkingDirectoryPopupViewModel
  at 53–62 add **real exact target ownership**. Their adapters at
  [ApplicationViewModel][app-vm] 457–470 are legitimate captured owner handles,
  not this alias; retain them.

#### F4 — B1 structural cutover: test-only old catalog constructor delegates wholesale

- Origin: `fc9f1936`.
- [RpcSessionCatalog][rpc-catalog] 10–19 is
  `SessionCatalogViewModel by DefaultSessionCatalogViewModel(...)`, with no
  additional state, failure, target or lifecycle policy and no navigation
  interactions. It preserves the old concrete construction name.
- True production [Application][application] 102–105 constructs Default directly
  with the dependencies/interactions factory. Current wrapper constructors
  occur in [SessionRepositoryViewModelTest][repository-test] 13/39 and
  [RpcSessionViewsTest][rpc-test] 53/245, not production.
- The shell is not the RPC/storage boundary; **RpcSessionCatalogDependencies**
  22–40 is that real boundary and must stay. Nor does this show catalog spec
  is unused: the real production implementation owns its actual state.
- Smallest ablation: remove only forwarding construction spelling; tests
  construct true Default impl with actual RpcSessionCatalogDependencies.
  Production is unchanged. Keep real RPC dependencies and contract serialization.
- Coordinator may combine F3/F4 under one “residual compatibility surface”
  repair gate, but their precise artifacts and removal criteria differ.

### Suspected blockers and identity/lifecycle limits

#### F5 — B2: root tab identity cache keeps departed Session handles indefinitely

- Origin: `821b6627`.
- [SessionTreeCliScreen][tree] 116–122 remembers a strong
  `MutableMap<SessionViewModel, SessionTabIdentity>`. `getOrPut` is the only
  mutation; departed tabs/drafts are never removed. Repeated create/materialize/
  close retains those original handles until the whole screen is disposed,
  despite root child close.
- This is definite unbounded key retention from static code; whether retained
  closed objects keep consequential storage/jobs and how large the impact is
  require measurement. Hence B2, not claimed runtime leak magnitude.
- The mapped ID is a legitimate **surface admission identity**, not a backend
  Session ID/nonce. Current targetFor maps through current rendered entries,
  so retention alone does **not** prove stale cross-target mutation.
- Smallest ablation/experiment: prune only absent handles (or use existing exact
  handle identity without a permanent registry). Repeated materialization/close
  should make remembered keys plateau at open tabs; retained old callback should
  still fail admission. Compare unchanged labels/order/selected viewport/actions.
- [ComposerOwnerId spec][composer-spec] 10–18 also claims an exact owner address,
  while [draft][draft] uses `"session-draft"` for every draft and
  [RPC Composer][rpc-composer] 52 uses `"session-$index"` across binding recovery.
  Current actual safety is fixed binding closures plus exact VM/revision,
  **not this value**. Record the nominal-identity mismatch; no evidence here
  that two equal values redirect current callbacks. Do not invent permanent
  backend Session IDs or replace approved index/cache-nonce semantics.

#### F6 — B2 retained original risk: evicted non-releasable lazy History loaders

- [Message Impl][message-impl] 17–23 and equivalent Plan/Request/Suggest item
  implementations attach **LAZY** loading jobs to the shared owner context.
  They implement LoadableHistoryItem, not ReleasableHistoryItem.
- [load context release][history-context] 171–172 silently does nothing for
  these kinds. [aggregate eviction][history-impl] 351–357 removes cached values
  and calls that release; all-release 387–390 does the same.
- A structurally created but never visited item can therefore leave an
  unstarted child Job attached to the History owner after cache eviction or
  generation replacement. Completed read jobs differ; Tool/Patch cancel their
  loaders and WorkGroup collapse cancels its initial loader.
- Original Message lazy job predates extraction (`9fe4cd126`); parent/current
  aggregate comparison preserves this path. **Not an introduced 821 regression.**
- Minimum experiment: fake genuine source with repeatedly changing/evicted
  windows, intentionally never materialize every structural row; count owner
  `Job.children`/retained item reachability after eviction/generation changes.
  Ablate only cancellation of evicted never-started loaders; count should
  plateau. Then verify real viewport demand still loads retained rows.
- The historical 1000-item follow/round-trip five-second timeout is already
  recorded in the extraction task; rerun pass does not close it. This audit
  neither reproduces it nor establishes F6 (or any other path) as its cause.
  Do not count that record as a newly discovered bug.

### Unmigrated root races: separate from component-introduced regression

- **Queued select/materialize by index — B2 root risk, pre-existing.**
  [root renderer][tree] 387–390 converts captured exact target to index before
  `scope.launch { selectTab(index) }`; [root owner][app-vm] 105–112 validates only
  bounds under commandMutex. NewSession Enter similarly converts target before
  launch at root 549–552; owner 165–184 resolves draft from the later slot.
  Another earlier queued command can change slots before admission.
  Parent versions already use index-based root select/materialize; do not
  misclassify the retained root API as a new tab/NewSession implementation bug.
  Exact close/fork/popup commands have different handle guards.
- Minimum fixture: block root commandMutex with deferred work, queue Enter or
  selection, mutate/remove a preceding tab via an earlier command, then release.
  Verify captured target or explicit rejection rather than silently using a new
  slot. Renderer-only fake callbacks do not exercise this race.
- **Registry release/open — B2 root risk, pre-existing.**
  [DefaultPersistedSessionViewModelRegistry][sessions] 31–34 removes under its
  mutex, then invokes `views.release(index)` outside that lock. A concurrent
  direct registry open in between can obtain/create a view subsequently
  released by the older close. Root Application commandMutex serializes normal
  commands, mitigating that route; independent callers require a deferred test.
  The original RPC registry path at `c1f9fdfe` already had this sequence.
- Minimum fixture: pause release after registry removal, open the same index,
  then finish release; assert reopened handle is usable. No backend permanent
  index uniqueness is assumed, and no resource/race experiment was executed.
- Catalog open/fork renderer catch-and-ignore comments rely on existing parent
  failure reporting. Full root failure visibility was not runtime validated;
  do not manufacture a new catalog extraction regression from an old catch.

### Preserved legitimate boundaries; debt, tests and platform limits

- **History debt D:** spec exposes Compose `Stable`, custom LazyList types and
  Job-bearing loading states. This mixed view/viewmodel boundary is expressly
  retained in [History aggregate extraction task][history-task], not newly
  invented permission to spread renderer APIs elsewhere. Mixed root
  `agent-state-contract`/app shared dependencies remain incomplete migration,
  not sufficient evidence of unused specs.
- Source ports, original serialized models, derived presentation, render-local
  widget state and pure visual status slots were checked rather than condemned
  by name. No new parallel authoritative Session settings/draft/history model
  was found in the owned components.
- Picker effect with selected path and delete Boolean are meaningful results,
  not single-value “receipts” replacing original behavior. Prepared-content
  Agent submit is separate from draft submit. Suggested task batch IDs/results
  remain original protocol models; do not invent replay/idempotency guarantees.
- Runtime config writes are source CAS commands, not optimistic mirrors.
  Suggestion cwd adapter merges current same-call configuration; root
  rename/cwd adapters carry owning targets. These are useful adaptations.
- Pending input/suggestion accepted work deliberately outlives caller wait and
  observer close, using owner scope. Original internal pending-clearing after
  completion remains documented, and backend deduplication/nontransactional
  limitations must not be silently recast as a new migration regression.
- UI source comparisons cover real output/actions: stored history item branches,
  active panels, picker loading/filter/confirm, rename Enter variants, delete
  Cancel autofocus/Boolean, catalog menus, tab labels/scroll/controls, NewSession
  multiline/undo/paste, runtime leaf/menu Escape, and Composer newline/status.
  F1 demonstrates why correct standalone snapshots are not host acceptance.
- Component VM tests invoke models directly for state/cancellation mechanics.
  Mosaic tests generally use real key/mouse/wheel dispatch but often fake child
  models/dependencies. Neither alone proves exact Application/RPC completion.
  Explicit exceptions/gaps are noted per row; no direct call was counted as
  renderer callback evidence.
- Read all owned spec/impl build scripts: spec types are actually imported,
  implemented, constructed or dispatched in production. `settings.gradle.kts`
  discovers those real split projects; there is no demand for a VM in pure
  tab/summary contracts.
- [KMP host configuration][host-targets] 22–46 declares JVM, linuxX64,
  linuxArm64, macosArm64 and mingwX64 and shared Mosaic hierarchy. Current
  components do not establish JS/browser acceptance. No native platform test,
  real terminal/CLI, filesystem permission/symlink matrix, transport failure
  injection, heap/job count, live provider or IDE was run.
- Prior task records list targeted JVM successes and Integration compilation,
  not a new execution result from this Session. Existing build/report artifacts
  were not treated as fresh evidence. The known History timeout stays open.
- U limits: exhaustive runtime scheduling/exception frequencies, whole backend
  RPC correctness beyond the consulted hosts, full root failure UI and actual
  all-platform interaction acceptance. Other reviewers own those backend/root
  scopes; this report is not whole-repository signoff.

### Handoff

- **REVIEW READY** for coordinator independent recheck at the pinned baseline.
- Four confirmed findings, two suspected lifecycle findings, plus separately
  classified pre-existing root races. Static wiring coverage: **13/13** owned
  components and **9/9** real History item kinds.
- Only this assigned outer child report was written. No fixes or new migration
  authorized/performed. Minimal ablations above are diagnostic proposals,
  not experiments claimed to have passed.
- Report file-URI targets/line bounds and reference definitions were checked;
  scoped `git diff --check` passed. No temporary files or acquired resources
  were created.

### Evidence links

[application]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L55
[app-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L105
[sessions]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt#L15
[rpc-views]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionViews.kt#L142
[binding]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionBinding.kt#L54
[presentation]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentPresentation.kt#L62
[agent]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentViewModel.kt#L24
[agent-screen]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L69
[tree]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L115
[popups]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/contract/application/src/commonMain/kotlin/io/github/stream29/kodex/app/application/contract/ApplicationPopupState.kt#L39
[composer-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/ComposerViewModel.kt#L292
[composer-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/ComposerViewModel.kt#L26
[rpc-composer]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcComposer.kt#L32
[composer-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/agent/ComposerView.kt#L57
[composer-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/ComposerInput.kt#L28
[composer-vm-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/agent/ComposerViewModelTest.kt#L49
[composer-ui-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/agent/ComposerViewTest.kt#L26
[rpc-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionViewsTest.kt#L264
[backend-runtime]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendAgentRuntimeRpc.kt#L104
[history-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/AgentHistoryViewModel.kt#L21
[history-source]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistorySource.kt#L20
[history-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L242
[history-context]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/HistoryItemLoadContext.kt#L111
[history-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt#L88
[history-model-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryModelsTest.kt#L62
[history-entry-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryEntryInteractionTest.kt#L81
[history-window-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L170
[history-anchor-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryAnchorTest.kt#L64
[index-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/HistoryIndexViewModel.kt#L66
[index-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexViewModel.kt#L48
[index-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/historyindex/HistoryIndexComponent.kt#L98
[index-vm-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexComponentTest.kt#L1
[index-ui-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/historyindex/HistoryIndexComponentTest.kt#L119
[new-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/session/contract/NewSessionViewModel.kt#L18
[new-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModel.kt#L14
[draft]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionDraft.kt#L27
[new-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/NewSessionScreen.kt#L31
[new-vm-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModelTest.kt#L16
[new-ui-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/application/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionScreenTest.kt#L18
[path-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/pathpicker/contract/DirectoryPickerViewModel.kt#L21
[path-deps]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/pathpicker/contract/DirectoryPickerDependencies.kt#L13
[path-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/pathpicker/DirectoryPickerViewModel.kt#L27
[path-browser]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/pathpicker/DirectoryPickerBrowser.kt#L16
[path-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/pathpicker/DirectoryPickerPopup.kt#L52
[path-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/pathpicker/DirectoryPickerDependencyTest.kt#L17
[path-ui-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/pathpicker/DirectoryPickerPopupTest.kt#L58
[input-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/RequestUserInputViewModel.kt#L131
[input-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/RequestUserInputViewModel.kt#L33
[input-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/agent/RequestUserInputPanel.kt#L35
[input-vm-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/agent/RequestUserInputViewModelTest.kt#L1
[input-ui-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/agent/RequestUserInputOptionsTest.kt#L166
[runtime-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/runtimeconfiguration/RuntimeConfigurationViewModel.kt#L33
[runtime-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/runtimeconfiguration/RuntimeConfigurationViewModel.kt#L21
[rpc-runtime]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcRuntimeConfiguration.kt#L14
[status-bar]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/RuntimeStatusBar.kt#L67
[runtime-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/runtimeconfiguration/RuntimeConfiguration.kt#L77
[runtime-vm-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/runtimeconfiguration/RuntimeConfigurationViewModelTest.kt#L100
[runtime-ui-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/runtimeconfiguration/RuntimeConfigurationTest.kt#L51
[catalog-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/sessioncatalog/contract/SessionCatalogViewModel.kt#L1
[catalog-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/sessioncatalog/DefaultSessionCatalogViewModel.kt#L26
[rpc-catalog]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionCatalog.kt#L10
[catalog-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessioncatalog/SessionCatalogPopup.kt#L74
[catalog-vm-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/sessioncatalog/SessionCatalogInteractionTest.kt#L21
[catalog-ui-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessioncatalog/SessionCatalogPopupTest.kt#L88
[repository-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/session/src/commonTest/kotlin/io/github/stream29/kodex/cli/session/SessionRepositoryViewModelTest.kt#L13
[delete-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-delete/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/sessiondelete/contract/SessionDeleteViewModel.kt#L10
[delete-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-delete/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/sessiondelete/SessionDeleteViewModel.kt#L6
[delete-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-delete/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessiondelete/SessionDeletePopup.kt#L28
[delete-vm-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-delete/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/sessiondelete/SessionDeleteViewModelTest.kt#L13
[delete-ui-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-delete/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessiondelete/SessionDeletePopupTest.kt#L57
[rename-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-rename/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/sessionrename/contract/SessionRenameViewModel.kt#L16
[rename-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-rename/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/sessionrename/SessionRenameViewModel.kt#L9
[settings-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/SessionSettingsViewModel.kt#L125
[rename-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-rename/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessionrename/SessionRenamePopup.kt#L42
[rename-vm-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-rename/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/sessionrename/SessionRenameViewModelTest.kt#L19
[rename-editor-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-rename/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessionrename/SessionRenameEditorTest.kt#L18
[rename-ui-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-rename/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessionrename/SessionRenamePopupTest.kt#L57
[tabs-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/sessiontabbar/contract/SessionTabBar.kt#L9
[tabs-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessiontabbar/SessionTabBar.kt#L57
[tabs-ui-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessiontabbar/SessionTabBarTest.kt#L24
[suggest-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/SuggestSubagentTaskViewModel.kt#L69
[suggest-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskViewModel.kt#L34
[suggest-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskPanel.kt#L67
[suggest-cwd]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/SuggestedWorkingDirectoryAdapter.kt#L8
[suggest-vm-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskFeedbackTest.kt#L1
[suggest-ui-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskPanelTest.kt#L97
[suggest-config-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskConfigurationTest.kt#L82
[cwd-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/working-directory/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/workingdirectory/contract/WorkingDirectoryViewModel.kt#L18
[cwd-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/working-directory/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/workingdirectory/WorkingDirectoryViewModel.kt#L16
[cwd-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/working-directory/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/workingdirectory/WorkingDirectoryPopup.kt#L21
[cwd-vm-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/working-directory/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/workingdirectory/WorkingDirectoryViewModelTest.kt#L30
[cwd-ui-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/working-directory/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/workingdirectory/WorkingDirectoryPopupTest.kt#L60
[item-message]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/MessageHistoryItemViewModel.kt#L10
[item-reasoning]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/ReasoningHistoryItemViewModel.kt#L11
[item-compaction]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/ContextCompactionHistoryItemViewModel.kt#L9
[item-tool]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/ToolHistoryItemViewModel.kt#L26
[item-patch]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/PatchHistoryItemViewModel.kt#L9
[item-plan]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/PlanUpdateHistoryItemViewModel.kt#L9
[item-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/RequestUserInputHistoryItemViewModel.kt#L12
[item-suggest]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/SuggestSubagentTaskHistoryItemViewModel.kt#L9
[item-group]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/WorkGroupHistoryItemViewModel.kt#L13
[message-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/MessageHistoryItemViewModel.kt#L13
[tool-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/ToolHistoryItemViewModel.kt#L14
[patch-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/PatchHistoryItemViewModel.kt#L13
[plan-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/PlanUpdateHistoryItemViewModel.kt#L12
[input-history-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/RequestUserInputHistoryItemViewModel.kt#L12
[suggest-history-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/SuggestSubagentTaskHistoryItemViewModel.kt#L12
[group-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/WorkGroupHistoryItemViewModel.kt#L17
[host-targets]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt#L22
[history-task]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-04-extract-history-aggregate-component.md#L1
