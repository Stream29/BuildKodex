REVIEW READY

# Task Tree

- `Verify the pinned source baseline`()
- `Trace all remaining utility topics and actual consumers`()
- `Review legitimate spec-only APIs, ownership and tests`()
- `Publish the independently evidenced report`()

# Details

- Status: **REVIEW READY** — scoped static review finished; not a whole-repository pass.
- [共同规范/分级/基线](2026-10-07-final-reaccept-spec-impl-repository.md)。
- 范围 23 项目：utils 排除平台线五域后全部项目；包括 coroutines、external-url、
  filesystem-lease、host-test-support、images、kotlinx-io-coroutines/serialization、
  ktor-client-ext、logging、patch、read-write-mutex、rpc-exception、search-index、terminal-text。
- lease 的真实取消/清理缺陷和契约在本线核对；UI/Home 线审其 Home 影响，根因不重复。
  检查纯函数、serializer、actual 与 fixture 的真实归属，不发明接口配对。
- 仅写本报告，禁止源码修复、其他文件、Gradle/IDE/资源及 Git 操作。

## Baseline, method and checkpoints

- Initial and post-trace checks: HEAD `6b7129fa0e719dde8f88a0034b9d572679984a03`,
  branch `refactor/spec`, inner `git status --short` empty. Outer changes predate
  this review and are not evidence of inner source dirtiness.
- Final handoff check repeats the pinned HEAD and empty inner porcelain status.
  All report file URI targets and referenced line starts were checked for
  existence/range; coverage table independently counted 23 rows.
- Loaded AGENTS, change/checklist/kanban/planning/document/ask-user skills, Draft,
  parent executable, module-boundary and coroutine-resource checklists, plus the
  Home lease planning record. No authorization inferred from those plans.
- Checkpoint 1 inventoried 23 build files; checkpoint 2 recorded lease publication,
  join-only cleanup, curl detached cleanup and the actual mutex repair. This
  checkpoint completes the independent source/history/caller/test trace.
- Read-only filesystem/Git inspection only. No Gradle, IDE, test execution,
  network, device, service, checkout, branch, commit or push. Only this report
  was edited with `apply_patch`; no temporary files or acquired resources remain.
- History basis: inspected `6606efe9` and its parent, `e89c5094`, relevant
  `git log --follow`, and current source. The many 100% source moves are evidence
  of continuity, **not a substitute for checking the current implementation**.

## Per-project coverage: 23 / 23

- Each row names an actual build file, not a virtual root. Detail IDs below provide
  declaration → implementation/factory → caller → test → history evidence.
- Project grades: **R 10 / D 9 / B1 3 / B2 1 / U 0**. These are project
  classifications, not a root-cause count: lease impl contains L1/L2; curl impl
  contains C1 and the additional B2 C2; Home shares L1/L2.
- `R` is static key-path reacceptance, not a runtime/platform certification.
  `D` is an explicit legitimate boundary or documented debt. `B1` and `B2`
  follow the parent definitions. No unlisted project is implicitly passed.

| # | Project / exact build file | Grade | Detail and dependency check |
| --- | --- | --- | --- |
| 1 | [utils-coroutines-spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/coroutines/spec/build.gradle.kts#L1) | D | T1; legitimate pure executable contract; owner/error KDoc debt |
| 2 | [utils-external-url-spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/spec/build.gradle.kts#L1) | R | T2; real launcher result model, not an unused interface |
| 3 | [utils-external-url-impl](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/impl/build.gradle.kts#L8) | R | T2; spec API + real process impl; JVM/native actuals |
| 4 | [utils-filesystem-lease-spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/spec/build.gradle.kts#L1) | D | T3, D1; real resource identity, seriously incomplete lifetime/error contract |
| 5 | [utils-filesystem-lease-impl](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/build.gradle.kts#L9) | B1 | T3, L1/L2; actual spec return types, unresolved preexisting defects |
| 6 | [utils-host-test-support-spec](file:///home/stream/ACodeSpace/local/kodex-e2e-20261007-7b23c481/evidence/source-baseline-6b7129fa/utils/host-test-support/spec/build.gradle.kts#L8) | D | T4; legitimate test dependency contract, no invented FQCN |
| 7 | [utils-host-test-support-impl](file:///home/stream/ACodeSpace/local/kodex-e2e-20261007-7b23c481/evidence/source-baseline-6b7129fa/utils/host-test-support/impl/build.gradle.kts#L8) | D | T4; test-only dependency forwarding artifact, no production implementation |
| 8 | [utils-images-spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/build.gradle.kts#L1) | D | T5, D1; pure policy + real transformer contract, throwing operation KDoc gaps |
| 9 | [utils-kotlinx-io-coroutines-spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/spec/build.gradle.kts#L8) | D | T6, D1; FS/raw ownership and error documentation incomplete |
| 10 | [utils-kotlinx-io-coroutines-impl](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/build.gradle.kts#L25) | R | T6; real platform FS, scoped open/use/close; no spec→impl main edge |
| 11 | [utils-kotlinx-io-serialization-spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-serialization/spec/build.gradle.kts#L8) | R | T7; actual serializer and stable descriptor/string wire operation |
| 12 | [utils-ktor-client-ext-spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/spec/build.gradle.kts#L8) | D | T8, D1; real Ktor plugin/extensions; cold-flow failure docs incomplete |
| 13 | [utils-ktor-client-ext-impl](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/build.gradle.kts#L8) | B1 | T8, C1; real native engine; C2 additionally requires experiment |
| 14 | [utils-logging-spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/logging/spec/build.gradle.kts#L8) | R | T9; pure scoped logger behavior, no resource owner |
| 15 | [utils-logging-impl](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/logging/impl/build.gradle.kts#L21) | B2 | T9, F1; explicit process-wide writer, test isolation remains unverified |
| 16 | [utils-patch-spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/spec/build.gradle.kts#L9) | R | T10; authoritative parser/models/matcher; FS dependency is spec only |
| 17 | [utils-patch-impl](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/impl/build.gradle.kts#L8) | D | T10, D1; real borrowed-FS application; operation guarantees still impl-only |
| 18 | [utils-read-write-mutex-spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/spec/build.gradle.kts#L8) | R | T11; actual State/Mutex contract owns repaired owner rejection |
| 19 | [utils-read-write-mutex-impl](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/impl/build.gradle.kts#L8) | D | T11; real repair accepted statically; unused SafeRw helper remains |
| 20 | [utils-rpc-exception-spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/rpc-exception/spec/build.gradle.kts#L9) | R | T12; authoritative closed wire hierarchy + restoration code |
| 21 | [utils-search-index-spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/spec/build.gradle.kts#L1) | R | T13; actual fixed-snapshot contract is clear; impl violates it |
| 22 | [utils-search-index-impl](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/impl/build.gradle.kts#L21) | B1 | T13, S1; real JS/Lucene actuals retain caller list |
| 23 | [utils-terminal-text-spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/terminal-text/spec/build.gradle.kts#L8) | R | T14; pure grapheme/cell policy; external Unicode dependencies not public models |

## Build/source-set boundary checks

- All nine impl main dependency blocks expose their actual matching spec;
  spec main graphs inspected here have no project dependency on impl. A
  spec project containing pure code, a serializer, or a dependency contract is
  legitimate. There is no requirement to add an interface to these projects.
- Physical layout/inclusion is real:
  [settings L34–48,63](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L34)
  maps `utils/<topic>/<spec|impl>` into hyphenated Gradle project names.
- Shared targets include JVM/JS/Wasm/Linux/macOS/MinGW/iOS
  [shared convention L22–45](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-shared.gradle.kts#L22);
  host adds Node JS to JVM/four native targets
  [host L11–23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-host.gradle.kts#L11),
  [host target helper L22–32](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt#L22).
  External URL and RPC exceptions use CLI host targets without JS
  [CLI convention L9–12](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-cli.gradle.kts#L9).
- Actual source-set distinctions matter: IO blocking=JVM+POSIX, Node and MinGW
  are separate actuals; search `luceneMain`=JVM+native and JS fallback;
  logging `fileLoggingMain`=JVM+native (no JS logging initializer);
  custom curl is `linuxMain`, not macOS/Windows. No current source was
  assumed to cover targets outside these declarations.
- `e89c5094` moved SSE JVM server test dependencies to the **impl test set**;
  current build confirms this at
  [L13–16](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/build.gradle.kts#L13).
  Pure spec tests do not transitively need that local server fixture.
- Test framework injection uses Kotlin test/TestBalloon only
  [helper L50–55](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt#L50).
  MockEngine comes from the explicit support dependency, not hidden production
  fixture source. No owned project introduces a code generation task or new
  resource format; curl registration is eager Kotlin initialization, not a
  fabricated missing service resource.
- Dependency versions are repository declarations, not online verification:
  [catalog L3–10,24,28](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/gradle/libs.versions.toml#L3)
  and engine bundles
  [L83–87](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/gradle/libs.versions.toml#L83).
  Lucene composite substitution is real at
  [settings L25–28](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L25).
  External library internals were not reaudited.

## Trace evidence, original → spec → implementation → caller → tests

### T1 — coroutines

- Original `utils/coroutines/src/.../ChildScope.kt` and
  `RunCatchingCancellable.kt` moved 100% to spec in `6606efe9`.
  `io.github.stream29.kodex.utils.coroutines.childScope`,
  `supervisorChildScope`, `cancelAndJoin` are real executable contracts at
  [ChildScope L22–53](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/coroutines/ChildScope.kt#L22);
  the original `Job(parent)` / `SupervisorJob(parent)` relationship is intact.
  Rejecting a caller-supplied Job prevents a second authority over ownership.
- Production renewable lease uses the exact helper
  [lease L26–34](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseImpl.kt#L26).
  No DTO/interface adapter is involved.
- `runCatchingCancellable` preserves cancellation, rather than converts it to a
  Result failure
  [L9–11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/coroutines/RunCatchingCancellable.kt#L9).
- Actual helper tests check parent cascade, sibling failure versus supervisor
  isolation and Job rejection
  [ChildScopeTest L18–70](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/coroutines/spec/src/commonTest/kotlin/io/github/stream29/kodex/utils/coroutines/ChildScopeTest.kt#L18);
  cancellation propagation is asserted at
  [RunCatchingCancellableTest L29](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/coroutines/spec/src/commonTest/kotlin/io/github/stream29/kodex/utils/coroutines/RunCatchingCancellableTest.kt#L29).
- D1: ownerless scope/Job argument rejection lacks `@throws` on the throwing
  functions; tests omit owner-without-Job and child-cleanup completion barriers.
  Minimal ablation: keep these pure helpers, do not add an impl project.

### T2 — external-url

- Original top-level `openExternalUrl` split in `6606efe9`; `e89c5094` removed
  unused `ExternalUrlOpener` / `HostExternalUrlOpener`, restoring the real
  top-level path. Current spec really owns
  `io.github.stream29.kodex.utils.externalurl.OpenExternalUrlResult`
  [L3–12](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/externalurl/ExternalUrl.kt#L3).
- Actual function rejects blanks and delegates to the actual host operation
  [L15–20](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/externalurl/ExternalUrl.kt#L15);
  JVM selects `open` / `rundll32.exe` / `xdg-open`
  [JVM L5–34](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/externalurl/ExternalUrl.jvm.kt#L5);
  native selects the equivalent commands
  [native L9–34](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/impl/src/nativeMain/kotlin/io/github/stream29/kodex/utils/externalurl/ExternalUrl.native.kt#L9).
- The real process operation awaits exit code, closes process/client in finally,
  propagates CancellationException and redacts the URL from ordinary error text
  [ExternalUrlProcess L12–32](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/externalurl/ExternalUrlProcess.kt#L12).
- Production default `io.github.stream29.kodex.cli.app.SessionTreeCliScreen`
  calls that function, not the removed interface
  [L111–116](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L111).
  Local test only covers blank input
  [ExternalUrlTest L7](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/externalurl/ExternalUrlTest.kt#L7).
- Gap/experiment: isolated host launcher success/nonzero/cancel and credential
  redaction, with process completion observed. This is not a real-browser
  acceptance. Minimal ablation: removed wrappers should stay removed; the result
  model and real host factory already preserve responsibilities.

### T3 — filesystem-lease

- Original `filesystem-lease/contract` renamed to spec in `6606efe9` (100%);
  real impl algorithms did not change. FQCN
  `io.github.stream29.kodex.utils.filesystemlease.FileSystemLease` remains
  `AutoCloseable, CoroutineScope`
  [spec L5–9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLease.kt#L5).
  This is a genuine resource identity, not a marker requiring replacement.
- Factory `CoroutineScope.FileSystemLease` returns
  `RenewableFileSystemLease`, directly implementing that spec
  [L20–45](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseImpl.kt#L20).
  Successful delivered leases have owner-bound renewable children and
  NonCancellable release, but L1/L2 remain.
- Read factory shares only the same owner's reference, rejects another owner,
  decrements under a mutex and closes the real lease only on the final reference
  [ReadLease L41–83](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemReadLeaseImpl.kt#L41).
  `SharedReadLease` is a necessary per-handle resource, not an unused wrapper
  [L97–115](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemReadLeaseImpl.kt#L97).
- Write/guard path uses the same acquisition operation and join-only release
  [WriteLease L13–43](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemWriteLeaseImpl.kt#L13),
  [guard L33–54](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseSupport.kt#L33).
  Heartbeat serialized fields remain `pid`, `acquiredAt`, `expiresAt`
  [L151–164](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseSupport.kt#L151);
  release checks exact pid+acquiredAt
  [L106–114](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseSupport.kt#L106).
- Real consumers are Home `prepareKodexHome`
  [Migration L78–133](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigration.kt#L78),
  `FileSystemKodexAgentSession`
  [L33–42](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexAgentSession.kt#L33),
  and Session repository deletion/repair
  [L238–266](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt#L238),
  [L415–439](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt#L415).
  Canceling the Session scope cannot release an owner never handed to a lease.
- Actual tests cover delivered exclusive/read/write owner completion, sharing,
  mutual exclusion and malformed owner fail-closed
  [LeaseImplTest L26–106](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseImplTest.kt#L26).
  JVM process fixture checks real cross-process exclusion
  [LeaseProcessTest L36](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/jvmTest/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseProcessTest.kt#L36).
  Neither proves safe publication or observed cleanup failure.

### T4 — host-test-support

- Original single build artifact moved unchanged to spec in `6606efe9`; new impl
  only reexports it. No Kotlin declaration, implementation or production factory
  exists in either artifact; claiming an FQCN here would be fabricated.
- Spec's actual contract is transitive `io.ktor:ktor-client-mock` at its build
  L8; impl's actual responsibility is the build L8 forwarding dependency.
  Production source does not consume this artifact; OpenAI **commonTest** does
  [client build L20–24](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/build.gradle.kts#L20).
- Real test creates MockEngine directly, validates request form/URL and closes
  its client
  [OpenAiLoginClientTest L21–55](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonTest/kotlin/io/github/stream29/kodex/openai/client/OpenAiLoginClientTest.kt#L21).
  MockEngine is external dependency code, not a hidden duplicate Kodex client.
- D: dependency-only test support is legitimate. Minimal ablation, if artifact
  reduction is desired, is to consume spec directly and remove only the impl
  forwarding build, retaining MockEngine/test scope. No fake support interface,
  fixture code or production dependency is needed. No runtime isolation result.

### T5 — images

- All original image policy/container/parser files moved 100% to spec in
  `6606efe9`. `e89c5094` deleted the unused `PromptImageCodec` contract and put
  guarantees on the existing real
  `io.github.stream29.kodex.utils.images.PromptImageTransformer.transform`
  [L49–69](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/PromptImageProcessing.kt#L49).
  No second codec authority remains.
- `EncodedImage` and `PromptImageTransformRequest` defensively copy input and
  output bytes
  [L22–46](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/PromptImageProcessing.kt#L22).
  Actual `toPromptImage` computes plan, skips unnecessary transform, then validates
  transformed MIME/dimensions
  [L82–124](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/PromptImageProcessing.kt#L82).
  Dimension policy is pure
  [PromptImageDimensions L38–103](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/PromptImageDimensions.kt#L38).
- Actual filesystem bridge calls this policy
  [PromptImageCodecIo L14–28](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/codec/PromptImageCodecIo.kt#L14);
  production `io.github.stream29.kodex.tool.viewimage.ViewImageToolClient.view`
  supplies `HostPromptImageTransformer` and reads through it
  [L11–35](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/view-image/src/commonMain/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageToolClient.kt#L11).
  Platform codec verification is owned by the platform reviewer, not double-counted.
- Tests actually check preserve/resize/GIF policy and data-URL path
  [PromptImageProcessingTest L14–77](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonTest/kotlin/io/github/stream29/kodex/utils/images/PromptImageProcessingTest.kt#L14),
  malformed/oversize data URLs
  [ImageDataUrlsTest L30–43](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonTest/kotlin/io/github/stream29/kodex/utils/images/ImageDataUrlsTest.kt#L30).
- D1: throwing header/data-URL/`toPromptImage` operations lack attached `@throws`;
  exception-class descriptions alone do not meet the checklist. Gap: explicit
  byte alias mutation and wrong-transform metadata assertions in this pure
  suite. Minimal ablation: retain actual transformer + pure policy, not the
  removed codec interface.

### T6 — kotlinx-io-coroutines

- Original raw contract code moved 100% to spec; FS contract lost only platform
  default declaration, now a real impl `expect` binding
  [SystemCoroutineFileSystem L5–10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/SystemCoroutineFileSystem.kt#L5).
  FQCN `io.github.stream29.kodex.utils.kotlinxiocoroutines.CoroutineFileSystem`
  owns scoped source/sink use and private-file operations
  [spec L29–125](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineFileSystem.kt#L29).
- JVM factory selects `BlockingCoroutineFileSystem` and exclusive `CREATE_NEW`,
  metadata/fingerprint/private chmod hooks
  [JVM L15–58](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/SystemCoroutineFileSystem.jvm.kt#L15).
  Blocking opens and use/finally are **inside** IO dispatcher scope, not returned
  resources across a cancellable dispatcher handoff
  [Blocking L41–60](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/blockingMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/BlockingCoroutineFileSystem.kt#L41).
- Native POSIX selects the same adapter, with `fopen(...x)` exclusive sink
  [POSIX L26–56](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/posixMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/SystemCoroutineFileSystem.posix.kt#L26).
  Linux x64/arm64 use `st_mtim`; macOS uses `st_mtimespec`; each has a real
  private-file chmod actual. Native dispatcher is elastic IO
  [L8–10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/nativeMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/PlatformCoroutineFileSystem.kt#L8).
  MinGW opens/use-closes Win32 handles inside IO scope
  [L152–185](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/mingwX64Main/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/MingwCoroutineFileSystem.kt#L152).
- Node shields only uncancellable open handoff, enters use/finally, restores caller
  cancellation before block
  [L162–211](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/NodeCoroutineFileSystem.kt#L162).
  Error conversion preserves CancellationException
  [L335–337](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/NodeCoroutineFileSystem.kt#L335).
- `CoroutineCloseable.use` closes NonCancellably and preserves original failure
  with suppressed cleanup failure
  [RawIo L40–56](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineRawIo.kt#L40).
  `copyTo` borrows both streams and does not close them
  [L60–75](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineRawIo.kt#L60).
  Process streams intentionally stay process-client-owned
  [process spec L19–39](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClient.kt#L19);
  general raw IO's “must close” text must not override a specific borrowed owner.
- Production callers include lease writes at T3 and patch application at T10;
  they pass a borrowed `CoroutineFileSystem`, not an owned FS handle. Image
  filesystem bridge at T5 also uses this scoped byte API.
- Actual close tests inject cancellable close and primary/cleanup failures
  [FSTest L158–214](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineFileSystemTest.kt#L158).
  Real descriptor suites observe owner completion at
  [JVM L23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/jvmTest/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineFileDescriptorTest.kt#L23),
  [Linux L33](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/linuxX64Test/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineFileDescriptorTest.kt#L33),
  [Node open-handoff L23–52](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/jsTest/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineFileDescriptorTest.kt#L23).
  Node descriptor test relies on Linux `/proc`, not cross-platform Node proof.
- D1/gaps: scoped blocks' borrowed/non-escape rule, copy ownership, EOF/zero-byte
  semantics and IOException conditions need normative documentation. Direct raw
  `close()` dispatcher implementations are cancellable, while **scoped use**
  correctly shields them; no claim that arbitrary escaped raw handles are safe.
  Experiment: cancellation at acquisition and cleanup fault matrix on MinGW/macOS,
  plus borrowed streams staying open after copy. Do not add another IO wrapper.

### T7 — kotlinx-io-serialization

- Original serializer and build moved 100% in `6606efe9`. Actual
  `io.github.stream29.kodex.utils.kotlinxioserialization.PathAsStringSerializer`
  is spec code, not a DTO pretending to be an implementation:
  [L12–21](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-serialization/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxioserialization/PathAsStringSerializer.kt#L12).
- Descriptor remains `kotlinx.io.files.Path`, primitive STRING; encode is
  `Path.toString()`, decode is `Path(string)`. No resolver, filesystem access,
  ownership or persisted-field renaming was introduced.
- Production `McpServerConfiguration.Stdio.workingDirectory` explicitly selects
  this serializer and retains `working_directory`
  [McpSettings L143–150](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpSettings.kt#L143).
  OpenAI compaction path model also selects it
  [CompactionModels L68](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/CompactionModels.kt#L68).
- No local serializer test source exists. Evidence is unchanged real serializer
  and actual annotated callers, not a claimed successful runtime roundtrip.
  Needed experiment: descriptor + golden JSON + relative/Unicode/Windows
  path roundtrip across targets; path normalization is platform behavior, not
  a newly introduced migration policy. Minimal ablation: keep serializer in spec;
  do not create an empty impl or fake interface.

### T8 — ktor-client-ext

- `6606efe9` moved original headers/SSE/plugin code unchanged to spec and all
  Linux curl code unchanged to impl. `e89c5094` corrected only test dependency
  placement. Real spec operations
  `io.github.stream29.kodex.utils.ktorclientext.postSseEvents`,
  `postSseEventsWithHeaders` return a cold channelFlow and use official Ktor
  scoped SSE session
  [SseRequests L24–80](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/SseRequests.kt#L24).
  Client is borrowed; collector owns collection/session, not the whole HttpClient.
- `SseCompatibility` only marks selected requests and repairs missing Content-Type
  for HTTP 200; supplied content type and other statuses are not overwritten
  [L26–59,69–75](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/SseCompatibility.kt#L26).
  Header nullable removal and exact originator/account/version names remain at
  [HeadersBuilder L10–23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/HeadersBuilderExtensions.kt#L10),
  [HttpHeaders L6–15](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/HttpHeadersExtensions.kt#L6).
- Real production `io.github.stream29.kodex.openai.client.OpenAiClient`
  installs SSE/compatibility and consumes both stream helpers
  [L136–142](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L136),
  [L370–389](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L370),
  [L518–526](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L518).
- Linux engine FQCN
  `io.github.stream29.kodex.utils.ktorclientext.kodexcurl.KodexCurlClientEngine`
  is actually selected by eager `engines.append(KodexCurl)` and factory
  [KodexCurl L24–41](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurl.kt#L24).
  Execute adapts real body channel/response to Ktor; engine close reaches processor
  [Engine L29–79](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlClientEngine.kt#L29).
- Native header ownership transfers `takeHeaders()` → RequestHolder; holders
  dispose slist, StableRefs and cancellation registration
  [Raw L70–78](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlRaw.kt#L70),
  [Multi L29–45,144–158](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt#L29).
  Cancel requests carry a generation token and verify exact holder identity
  [L225–236](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt#L225).
  Those are real lifecycle/anti-reuse responsibilities, not removable wrappers.
- Request body WriteChannelContent uses `writer(coroutineContext)`, so despite
  the spelling `GlobalScope.writer`, the current Job is supplied
  [Raw L111–124](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlRaw.kt#L111).
  Do not conflate this with **jobless detached cleanup** in C1.
  Response body has `Job(callContext)` and owned ByteChannel
  [HttpResponseBody L22–31](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlHttpResponseBody.kt#L22).
- JVM tests exercise real local SSE with inactivity timeout, heartbeats, wrong/
  missing content types and non-success responses
  [SseRequestsJvmTest L41–167](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/jvmTest/kotlin/io/github/stream29/kodex/utils/ktorclientext/SseRequestsJvmTest.kt#L41).
  Linux test proves intended default engine assertion and connection-failure
  path only, not shutdown/streams
  [DefaultEngineTest L15–36](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxX64Test/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlDefaultEngineTest.kt#L15).
- D1: plugin missing/invalid timeout errors occur on Flow collection and lack
  `@throws`; successful function return is not successful network acquisition.
  C1/C2 below block blanket native acceptance. Additional gaps: cancellation
  under paused backpressure, request body ownership and WebSocket close path
  (advertised capability is not WS runtime acceptance).

### T9 — logging

- Original scoped logger moved 100% to spec, file logger/test 100% to impl in
  `6606efe9`. `io.github.stream29.kodex.utils.logging.global/session/agent/tool`
  call the actual private `ScopedKLogger`, flatten inherited payload instead of
  building competing scope owners
  [ScopedLogging L12–90](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/logging/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/logging/ScopedLogging.kt#L12).
- Production `buildMasterAgentRuntime` composition uses session and tool contexts
  [Composition L44](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L44),
  [L191](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L191).
- Actual `initializeLogging` creates Kermit rolling writer and `KermitAppender`,
  sets process-global KotlinLogging config; mapping handles OFF/markers/payload
  [ApplicationLogging L16–70](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/logging/impl/src/fileLoggingMain/kotlin/io/github/stream29/kodex/utils/logging/ApplicationLogging.kt#L16).
  CLI calls this before application creation
  [Main L24–31](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L24).
  This is actual process bootstrap, not a Session-owned logger.
- Pure tests use independent RecordingLoggerFactory instances and check payload
  override, lazy evaluation and markers
  [ScopedLoggingTest L14–115](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/logging/spec/src/commonTest/kotlin/io/github/stream29/kodex/utils/logging/ScopedLoggingTest.kt#L14).
  Impl test exercises real writer but mutates global state without restoration
  [ApplicationLoggingTest L25–42](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/logging/impl/src/fileLoggingTest/kotlin/io/github/stream29/kodex/utils/logging/ApplicationLoggingTest.kt#L25).
  F1 below prevents calling the file fixture independent.
- D: process-lifetime writer retention is explicitly acknowledged by that test;
  no new Session/Application wrapper should be invented. If repeat initialization
  or owner-scoped shutdown is required, first define real Kermit disposal/
  completion support; this review did not inspect Kermit's internal resources.

### T10 — patch

- Original models/parser/matcher moved to spec in `6606efe9`; pure matcher uses
  four tolerance passes/KMP and original path/wire shapes remain.
  Real FQCNs `io.github.stream29.kodex.utils.applypatch.Patch`, `Hunk`,
  `PatchApplyResult` are here
  [PatchModels L15–85](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/PatchModels.kt#L15).
  `environment_id`, `move_path`, `change_context`, `affected_paths` are retained.
- Actual `String.parsePatch` owns documented syntax failure and normalizes text
  without FS access
  [PatchParser L14–38](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/PatchParser.kt#L14);
  matching comes from spec
  [FindLineSequence L21–50](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/FindLineSequence.kt#L21).
- `e89c5094` genuinely removed `PatchApplier` / `PatchApplierImpl`; current
  `Patch.applyToFileSystem` directly applies real hunks, no forwarding object
  [PatchFileSystem L20–24,70–101](file:///home/stream/ACodeSpace/local/kodex-e2e-20261007-7b23c481/evidence/source-baseline-6b7129fa/utils/patch/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/PatchFileSystem.kt#L20).
  Injected FS is borrowed; relative/absolute resolution, overwrite, move and
  partial commit are explicit, not a transaction accidentally promised.
- Production `io.github.stream29.kodex.tool.applypatch.ApplyPatchToolClient`
  uses parser then that actual operation
  [L16–19](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/apply-patch/src/commonMain/kotlin/io/github/stream29/kodex/tool/applypatch/ApplyPatchToolClient.kt#L16).
- Tests exercise the real default FS path, not a fake applier: isolated temp root,
  scoped setup source/sink, teardown
  [PatchFileSystemTest L21–73](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/applypatch/PatchFileSystemTest.kt#L21).
  Partial commit/overwrite semantics are asserted
  [RustBehavior L295–362](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/applypatch/ApplyPatchBehaviorRustTest.kt#L295).
  Parser strict/CRLF/environment tests remain in spec
  [EdgeCase L70–109](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/spec/src/commonTest/kotlin/io/github/stream29/kodex/utils/applypatch/PatchParserEdgeCaseRustTest.kt#L70).
- D1: application operation contract now exists only in impl KDoc. Remedy must
  retain the real function/borrowed FS semantics, not resurrect unused applier
  interfaces. Minimal boundary experiment: put the explicit-FS operation in
  spec and keep default system-FS binding in impl only if this reduces, rather
  than duplicates, API responsibility. No authorized code change here.

### T11 — read-write-mutex

- Original combined interface/impl split in `6606efe9`; exact FQCN
  `io.github.stream29.kodex.utils.ReadWriteMutex` owns State/reader/writer.
  Real impl factory instantiates private `ReadWriteMutexImpl`
  [impl L10–16](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/ReadWriteMutex.kt#L10).
- `e89c5094` really repairs mismatched writer owner **before** publishing Free:
  [unlock L78–86](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/ReadWriteMutex.kt#L78).
  Spec states rejection leaves both lock and state unchanged
  [spec L31–38](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/ReadWriteMutex.kt#L31).
  Cancellation waiting for writer releases canRead in catch
  [impl L62–70](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/ReadWriteMutex.kt#L62).
- Actual production `InMemoryIndexVersioned` uses reader/writer withLock rather
  than a substitute state API
  [InMemoryStorage L114–131](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/inmemory/InMemoryKodexAgentStorage.kt#L114).
- Regression test checks wrong-owner rejection, State.Write, held owner,
  reader exclusion, then correct unlock/read acquisition
  [MutexTest L21–33](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/ReadWriteMutexTest.kt#L21).
  This test would fail if the pre-validation lines were removed: meaningful
  minimal ablation, not merely an `is ReadWriteMutex` type assertion.
  Wait/cancel paths also have assertions
  [L55–151](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/ReadWriteMutexTest.kt#L55).
- D: `io.github.stream29.kodex.utils.SafeRw` remains public in impl
  [SafeRw L8–32](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/SafeRw.kt#L8);
  current tree search finds only its own tests, no production caller. This is
  unused convenience debt, not another deployed mutex owner. Minimal ablation:
  remove helper+dedicated tests if no external API commitment, preserve actual
  ReadWriteMutex and its production caller. Gap: multithread stress and deprecated
  onLock/reader owner semantics; do not claim an exact ordinary Mutex substitute.

### T12 — rpc-exception

- All original six declarations + restoration code moved 100% in `6606efe9`.
  Actual `io.github.stream29.kodex.utils.rpcexception.RemoteException` is sealed,
  serializable; computed message uses base serializer, no recursive backing field
  [L6–16](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/rpc-exception/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/rpcexception/RemoteException.kt#L6).
  Known fieldless variants retain FQCN discriminators; no migration rename.
- Actual `restoreRemoteException` rethrows cancellation, only decodes top-level
  brace-starting messages, preserves original malformed/unknown failures
  [L15–27](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/rpc-exception/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/rpcexception/RestoreRemoteException.kt#L15).
  Not a retry/activation/metadata replacement API.
- Real production `io.github.stream29.kodex.rpc.client.RestoringRpcClient`
  adapts unary operations and **upstream Flow catch**, not downstream collection
  [L24–34](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/rpc/client/RestoringRpcClient.kt#L24).
- Exact wire/discriminator tests and no-retry/cancellation/downstream boundaries
  are real
  [RestoreTest L45–52,136–174](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/rpc-exception/spec/src/commonTest/kotlin/io/github/stream29/kodex/utils/rpcexception/RestoreRemoteExceptionTest.kt#L45),
  [L178–244](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/rpc-exception/spec/src/commonTest/kotlin/io/github/stream29/kodex/utils/rpcexception/RestoreRemoteExceptionTest.kt#L178).
  Real transport restoration tests exist downstream
  [RestoringRpcClientTest L33–52](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/client/src/commonTest/kotlin/io/github/stream29/kodex/rpc/client/RestoringRpcClientTest.kt#L33).
- Minimal ablation: retain executable spec hierarchy/restorer; no impl project or
  fake remote-error interface. Gap: runtime wire acceptance belongs to RPC reviewer;
  this line does not convert static fixture source into transport pass.

### T13 — search-index

- Original `SearchIndex`/`SearchDocument` moved to spec; factory removed from
  spec into impl in `6606efe9`; both actuals moved 100%. The original fixed
  snapshot KDoc predates migration and remains
  [spec L8–15](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/searchindex/SearchIndex.kt#L8).
- Actual `io.github.stream29.kodex.utils.searchindex.createSearchIndex`
  directly calls the platform factory
  [Factory L6–9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/searchindex/SearchIndexFactory.kt#L6).
  `FallbackSearchIndex` (JS) and `LuceneSearchIndex` (JVM/native) implement spec
  [JS L3–22](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/searchindex/PlatformSearchIndex.kt#L3),
  [Lucene L16–56](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/impl/src/luceneMain/kotlin/io/github/stream29/kodex/utils/searchindex/PlatformSearchIndex.kt#L16).
  Lucene writer use-closes after indexing; directory/reader/analyzer are retained
  without a local close operation. Whether retained heap-only index objects require
  an external lifecycle contract remains a library-boundary question, not a
  proven OS-handle leak in this audit.
- Production `io.github.stream29.kodex.tool.toolsearch.ToolSearchEngine`
  supplies a freshly mapped document list and validates positive limits
  [L8–35](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/tool-search/src/commonMain/kotlin/io/github/stream29/kodex/tool/toolsearch/ToolSearchEngine.kt#L8).
  It does not demonstrate caller mutation. S1 is a factory contract defect.
- Only local test checks two immutable-list documents and one search
  [SearchIndexTest L10–18](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/searchindex/SearchIndexTest.kt#L10).
  No mutable-source snapshot, nonpositive limit or JS/Lucene semantic parity
  test. Do not prescribe identical scoring where spec does not promise it.
- Minimal ablation: copy input list once at real factory, retaining existing
  interface/actual implementations; no extra manager/snapshot DTO/index wrapper.

### T14 — terminal-text

- Original entire pure library/tests moved 100% to spec in `6606efe9`.
  FQCN functions under `io.github.stream29.kodex.utils.terminaltext` own UTF-16
  grapheme offsets, terminal width and fitting prefix/suffix; actual algorithms
  are the spec code
  [TerminalText L6–63](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/terminal-text/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/terminaltext/TerminalText.kt#L6).
- Actual production `TextInput` uses segment offsets and fits available cell width
  [L395–409](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TextInput.kt#L395),
  [L449](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TextInput.kt#L449).
  Session tab geometry also uses real terminal width
  [TabBar L204–208](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessiontabbar/SessionTabBar.kt#L204).
- Actual tests include CJK, combining mark, ZWJ emoji, zero-width mark,
  source offsets and zero/too-narrow fit
  [TerminalTextTest L7–38](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/terminal-text/spec/src/commonTest/kotlin/io/github/stream29/kodex/utils/terminaltext/TerminalTextTest.kt#L7).
  These are pure policy tests, **not** renderer/input/terminal acceptance.
- Minimal ablation: keep this shared pure behavior; creating a terminal-text
  implementation interface would lose no concrete coupling because none exists.
  Gap: renderer cursor/hit-test and platform Unicode rendering remain downstream.

## Findings requiring coordinator decision

### L1 — B1, confirmed preexisting publication orphan; shared Home root

- Evidence: acquisition writes owner bytes at
  [LeaseImpl L90–94](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseImpl.kt#L90)
  **before** constructing the resource/cleanup child at L95. There is no
  rollback catch covering this gap. Guard/read/write/exclusive factories reuse it.
- Actual Home test writes real bytes then suspends before return; after cancellation
  and owner completion it deliberately asserts one read owner remains
  [LifecycleTest L270–297](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/KodexHomeHandleLifecycleTest.kt#L270).
  Test source is confirmation of the known baseline characterization, not this
  review executing it or evidence of a fix.
- Impact/spec implication: a failed acquisition may still publish an unowned
  live lock, delaying Home startup/migration and Session open/delete/repair
  until expiry. Guard publication can likewise orphan its acquisition lock.
  Owner Job alone is not evidence that failed acquisition registered cleanup.
- History: same write→return code exists at `6606efe9^`; impl had no source
  edit in this migration. **Not a new migration regression.**
- Minimal repair/experiment: within existing acquisition, capture this precise
  heartbeat identity, rollback unhanded publication and await finite
  NonCancellable release; preserve original/suppressed failures. Convert the
  existing fault probe to zero-orphan invariant and run guard/read/write/exclusive
  publication barriers. Do not change frozen Home/heartbeat protocol, delete an
  unknown owner, or add a parallel LeaseManager. UI/Home impact must be merged
  with this root, not counted independently.

### L2 — B1, confirmed preexisting cleanup result is not observed

- Evidence: `closeAndJoin` only closes then `Job.join()`
  [Support L23–25](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseSupport.kt#L23).
  Release runs in a launched child under supervisor; join waits termination,
  not the child's cleanup error. Home duplicates this pattern
  [HeldLeaseKodexHomeHandle L29–31](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigration.kt#L29).
- Actual Home primary+delete failure characterization receives cleanup only in
  background handler, asserts **empty suppressedExceptions** and remaining write lock
  [LifecycleTest L232–266](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/KodexHomeHandleLifecycleTest.kt#L232).
- Impact: guard/final-read/write cleanup and Session repository use/release waiters
  cannot distinguish released resource from failed physical deletion just by join.
  Success path can hide cleanup failure; primary failure keeps identity but misses
  cleanup suppression. This is distinct from L1's never-registered owner.
- History: join-only impl exists before `6606efe9`; new Home spec/tests expose,
  rather than cause, the old issue. **Characterization-green does not fix it.**
- Minimal repair: the actual lease needs one observable awaited release result
  consumed by Home/Session waiters, retaining request-only close semantics and
  per-read-handle reference ownership. Existing `FileSystemLease` spec should
  own that lifecycle/error contract; not an external wrapper/result journal.
  Experiment: successful action+cleanup fail, primary+cleanup fail, canceled
  waiter then re-wait, repeated close, exact suppressed identity on each target.
  Share this root with UI/Home reviewer.

### C1 — B1, preexisting native curl cleanup lies outside resource owner

- Evidence: real engine closes superclass then processor
  [Engine L77–79](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlClientEngine.kt#L77).
  Processor `GlobalScope.launch(curlDispatcher)` has no supplied Job and first
  **joins engine owner**, only then disposes task/native API and dispatcher
  [Processor L106–122](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlProcessor.kt#L106).
  Thus owner completion is not cleanup completion by construction.
- Production chain is T8's eager Linux factory → default HttpClient →
  OpenAiClient streams. Resources include single-thread dispatcher, multi/easy
  handles, StableRefs and queued request headers, not just in-memory DTOs.
- History: processor is a 100% move in `6606efe9`; `git log --follow` also points
  to `3a03e3e6` / `f4be9653` before the migration. Same detached shutdown exists
  at `6606efe9^`. This is baseline lifecycle debt, not a new split regression.
- Tests/impact: Linux test only calls client.close and does not await/assert native
  resource release; JVM SSE tests cannot validate this Linux implementation.
  Exact real shutdown leak/hang counts are unmeasured, but the checklist's
  owner-completion ordering is demonstrably violated.
- Minimal repair/ablation: move the existing processor's cleanup into its real
  structured owner shutdown with finite NonCancellable release and observed
  failure, preserve serialization on its dispatcher, remove detached cleanup
  rather than add a new engine abstraction. Experiment: gated native cleanup,
  assert engine owner cannot finish first; active stream/backpressure close,
  queued canceled request and cleanup-fault matrix. No experiment run here.

### S1 — B1, preexisting fixed-snapshot contract retains caller-owned list

- Evidence: both real actual constructors store original `documents` directly
  [JS L6–17](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/searchindex/PlatformSearchIndex.kt#L6),
  [Lucene L19–20,51–55](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/impl/src/luceneMain/kotlin/io/github/stream29/kodex/utils/searchindex/PlatformSearchIndex.kt#L19);
  factory has no snapshot copy. Fixed document snapshot is explicit in spec T13.
- Impact: with a mutable input list, JS searches new content; Lucene searches
  old text but resolves hit ids against the mutated list. Clearing the original
  list statically yields no JS hits / null Lucene lookups; replacing its entry
  can return the wrong value for old indexed text. Current ToolSearch's fresh
  `map` avoids an identified mutation in today's production caller.
- History: both actuals moved 100%; fixed-snapshot KDoc already exists at
  `6606efe9^`. No new migration regression.
- Minimal repair: one `documents.toList()` snapshot at existing factory, no
  second index authority. Test mutable-list clear/replacement after construction
  on JS and JVM/native; assert same prior values/text. Current one-case test does
  not exercise this. Runtime confirmation pending, code-level contradiction clear.

### C2 — B2, post-header transfer errors may look like successful EOF

- Concrete evidence: completed path invokes `collectFailedResponse`, but
  `httpStatusCode != 0L` returns null before checking result code
  [Multi L405–435](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt#L405);
  completion finally closes response with no failure cause
  [L373–400](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt#L373).
  Header callback can already have handed success to the caller.
- Risk: truncated HTTP/SSE body after valid headers may produce ordinary EOF,
  not transport failure for OpenAI consumers. Current Linux connection-failure
  test has no headers and misses this branch. Native/Ktor behavior was not run.
- History: unchanged pre-migration curl code, not new split behavior.
- Needed experiment: isolated local server supplies valid 200 headers and
  advertised body/stream, then closes early; collect the real Linux default
  client body/SSE and inspect failure vs normal EOF. Also test explicit socket
  timeout separately because it has another removal path.
- Minimal possible fix, subject to experiment: propagate completed transfer
  error to existing body channel and preserve native cleanup; no custom second
  SSE parser or wrapper engine.

### F1 — B2, file-logging fixture is not proven independent

- Concrete facts: test uses production global initializeLogging, sets TRACE and
  leaves logger factory/appender/level/startup settings installed; no restoration
  or fixture close
  [ApplicationLoggingTest L25–42,105–107](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/logging/impl/src/fileLoggingTest/kotlin/io/github/stream29/kodex/utils/logging/ApplicationLoggingTest.kt#L25).
  Its comment explicitly retains the real writer for test-process lifetime.
- Why B2 rather than a manufactured production failure: this impl currently has
  one such test and Gradle test executables may isolate it. No observed parallel
  test corruption is claimed. A random build temp path isolates files, **not
  process-global state or writer lifecycle**. Pure scoped logger tests are
  independently constructed and do not have this issue.
- History: file logging test is a 100% move in `6606efe9`; this is not a newly
  introduced test-state change.
- Needed experiment: run two initializer invocations plus sentinel logger test
  in the same process, varying order/concurrency; assert prior config restored
  and clarify retained writer disposal. If actual external writer lacks scoped
  disposal, use explicit subprocess isolation for bootstrap integration, not a
  pretend `closeWith` that cannot close its file. Restore existing global config
  in finally where supported; do not remove the meaningful real-file test.

### D1 — normative contract documentation remains incomplete

- Lease spec has no lifetime/close/ownership/error KDoc at all. Actual owner Job,
  request-only close, awaited completion, sharing and cleanup failure are currently
  learned from impl/tests; L1/L2 make this more than decorative documentation.
- Raw FS spec lacks explicit borrowed block/non-escape/EOF/error contracts;
  scoped use code is real, and pure helpers/serializers need no fake impl.
- Images has real exception classes but throwing functions lack `@throws`
  [DataUrls L35–74](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/ImageDataUrls.kt#L35),
  [ImageInfo L16–37](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/ImageInfo.kt#L16).
  Transformer operation docs were genuinely improved in `e89c5094`.
- Coroutine owner validation and SSE collection-time validation also need throwing
  operation KDoc; patch application's behavior/errors remain impl-only after
  wrapper ablation. Do not restore unused interfaces merely to fill directories.
- History: lease/FS/image/coroutine/SSE gaps largely predate the split;
  `e89c5094` deliberately removed patch/opener forwarding interfaces, so any
  remaining documentation relocation must target **real existing operations**.
  This grade records contract closure debt separately from demonstrated behavior.

## Qualified verdict and remaining experiments

- Physical layout: 23 actual assigned build files accounted for, 14 spec + 9 impl.
  Direction of these spec main project edges is clean; legitimate executable
  specs, serializers, models and test dependency artifacts remain legitimate.
- Contract takeover: real factory/spec/caller chains verified; mutex repair and
  removed external-url/patch/image-codec wrappers are genuine, not assumed.
  Lease lifecycle and other operation KDoc still prevent complete contract closure.
- Behavior: **not an all-pass**. L1/L2 are the confirmed unfixed preexisting lease
  defects; C1 and S1 are additional statically evidenced preexisting lifecycle/
  contract defects. Do not rebrand them as new migration regressions or count
  Home's same roots twice. C2 and F1 require targeted experiments.
- Runtime: no tests run by this reviewer; historical greens were not inherited.
  JVM/native/JS/MinGW/macOS acceptance, real browser launch, native backpressure/
  close and renderer tests remain separate from source trace acceptance.
- Minimum coordinator queue: lease safe-handoff + observed cleanup matrix;
  curl owner-before-release gate and truncated-stream test; mutable search
  snapshot test; logging in-process isolation sentinel; remaining platform
  FS/serializer/borrowed IO tests. Centralize execution rather than start
  parallel resource-consuming experiments from this child.
