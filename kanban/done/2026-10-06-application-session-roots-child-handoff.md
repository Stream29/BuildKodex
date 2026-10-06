# Task Tree

- `Load approved root cutover and original contracts/suites`()
- `Publish exact API and integration dependencies`()
- `Copy assigned roots and adapt exact-target/lifetime behavior`()
- `Add gated regression coverage and statically review assigned files`()
- `Hand off source completion with central build pending`()

# Details

## Historical handoff status

- 本文记录独立子线交接时的状态，不作为当前 registry 算法或中央验收状态。
  当时未运行的竞态实验后由协调者实际复现并最小修正；
  最终构建、测试、真实 CLI 及用户范围验收见
  [主任务](2026-10-06-hard-migrate-frontend-ownership-roots.md#scoped-acceptance) 和
  [root 线中央证据](2026-10-06-cutover-application-and-session-roots.md#coordinator-final-handoff-and-verification)。
  随已验收整批归档，以下 pending/原算法描述仅为历史交接记录。

## Ownership

- Child of [Application/Session roots](2026-10-06-cutover-application-and-session-roots.md),
  on shared `Kodex/refactor/spec`, baseline `034b778c`.
- Own only `Kodex/app/spec/{application,session}`,
  `Kodex/app/impl/{application,session}`, and this handoff.
- Main owns `Application.kt` scoped assembly, raw RPC, root View, other consumers,
  all shared Gradle registration, old-file deletion, and centralized validation.
- Preserve originals for other lanes to read. Do not copy `Application.kt` or
  `SessionViewModelKoin.kt`; do not launch Gradle/IDE, commit, or allocate devices.

## Early integration contract

- Retain original FQCNs and typed factories/registry. The only root API replacements:
  - `suspend fun selectTab(target: SessionViewModel): Boolean`
  - `suspend fun materializeNewSession(target: NewSessionViewModel): PersistedSessionViewModel?`
- Both locate `===` inside the original `commandMutex`. Removed/consumed targets
  return false/null; a moved but still-owned instance addresses its current slot.
  Closed Application retains the original `IllegalStateException` admission rule.
  Actual materialization/create/storage/transport failures and cancellation propagate.
- No index overload or materialization result cache. Successful replacement preserves
  list size/selection and closes the captured draft's popup and draft.
- Session directly publishes `override val rootAgent = view.agent`; no mutable Agent,
  wrapper factory, presentation collector, or independent borrowed-Agent close.
  Admission uses raw `view.current()` and captured `agent.settings === binding.settings`.
  `close()` calls `view.close()` before local owner cancellation.
- Keep name/settings last-known read caches across recovery; preserve timestamps and
  exact source/generation fork checks. History fork captures the raw current binding
  once, validates the borrowed Agent/settings identity, then calls that binding.
  Raw RPC owner must withdraw `agent` on
  replacement/close and retain the single actual Agent owner.

## New module registration and dependencies

- Register `:app-spec-application`, `:app-spec-session`,
  `:app-impl-application`, `:app-impl-session` at their matching new directories.
- Spec convention: `kodex.kmp-cli`; implementation: `kodex.kmp-viewmodel`.
- Application spec retargets old Agent/Settings/Session dependencies to
  `:app-component-agent-spec`, `:app-component-settings-spec`,
  `:app-spec-session`, and explicitly depends on `:app-component-session-sidebar-spec`;
  existing New Session, popup-child, and value dependencies remain.
- Session spec → `:app-component-agent-spec`, OpenAI models, coroutines.
- Session impl → `:app-impl-rpc`, `:app-spec-session`, actual catalog spec,
  coroutine dependencies; no Agent marker module.
- Application impl retargets original real assembly/child dependencies to
  `:app-impl-rpc`, `:app-spec-application`, `:app-spec-session`,
  `:app-component-settings-spec`, `:app-impl-session`,
  `:app-component-settings-impl-viewmodel`; remove empty `:app-viewmodel-agent`.
  Retain actual assembly dependencies because main moves `Application.kt` here.
- Test dependencies retain original RPC/Session/storage fixtures; remove empty
  Agent marker dependency. Main retargets fixture production imports/dependencies.

## Required main integration

- Move original `Application.kt` into Application impl, retaining `KodexApplication`,
  `withKodexApplication`, and startup fault-injection names. Copied original startup
  tests must keep using this seam, not a substitute Application fixture.
- Convert all other UI/RPC/CLI/Docs test callers to exact target API and handle nullable
  stale materialization results. Original package imports remain valid after relocation.
- Retarget `RpcSessionDraft` final resume to the exact `view.agent` child.
- Raw `RpcSessionViews.release(view)` already exists in baseline. Registry and RPC views
  retain distinct handle-vs-heartbeat roles. Execute the gated release/open experiment
  below before making or claiming its conditional minimal fix.
- Keep centralized JVM/root/CLI/Integration tests and original recordings.

## Release/open experiment: central gate

- `sessionBindingOwnershipTest` now includes
  `gated registry release cannot close a fresh view registered by an opener waiting on its mutex`.
- It uses the real registry mutex and raw view map, not a parallel registry model:
  a Closed observer begins raw exact-view release and queues `registry.open`;
  unlocking the registry mutex admits that opener before the original index release.
  The initial actual unarchive has completed; only the second idempotent unarchive
  Unit is returned inline by the transport decorator to remove unrelated RPC scheduling.
- Source interleaving predicts the unchanged `views.release(sessionIndex)` can
  close that replacement. This is **not runtime confirmation**: no test was run here.
  Central must execute this test against the copied original release first.
- If centrally confirmed, approved minimal correction is only in
  `SessionViewModels.kt`: after removing/closing the old handle under the original
  mutex, release its exact `handle.view` using existing `views.release(view)`, then
  join that handle; no new map/lease/framework. An absent handle must not trigger a
  late index-based release against a newly admitted opener.
- Keep the success assertion (replacement Ready/non-null Agent) as the post-fix
  regression. Main may apply that exact minimal correction after this child handoff
  if central reproduction happens after the child finishes; document actual outcome.
- Until that confirmation/correction, this test is an expected integration gate,
  not a reported passing test or a completed race fix.

## Validation status

- Source copies are present: seven original contract files, three Application
  implementation files, SessionViewModels, all ten original Application suites
  plus ApplicationFixture, and all three original Session suites. Application.kt
  was independently moved by main and is not a child edit.
- All copied Application test consumers now use captured targets and assert
  non-null successful materialization. Startup tests retain the original host seam.
- Added `exactTargetCommandTest` drives the real Application commandMutex with
  deferred child/registry gates and real RPC drafts: duplicate, moved, removed,
  equality-equal foreign target, queued/in-flight cancellation, actual allocation
  followed by child-open failure, and serialized root shutdown.
- Added Session gates cover borrowed-flow identity, last-known caches through
  recovery, foreign old-Agent fork, cancelled initial opener, parent owner
  cancellation, NonCancellable shutdown cleanup, and release/open interleaving.
- Session test build adds real server/in-memory and coroutine-filesystem dependencies
  for the isolated decorated-client fixture; no shared build files were edited.
- Read-only baseline comparison verified every original suite/test declaration was
  retained. ApplicationFixture, Sidebar/Suggestion adapter implementation and their
  unchanged suites, and all three Session suites match baseline exactly.
- Static review verified balanced Kotlin delimiters, no retired coordinates in the
  four assigned builds, no trailing whitespace, and resolved handoff links. This
  is not Kotlin compilation. Repository `git diff --check` also returned clean.
- Source review caught/fixed one copied trailing comma outside a call. Remaining
  implementation changes are the two exact-target APIs, borrowed Agent ownership,
  preserved cache comments, and single captured-binding history fork admission.
- No build/test execution by this child.
- Central compilation and test execution belong to main; completion of this child
  will not imply a successful build, race experiment, UI comparison, or acceptance.
- Child source/handoff work is complete. Runtime release/open confirmation and its
  conditional exact-view correction remain a main integration gate; registry release
  is deliberately still the original code. Old-file cleanup remains main-owned.

## Source entry points

- [Exact-target implementation](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L105)
- [Materialization replacement](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L166)
- [Borrowed Agent and cache](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt#L60)
- [Session lifetime/admission](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt#L90)
- [Eight exact-target regression declarations](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/ExactTargetCommandTest.kt#L19)
- [Five Session regression declarations](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/session/src/commonTest/kotlin/io/github/stream29/kodex/cli/session/SessionBindingOwnershipTest.kt#L34)
