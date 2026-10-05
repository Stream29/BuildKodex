# Task Tree

- `Align MCP effect mounting with retained Settings owner`() // source implemented
- `Correct Hook guarantee and source-neutral Login copy`() // source implemented
- `Remove deletion alias and old Catalog forwarding constructor`() // source implemented
- `Add root callback probes and publish static handoff`() // lane source checkpoint below
- `Record user acceptance and commit scoped repair`()

# Details

- Parent: [Reacceptance and authorized repairs](../executable/2026-10-05-reaccept-spec-impl-refactor.md).
- 用户接受当前修复并授权分批提交；内层提交 `a1adf2f2`，本轮未推送。
- Own SettingsPopup and root SessionTreeCliScreen, MCP Settings component, Hook spec/doc-focused tests, OpenAI Login renderer/tests, ApplicationPopupState, RpcSessionCatalog and focused tests.
- Keep one MCP effect consumer alive for exact Settings owner, including Login.returnTo; no global URL service or second consumer.
- Preserve Hook stale/same-name no-op behavior; correct reporting KDoc rather than silently alter persistence.
- Login completion copy is neutral; do not change source selection/protocol.
- Remove no-behavior Delete alias/Catalog wrapper; preserve actual RpcSessionCatalogDependencies.
- Prune departed tab identity keys locally; retain stale callback admission and immutable tab spec.
- Two-field stale draft concern: reproduce before modifying; field updates against current exact token draft may replace composition-copy only if confirmed.
- No application registry rewrite, no Composer/AgentRuntimeScreen edits, no builds/commits/push/branch/IDE operations.
- Handoff lists exact source changes and root-level renderer tests; central validation pending.

## Integration handoff — early, implementation in progress

- Root `SessionTreeCliScreen` will mount one `McpSettingsEffects` keyed by the exact
  `ApplicationPopupState.Settings` handle (current Settings or `Login.returnTo`).
  `SettingsPopup` will no longer mount it. Settings → Login keeps that mount.
  Complete root unmount still cancels its captured in-flight effect, as before;
  real Settings close separately cancels VM-owned login jobs.
- Planned signature: append defaulted `openMcpUrl: suspend (String) -> Boolean`
  to `SessionTreeCliScreen`, defaulting to the existing global `openExternalUrl`.
  This is a renderer test seam, not a URL service or backend dependency.
- **Main owns host build edits:** `:app-view-application` `mosaicMain` needs direct
  `implementation(project(":app-component-mcp-settings-impl-view"))` and
  `implementation(project(":utils-external-url-impl"))`; root tests need
  `:app-viewmodel-application`, `:app-component-mcp-settings-impl-viewmodel`
  in test dependencies. Existing RPC test support may expose the other fixtures.
- Delete field becomes `SessionDeleteViewModel` directly; remove
  `DeleteSessionPopupViewModel` alias. Delete `RpcSessionCatalog` forwarding
  constructor only; all four test constructions become
  `DefaultSessionCatalogViewModel(scope, RpcSessionCatalogDependencies(...))`.
  Real dependencies and production construction stay intact.
- Planned tests: genuine root Settings → Login → return with gated MCP prepare
  and opener; captured old-attempt failure versus newer attempt; true owner
  closure; tab identity churn/stale admission; source-neutral Login snapshot;
  real Hook adapter stale/same-name unchanged values and admitted/released queue.
- Hook/MCP burst callback renderer tests will be added as reproduction probes.
  No field-update API change is authorized by an unrun probe; unresolved risk
  will be recorded if centralized execution has not confirmed it.
- No build or test execution in this lane. Central compilation/JVM execution,
  Native/CLI and platform acceptance are pending; inventory will be finalized
  below when source edits finish.

## Integration handoff — source checkpoint

- Host `mosaicMain` requirements from the early handoff are now present in the
  shared build file, edited by main (not this lane).
- **Additional host test dependency needed:** `:app-view-application`
  `mosaicTest` needs `implementation(project(":app-migration-impl"))` for
  `prepareKodexHome` in the real application/root fixture. Mock client is already
  exposed by `:app-test-support-rpc`; Login/MCP specs are exposed by host contracts.
- **Catalog test dependency needed:** `:app-viewmodel-session` `commonTest`
  needs `implementation(project(":app-component-session-catalog-impl-viewmodel"))`
  now that its tests construct the true implementation instead of the old RPC
  forwarding name. RPC module already has this implementation dependency.
- New root suite is `settingsLoginLifetimeTest` in `SettingsLoginLifetimeTest.kt`.
  It mounts the full production `SessionTreeCliScreen`, invokes actual MCP and
  Authentication mouse callbacks and Login Escape return, and forwards popup
  opening/dismissal to real Application commands. Only MCP dependencies are gated
  test ports, bound to original Settings closure. A test-only display binding
  substitutes that child; it does not claim live backend OAuth coverage.
- Root cases: gated preparation while Login is visible; gated opener retained
  through return (one call); delayed old opener failure versus a newer attempt;
  whole-renderer removal cancels its exact effect without closing Settings;
  a fresh attempt survives Login transition and cancels on real target closure.
  Additional churn test calls the production pruning helper with real handles,
  retains an old exact close callback and verifies real root rejection after
  replacement. It is not a queued-index selection-race test.
- New RPC case uses the actual Hook adapter, real frontend store and original
  `SettingsUpdateQueue`: blocked queue admits stale edit, same-name Add, valid
  Add and stale Delete; handle release and queue close retain admitted intent;
  stale/conflicting values stay unchanged and the successful queue clears failure.
- **Unrun reproduction probes:** Hook Name → Command and MCP Name → URL dispatch
  all genuine renderer key events before any test frame. They record callback
  frame identity and assert both fields survive. These may expose HM2 as failing
  tests in central validation; no field API/implementation changed speculatively.
  Do not count them as passed or claim a runtime reproduction yet.

## Coordinator integration constraint

- Aligning the effect's root mount with retained Settings already prevents
  Settings → Login from cancelling its composition. Preserve the existing
  `CancellationException` → exact `effect.cancel()` behavior of
  `McpSettingsEffects`; do not weaken that callback contract merely to repair
  an incorrect mount. True owner removal and standalone effect-handler teardown
  must still cancel their captured operation. Central review will restore this
  branch if the lane changed it.

## Coordinator reconciliation

- The lane assignment mixed true Settings operation ownership with renderer-effect
  cancellation. The intended repair is retention through Settings → Login, not
  changing baseline behavior when the whole root renderer disappears.
- Coordinator retained the original captured `effect.cancel()` on in-flight opener
  cancellation and updated the root test to distinguish those two lifetimes.
  No `state.closed` check, global opener service or extra operation owner was added.
- The burst probes initially timed out without processing input: Mosaic only drains
  queued events when a frame is sent. They now enqueue the whole burst and send one
  frame, which drains both real callbacks before recomposition. A recorded composition
  witness still verifies the claimed boundary. Behavioral reproduction is pending.

## Final source/test inventory and validation limits

| Artifact | Lane change and central check |
| --- | --- |
| [Root signature/mount](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L108) | Defaulted renderer opener appended; one mount at retained Settings handle; prune helper at L1035. No tab callback/label or Shell business changes. |
| [Settings host](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt#L23) | Removed the old short-lived MCP consumer and unused opener imports. |
| [MCP effect boundary](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/McpSettingsComponent.kt#L105) | Root-retention KDoc and original exact cancellation branch preserved. No MCP backend/core change. |
| [Hook spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/hooksettings/HookSettingsViewModel.kt#L8) | Admission, stale/same-name Add no-op and failure semantics match the actual adapter; persistence unchanged. |
| [Delete popup field](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/contract/application/src/commonMain/kotlin/io/github/stream29/kodex/app/application/contract/ApplicationPopupState.kt#L39) | Direct `SessionDeleteViewModel`; removed compatibility alias, retained real Rename/Cwd target adapters. |
| [Real Catalog RPC dependencies](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionCatalog.kt#L7) | Forwarding class removed. Four constructions changed across SessionRepositoryViewModelTest and RpcSessionViewsTest; only constructor/import hunks in the latter belong to this lane. |
| [Root test suite](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/application/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SettingsLoginLifetimeTest.kt#L33) | Five new cases (two generated gated phases, old failure/new attempt, removal/real Login-owner close, churn/stale close). Actual root renderer and Application commands; MCP port substitution only. |
| [Hook adapter/queue test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/RpcSettingsTest.kt#L40) | One new delayed real-store/adapter/queue case; released admissions drain and stale/conflicting writes leave values unchanged. |
| [Hook burst probe](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/HookSettingsComponentTest.kt#L23), [MCP burst probe](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/McpSettingsComponentTest.kt#L24) | Two new genuine-renderer pre-frame probes. Unconfirmed HM2; no speculative field/draft API changes or copied authoritative state. |
| [Login completion](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/OpenAiLoginPopup.kt#L126), [render assertion](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/OpenAiLoginRenderContractTest.kt#L52) | Neutral `Sign-in complete`; strengthened existing Completed snapshot assertions, no source query/switch. |

- Static reference search: one production MCP consumer; no remaining Kotlin
  `DeleteSessionPopupViewModel` or `RpcSessionCatalog(...)` construction.
- Scoped tracked whitespace check and new root-file whitespace check pass.
  Source review checked exact keys, callback captures, queue release and test
  cleanup. New Mosaic probes cancel their own test renderer in `finally`.
- Eight new cases are written, **none executed here**; existing Login and Catalog
  tests are amended, not newly passed. Central targets:
  `:app-view-application:jvmTest`,
  `:app-component-{hook-settings,mcp-settings,openai-login}-impl-view:jvmTest`,
  `:app-viewmodel-rpc:jvmTest`, `:app-viewmodel-session:jvmTest`.
  Resolve the two outstanding host/test dependency requirements above first.
- Native/CLI/live OAuth, actual heap reachability and queued-index root races
  remain unvalidated/outside this lane. HM2 requires executing the corrected probes
  before any field-specific repair. Central results supersede the unrun lane checkpoint.
- No Gradle, IDE/resources, commits/push/branch operations, audit-report or master
  document edits. No temporary files or acquired shared resources remain.

## Central validation (supersedes unrun lane checkpoint)

- Root Settings/Login **5/5**, MCP renderer **25**, Hook renderer **10**, Login renderer **9**,
  RPC ViewModel **68**, Session ViewModel **8** tests passed.
- Corrected burst probes genuinely reproduced HM2 in both components: second-field input
  reset the name to empty, with both callbacks witnessed in the same composition.
- Hard-cutover `updateDraft(token, update)` now applies a pure synchronous update once to
  the latest VM draft. Stale/closed tokens do not invoke it; throwing updates leave state intact.
  All production and test callers changed; no old whole-draft overload or View authority remains.
- Hook VM **13**, MCP VM **28**, Settings VM **12** tests passed, including invalid/stale/closed
  updates, same-frame renderer edits and original admission/queue behavior.
- Root teardown fixture now waits for the actual removal frame before checking cancellation;
  no Settings → Login frame disposes its retained owner. The original captured-effect cancellation
  branch stays intact, and real tab closure separately cancels the retained Settings child.
- Removed alias/forwarding class and departed identity keys; no protocol or global registry change.
- Host test dependencies were added centrally. Both required build edges are present.
- Other Native/CLI/live OAuth/heap/root queued-index experiments remain open; no commit/push.
