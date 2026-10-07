# Task Tree

- `Restore actual Composer output and ordinary-failure boundary`() — implementation complete
- `Add real host keyboard and bound command regressions`() — centrally validated
- `Publish handoff for centralized regression`() — complete
- `Record user acceptance and commit scoped repair`()

# Details

- Parent: [Reacceptance and authorized repairs](2026-10-05-reaccept-spec-impl-refactor.md).
- 用户接受当前修复并授权分批提交；内层提交 `a1adf2f2`，本轮未推送。
- Own Composer component source/tests, `AgentRuntimeScreen.kt`, `RpcComposer.kt`, `RpcAgentPresentation.kt`, `RpcSessionViews.kt` and focused related tests.
- Preserve single submit algorithm, fixed binding, caller cancellation and SessionNotActive handling.
- Restore visible status/error without duplicate pending preview; no no-op failure outlet.
- Ordinary async resume failure must reach original caught failure policy, not kill Session binding.
- Real host UI Enter and injected bound failure tests are required; direct VM tests alone insufficient.
- Do not modify root `SessionTreeCliScreen.kt` or Settings work-line sources.
- No Gradle/commit/push/branch/IDE/resource actions; exact API and test handoff in this file.

## Integration handoff

- **IMPLEMENTATION READY — PENDINGCENTRAL**, Session 504. No Gradle or runtime tests
  executed in this lane; completion does not depend on waiting for the central compile.
- Exact RPC factory signature:
  `createRpcComposerViewModel(binding: RpcSessionBinding, ownerScope: CoroutineScope, resumePort: ComposerResumePort, cancellationPort: ComposerCancellationPort, failureReporter: ComposerFailureReporter): ComposerViewModel`.
- Exact internal constructor:
  `RpcAgentPresentation(binding: RpcSessionBinding, scope: CoroutineScope, services: RpcServices, models: StateFlow<List<ModelInfo>>, onCreated: (List<CreatedSuggestedSession>) -> Unit)`.
  It now constructs its own
  `composer` after establishing its local owner and failure sink. `RpcSessionView` borrows
  that exact child; no staged adapter, parallel submit algorithm or global callback is used.
- Resume/cancel callbacks call presentation `resume()`/`stop()` → existing `operate`.
  The typed Composer failure reporter records into existing presentation `failure`.
  Required port callbacks are `resume(ownerId: ComposerOwnerId): Unit`,
  `cancel(ownerId: ComposerOwnerId): Unit`,
  `report(ownerId: ComposerOwnerId, failure: ComposerFailure): Unit`.
  RPC callback closures capture this presentation, never a mutable selected owner.
- `ComposerView` replaces `showAuxiliary` with independent
  `showPendingSteer: Boolean = true`, `showFailure: Boolean = true`.
  `composerStatusLine(state: ComposerState, showFailure: Boolean = true): String?` is
  public renderer-local copy for exact host row budgeting.
- AgentRuntimeScreen hides only duplicate preview/Composer failure copy, retains status,
  and renders the existing Agent notification once (including asynchronous resume errors).
- Main-owned build integration: `:rpc-impl-server`, `:rpc-impl-in-memory` are needed in
  `:app-view-application` `mosaicTest.dependencies`; the shared build file now contains both.
  This lane did not edit that build file.
  Existing `:app-test-support-rpc` exposes the remaining frontend/mock fixture dependencies.
- `hostTestsPassed = PENDINGCENTRAL`; `rpcBindingTestsPassed = PENDINGCENTRAL`;
  `composerRendererTestsPassed = PENDINGCENTRAL`.
  Historical build outputs are not evidence for this repair.

### Production behavior and ownership review

- [Factory](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcComposer.kt#L33):
  append/steer still use the exact binding; mandatory callback arguments replace both direct
  uncaught launches and the empty failure reporter.
- [Presentation assembly](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentPresentation.kt#L46):
  owner and existing failure sink precede child construction. Ordinary asynchronous
  resume/stop errors go through unchanged [operate](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentPresentation.kt#L207);
  cancellation still rethrows. Binding invalidation remains responsible for SessionNotActive/
  SessionNotFound recovery; no command replay or SupervisorJob workaround was introduced.
- [Real host](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L82):
  status rows are counted from the same renderer function that draws them, and added to the
  child height. Agent notification renders once at [line 180](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L180);
  Composer hides only its failure copy and duplicate pending preview, not status.
- The only revision/draft submit algorithm remains `ComposerViewModelImpl.submit`.
  No implementation change to it; failed append preserves text/cursor and the existing
  failure-revision increment. Persisted append still clears before resume admission.
- Port KDoc in the Composer spec now explicitly distinguishes asynchronous admission from
  execution success and describes caught owner failure/cancellation semantics; `@throws`,
  identity, closed/late-result and revision rules remain authoritative.
- This lane did not edit SessionTreeCliScreen, Settings, build files, branches, commits,
  IDE or shared compute resources. Other lanes' current shared-tree diffs are not this repair.

### Test-source handoff

- [ComposerHostRepairTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/application/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/ComposerHostRepairTest.kt#L53)
  adds six tests through **real AgentRuntimeScreen Enter**, real backend JSON RPC,
  RpcSessionViews/Binding/Presentation, and persisted root-Agent registry:
  - gated append: `Submitting…`, one visible/reported error, retained text/cursor,
    authoritative failed revision, zero resume, unchanged backend and Ready/root identity;
  - ordinary resume failure: original `Session operation failed.` notification/detail,
    one rendered outlet, already-cleared draft, one persisted append, Ready/root identity,
    subsequent rename still works, dismissal works, no retry;
  - running steer: exact original `Submit to steer` hint and one pending-preview header
    before/after actual Enter;
  - SessionNotActive resume: new binding/root child, old Composer closed, append not replayed;
  - cancelled resume: accepted append remains, no ordinary failure notification, binding Ready;
  - renderer unmount after resume admission: owner-held command completes, same root survives.
- [RpcSessionViewsTest additions](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionViewsTest.kt#L296)
  add four actual-binding tests: stale/closed zero-command admission, caller cancellation
  restoring editing without Stop, lost accepted-append reply without rollback/retry, and
  root Stop/Composer cancel using the original caught boundary.
  These supplement, not replace, the real-host tests.
- [ComposerViewTest addition](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/agent/ComposerViewTest.kt#L157)
  covers independent output switches; it is an isolated renderer check, not end-to-end evidence.
- Central targets: `:app-view-application:jvmTest` (`composerHostRepairTest`),
  `:app-viewmodel-rpc:jvmTest` (`rpcSessionViewsTest`),
  `:app-component-composer-impl-view:jvmTest` (`composerViewTest`),
  `:app-component-composer-impl-viewmodel:jvmTest` (`composerViewModelTest` unchanged algorithm).
  Native targets remain unverified and are for the coordinator's validation policy.
- Static review: scoped `git diff --check` clean; all factory callers and constructor assembly
  updated; no `showAuxiliary` remains in app Kotlin sources. No temporary files/resources
  created by lane execution. New test fixtures include owner shutdown and directory cleanup
  for when central validation executes them.

## Central validation (supersedes pending lane checkpoint)

- Real `composerHostRepairTest`: **6/6** passed, including append/resume failure,
  running steer, inactive-binding recovery, cancellation and renderer unmount.
- RPC ViewModel **68**, Composer VM **10**, Composer View **6** tests passed;
  full Application renderer regression **102** tests passed.
- Coordinator fixed the test harness, not the command policy: Mosaic events need a frame
  before awaiting their RPC outcome, and snapshot predicates inspect actual current drawing
  before requesting another frame. Every new renderer fixture cancels in `finally`.
- The real failure outlet is exercised through keyboard Enter and the actual binding.
  No empty reporter, parallel submit algorithm or third failure authority remains.
- No commit/push; other Native/CLI/live service checks remain as in the parent task.
