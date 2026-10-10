# Task Tree

- `Review the real fixture send-before-cancel boundary`() // complete
- `Record whether the isolated readiness change preserves the token assertions`() // complete

# Details

- Parent: [full import repair](../executable/2026-10-08-repair-full-model-idea-import-oom.md).
- Fixed [test copy](file:///tmp/kodex-native-fixture-readiness-review-20261009/KodexCurlTerminalCleanupTest.kt):
  bounded existing `sent.await()` after token assertions, before cancellation.
- Original103-test gate: one Native `sendAll` EPIPE; accepted single-case passes.
  Neither establishes unique cause or baseline reliability.
- Review only; production sources unchanged. No build/IDE/process/network writes.
  Coordinator owns runtime counterproof and preserves the original failed gate.

## REVIEWREADY — independent static verdict

- **R:** No static blocker to the isolated one-test-file runtime countertest:
  genuine fixture send-readiness repair, not an OOM repair or adoption.
  Existing real socket writes, native handles, token assertions and cleanup remain.
- Scope: supplied actual-owner candidate140/63, model216/1344/5026 passed;
  supplied functional103-test run has one Native server-send EPIPE failure.
  These runtime receipts were not independently executed in this review.
- Inputs: loaded Gradle skill, Spec/Impl boundaries, Curl engine contract and
  coroutine-resource lifecycle checklist. No builds/tests, IDE, process/device
  operations, network writes, Git or credentials; only this task file edited.

### Fixed bytes and exact delta

- [Original local test, lines292–330](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxX64Test/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlTerminalCleanupTest.kt#L292)
  SHA-256: `76d147483555f3981bc85f4c6a143bb68d65c1bcf12148c671214a2bc2417447`.
- [Fixed isolated copy, lines318–334](file:///tmp/kodex-native-fixture-readiness-review-20261009/KodexCurlTerminalCleanupTest.kt#L318)
  SHA-256: `152612ac090404d3de2a8e97963ec5b7396d7436b9342ec39ef12536f5f27cc5`.
- Direct full-file diff: one hunk, exactly four added lines328–331
  (three comments, `withTimeout(5.seconds) { fixture.sent.await() }`);
  zero deletions or other changes. Accepted5b attribution is supplied context,
  not an independently checked Git identity.

### B1 — confirmed source boundary

- [Real fixture, lines52–75](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxX64Test/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/NativeLoopbackHttpFixture.kt#L52):
  `sent` succeeds only after both real header/body `sendAll` calls; peer then
  awaits `allowPeerClose`. It is not client consumption or parent-body rendering.
- Concrete allowed old schedule: retire old handle; schedule one current request;
  drain stale then valid unpause, finish zero/one assertions; server has received
  request headers but not finished response writes; current cancellation and
  native teardown close its peer before server writes finish, allowing EPIPE at
  [sendAll lines101–108](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxX64Test/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/NativeLoopbackHttpFixture.kt#L101).
  Thus old assertions do not order send before teardown; the added await does.
- [Actual handler lines292–355](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt#L292):
  admission captures the real handle/private queue; identity-checked unpause
  drain precedes `curl_multi_perform`, so it can precede connection readiness.
  Candidate retains retired-token inequality, forced current-address/stale-token
  rejection and valid actual-handle native unpause, all BEFORE readiness wait.
- Production caller is [Processor lines209–222](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlProcessor.kt#L209):
  real schedule/cancellation task routing. The test instead uses existing
  `withActualHandler` and direct cancellation callback; it tests the same native
  handler/queue, not the entire processor/business layer or parent-body renderer.

### B2 — specific remaining failure boundary

- If unpause/real CURLE failure terminates this request BEFORE body can send,
  readiness cannot succeed: fixture failure completes `sent` exceptionally
  (lines59–61), or absent progress fails its five-second await/native timeouts.
  Server-child failure may cancel the waiting scope; neither path becomes green.
- Existing catch cannot erase an already exceptional `sent`; candidate adds no
  catch-ignore/retry, delay increase, engine disable, fixture API or new gate.
  [Candidate cleanup lines332–334,422–431](file:///tmp/kodex-native-fixture-readiness-review-20261009/KodexCurlTerminalCleanupTest.kt#L332)
  retains current-call cancellation and NonCancellable handler cleanup;
  fixture lines71–75 retain release, server join and listener close.

### U — unproved and acceptance boundaries

- Static schedule is not unique historical-root proof; supplied accepted5b
  single-case pass is not baseline-flakiness proof or plugin-order invariance.
  `sent` also cannot certify every later CURL outcome; existing pause return
  handling and discarded current-response promise are unchanged test limits.
- Coordinator owns remote full Native/JVM/Node validation and preserved failed
  control. Even a pass requires genuine GUI import/navigation at unchanged
  targets/API/tests and4GiB before adoption; this finite review waits for no run.
