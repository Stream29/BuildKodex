# REVIEW READY — independent narrow review, Session 559

- **R: static delta is ready for coordinator-controlled CI; no confirmed B1
  blocker found in the sealed five files.**
- **B2/U: Windows regression execution and all original three-host consumer
  gates remain mandatory before the writer. No publication or default-consumer
  acceptance is certified here.**
- Final handoff for asynchronous coordinator reading; no runtime work awaited.

# Task Tree

- `Verify the frozen five-file source and original fork semantics`()
- `Independently review the return-value correction and real regression gate`()
- `Report scoped deployment readiness and remaining Windows runtime gates`()

# Details

- Parent: [Mosaic resize repair](2026-10-07-fix-mosaic-windows-resize-package-gate.md).
- Read-only review; write only this report. Do not run tests, builds, devices,
  publication, Git operations or edit implementation.
- Fixed source: `/tmp/kodex-mosaic-resize-review-20261008`, five files,
  `SHA256SUMS` digest
  `ba2c0e566bee750c26188cd89e7ab8a84dfe4269e0595f1cb2425b304a246190`.
- Fork commit `78f94c4c2e136d908b838f09cd3231dc37ee67fd`,
  original `7b1a412918f7bc24db847343480f010242939788`.
- Scope: one Win32 return predicate, two real Windows Native tests, targeted
  producer task and mandatory XML receipt. All original consumer gates remain.
- Coordinator ran 78 offline cases on Xiaoxin. Real Windows execution remains
  mandatory before sealed publication; offline tests do not prove Native behavior.
- MCP/Lucene existing versions are remotely verified and must not be republished
  with a changed recipe. No default consumer switch has occurred.

## Evidence boundary

- Loaded AGENTS, associated change/kanban/planning/checklist/document/workspace/
  toolchain/IDE/Gradle skills, Draft, this outer card and its parent repair card.
  Runtime/device restrictions supersede any general build/IDE workflow.
- Current delta evidence is **only** the frozen five-file snapshot.
  [Manifest](file:///tmp/kodex-mosaic-resize-review-20261008/SHA256SUMS#L1)
  SHA-256 equals
  `ba2c0e566bee750c26188cd89e7ab8a84dfe4269e0595f1cb2425b304a246190`.
  Initial and final `sha256sum -c SHA256SUMS`: **5/5 OK**.

| Frozen file | Verified SHA-256 |
| --- | --- |
| README.md | `8ab7bed2d2cb140a8bb34e03d57c3202bf076412cfdd82f85d7aacaafc21aeb8` |
| WindowsConsoleResizeTest.kt | `aaa090c37c06faa0886b999238f8d989041b97934ef621b996c2d4daa7e38d19` |
| mosaic-test-windows.c | `748d6068bb35c38e1d8c16644395f09974242953286813eb9f3158743905e45e` |
| pipeline.py | `88c1ce98160db9684811e4797bbb07ce24c360711683ffb91920de8b17af898b` |
| test_admission.py | `f58fffa3e97817d7814b364c9b6cc5405e99fbd43b9a18becb56eed3f77db886` |

- Shared dirty `Kodex/` paths were consulted as API/location context, not as a
  clean checkout or current-delta proof. Relevant original behavior was also
  read from commit-addressed primary sources: Mosaic `7b1a412918f7...` and Kodex
  `8772acf`. Local context links below are location mappings, not immutable
  worktree attestations.
- Signed push `78f94c4c2e136d908b838f09cd3231dc37ee67fd`, its tree and unchanged
  upstream/trunk are coordinator-provided provenance from
  [parent lines 27–34](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-fix-mosaic-windows-resize-package-gate.md#L27).
  No independent signature, ancestry or live remote-head verification was run.
- Executed no Python tests/imports, compiler, Gradle, builds, runtime probes,
  Git commands, network writes, SSH/device/VM/IDE or process-control operations.
  Only this report was written, via `apply_patch`; no temporary files or
  persistent resources were created.

## R — return predicate and real Native tests

- The original helper treated `SetConsoleWindowInfo == 0` as success.
  The fixed [C lines 35–44](file:///tmp/kodex-mosaic-resize-review-20261008/mosaic-test-windows.c#L35)
  returns Mosaic success `0` only for Win32 **nonzero** success; otherwise it
  immediately returns `GetLastError()`. This is the intended minimal correction,
  not clearing errors or converting real failure into success.
  [Original pinned C](https://github.com/Stream29/mosaic/blob/7b1a412918f7bc24db847343480f010242939788/mosaic-tty/src/commonMain/c/mosaic-test-windows.c#L35),
  [Microsoft return contract](https://learn.microsoft.com/en-us/windows/console/setconsolewindowinfo#return-value).
- The helper is shared by initialization at
  [C lines 72–77](file:///tmp/kodex-mosaic-resize-review-20261008/mosaic-test-windows.c#L72)
  and explicit resize at
  [C lines 351–361](file:///tmp/kodex-mosaic-resize-review-20261008/mosaic-test-windows.c#L351).
  A failed resize returns before the explicit input-buffer resize record.
  There is no new Kotlin-side dimension guard or replacement C API.
- The real Native call chain is
  `com.jakewharton.mosaic.tty.TestTerminal.resize`
  → `mosaic_test_resize` → `mosaic_test_resize_internal`
  → `SetConsoleWindowInfo`; nonzero Mosaic errors return to
  `com.jakewharton.mosaic.tty.throwIoe(UInt)`.
  [Native wrapper mapping, lines 148–153](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/src/nativeMain/kotlin/com/jakewharton/mosaic/tty/TestTerminal.kt#L148),
  [original pinned wrapper](https://github.com/Stream29/mosaic/blob/7b1a412918f7bc24db847343480f010242939788/mosaic-tty/src/nativeMain/kotlin/com/jakewharton/mosaic/tty/TestTerminal.kt).
- `IOException` in this test resolves to
  **`com.jakewharton.mosaic.tty.IOException`**, the Native actual exception,
  not `java.io.IOException`. `throwIoe` uses `error.toString()`, so expected
  message `"87"` matches the existing producer of the exception.
  [Exception mapping, line 3](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/src/nativeMain/kotlin/com/jakewharton/mosaic/tty/IOException.kt#L3),
  [throwIoe mapping, lines 144–145](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/src/nativeMain/kotlin/com/jakewharton/mosaic/tty/Tty.kt#L144),
  [pinned exception](https://github.com/Stream29/mosaic/blob/7b1a412918f7bc24db847343480f010242939788/mosaic-tty/src/nativeMain/kotlin/com/jakewharton/mosaic/tty/IOException.kt),
  [pinned throwIoe](https://github.com/Stream29/mosaic/blob/7b1a412918f7bc24db847343480f010242939788/mosaic-tty/src/nativeMain/kotlin/com/jakewharton/mosaic/tty/Tty.kt).
- [Test lines 11–31](file:///tmp/kodex-mosaic-resize-review-20261008/WindowsConsoleResizeTest.kt#L11)
  bind a real terminal inside each test and use `AutoCloseable.use`; successful
  binding is closed even if assertions throw. No persistent class fixture,
  mock terminal, replacement library or new native wrapper is introduced.
- `successfulResizeDoesNotReturnStaleLastError` seeds error3 **after binding**,
  resizes to `80×24`, and checks both actual dimensions. Native `Tty.currentSize`
  calls `mosaic_tty_current_terminal_size`, whose Windows implementation reads
  `GetConsoleScreenBufferInfo`, rather than cached test input.
  [Tty mapping, lines 106–112](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/src/nativeMain/kotlin/com/jakewharton/mosaic/tty/Tty.kt#L106),
  [C size mapping, lines 263–271](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/src/commonMain/c/mosaic-tty-windows.c#L263).
- Existing common tests already bind/close this API and assert default
  `80×24` plus `resize(90,30,0,0)` changing `currentSize()`.
  [Common-test mapping, lines 189–195](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/src/commonTest/kotlin/com/jakewharton/mosaic/tty/TtyTest.kt#L189),
  [pinned common tests](https://github.com/Stream29/mosaic/blob/7b1a412918f7bc24db847343480f010242939788/mosaic-tty/src/commonTest/kotlin/com/jakewharton/mosaic/tty/TtyTest.kt).
- Native interop imports are consistent with the original `mingwX64` target,
  default hierarchy and common `kotlin.test` dependency.
  `ExperimentalForeignApi` is opted in. Win32 `SetLastError` accepts a 32-bit
  unsigned `DWORD`; `.toUInt()` provides that type without assuming the signed
  type of the imported error macro. Numeric `.toString()` yields `"87"` for
  either signed or unsigned representations of this positive constant.
  [Pinned build](https://github.com/Stream29/mosaic/blob/7b1a412918f7bc24db847343480f010242939788/mosaic-tty/build.gradle),
  [Kotlin 2.3.21 Windows platform definition](https://github.com/JetBrains/kotlin/blob/v2.3.21/kotlin-native/platformLibs/src/platform/mingw/windows.def),
  [MinGW SetLastError declaration](https://github.com/mingw-w64/mingw-w64/blob/master/mingw-w64-headers/include/errhandlingapi.h#L45).
- Read-only string inspection of the installed 2.3.21 MinGW platform metadata
  also finds both error constants and `SetLastError` in `platform.windows`.
  This confirms symbol presence, **not** compilation of the new test.
  [Platform definition, lines 1–8](file:///home/stream/.konan/kotlin-native-prebuilt-linux-x86_64-2.3.21/konan/platformDef/mingw_x64/windows.def#L1).
- `failedResizeReportsActualWin32Error` passes **columns=0** (not pixel width):
  C produces `Left=0, Right=-1`. Win32 rejects this inverted rectangle before
  Mosaic writes its input-buffer record. Microsoft documents this invalid
  rectangle as failure; its console implementation returns `E_INVALIDARG`
  before viewport/buffer resizing. Error87 is therefore a well-founded expected
  runtime result, not a Mosaic precondition invented by the test.
  The API documentation alone does not guarantee that exact code for every
  possible host/handle failure.
  [Microsoft rectangle rules](https://learn.microsoft.com/en-us/windows/console/setconsolewindowinfo#remarks),
  [Microsoft console validation](https://github.com/microsoft/terminal/blob/main/src/host/getset.cpp#L833),
  [Win32 error3/error87 meanings](https://learn.microsoft.com/en-us/windows/win32/debug/system-error-codes--0-499-).

## R — producer, mandatory XML and sealed evidence

- Producer chain:
  `pipeline.main()` with `command=build` → `pipeline.build(args)` →
  original publication `task_set(fork,host)` → successful `run(command)` →
  Windows-only real `:mosaic-tty:mingwX64Test --tests
  com.jakewharton.mosaic.tty.WindowsConsoleResizeTest` →
  `pipeline.windows_resize_test_receipt(...)` → seal.
  [Dispatch lines 472–476](file:///tmp/kodex-mosaic-resize-review-20261008/pipeline.py#L472),
  [build lines 249–281](file:///tmp/kodex-mosaic-resize-review-20261008/pipeline.py#L249).
  Publication here is to the isolated **file staging repository**, not Maven PUT.
- The second command removes only the publication-task suffix and reuses the
  same explicit JVM, private homes, version/init script and no-scan/cache settings.
  It runs **after publication**, not as a compile-only surrogate.
- `org.jetbrains.kotlin.gradle.targets.native.tasks.KotlinNativeHostTest`
  inherits `KotlinNativeTest`, which inherits
  `org.jetbrains.kotlin.gradle.tasks.KotlinTest`, then Gradle `AbstractTestTask`.
  Thus `--tests` is supported, and the filter reaches the Native runner.
  KotlinTest explicitly disables failure on no matching tests: Gradle exit
  success alone would be insufficient.
  [KGP 2.3.21 Native task](https://github.com/JetBrains/kotlin/blob/v2.3.21/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/targets/native/tasks/KotlinNativeTest.kt#L29),
  [KotlinTest filter](https://github.com/JetBrains/kotlin/blob/v2.3.21/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/tasks/KotlinTest.kt#L49),
  [Gradle 9.6.1 --tests option](https://github.com/gradle/gradle/blob/v9.6.1/platforms/software/testing-base/src/main/java/org/gradle/api/tasks/testing/AbstractTestTask.java#L682).
- [Receipt lines 122–136](file:///tmp/kodex-mosaic-resize-review-20261008/pipeline.py#L122)
  require exactly two testcase entries, both expected methods, and no
  `failure`, `error` or `skipped`. No reports, one method, a repeated single
  method or extra/duplicate testcase entries cannot satisfy ordinary real
  output. Malformed XML also aborts rather than becoming success.
- [Offline fixture lines 58–77](file:///tmp/kodex-mosaic-resize-review-20261008/test_admission.py#L58)
  exercise missing output, valid two-pass output, failure, skipped, missing
  method and repeated method. The parser also rejects `<error/>` by inspection;
  this particular fixture does not explicitly test that tag.
- Matching uses class/method **substrings**, accommodating Native name suffixes;
  it is not an exact-FQCN validator for arbitrary hostile XML. The producer
  invocation itself selects the exact FQCN in a fresh extracted tree. No
  unrelated-class output bypass is evidenced in this reviewed call chain.
- Only after successful test/receipt processing is
  `toolchain["windowsResizeRegression"]` added, then included in `toolchain.json`
  and `stage.json`; merge carries it into manifest host toolchain evidence.
  [Receipt insertion lines 284–286](file:///tmp/kodex-mosaic-resize-review-20261008/pipeline.py#L284),
  [sealing lines 317–344](file:///tmp/kodex-mosaic-resize-review-20261008/pipeline.py#L317),
  [merge lines 369–402](file:///tmp/kodex-mosaic-resize-review-20261008/pipeline.py#L369).
- This is a trusted-producer **summary receipt**, not retained raw XML or a
  standalone signature. `verify_bundle` does not separately require the
  Windows receipt field; the enforcement point is `build` before its upload
  can succeed. Do not describe downstream verification as a second independent
  test execution or receipt-presence validator.
- Original controlled-CI context still has `build → merge → three-host smoke →
  publish`, with publish dependent on all smoke jobs. The immutable baseline
  retains Java21 JNI, Java25 FFM and real Native probes; receipts bind all three
  hosts to the manifest before writer calls. This five-file delta removes none.
  [Pinned workflow](https://github.com/Stream29/Kodex/blob/8772acf/.github/workflows/fork-packages-mosaic.yml),
  [pinned gate contract](https://github.com/Stream29/Kodex/blob/8772acf/scripts/fork-packages/contract.py#L70),
  [pinned writer receipt check](https://github.com/Stream29/Kodex/blob/8772acf/scripts/fork-packages/publisher.py#L110).

## B1 — concrete static blockers

- **None established in this scope.** The corrected return predicate, actual
  Native exception/API use, supported targeted task and required two-test XML
  observation are consistent.
- Do not promote hypothetical headless-console failures, unrun compilation or
  an unobserved non87 result into confirmed source defects. They remain U and
  must fail the real gate if encountered.

## B2 — coordinator-owned gated checks

- Controlled Windows producer must compile/link/run the real test class and
  emit both successful methods; inspect actual XML, task logs and sealed
  `hosts.windows.toolchain.windowsResizeRegression`. Missing/skipped reports
  are failure, never permission to remove the receipt.
- Rerun every original Linux/macOS/Windows consumer gate against the **new**
  bundle: Java21 JNI, Java25 FFM, host Native, plus Linux Arm64 compile-only.
  Require matching identity/manifest receipts before writer admission.
- Derive a new immutable version from the new gitlink:
  **`0.19.0-SNAPSHOT-kodex.78f94c4c2e13`**, not the old commit version.
  [Identity lines 64–74](file:///tmp/kodex-mosaic-resize-review-20261008/pipeline.py#L64),
  [parent version](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-fix-mosaic-windows-resize-package-gate.md#L32).
- Keep MCP **453/453** and Lucene **133/133** remote audits as coordinator
  context. Do not republish those existing versions with this changed recipe.
  [README immutable-version boundary](file:///tmp/kodex-mosaic-resize-review-20261008/README.md#L232).
- README/security-policy acceptance remains prior-review context, not a new
  security audit. Preserve the established fail-sealed writer and separate
  default-consumer acceptance boundary.

## U — evidence not supplied by static review

- Actual Windows console availability, buffer/window limits, stale-last-error
  behavior across the Native call and exact failure87 on the CI host.
  Successful binding itself performs an `80×24` resize; inability to bind or
  resize must fail, not skip, mock or weaken the test.
- Coordinator reports **78 offline cases passed remotely**. This review did
  not rerun them; synthetic XML does not prove Native compilation or Win32
  behavior. Likewise the historical Windows FFM/Native green result is not
  proof of the repaired error contract.
- Parent records the original Java21 JNI `Jni.testResize` IOException3 failure.
  The JNI chain is `TestTerminal.resize → com.jakewharton.mosaic.tty.Jni.testResize
  → Java_com_jakewharton_mosaic_tty_Jni_testResize → mosaic_test_resize`;
  nonzero errors throw. User-supplied history says the original `WindowsNative`
  smoke ignored resize errors; that weaker observation is not reusable proof.
  [JNI mapping lines 1093–1105](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/src/jvmMain/c/com_jakewharton_mosaic_tty_Jni.c#L1093),
  [recorded original failure](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-fix-mosaic-windows-resize-package-gate.md#L15).
- Current pin cleanliness/ancestry/signature, controlled CI completion and new
  remote package bytes were not independently verified. Full Kodex regressions,
  IDEA performance/gain, default-consumer switching and Linux Arm64 runtime
  remain outside this review.

## Ablation and final decision

- Reverting only the predicate restores false success on an invalid rectangle;
  the failure test specifically prevents treating that as an acceptable repair.
- Dropping failure87 coverage would leave an always-success “fix” insufficiently
  checked. Dropping actual-size checks would weaken real console-state evidence.
- Dropping the XML receipt would permit the Native task's no-match success to
  masquerade as two tests passed. Dropping Java21 JNI would omit the historically
  failing consumer path; Java25/Native cannot replace it.
- Extra APIs, mocks, native wrappers and broad source changes are unnecessary
  for this authorized minimal delta. No such implementation is requested here.
- Whole-report change/ablation review completed: facts, expected outcomes and
  unverified runtime results are separated; no source edits or gate relaxation.
- **REVIEW READY / scoped static CI readiness, B1 none; mandatory B2/U gates
  open.** Coordinator may read asynchronously before controlled CI; this report
  neither runs that CI nor grants publication/default-consumer approval.
