REVIEW READY — independent read-only frontend ownership audit; qualified verdict, not full acceptance.

# Task Tree

- `Verify the pinned source baseline and inventory()`
- `Trace all fifty actual frontend projects()`
- `Review ownership, admission, failure and renderer-input evidence()`
- `Publish findings and unresolved verification gates()`

# Details

## Scope, baseline and evidence limits

- Parent: [final repository reacceptance][parent]. Exactly **50 actual build files**:
  42 builds in 15 non-Settings components, plus eight roots/CLI/RPC fixture builds.
- Inner repository: `/home/stream/ACodeSpace/push/BuildKodex/Kodex`;
  branch `refactor/spec`, HEAD `6b7129fa0e719dde8f88a0034b9d572679984a03`.
  Read-only HEAD/status checks found the pinned HEAD and an empty inner status.
  Outer unrelated changes belong to other work; they were not modified.
- Loaded outer AGENTS, buildkodex-change, checklist/kanban/programmatic-planning
  skills, ask-user, document-editing and IDE-collaboration guidance; Draft, parent
  executable, closure planning, frontend/spec-impl/coroutine/RPC timeline/session
  checklists and relevant prior audits/done records. Modern frontend guidance takes
  precedence over explicitly historical CLI Session design sections.
- Only this outer child report was written, using `apply_patch`. No code fixes,
  other documentation writes, builds, tests, CLI/PTY, IDE, devices, network,
  Git mutation or shared-resource operations. No temporary files/resources created.
- `R` means reviewed source/critical chain, **not** fresh runtime/platform certification.
  `D` means explicit retained debt/exception; `B1` means actual source-chain error;
  `B2` means a specific doubt with a missing discriminating experiment.
  `U` applies to the unperformed validation gates below, not silently to green.
- Settings/account/OpenAI Login internals are the Settings lane. History View is
  counted once here; its migrated `cli.patch` specialization is the UI/Home lane.
  This report covers History's aggregate, all item dispatch and whole UI actions,
  not a second Patch-specific acceptance.

## Verdict for the coordinator

- **Physical/ownership cutover is real on the reviewed production chain.**
  The RPC Agent implementation absorbed the former Presentation wrapper;
  Session directly borrows the raw view's Agent flow. Application owns exact
  tab targets and parent registry admission. No parallel AgentHistoryAggregate,
  Timeline VM, independent Shell project, or nine fake item projects are needed.
- **Do not declare frontend behavior fully accepted:** F1 failure swallowing and
  F2 detached draft Composer lifetime remain `B1`. F3 saturated History navigation
  and F4 equality-based root keys remain `B2`.
- Seven builds carry documented debt, chiefly pending-tool late-success behavior
  and narrow-screen sidebars. Characterization tests are evidence of those
  behaviors, not evidence that their consequences have been repaired.
- Per-build primary verdicts: **36 R / 7 D / 5 B1 / 2 B2 / 0 U = 50**.
  Several builds share a root cause; these are **not five independent B1 defects**.
  Fresh execution, platform actuals and the experiments below remain **U**.

## Every actual build in coverage

Each row names its own build, responsibility and evidence chain below. A component
chain covers its spec, actual factory/implementation, production consumer and tests;
the row verdict applies to that project's responsibility, not a blanket directory pass.

| # | Actual build file | Verdict | Responsibility / evidence |
| --- | --- | --- | --- |
| 01 | [agent/spec/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/spec/build.gradle.kts#L1) | R | C1: actual Agent/settings, independent runtime facts, child ownership, Shell contract |
| 02 | [agent/impl/view/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/impl/view/build.gradle.kts#L1) | R | C1: real Agent screen/status predicates, one preview/error outlet; root History catches are F1, not this outlet |
| 03 | [composer/spec/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/spec/build.gradle.kts#L1) | R | C2: owner/revision/draft/admission and typed failure contract |
| 04 | [composer/impl/viewmodel/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/impl/viewmodel/build.gradle.kts#L1) | R | C2: sole draft VM, fixed ports, canceled/failed/accepted submission semantics |
| 05 | [composer/impl/view/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/impl/view/build.gradle.kts#L1) | R | C2: input buffer/cursor/layout belong to View; captured revision reaches VM |
| 06 | [history/spec/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/build.gradle.kts#L1) | R | C3: original cross-timeline owner, portable window/viewport/effect and nine item contracts |
| 07 | [history/impl/viewmodel/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/build.gradle.kts#L1) | B2 | C3 / F3: ignored navigation `trySend` failure can strand newest intent; ordinary bounded paging retained |
| 08 | [history/impl/view/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/build.gradle.kts#L1) | R | C3: genuine scroll classification, measured exact effect consumption, all item/UI dispatch; cli.patch specialization excluded |
| 09 | [history-index/spec/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/spec/build.gradle.kts#L1) | R | C4: nonce/revision, per-consumer read handles, Check out is navigation |
| 10 | [history-index/impl/viewmodel/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/impl/viewmodel/build.gradle.kts#L1) | R | C4: original sparse scans, guarded publication and released IO handles |
| 11 | [history-index/impl/view/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/impl/view/build.gradle.kts#L1) | R | C4: independently borrowed sidebar reads, real hover/menu/keyboard/wheel |
| 12 | [new-session/spec/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/spec/build.gradle.kts#L1) | R | C5: exact original child, stable Composer/configuration, no speculative persistence |
| 13 | [new-session/impl/viewmodel/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/viewmodel/build.gradle.kts#L1) | B1 | C5 / F2: scoped draft owner does not close parentless Composer on owner termination |
| 14 | [new-session/impl/view/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/view/build.gradle.kts#L1) | R | C5: borrowed original draft and status child; disposal does not own tab lifetime |
| 15 | [path-picker/spec/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/spec/build.gradle.kts#L1) | R | C6: browser/load/validation/effect capability contract, no framework geometry |
| 16 | [path-picker/impl/viewmodel/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/impl/viewmodel/build.gradle.kts#L1) | R | C6: actual filesystem browser, child Job and latest-request guards |
| 17 | [path-picker/impl/view/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/impl/view/build.gradle.kts#L1) | R | C6: real popup navigation/filter/confirm/retry; supplied VM owns IO |
| 18 | [request-user-input/spec/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/spec/build.gradle.kts#L1) | D | C7 / D2: explicit legacy late-success and canceled-dependency policy |
| 19 | [request-user-input/impl/viewmodel/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/impl/viewmodel/build.gradle.kts#L1) | D | C7 / D2: new visible call can lose submit capability after old success |
| 20 | [request-user-input/impl/view/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/impl/view/build.gradle.kts#L1) | D | C7 / D2: genuine input, but current branch cannot repair internal stale pending event |
| 21 | [runtime-configuration/spec/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/spec/build.gradle.kts#L1) | R | C8: atomic model/effort/tier, source-authoritative projection and ownership |
| 22 | [runtime-configuration/impl/viewmodel/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/impl/viewmodel/build.gradle.kts#L1) | R | C8: exact target, caller-owned waits, close cancels waits not borrowed source |
| 23 | [runtime-configuration/impl/view/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/impl/view/build.gradle.kts#L1) | R | C8: original nested menu/focus/fallback/no-spacing presentation |
| 24 | [session-catalog/spec/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/spec/build.gradle.kts#L1) | R | C9: data vs navigation capabilities; failures throw, not empty success |
| 25 | [session-catalog/impl/viewmodel/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/viewmodel/build.gradle.kts#L1) | R | C9: exact opening/delete child; reload restores prior state then rethrows |
| 26 | [session-catalog/impl/view/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/view/build.gradle.kts#L1) | B1 | C9 / F1: false “host registry reports” premise swallows open/fork failures |
| 27 | [session-delete/spec/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-delete/spec/build.gradle.kts#L1) | R | C10: stable captured index/name and true/false result meaning |
| 28 | [session-delete/impl/viewmodel/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-delete/impl/viewmodel/build.gradle.kts#L1) | R | C10: actual bound deletion, no duplicate catalog/tab registry |
| 29 | [session-delete/impl/view/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-delete/impl/view/build.gradle.kts#L1) | R | C10: Cancel-first focus and exact active-child result guard |
| 30 | [session-rename/spec/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-rename/spec/build.gradle.kts#L1) | R | C11: sole short-lived name draft and captured rename port |
| 31 | [session-rename/impl/viewmodel/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-rename/impl/viewmodel/build.gradle.kts#L1) | R | C11: normalization/bound edit/error propagation; no artificial scope required |
| 32 | [session-rename/impl/view/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-rename/impl/view/build.gradle.kts#L1) | R | C11: widget cursor/undo/focus and late result guard, genuine Enter |
| 33 | [session-sidebar/spec/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-sidebar/spec/build.gradle.kts#L1) | R | C12: actual shared SidebarSettings contract; borrows Agent children |
| 34 | [session-sidebar/impl/view/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-sidebar/impl/view/build.gradle.kts#L1) | D | C12 / D1: Shell/Index real composition; inherited one-column center policy |
| 35 | [session-tab-bar/spec/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/spec/build.gradle.kts#L1) | R | C13: immutable unique identities and captured host callbacks, no pretend VM |
| 36 | [session-tab-bar/impl/view/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/impl/view/build.gradle.kts#L1) | R | C13: stateless production renderer; F4 concerns root's identity adapter |
| 37 | [suggest-subagent-task/spec/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/spec/build.gradle.kts#L1) | D | C14 / D2–D3: explicitly nontransactional batch and late-success semantics |
| 38 | [suggest-subagent-task/impl/viewmodel/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/impl/viewmodel/build.gradle.kts#L1) | D | C14 / D2–D3: actual create→complete→resume, explicit retry can create again |
| 39 | [suggest-subagent-task/impl/view/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/impl/view/build.gradle.kts#L1) | D | C14 / D2–D3: business configuration/rejection not cloned; legacy Failed has no added banner |
| 40 | [working-directory/spec/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/working-directory/spec/build.gradle.kts#L1) | R | C15: exact browser ownership and captured selection capability |
| 41 | [working-directory/impl/viewmodel/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/working-directory/impl/viewmodel/build.gradle.kts#L1) | R | C15: real transfer/apply/close, no settings mirror |
| 42 | [working-directory/impl/view/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/working-directory/impl/view/build.gradle.kts#L1) | R | C15: exact selection effect and renderer-lifetime callback guard |
| 43 | [app/spec/application/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/spec/application/build.gradle.kts#L1) | R | A: actual navigation/popup/factory contracts, exact targets under admission |
| 44 | [app/spec/session/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/spec/session/build.gradle.kts#L1) | R | S: stable handle, nullable borrowed Agent, registry release/shutdown contract |
| 45 | [app/impl/application/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/build.gradle.kts#L1) | B2 | A / F4: exact select/materialize hold; deletion's equality-based selection needs equal-twin host test |
| 46 | [app/impl/session/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/session/build.gradle.kts#L1) | R | S: rootAgent flow identity, serialized release/open and canceled-caller cleanup |
| 47 | [app/impl/rpc/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/build.gradle.kts#L1) | B1 | RPC / F2: real protocol/cache/Agent composition is justified; RpcSessionDraft scope escapes owner |
| 48 | [app/impl/view/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/build.gradle.kts#L1) | B1 | V / F1; also F4 B2: production callbacks swallow exceptions; History retention itself is exact |
| 49 | [app/cli/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/build.gradle.kts#L1) | R | CLI: original Native entry, real host, contract-typed renderer and title cleanup |
| 50 | [app/test-support/rpc/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/test-support/rpc/build.gradle.kts#L1) | B1 | Fixture / F2: real JSON/backend fixture, but “owns all frontend jobs” excludes detached draft Composer |

## Actual roots, factories, static types and ownership

### A — Application / production host

- Contract FQCN `io.github.stream29.kodex.app.application.contract.ApplicationViewModel`
  [11–36, 52–64, 129–151][application-spec]; navigation permits distinct referential
  children, not equality deduplication [27–35][navigation-spec]. Popup variants
  retain exact borrowed targets; Login retains its Settings return target [12–76][popup-spec].
- Implementation FQCN `io.github.stream29.kodex.cli.app.ApplicationViewModelImpl`
  [56–66][application-impl]. `selectTab` resolves `===` inside the original mutex
  at 105–114; `materializeNewSession` resolves exact draft at 166–182, replaces
  the same slot without selecting a newly current tab, closes the consumed draft.
- Original parent registry owns index admission: open 83–103, close/archive
  127–144, accepted suggested-index opening 156–164. Indexes address persisted
  storage; local tab/Agent/popup identities are captured instances. No extra
  queued-index resolver or third registry was added. Actual factory failure escapes;
  `null` means stale/consumed materialization target, not failed allocation.
- Popup factories/callbacks at 184–319 bind the opening, rename target, deletion
  index and working-directory Agent/call. Target adapters at 454–467 add those
  duties while delegating the actual child: they are not useless wrapper VMs.
- Production FQCN `io.github.stream29.kodex.cli.app.withKodexApplication`
  [55–65, 71–154][host] creates backend→JSON transport→Restoring RPC→views→
  registry→draft factory→Application. `KodexApplication.viewModel` at 42–46 is
  the **spec interface**, not an implementation-only renderer API.
- Settings concrete source selection at 107–110 is a legitimate composition-root
  factory branch. It is not a VM resolving the currently selected implementation.
  Callback 90–96 opens accepted child indexes separately, catches/logs navigation
  failure without replaying creation. It does not report unrelated catalog failures.
- Shutdown [159–185][host] preserves primary/suppressed failures, explicitly closes
  local children, joins frontend owner in NonCancellable, shuts registry, then exits
  transport/backend and releases Home last. Normal CLI shutdown explicitly closes
  drafts: F2 does **not** establish that every normal CLI exit leaks.
- [ExactTargetCommandTest][exact-tests] at 20–228 uses real frontend/RPC and gated
  factory/command seams: moved/closed/equal targets, duplicate submission, queued
  and post-allocation cancellation, failure identity retention and shutdown.
  These are **direct command tests**, not renderer Enter/menu acceptance.
  [SessionActionComponentOwnershipTest][action-tests] covers captured action children.
- Minimum ablation: remove the old slot-based command path, not the parent mutex,
  registry or bound factories. Gated select/materialize must still target the same
  child, return stale/no-op only appropriately, propagate real failure and never
  allocate another Session after a known successful allocation. F4 remains separate.

### S / RPC — Session, raw binding and merged Agent

- Spec FQCNs `io.github.stream29.kodex.app.session.contract.SessionViewModel`,
  `PersistedSessionViewModel`, `PersistedSessionViewModelRegistry` are the actual
  contracts [Session][session-spec], [Persisted][persisted-spec], [Registry][registry-spec].
  Close is local, not backend Stop; root Agent may disappear/rebind; registry
  shutdown accepted cleanup survives caller cancellation.
- `io.github.stream29.kodex.cli.session.DefaultPersistedSessionViewModelRegistry`
  [15–54][session-impl] owns stable tab handles. Open/release are serialized through
  exact old raw-view release; removed handle join happens afterward.
- `io.github.stream29.kodex.cli.session.RpcPersistedSessionViewModel` [58–124][session-impl]
  implements the spec; **line 66 assigns `rootAgent = view.agent`**, not a mapping
  wrapper, permanent Agent or hidden cast. Name/settings retain read-only last-known
  values during recovery, not a second writable backend authority.
- `io.github.stream29.kodex.cli.rpc.RpcSessionViews` [39–90][rpc-views] reuses
  initialization independently of one opener's cancellation; failed first opens
  evict only the exact failed view. `RpcSessionView` [100–222][rpc-views] owns
  keepalive/read rebinding: withdraw Agent→close original children→join old binding
  before reactivation. Recovery retries observation, **not accepted commands**.
- Factory FQCN `io.github.stream29.kodex.cli.rpc.createRpcAgentViewModel`
  [59–74][rpc-agent] returns **AgentViewModel**; sole private
  `io.github.stream29.kodex.cli.rpc.RpcAgentViewModel` implements it.
  Children are constructed once per binding at 91–173 and closed with that owner
  at 175–186. Former `RpcAgentPresentation`/`RpcAgentPresentationViewModel`
  path is retired, not retained behind a forwarding shell.
- `io.github.stream29.kodex.cli.rpc.RpcSessionBinding` [36–84, 87–190][rpc-binding]
  remains necessary: six original timelines, nonce-aware reads, address-bound
  commands and cached backend facts. `RpcStorage` implements the actual observable
  storage contract, not another History VM. `RpcOutput` [21–80][rpc-output] shares
  current response replay and switches nonce by cancel/join; structured token total
  is a legitimate nullable derived projection, not a new runtime aggregate.
- Genuine RPC tests [RpcSessionViewsTest][rpc-tests] cover missing/failed opens,
  nonce rollback, caller-canceled append, lost accepted append reply, caught Stop,
  rebind without append replay, shared output and exact history confirmation.
  [SessionBindingOwnershipTest][binding-tests] at 35–232 covers direct flow identity,
  real upstream inactivity, owner cancellation, canceled shutdown drain and gated
  release/open race. It injects RPC scheduling/failures, not successful fake VM callbacks.
- Minimum ablation: Session must continue directly borrowing `view.agent`; do not
  delete the raw binding or handle registry to achieve a superficial single class.
  Removing exact-view release/its mutex should make the original gated opener fail.
  Keeping it must preserve same handle, local close/no Stop and backend turn lifetime.

### V / CLI / Fixture — actual consumers, not test-only wiring

- `io.github.stream29.kodex.cli.app.SessionTreeCliScreen` [120–185, 402–415,
  558–637][root] observes the Application spec. Tab callbacks capture a target
  **before** launching; New Session Enter calls `materializeNewSession(selected)`
  at 569. Selected Agent is borrowed, not reconstructed by the root.
- `io.github.stream29.kodex.cli.app.rememberHistoryRenderStates` [18–46][history-retention]
  subscribes to every persisted tab, including hidden binding withdrawal. Pair
  pruning and lookup use **`===` History VM**, never storage index/equality.
  `io.github.stream29.kodex.cli.history.AgentHistoryViewState` [16–39][history-view-state]
  owns only LazyList/interaction widget state and a mount-scoped listener; no
  business window, cache, IO Job or another Application registry.
- Root retains state through actual tab switches and prunes a replaced/closed
  History. [HistoryRenderStateTest][retention-tests] has hidden withdrawal through
  a **manually mutable delegated rootAgent seam**, and equal-History helper tests.
  It is not itself a full hidden RPC rebind + real-tab-input test. Complementary
  real RPC rebind evidence is S; actual native tab-roundtrip evidence is historical.
  Do not substitute the former for the latter. F4 concerns other equality-based keys.
- `io.github.stream29.kodex.cli.app.main` [11–49][cli] prepares Home, enters
  `withKodexApplication`, runs Mosaic with the spec-typed root and restores terminal
  title in finally; the application/run-loop cancellation catch rethrows.
  Early Home preparation/logging failure handling is not a fresh cancellation
  acceptance and delegates the Home protocol to the UI/Home lane.
  [CLI coroutine logger][cli-logger]
  only logs otherwise unobserved scope failures: it cannot see swallowed F1 failures.
- `io.github.stream29.kodex.app.test.RpcFrontendFixture` [29–54, 66–113][fixture]
  creates genuine backend/Home/filesystem/JSON transport; only model/OAuth external
  APIs are mocked. Renderer tests can supply one interaction dispatcher. Its project
  is legitimate test infrastructure, not an empty paired spec. Production consumers
  do not use it as an application service. F2 invalidates its universal ownership claim.
- Minimum ablation: remove state lookup by storage index/equality, retaining the
  renderer widget state per exact live History owner. Removing root retention should
  break actual tab-roundtrip output/anchor; pruning a hidden withdrawn Agent must
  release the old pair without closing backend storage. Fixture must use the same
  real factories and must close all scoped children; no fake success substitutes.

## Component chains and minimum responsibility-preserving ablations

### C1 — Agent / Shell composition

- Contract `io.github.stream29.kodex.app.agent.contract.AgentViewModel`
  [95–238][agent-spec] → merged RPC factory/VM above → raw view→Session→root →
  `io.github.stream29.kodex.cli.app.AgentRuntimeScreen` [49–210][agent-screen].
  Root passes actual Agent; History/Composer/Index/pending/configuration children
  are spec-typed stable properties. Agent screen suppresses duplicate Composer
  failure and pending-steer preview, showing one existing notification outlet.
- State/running remain separate facts. Stop wins whenever running, including
  compaction; idle ToolPending shows Clear pending, other idle states Resume.
  Compact/history editing visual predicates do not replace backend admission.
  Null token count omits the counter. Host pending branches disable ordinary Composer.
- Shell contract `io.github.stream29.kodex.app.agent.contract.AgentShellSession`
  and registry [13–42][shell-spec] → original RPC ShellView [189–196, 317–327][rpc-agent]
  → C12's real Sidebar. Same-id retained handles expose completion; completed
  registry entries are not deleted by presentation filtering. Close validates
  **same handle** before command; no independent Shell build or invented manager.
- [ComposerHostRepairTest][agent-ui-tests] sends actual Enter into actual Agent
  host with RPC failure injection, checks one error line, retained draft/binding
  and no replay; running steer, cancellation, remount and right-click paths also
  use input/real models. RPC Shell projection tests use injected registry snapshots,
  not a live terminal process certification.
- Minimum ablation: keep one RPC Agent implementing the spec; remove any second
  failure-forwarding Presentation wrapper, not binding/child owners. Suppress
  sibling Composer preview/error in this host only; genuine Enter must still
  report once, retain failed input and leave ordinary-failure binding Ready.

### C2 — Composer

- `io.github.stream29.kodex.app.agent.contract.ComposerViewModel` [280–372][composer-spec]
  → `io.github.stream29.kodex.cli.agent.createComposerViewModel` [26–34][composer-impl]
  → private `io.github.stream29.kodex.cli.agent.ComposerViewModelImpl` [42–283][composer-impl].
  Fixed runtime/append/steer/resume/cancel/failure ports are supplied by
  `io.github.stream29.kodex.cli.rpc.createRpcComposerViewModel` [33–82][rpc-composer].
- Agent creation owns actual VM; `io.github.stream29.kodex.cli.agent.ComposerView`
  [49–120][composer-view] / `io.github.stream29.kodex.cli.app.ComposerInput`
  [28–78][composer-input] hold widget edits and invoke captured revision.
  Runtime is read at admission, not from delayed displayed projection.
  Append success clears exact draft before caught asynchronous resume; failure
  retains input/revision appropriately; cancellation is not ordinary failure.
- [ComposerViewModelTest][composer-tests] uses fixed fake ports for model admission/
  failure/cancellation; isolated View tests use actual keys with a fake VM.
  C1's RPC-host Enter tests supply the missing production error outlet evidence.
- Minimum ablation: do not create a sibling draft/error authority. Removing
  revision/fixed-owner guard must break stale submission tests; removing real
  failure reporter must break one-notification host Enter tests. Preserve close
  behavior for accepted work and observation cleanup. Draft parenting defect is F2.

### C3 — Original cross-timeline History, portable viewport/effect, all nine items

- Actual contract `io.github.stream29.kodex.app.history.contract.AgentHistoryViewModel`
  [129–230][history-spec] plus original
  `io.github.stream29.kodex.cli.history.AgentHistorySource` [source][history-source]
  → `io.github.stream29.kodex.cli.history.createAgentHistoryViewModel` [857–861][history-impl]
  → `io.github.stream29.kodex.cli.history.AgentHistoryViewModelImpl` [47–834][history-impl].
  RPC Agent at 108–112 supplies real storage/latest/display/nonce and a dedicated
  child scope. History owns cross-timeline grouping, sparse/chunk reads, current
  bounded item identities and lazy payload lifetimes. No new Aggregate/Timeline VMs.
- `io.github.stream29.kodex.cli.history.AgentHistoryView` [97–358][history-view]
  receives **the spec** from C1, reports copied portable visible children of a
  captured window, dispatches nine original variants and forwards right-click
  actual item/index/generation. VM validates exact window/items at 202–225;
  acknowledgment CAS at 228–240 cannot clear a newer same-target effect.
- Replacement at 466–521 withdraws effects, invalidates caches and releases owned
  reads; publish at 800–834 prunes current viewport and evicted targets. VM has no
  Compose/Mosaic/LazyList state/imports in its main ownership implementation.
- Renderer [132–200][history-view] classifies consumed pointer/keyboard vertical
  scroll synchronously through a mount-scoped listener. Focus/programmatic layout
  is not user follow intent. Disposal unbinds and reports empty current viewport.
  Latest/item effects wait for measured layout over frames, verify generation,
  effect instance and target item identity, account for transient prefix and only
  acknowledge the current window. Late mount retains pending effect; unmount
  cancels widget wait, not History. Repeated same target gets a new effect object.
- [HistoryViewportAndScrollEffectTest][viewport-tests] uses the original real
  in-memory VM for repeated targets, stale ack, nonce/revert/close and copied
  sparse/top-level-group viewport retention. These are **model calls**, not input.
  [AgentHistoryRendererStateTest][history-renderer-tests] uses actual wheel/keys,
  real VM late mount, fake dynamic-prefix model, exact retained-state unmount/
  remount and measured pointer WorkGroup expansion. The remount toggle is a host
  seam, not a production tab command. Root and historical native evidence supplement it.
- [DocsHistoryRecordingTest][docs-history] at 63–161 sends actual motion/right-click/
  Check out/latest into real Sidebar/History/RPC seeded storage; asserts Check out
  leaves storage unchanged. At 164 onward real dialog cancels/reverts disposable
  storage, but its assembled callback/tab adapter is **not the complete current
  SessionTreeCliScreen failure path**; it does not clear F1.

Nine variants belong to these three existing History builds, not nine extra projects.
FQCNs in the contract column have prefix
`io.github.stream29.kodex.app.history.contract.item.`; lazy implementations have
prefix `io.github.stream29.kodex.cli.history.`. The suffixes below are exact declarations.

| Original contract FQCN suffix / exact source | Actual owner/implementation evidence | Actual rendering / smallest discriminating ablation |
| --- | --- | --- |
| [MessageHistoryItemViewModel][item-message] | [MessageHistoryItemViewModelImpl][impl-message]: lazy stable event/timestamp/turn duration; releasable initial loading | StoredHistoryContent 540 onward; cancel evicted payload Job while preserving same displayed content/model |
| [ReasoningHistoryItemViewModel][item-reasoning] | Immutable actual spec class constructed by original descriptor factory at History VM 300–301 | Think header at renderer 527–532; no empty implementation wrapper to remove |
| [ToolHistoryItemViewModel][item-tool] | [ToolHistoryItemViewModelImpl][impl-tool]: lazy event and owned expanded tool/raw work, release collapses/cancels | Original content/tool disclosure dispatch; remove only leaked read, retain shell registry/content rendering |
| [PatchHistoryItemViewModel][item-patch] | [PatchHistoryItemViewModelImpl][impl-patch]: actual patch payload/lazy lifetime, not a timeline aggregate | Original Patch branch; specialist cli.patch line paging/diff acceptance belongs UI/Home, not a second project here |
| [PlanUpdateHistoryItemViewModel][item-plan] | [PlanUpdateHistoryItemViewModelImpl][impl-plan]: original completed plan payload, releasable loading | Original Plan branch; canceled eviction must not republish payload |
| [ContextCompactionHistoryItemViewModel][item-compaction] | Immutable spec value constructed at History VM 318–319; same original storage boundary/elapsed fact | Context compacted header 533–538; preserve boundary mapping, do not manufacture a DTO/VM pair |
| [RequestUserInputHistoryItemViewModel][item-input] | [RequestUserInputHistoryItemViewModelImpl][impl-input]: completed stored answer, not active C7 draft owner | Read-only completed question branch; release loading, retain answer ordering/content |
| [SuggestSubagentTaskHistoryItemViewModel][item-suggestion] | [SuggestSubagentTaskHistoryItemViewModelImpl][impl-suggestion]: completed stored response, not active C14 batch editor | Completed sessions/feedback branch; preserve raw accepted/rejected event |
| [WorkGroupHistoryItemViewModel][item-group] | [WorkGroupHistoryItemViewModelImpl][impl-group] plus original History group cache/key/readChunkChildren | StoredHistoryWorkGroup 426–483 actual expand/collapse; release nested expansion/cached children, not grouping owner |

- [HistoryItemLoadContext][history-context] resolves original index/work/timestamp
  with generation guards. Aggregate cache prune at 379–418 releases evicted leaves
  and collapses groups. Message/Plan/Request/Suggestion now implement the release
  hook; Tool/Patch release and group collapse remain actual responsibilities.
  [HistoryItemReleaseTest][release-tests] checks those four dormant initial jobs.
- Minimum ablation: retain the single original VM, bounded structural algorithm
  and nine item representations. Remove renderer state from VM only; exact copied
  viewport and pending-effect slot must still govern original chunk retention.
  Disable release alone to discriminate payload lifetime, not reroute source reads
  or loosen bounds/timeouts. F3 is a separate navigation-admission doubt.
- **Historical 1,000-item stress timeout remains a risk**, not a root cause proved
  to be these leaf jobs, F3 or Mosaic. Current bounded-window/source tests keep
  original count/bound/timeout assertions; a historical pass is not all-pressure
  certification. No causal claim or unauthorized algorithm change is made here.

### C4 — History Index

- `io.github.stream29.kodex.app.agent.contract.HistoryIndexViewModel`
  [66–182][index-spec] → `io.github.stream29.kodex.cli.agent.createHistoryIndexViewModel`
  [51–56][index-impl] → `io.github.stream29.kodex.cli.agent.HistoryIndexViewModelImpl`.
  RPC Agent 113–128 supplies actual index/timestamp/externalWrite/nonce and binds
  scroll to the **same History VM**; no storage mutation is named Check out.
- C12 borrows `agent.historyIndex`; actual
  `io.github.stream29.kodex.cli.historyindex.HistoryIndexSidebarBody`
  [98–217][index-view] owns local follow/list/hover geometry. Each side acquires/
  releases its own row/detail/timestamp handles. Late/uncooperative results cannot
  write a released revision; failures are explicit, cancellation not Failed.
- [HistoryIndexComponentTest VM][index-tests] covers sparse data/equal boundaries/
  decreasing nonce, independent consumers, delayed reads/release, failure and
  owner cancellation. [View tests][index-ui-tests] use genuine mouse/wheel/Enter
  with fake callback child; C3 Docs supplies original real storage/RPC Check out.
- Minimum ablation: remove only duplicate source authorities, **not** independent
  read handles/revision guards. Delayed nonce replacement and two sidebars must
  retain exact current reads and navigate original History without reverting storage.

### C5 — New Session hard cut

- `io.github.stream29.kodex.app.session.contract.NewSessionViewModel`
  [18–68][new-spec] → actual
  `io.github.stream29.kodex.cli.newsession.DefaultNewSessionViewModelFactory`
  / `RpcNewSessionViewModel` [14–60][new-impl] → original
  `io.github.stream29.kodex.cli.rpc.RpcSessionDraft` [27–110][draft].
  Host constructs factory at 98, parent materialization A consumes this exact child.
- `io.github.stream29.kodex.cli.app.NewSessionScreen` [31–98][new-view] borrows it;
  root supplies status slot and exact captured submit callback at 562–569.
  Display label is local; explicit name/configuration/Composer are consumed once,
  known allocation is retained after later failure. No component wrapper,
  creation receipt model, new page state or replay create command.
- [NewSessionViewModelTest][new-tests] directly uses real fixture/factory for
  allocation/no premature Composer persistence, tuple config/close and name behavior.
  [NewSessionScreenTest][new-ui-tests] uses actual Enter/paste/cursor/undo and
  verifies unmount does not close borrowed draft. Enter's callback capture test is
  not production queued Application materialization; A tests that separately.
- Minimum ablation: retain original draft identity/allocation mutex and consumed
  index. Remove only detached scope in F2, parenting the real Composer to the
  original draft owner and closing on termination; no new draft lease/wrapper.

### C6 — Path Picker

- `io.github.stream29.kodex.app.pathpicker.contract.DirectoryPickerViewModel`
  [21–106][path-spec] / DirectoryPickerBrowser capability →
  `io.github.stream29.kodex.app.pathpicker.createDirectoryPickerViewModel`
  [208–224][path-impl] → `DirectoryPickerViewModelImpl` [27–195][path-impl].
  Default entry constructs actual SystemDirectoryPickerBrowser [20–68][path-browser],
  with SystemCoroutineFileSystem IO and cancellation rethrow, not fake directories.
- Host directory factory at 99 transfers original child to C15/Settings;
  `io.github.stream29.kodex.cli.pathpicker.DirectoryPickerPopup` [52 onward][path-view]
  renders load/validation errors, filter/navigation and exact selection effect.
  Latest request IDs guard asynchronous load/confirm; close closes effects/cancels
  owned work, never borrowed caller scope.
- [DirectoryPickerDependencyTest][path-tests] uses dependency gates, not platform
  actual certification. [Popup tests][path-ui-tests] use genuine keys/clicks;
  common browser tests and Linux-specific source exist, but no actual ran here.
- Minimum ablation: replace only browser source with a gated deterministic port;
  late old navigation must not publish/confirm after a new request or close.
  Preserve filesystem result distinctions, VM-owned filter query and View-owned
  input/focus geometry.

### C7 — Request User Input

- `io.github.stream29.kodex.app.agent.contract.RequestUserInputViewModel`
  [131–234][input-spec] → `io.github.stream29.kodex.cli.agent.createRequestUserInputViewModel`
  [33–46][input-impl] → `RequestUserInputViewModelImpl` [49–213][input-impl].
  RPC Agent 129–140 binds first backend pending projection, actual complete
  tool event and same-Agent resume; C1 borrows original child into
  `io.github.stream29.kodex.cli.agent.RequestUserInputPanel` [35 onward][input-view].
- Call/revision guards, question order, Other/free-form transformation and single
  business answer draft are actual. Owner async separates caller's await from
  accepted submission; close releases observation but accepted work drains in
  borrowed owner scope; owner termination cancels it.
- [RequestUserInputViewModelTest][input-tests] exercises busy/failure/dependency vs
  caller cancellation/close/owner cancellation. **217–243 explicitly asserts
  replacement remains visible but submits StaleCall after old success**.
  [RequestUserInputOptionsTest][input-ui-tests] sends genuine question keys with
  fixture ports; it does not repair this legacy behavior. See D2.
- Minimum ablation: keep actual tool protocol and same-Agent command ports;
  compare clearing internal pending event against the captured event/call only.
  In an authorized behavior change, new visible call must remain actionable after
  old late success without replaying completion. Not a second pending VM.

### C8 — Runtime Configuration

- `io.github.stream29.kodex.app.runtimeconfiguration.RuntimeConfigurationViewModel`
  [33–143][runtime-spec] → `createRuntimeConfigurationViewModel` /
  `DefaultRuntimeConfigurationViewModelFactory` [21–37][runtime-impl] →
  private `io.github.stream29.kodex.app.runtimeconfiguration.DefaultRuntimeConfigurationViewModel`.
- Real `io.github.stream29.kodex.cli.rpc.createBoundRuntimeConfigurationViewModel`
  [15–32][rpc-runtime] binds exact Agent/draft once, projects observed settings,
  sends atomic tuple to captured target; never re-resolves selected tab.
  Agent/New Session original stable property reaches
  `io.github.stream29.kodex.cli.runtimeconfiguration.RuntimeConfigurationTriggers`
  and `RuntimeConfigurationMenus` [33 onward][runtime-view].
- VM owns source observation and tracked caller child waits, close cancels those
  waits/owner, not borrowed settings. Successful reply does not impersonate a new
  source snapshot. Missing current model/custom effort/tier remains visible;
  menu fallback does not silently correct persisted settings.
- [RuntimeConfigurationViewModelTest][runtime-tests] covers tuple/source receipt,
  caller cancel, close/owner cancel and old target. [View test 51–99][runtime-ui-tests]
  sends click→Right→Right→Down→Enter, one tuple/mode write, fake VM.
  RPC tests at 267–287 cover exact real target and unrelated-field retention.
- Minimum ablation: keep one bound target and source-authoritative projection;
  remove model/effort intermediate writes, not tuple port. Real menu traversal
  must make no writes before tier selection and one exact tuple afterward.

### C9 — Session Catalog

- `io.github.stream29.kodex.app.sessioncatalog.contract.SessionCatalogViewModel`
  [72–214][catalog-spec] + Dependencies/Interactions →
  `io.github.stream29.kodex.app.sessioncatalog.DefaultSessionCatalogViewModel`
  [26–144][catalog-impl]. Host actual factory at 102–105 supplies
  RpcSessionCatalogDependencies; Application 184–207 binds exact opening/navigation.
- `io.github.stream29.kodex.cli.sessioncatalog.SessionCatalogPopup`
  [63–180][catalog-view] receives spec, real row/menu and existing Delete child.
  Row equality is intentional loaded-data equality under StateFlow conflation,
  unlike local tab identity. Mutation→reload preserves prior loaded snapshot on
  failure and **rethrows**; cancellation links owner and caller wait.
- [SessionCatalogInteractionTest][catalog-tests] specifically tests actual
  exception escape/no-dismiss/no-replay, equal refreshed rows and stale child.
  [Popup tests][catalog-ui-tests] use actual keys/pointer with fake ports.
  Their retained-popup checks do not prove a real failure outlet: F1 contradicts
  the renderer's report-by-registry assumption.
- Minimum ablation: connect one real owning failure outlet (or propagate once at
  renderer command boundary), retaining exact opening/delete child and failure
  snapshot. Do not add another catalog failure VM or return empty Loaded success.

### C10 / C11 — Delete and Rename

- Delete: `io.github.stream29.kodex.app.sessiondelete.contract.SessionDeleteViewModel`
  [10–41][delete-spec] →
  `io.github.stream29.kodex.app.sessiondelete.createSessionDeleteViewModel` [6–22][delete-impl]
  → private `DefaultSessionDeleteViewModel`. Real Application 235–250 and Catalog
  56–83 bind the original deletion; no independent tab/catalog state. Renderer
  `io.github.stream29.kodex.cli.sessiondelete.SessionDeletePopup` [28–64][delete-view]
  starts Cancel-focused and reports exact active-child true/false result.
- Rename: `io.github.stream29.kodex.app.sessionrename.contract.SessionRenameViewModel`
  [16–43][rename-spec] →
  `io.github.stream29.kodex.app.sessionrename.createSessionRenameViewModel` [9–32][rename-impl]
  → private `DefaultSessionRenameViewModel`. Real Application 209–233 captures
  original Session target; `io.github.stream29.kodex.cli.sessionrename.SessionRenamePopup`
  [42–100][rename-view] owns cursor/selection/undo and sends bound trim/rename.
- These small implementations legitimately have no observation scope: synchronous
  local active/draft state plus caller suspend port is a real responsibility,
  not grounds for another manager/empty infrastructure interface.
- [Delete VM][delete-tests], [Delete popup][delete-ui-tests],
  [Rename VM][rename-tests], [Rename popup][rename-ui-tests] inspect failure/cancel/
  replacement/close and actual button/Enter behavior. Application action tests
  supply actual captured factory/target evidence; leaf UI ports alone are not RPC.
- Minimum ablation: remove redundant root title/delete state, not these bound
  commands or widget buffers. Late result on replaced popup must not consume a
  new opening; deletion false stays false, actual failures propagate, rename edits
  original target only. Do not fabricate a scope for pure local state.

### C12 — Sidebar and actual Shell/History Index composition

- `io.github.stream29.kodex.app.application.contract.SidebarSettingsViewModel`
  [26–54][sidebar-spec] →
  `io.github.stream29.kodex.cli.app.SidebarSettingsViewModelImpl` [12–39][sidebar-impl]
  constructed in host 154. Content comes from original frontend preference owner;
  widths are invocation-local, combine projection not a second persistence store.
- `io.github.stream29.kodex.cli.app.SessionSidebar` [75–190][sidebar-view]
  is the real None/TerminalSessions/HistoryIndex renderer. Root 169 onward keeps
  hover/pin/drag/anchors/popups/focus in View. Shell close menu captures original
  handle at 497–520, filters completion/sorts IDs at 654–666; it does not close
  Agent/registry on unmount or invent a Shell project.
- [SessionSidebarTest][sidebar-ui-tests] sends actual clicks/hover/wheel, including
  two sidebars, same-id replacement and completion-dismissed Close menu at 810–904;
  Shell snapshots are fixture registry inputs, not a running external process.
  [SidebarSettingsViewModelTest][sidebar-tests] covers actual preference/transient
  widths. Width baseline usability debt D1 remains.
- Minimum ablation: retain shared preference owner and exact borrowed children;
  remove no geometry into spec. Replacing same-ID Shell must not retarget an old
  menu; a completed process remains retained in raw registry, disappears only in UI.

### C13 — Tab Bar (legitimate spec + stateless View)

- `io.github.stream29.kodex.app.sessiontabbar.contract.SessionTabBarState`,
  `SessionTabIdentity`, `SessionTabPresentation`, `SessionTabBarCallbacks`
  [9–75][tabs-spec] are actual immutable UI-input/callback contracts. No mutable
  tab business VM is required beside Application.
- `io.github.stream29.kodex.cli.sessiontabbar.SessionTabBar` [58–180][tabs-view]
  is the real renderer; scroll/focus/geometry stay local. Production root builds
  presentations/identity→captured-target adapter at 135–158, callbacks at 407–415.
  F4 flags equality in that **host adapter**, not missing TabBar implementation.
- [SessionTabBarTest][tabs-ui-tests] uses genuine click/keys/wheel over stateless
  callbacks; A's exact-target tests use real command/factory path separately.
  Neither demonstrates equal local children mounted in the full production root.
- Minimum ablation: no second tab registry/VM; retain immutable unique identity
  callbacks, make host identity referential where required. Removing target capture
  should break moved/closed target admission, not be masked by current selected index.

### C14 — Suggest Subagent Task

- `io.github.stream29.kodex.app.agent.contract.SuggestSubagentTaskViewModel`
  [69–207][suggest-spec] →
  `io.github.stream29.kodex.cli.agent.createSuggestSubagentTaskViewModel` [34–50][suggest-impl]
  → `SuggestSubagentTaskViewModelImpl`. RPC Agent 141–173 supplies actual batch
  creation/complete/resume/defaults/catalog and accepted-index navigation callback.
  C1 borrows actual child into
  `io.github.stream29.kodex.cli.agent.SuggestSubagentTaskPanel` [83 onward][suggest-view].
- Actual single business draft owns feedback/reject/configuration; menus/anchor/
  cursor are renderer state. Field edits merge latest same-call tuple/cwd/mode.
  Default configuration sampled once per new call, catalog changes not silent edits.
  Owner async admits once by call/revision, caller cancellation stops awaiting;
  close releases observer but accepted batch belongs to original owner.
- [SuggestSubagentTaskFeedbackTest][suggest-tests] preserves protocol/field-merge/
  create→complete→resume, partial failure/explicit retry, late replacement and
  owner cancellation. [Panel test][suggest-ui-tests] sends real wheel/click/
  feedback typing/Enter with ports. Neither certifies live provider child creation.
  Late success and partial side effects are D2–D3, not repaired by green snapshots.
- Minimum ablation: retain one editor and real backend batch port; no adapter that
  clones configuration into parent. Test captured-event clearing independently;
  any future receipt/idempotency policy needs separate authorization, not invented
  “transactional” wrapper or automatic compensation/replay.

### C15 — Working Directory

- `io.github.stream29.kodex.app.workingdirectory.contract.WorkingDirectoryViewModel`
  [18–83][working-spec] →
  `io.github.stream29.kodex.app.workingdirectory.createWorkingDirectoryViewModel`
  [16–37][working-impl] → private `DefaultWorkingDirectoryViewModel`.
  Exact DirectoryPicker is owned/transferred, selection invokes one captured port,
  success closes owned picker; failures/cancellation do not invent a cwd fallback.
- Application 269–304 binds Agent/draft target; suggested-child adapter binds
  current call ID, not source-Agent cwd. Settings' local revision queue/temporary
  browser host is a specialized Settings responsibility, not duplicated here.
- `io.github.stream29.kodex.cli.workingdirectory.WorkingDirectoryPopup`
  [22–57][working-view] borrows nested picker with `closeOnDispose=false`, owns
  short parent lifetime and exact effect selection/active renderer guard.
- [WorkingDirectoryViewModelTest][working-tests] covers transfer/failure/cancel/
  consumed/close semantics; [Popup tests][working-ui-tests] send actual Enter/Escape
  and replacement; [Application ownership test][working-root-tests] verifies actual
  captured target without manual browser callbacks masquerading as a UI smoke.
- Minimum ablation: retain exact selection capability/picker owner, not another
  settings copy. Gated delayed selection after parent replacement must not change
  new target/call; remove duplicate root routing while preserving actual IO owner.

## Build/source-set/Koin/history review

- All 50 listed build declarations were inspected, separating main/test dependencies.
  Spec declarations use framework-free CLI convention, VM projects the existing
  [viewmodel convention][vm-convention], Views the shared Mosaic hierarchy.
  No new per-component target hierarchy, artificial nine item builds or Shell build.
- Koin compiler convention enables compileSafety/strictSafety/unsafeDslChecks.
  History/New Session/PathPicker annotation modules are component-scan declarations,
  e.g. [HistoryViewModelModule][history-koin]. **Production chain here is the
  explicit typed root factories**, not an imagined Koin resolver registration.
  Generated Koin/resolved configurations were not regenerated or executed here.
- app/impl/rpc's RPC-client and original History-VM dependencies are real protocol/
  component composition, allowed; its observable storage/state projections do not
  duplicate AgentRuntime/History ownership. Root View's genuine widget/helper
  dependencies and typed factory adapters are not faults merely for containing “State”.
- [CLI convention][cli-convention] keeps original Native executable and entrypoint.
  RPC fixture has real test-only consumers; no extra empty fixture spec is required.
  Whole resolved graph/substitution closure remains the coordinator's gate, not
  inferred from only explicit `project(...)` edges.
- Migration history checked read-only: `821b6627` real Agent/History hard migration;
  `9272f26f` root hard cut; `8297d388` exact-target/child ownership tests;
  `ec9bf1c4` portable History/View-state separation; pinned `6b7129fa` tests.
  Original FQCNs and cross-timeline History responsibilities remain, not invented
  DDD Aggregate/Timeline renaming.

## Findings: actual errors and specific doubts

### F1 — B1: ordinary History/catalog failures swallowed under a nonexistent reporting boundary

- **Evidence:** production root [931–970][root-history-failure] swallows rejected
  requestHistoryRevert, failed revert-and-edit and failed fork/open, claiming Agent/
  owning boundary reports. Actual RPC Agent [216–221, 307–310][rpc-agent-direct]
  simply validates/throws or awaits binding; binding [54–78][rpc-binding] only
  invalidates on SessionNotActive/NotFound and rethrows. Session fork [106–113][session-impl]
  and Application open [83–103][application-impl] similarly propagate, no notification.
  Root revertAndEdit [1328–1336][root-edit] awaits revert before Composer update.
- Catalog [58–60, 99–111, 155–165][catalog-view] makes the same false assertion.
  Actual catalog [42–54, 108–125][catalog-impl] and Application/registry propagate;
  failure reload restores old snapshot but produces no error outlet.
- **Impact:** ordinary backend/storage/admission failures disappear from these UI
  actions; menu may disappear or catalog stay open without any visible/reportable
  failure. Unchanged draft/data is not error reporting. Cancellation still rethrows;
  this is not a cancellation-to-failure misclassification.
- **Do not generalize:** asynchronous `confirmHistoryRevert`/resume/Stop use actual
  caught Agent `command` boundary [229–275][rpc-agent-direct] and notifications.
  Current Composer-host tests prove that boundary. They do not cover these direct
  throwing paths. CLI logger cannot log exceptions already swallowed.
- **History:** the root catches and parentless draft existed at `9272f26f^` under
  old `app/view/application` / `app/viewmodel/rpc` paths. Not introduced by the
  latest History portable-state migration; original assumption was not repaired
  by moving/merging source. Catalog swallowing is also current inherited behavior.
- **Test limit:** success Docs menus and isolated catalog exception/retained-popup
  tests do not inject failures through the full current root and verify one outlet.
- **Minimum experiment/ablation:** inject one ordinary RPC exception in real
  request/revert/fork/open; drive measured real context menu/catalog Enter or click.
  Assert exactly one owning error, unchanged unsent draft/current target, no replay,
  catalog not falsely successful. Remove false swallowed-report premise or wire
  existing owner outlet; do not add a parallel failure VM/History wrapper.
  Keep cancellation rethrow and backend admission. Coordinator should deduplicate
  cross-lane error-outlet observations against this root cause.

### F2 — B1: draft Composer is outside the supplied New Session/fixture owner

- **Evidence:** [RpcNewSessionViewModel 32–39, 42–60][new-impl] parents its own Job
  and configuration/name observations to supplied scope, but creates RpcSessionDraft
  without that scope and has no owner-completion draft-close hook.
  [RpcSessionDraft 36–60][draft] creates `CoroutineScope(SupervisorJob())` without
  parent and feeds it to real Composer. [Composer 61–77][composer-impl] starts three
  permanent StateFlow observers and closes on **that** detached Job's completion.
  Only explicit draft.close at 105–109 cancels it.
- **Impact:** canceling supplied factory owner terminates New Session projections
  but does not close Composer/its three observers; retained Composer still admits
  local edits. Draft setting edits check closed/allocation, not terminated owner.
  Fixture [64, 94–102][fixture] says it owns all frontend jobs until closeAndJoin,
  yet factory-created drafts escape it unless individually explicitly closed.
- **History:** parentless draft scope is byte-for-byte present at old path in
  `9272f26f^`; this is retained ownership debt, not a proven new refactor regression.
  Current production host's explicit app.close closes owned drafts on normal exit;
  do not claim its routine CLI shutdown has been reproduced leaking.
- **Test limit:** current New Session tests explicitly close models or exercise
  view unmount; no supplied-owner-cancel-only Composer closure assertion. Composer's
  own owner-cancel test cannot cover the wrong owner actually supplied by draft.
- **Minimum experiment/ablation:** real DefaultNewSession factory with isolated
  supplied Job; create unallocated draft, cancel/join that Job without model.close.
  Expect ComposerLifecycle.Closed, edits rejected, no surviving draft observers;
  also fixture.closeAndJoin with an unclosed factory draft. Parent existing draft/
  Composer scope to original owner and use original draft close on termination,
  preserving independent local close, no backend allocation/Stop, no wrapper/lease.

### F3 — B2: latest navigation intent can be dropped when History command buffer is full

- **Evidence:** [History VM 52, 244–258][history-send] uses bounded Channel.BUFFERED.
  Both navigation methods supersede `navigationIntent` and withdraw current effect,
  then ignore `trySend` result. A queued old navigation checks its identity and
  skips once superseded; a dropped newest command has no later effect-producing
  command. Older/newer paging at 172–198 correctly resets pending on send failure,
  so this is specifically navigation, not a blanket paging defect.
- **Impact hypothesis:** blocked structural IO plus a burst exceeding capacity can
  clear old effect/intent without delivering latest index/latest request. A newer
  follow flag alone is not proof of fulfillment when older history is mounted.
- **History:** current navigation/effect methods changed in `ec9bf1c4`; original
  command buffer existed before. This specific saturation consequence has **not
  been run here**. Not claimed as cause of historical stress timeout.
- **Test limit:** repeated-target VM tests await each effect sequentially; late-mount
  renderer test sends one request. Neither saturates while structural read is gated.
- **Minimum experiment/ablation:** gate original source read, enqueue e.g. 256
  alternating storage targets/latest intents, release gate, assert newest exact
  target/effect arrives and stale acknowledgments cannot consume it. Handle/coalesce
  admission within original single owner; do not add unbounded journal or second VM.

### F4 — B2: root tab adapter/deletion use equality where child contract permits referential twins

- **Evidence:** [root 135–158, 1036–1040][root] uses ordinary
  `MutableMap<SessionViewModel, SessionTabIdentity>`, getOrPut and toSet/retainAll.
  Distinct equality-equal children can receive the same identity; actual
  [SessionTabBarState 45–50][tabs-spec] rejects repeated identities.
  Application deletion [414–418][application-impl] uses `in`/indexOf equality to
  retain selection, unlike exact-target select/materialize. Spec navigation explicitly
  allows distinct `===` children, and [exact command tests 104–127][exact-tests]
  deliberately exercise equality-equal twins.
- **Qualified impact:** current real RpcNewSession/RpcPersisted concrete classes
  use reference equality, so **no production-user failure reproduced**. Broader
  spec/typed-factory substitutability and root equality-twin behavior are unverified.
  History state's own prune/lookup uses exact references and is **not this defect**;
  root Compose `key(tab)` likewise deserves inclusion in the twin host experiment.
- **History/test limit:** root tab registry is inherited; recent equal-target test
  calls VM directly, while equal-History test calls renderer-state helpers. Neither
  mounts equal local Session children in the full root and routes real tab input.
- **Minimum experiment/ablation:** actual Application factory with two distinct
  equal children, mount current root; assert unique tab IDs/no construction failure,
  genuine tab click and delete preserve exact surviving selection, departed identity
  pruned. Use referential host lookup/pruning and exact selected-child check in
  existing structures; do not invent another navigation registry or storage ID.

## Explicit debts / legitimate exceptions

### D1 — inherited narrow-screen dual Sidebar policy

- [Sidebar layout 819–888][sidebar-layout] retains only one center column minimum.
  [Prior actual CLI record 326–329][old-sidebar] reports 120→60 columns with two
  roughly 29-column sidebars leaving unusably narrow center; collapsing restores
  usability. Current policy/tests retain it, not resolved by component ownership.
- Separate layout-policy review/resize experiment is needed. Keep width/hover/
  geometry in View; a new geometry VM is not the fix.

### D2 — documented pending-tool late-success/cancellation consequences

- Request [177–186][input-spec], Suggest [139–144][suggest-spec] deliberately retain
  old success clearing internal pending event even after replacement while preserving
  newer visible snapshot. Current [Request tests 217–243][input-tests] and
  [Suggest tests 260–287][suggest-tests] assert the visible replacement then returns
  StaleCall/Stale on submit. This is **actual documented problematic behavior**,
  not an inferred snapshot overwrite or repaired capability.
- Dependency cancellation can leave Submitting until replacement/close; caller
  cancellation merely stops awaiting. Request blank failure message has explicit
  representation exception. Current contract accurately records these caveats;
  legacy behavior was in `7310a445` implementation, not created by root merge.
- Gate: review behavior policy and test replacement's *next genuine submit*,
  not just equality of displayed state. Existing capture/compare-clear responsibility
  can fix capability without a parallel pending authority, if separately authorized.

### D3 — suggestion creation and completion are separate commits

- [Spec 133–144][suggest-spec] explicitly does not promise exactly-once creation.
  [Tests 197 onward][suggest-tests] preserve created batch after completion failure
  and show explicit retry creates again. View has no new independent failure banner/
  receipt model; Failed return and editable state are not “no side effects.”
- Keep this policy explicit. Further retry/idempotency/UI feedback needs separately
  reviewed semantics and real batch failure input; do not falsely accept a transaction
  or automatically delete/replay created children.

### Legitimate boundaries that should not be “fixed” into empty pairs

- Raw RPC cache/binding, read-only Session settings retention, token/output
  projections, target-capturing popup adapters and renderer widget state have real
  responsibilities. No generic Adapter/Handle/State-name condemnation.
- Sidebar settings uses original application-scoped owner; Shell comes from real
  Agent registry; TabBar is immutable spec + stateless View; Reasoning/Compaction
  are concrete immutable spec representations; CLI/fixture are composition/
  infrastructure, not missing MVVM/spec implementations.

## Historical evidence versus current unperformed gates

- [Root hard-cut record][old-validation] reports 650 JVM cases across sixteen
  targets including UP-TO-DATE and actual isolated Linux Native CLI. It also
  records the independently reproduced registry-release race and original-mutex/
  exact-view correction. Current S source/test chain matches that responsibility.
- [UI/History/Home closure record][recent-validation] reports 612 cases including
  UP-TO-DATE; later five-target `--rerun` 402 cases plus one explicitly enabled
  Patch probe. Actual isolated native History scroll→tab roundtrip had identical
  complete plain output and latest returned newest. [History closure][history-done]
  separately records VM42/View71/Root54 and genuine input vs model seams.
- These are **historical records**, not tests executed by this read-only audit.
  Existing build outputs/cached green results were not used as a new acceptance.
  Patch-specialist assertions and Home lease root cause remain assigned UI/Home.
- **U — not reviewed as runtime acceptance:** all fresh build/test execution;
  Mac/Windows actuals, Desktop renderer, live OpenAI/OAuth/MCP/provider/real Shell
  process behavior, crash/restart and long-run performance; generated Koin/resolved
  external dependency variants; F1–F4 discriminating experiments.
- **U — stronger combined input gate still missing:** hidden *actual* RPC rebind
  while another tab is selected, then real production TabBar roundtrip and exact
  History-state pruning/anchor. Current source holds subscriptions/pruning; existing
  tests split real RPC binding behavior from delegated root flow/widget behavior.
  Historical actual native tab roundtrip covers tab preservation, not this whole race.
- Retained State compact-concurrent-settings and Home lease cancellation/cleanup
  failures belong their owners and remain baseline risks. No pass, source move or
  new release test here proves their root cause fixed. Historical History stress
  risk likewise remains open without causal attribution.

## Handoff

- **REVIEW READY** for the parent's asynchronous read and independent deduplication.
  Main can consume this report now; no repair/commit/next-stage authorization inferred.
- Final source baseline remains the pinned HEAD with clean inner status.
  All fifty build rows and current source/test links are individually recorded.
  Read-only report validation matched 50 unique build URIs to the exact inventory,
  checked URI existence/line bounds and defined references, and found no whitespace
  diagnostics. These checks are report integrity, not compilation/test execution.
  No source or other task/report was changed by this lane.

[parent]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-final-reaccept-spec-impl-repository.md#L50
[application-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/spec/application/src/commonMain/kotlin/io/github/stream29/kodex/app/application/contract/ApplicationViewModel.kt#L11
[navigation-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/spec/application/src/commonMain/kotlin/io/github/stream29/kodex/app/application/contract/ApplicationNavigationState.kt#L27
[popup-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/spec/application/src/commonMain/kotlin/io/github/stream29/kodex/app/application/contract/ApplicationPopupState.kt#L12
[application-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L56
[host]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L55
[exact-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/ExactTargetCommandTest.kt#L20
[action-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/SessionActionComponentOwnershipTest.kt#L1
[session-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/spec/session/src/commonMain/kotlin/io/github/stream29/kodex/app/session/contract/SessionViewModel.kt#L18
[persisted-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/spec/session/src/commonMain/kotlin/io/github/stream29/kodex/app/session/contract/PersistedSessionViewModel.kt#L22
[registry-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/spec/session/src/commonMain/kotlin/io/github/stream29/kodex/app/session/contract/PersistedSessionViewModelRegistry.kt#L16
[session-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt#L15
[rpc-views]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionViews.kt#L39
[rpc-agent]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentViewModel.kt#L59
[rpc-agent-direct]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentViewModel.kt#L216
[rpc-binding]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionBinding.kt#L54
[rpc-output]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcOutput.kt#L21
[rpc-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionViewsTest.kt#L223
[binding-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/session/src/commonTest/kotlin/io/github/stream29/kodex/cli/session/SessionBindingOwnershipTest.kt#L35
[root]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L120
[root-history-failure]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L931
[root-edit]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L1328
[history-retention]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/HistoryRenderState.kt#L18
[history-view-state]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewState.kt#L16
[retention-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/HistoryRenderStateTest.kt#L16
[cli]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L11
[cli-logger]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/CoroutineFailureLogging.kt#L7
[fixture]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/test-support/rpc/src/commonMain/kotlin/io/github/stream29/kodex/app/test/RpcFrontendFixture.kt#L29
[agent-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/AgentViewModel.kt#L95
[shell-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/AgentState.kt#L13
[agent-screen]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L49
[agent-ui-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/ComposerHostRepairTest.kt#L61
[composer-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/ComposerViewModel.kt#L299
[composer-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/ComposerViewModel.kt#L26
[rpc-composer]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcComposer.kt#L33
[composer-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/agent/ComposerView.kt#L49
[composer-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/ComposerInput.kt#L28
[composer-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/agent/ComposerViewModelTest.kt#L37
[history-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/AgentHistoryViewModel.kt#L129
[history-source]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistorySource.kt#L20
[history-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L47
[history-send]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L244
[history-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt#L132
[history-context]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/HistoryItemLoadContext.kt#L111
[viewport-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/history/HistoryViewportAndScrollEffectTest.kt#L37
[history-renderer-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryRendererStateTest.kt#L63
[docs-history]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsHistoryRecordingTest.kt#L63
[release-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/history/HistoryItemReleaseTest.kt#L16
[item-message]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/MessageHistoryItemViewModel.kt#L10
[item-reasoning]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/ReasoningHistoryItemViewModel.kt#L11
[item-tool]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/ToolHistoryItemViewModel.kt#L26
[item-patch]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/PatchHistoryItemViewModel.kt#L9
[item-plan]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/PlanUpdateHistoryItemViewModel.kt#L9
[item-compaction]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/ContextCompactionHistoryItemViewModel.kt#L9
[item-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/RequestUserInputHistoryItemViewModel.kt#L12
[item-suggestion]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/SuggestSubagentTaskHistoryItemViewModel.kt#L9
[item-group]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/WorkGroupHistoryItemViewModel.kt#L13
[impl-message]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/MessageHistoryItemViewModel.kt#L13
[impl-tool]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/ToolHistoryItemViewModel.kt#L14
[impl-patch]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/PatchHistoryItemViewModel.kt#L13
[impl-plan]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/PlanUpdateHistoryItemViewModel.kt#L12
[impl-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/RequestUserInputHistoryItemViewModel.kt#L12
[impl-suggestion]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/SuggestSubagentTaskHistoryItemViewModel.kt#L12
[impl-group]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/WorkGroupHistoryItemViewModel.kt#L17
[index-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/HistoryIndexViewModel.kt#L66
[index-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexViewModel.kt#L51
[index-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/historyindex/HistoryIndexComponent.kt#L98
[index-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexComponentTest.kt#L46
[index-ui-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/historyindex/HistoryIndexComponentTest.kt#L119
[new-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/session/contract/NewSessionViewModel.kt#L18
[new-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModel.kt#L14
[draft]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionDraft.kt#L36
[new-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/NewSessionScreen.kt#L31
[new-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModelTest.kt#L10
[new-ui-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionScreenTest.kt#L18
[path-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/pathpicker/contract/DirectoryPickerViewModel.kt#L21
[path-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/pathpicker/DirectoryPickerViewModel.kt#L208
[path-browser]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/pathpicker/DirectoryPickerBrowser.kt#L20
[path-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/pathpicker/DirectoryPickerPopup.kt#L52
[path-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/pathpicker/DirectoryPickerDependencyTest.kt#L17
[path-ui-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/pathpicker/DirectoryPickerPopupTest.kt#L58
[input-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/RequestUserInputViewModel.kt#L177
[input-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/RequestUserInputViewModel.kt#L33
[input-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/agent/RequestUserInputPanel.kt#L35
[input-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/agent/RequestUserInputViewModelTest.kt#L217
[input-ui-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/agent/RequestUserInputOptionsTest.kt#L166
[runtime-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/runtimeconfiguration/RuntimeConfigurationViewModel.kt#L33
[runtime-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/runtimeconfiguration/RuntimeConfigurationViewModel.kt#L21
[rpc-runtime]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcRuntimeConfiguration.kt#L15
[runtime-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/runtimeconfiguration/RuntimeConfiguration.kt#L33
[runtime-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/runtimeconfiguration/RuntimeConfigurationViewModelTest.kt#L100
[runtime-ui-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/runtimeconfiguration/RuntimeConfigurationTest.kt#L51
[catalog-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/sessioncatalog/contract/SessionCatalogViewModel.kt#L72
[catalog-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/sessioncatalog/DefaultSessionCatalogViewModel.kt#L26
[catalog-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessioncatalog/SessionCatalogPopup.kt#L58
[catalog-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/sessioncatalog/SessionCatalogInteractionTest.kt#L51
[catalog-ui-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessioncatalog/SessionCatalogPopupTest.kt#L88
[delete-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-delete/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/sessiondelete/contract/SessionDeleteViewModel.kt#L41
[delete-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-delete/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/sessiondelete/SessionDeleteViewModel.kt#L6
[delete-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-delete/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessiondelete/SessionDeletePopup.kt#L28
[delete-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-delete/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/sessiondelete/SessionDeleteViewModelTest.kt#L13
[delete-ui-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-delete/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessiondelete/SessionDeletePopupTest.kt#L57
[rename-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-rename/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/sessionrename/contract/SessionRenameViewModel.kt#L43
[rename-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-rename/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/sessionrename/SessionRenameViewModel.kt#L9
[rename-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-rename/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessionrename/SessionRenamePopup.kt#L42
[rename-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-rename/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/sessionrename/SessionRenameViewModelTest.kt#L19
[rename-ui-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-rename/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessionrename/SessionRenamePopupTest.kt#L57
[sidebar-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-sidebar/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/application/contract/SidebarSettingsViewModel.kt#L26
[sidebar-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/SidebarSettingsViewModel.kt#L12
[sidebar-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-sidebar/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt#L75
[sidebar-layout]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-sidebar/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt#L819
[sidebar-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/SidebarSettingsViewModelTest.kt#L1
[sidebar-ui-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-sidebar/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SessionSidebarTest.kt#L810
[tabs-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/sessiontabbar/contract/SessionTabBar.kt#L41
[tabs-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessiontabbar/SessionTabBar.kt#L58
[tabs-ui-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessiontabbar/SessionTabBarTest.kt#L24
[suggest-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/SuggestSubagentTaskViewModel.kt#L139
[suggest-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskViewModel.kt#L34
[suggest-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskPanel.kt#L83
[suggest-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskFeedbackTest.kt#L197
[suggest-ui-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskPanelTest.kt#L97
[working-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/working-directory/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/workingdirectory/contract/WorkingDirectoryViewModel.kt#L51
[working-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/working-directory/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/workingdirectory/WorkingDirectoryViewModel.kt#L16
[working-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/working-directory/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/workingdirectory/WorkingDirectoryPopup.kt#L22
[working-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/working-directory/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/workingdirectory/WorkingDirectoryViewModelTest.kt#L17
[working-ui-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/working-directory/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/workingdirectory/WorkingDirectoryPopupTest.kt#L79
[working-root-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/WorkingDirectoryComponentOwnershipTest.kt#L1
[vm-convention]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-viewmodel.gradle.kts#L22
[cli-convention]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-cli-executable.gradle.kts#L12
[history-koin]: file:///home/stream/ACodeSpace/local/kodex-e2e-20261007-7b23c481/evidence/source-baseline-6b7129fa/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/HistoryViewModelKoin.kt#L7
[old-validation]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-06-hard-migrate-frontend-ownership-roots.md#L261
[old-sidebar]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-06-hard-migrate-frontend-ownership-roots.md#L326
[recent-validation]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-cutover-ui-foundations-and-home-bootstrap.md#L116
[history-done]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-decouple-history-renderer-state.md#L86
