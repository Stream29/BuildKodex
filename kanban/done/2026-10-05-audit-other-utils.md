# Task Tree

- `Trace real contracts, implementations, consumers and historical extraction`()
- `Review behavior, tests and meaningful ablation candidates`()
- `Publish evidence and unreviewed coverage`()
- `Record coordinator review; preserve pinned audit evidence`()

# Details

- Parent: [Spec/impl reacceptance](2026-10-05-reaccept-spec-impl-refactor.md).
- Own review scope: `utils/{coroutines,external-url,filesystem-lease,host-test-support,images,
  kotlinx-io-coroutines,kotlinx-io-serialization,ktor-client-ext,logging,patch,
  read-write-mutex,rpc-exception,search-index,terminal-text}`.
- Report only in this file; source, other tasks and shared resources are read-only.
- Apply parent acceptance rules and pinned baseline; no independent fixes or builds.
- Status: **REVIEW READY** — independent static report, not release acceptance.

## Review report

### Baseline, method and limits

- Inner HEAD verified at entry and before publication:
  `821b66275aae2f0a9541fa2e9e03023d875c30b5`; branch `refactor/spec`;
  inner working tree clean. **No baseline mismatch.**
- Read AGENTS, associated workflows, parent task, module-boundary checklist,
  utility migration task and relevant URL/lease/Curl/search decisions.
  Used tracked inventory, numbered source, whole-repository symbol searches,
  migration diffs and earlier history. No Gradle, IDE, browser, network or process
  test was started. Existing build outputs are not a new test run.
- All fourteen domains were relocated/extracted by
  `6606efe93bfbb70d63bd93424f2f91385a0e08b4` (“organize spec and impl modules”).
  Original source is `6606efe9^:utils/<domain>/src/...`, except lease's existing
  `contract/` and `impl/`. Diff from `6606efe9` to pinned HEAD is empty for these
  fourteen directories. Introduction commits below distinguish inherited risks
  from migration additions.
- Excluded utility domains were read only as necessary consumers or bindings;
  their implementation acceptance belongs to the other lane.
- R = traced static path, not whole-domain/platform acceptance; D = explicitly
  recorded debt; B1 = confirmed boundary/decision failure; B2 = concrete unresolved
  risk with proposed confirmation; U = evidence not established.

### Complete scope coverage

| Domain | Classification | Fact source / actual use | Residual coverage |
| --- | --- | --- | --- |
| coroutines | R | Scope/cancellation functions directly used by Session/MCP owners | U: full caller shutdown and error KDoc |
| external-url | B1-URL | Result DTO used; new interface reached only through global forwarding API | U: real launch/cancel on JVM/native |
| filesystem-lease | R shape; B2-LEASE | Unified resource contract returned by three factories; Home/Session own it | U: lease-loss, stale takeover, acquisition cancellation |
| host-test-support | D-TEST-COORD | Deliberate test dependency re-export; no production interface required | U: resolved test variants |
| images | R | Pure facts plus genuinely used transformer contract | U: operation @throws; codecs/platforms in other lane |
| kotlinx-io-coroutines | R binding/close; B2-PRIVATE | Actual blocking, Node and Windows filesystem/raw-I/O implementations | U: Node modes, platform races/ACLs |
| kotlinx-io-serialization | R | KSerializer used by generated MCP/OpenAI serialization | U: cross-platform/malformed path cases |
| ktor-client-ext | R HTTP/SSE; B2-SSE-DEPS | Real Ktor extensions and registered native engine | U: test classpath, native stress, WebSocket host use/lifecycle |
| logging | R adapter; D-LOG-UPSTREAM | Scoped KLogger events reach real process-global Kermit appender | U: reinitialization/flush/close and bootstrap error contract |
| patch | R DTO/parser/algorithm; B1-PATCH | Added interface behind retained compatibility extension | U: hard-cutover evidence, full I/O @throws |
| read-write-mutex | R ordinary binding; B2-MUTEX-OWNER | Existing interface/single state used by SafeRw/Storage/MCP | U: owner/deprecated Mutex API, concurrent stress |
| rpc-exception | R | Serializable failures and actual RPC restoration assembly | U: full frontend retry policy/transport execution |
| search-index | R ordinary binding | Actual SearchIndex called by ToolSearchEngine in agent runtime | U: mutable snapshot, resource lifetime, ranking parity |
| terminal-text | R | Grapheme/cell facts used by CLI layout/input | U: all Unicode/emulator/platform rendering |

### Confirmed blockers and minimal ablations

#### B1-URL — migration manufactured an unnecessary opener interface

- **Introducing commit `6606efe9`; original capability `8cdb1d39`.**
  Original global `openExternalUrl` guarded blank input and invoked platform code.
  Migration added
  `io.github.stream29.kodex.utils.externalurl.ExternalUrlOpener` in
  [spec ExternalUrl.kt:14–29](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/externalurl/ExternalUrl.kt#L14).
- [Impl ExternalUrl.kt:9–20](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/externalurl/ExternalUrl.kt#L9)
  makes the old global function forward to `HostExternalUrlOpener.open`.
  Whole tracked-repository search for both opener type names finds only the
  declaration, singleton implementation and forwarding call. No second policy,
  interface-valued production binding or interface fake/test was found.
- **The implementation does implement and execute the interface method**; URL
  launching is not dead. The issue is unnecessary contract extraction contradicting
  [approved URL rule:5](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/external-url.md#L5):
  keep the global API; do not add this interface without multiple policies or
  implementations. JVM/native command branches are mechanisms of one host policy.
- Actual production:
  [OpenAiLoginPopup.kt:41,54–61](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/OpenAiLoginPopup.kt#L41)
  binds `::openExternalUrl` as a function callback, checks attempt identity, then
  reports launcher result. SettingsPopup also calls the global function.
- Only local
  [ExternalUrlTest.kt:7–11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/externalurl/ExternalUrlTest.kt#L7)
  tests blank input through the global entry; no host launch/substituted interface.
- **Minimal ablation:** remove ExternalUrlOpener/HostExternalUrlOpener; preserve
  meaningful result DTO spec and global host operation, restore direct guard/platform
  invocation. Alternatively obtain a separate policy decision and bind a genuinely
  needed abstraction. Do not move browser effects into the ViewModel.

#### B1-PATCH — explicit compatibility forwarding retained instead of cutover

- **Introducing commit `6606efe9`; original patch domain `bfd5bea9`.**
  [PatchApplier.kt:13–29](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/PatchApplier.kt#L13)
  adds `io.github.stream29.kodex.utils.applypatch.PatchApplier`.
- [PatchFileSystem.kt:7–26](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/PatchFileSystem.kt#L7)
  adds PatchApplierImpl, retains `Patch.applyToFileSystem`, labels it “compatibility
  convenience”, and allocates the concrete implementation solely to forward into
  the old private algorithm.
- [ApplyPatchToolClient.kt:11–19](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/apply-patch/src/commonMain/kotlin/io/github/stream29/kodex/tool/applypatch/ApplyPatchToolClient.kt#L11)
  still calls the extension. Whole-repository PatchApplier references are only
  spec and this impl file; no production client binds an interface-valued applier.
- Tests still call the extension, e.g.
  [ApplyPatchBehaviorRustTest.kt:25](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/applypatch/ApplyPatchBehaviorRustTest.kt#L25),
  PatchFileSystemTest and clean-event serialization suites.
  They reach the real algorithm/new implementing method, but do not prove cutover.
- **Not a duplicated DTO or demonstrated behavior regression.** Confirmed failure
  is the introduced no-semantic compatibility forwarding path under the parent
  audit's bridge prohibition; an `implements` line alone does not establish adoption.
- **Minimal hard cutover:** ApplyPatchToolClient consumes a PatchApplier supplied
  by composition and calls apply directly; remove compatibility extension and update
  utility/serialization tests. Preserve one Patch/Hunk/result model and real algorithm.

### Suspected blockers and confirmation gates

#### B2-SSE-DEPS — JVM tests moved, server test dependencies did not

- **Migration-specific, introduced by `6606efe9`.** Original utility build declared
  server CIO/core in jvmTest. Now
  [spec build:14–17](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/spec/build.gradle.kts#L14)
  retains them while
  [impl build:5–22](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/build.gradle.kts#L5)
  has no jvmTest server dependencies.
- [SseRequestsJvmTest.kt:19–27,188–192](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/jvmTest/kotlin/io/github/stream29/kodex/utils/ktorclientext/SseRequestsJvmTest.kt#L19)
  is now in impl, imports server routing/engine and starts embedded CIO.
  [Catalog:50,56–57,83](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/gradle/libs.versions.toml#L50)
  separates client bundle from server artifacts;
  [common-test convention:50–55](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt#L50)
  adds only Kotlin test/TestBalloon.
- Main `api(project(spec))` does not establish dependency on spec's test compilation.
  Static graph suggests unresolved server symbols; **no compiler failure was observed**.
- Authorized confirmation: inspect impl JVM test compile classpath, then compile
  this target. Minimal repair: move test dependencies with relocated tests (and
  coroutines-test if required), not export server engines in production API.

#### B2-LEASE — heartbeat ownership loss may leave an active lease Job

- **Inherited risk:** loop originally `ae37acf6`, current renewable shape `81d353af`;
  `6606efe9` preserved impl while relocating contract.
- [RenewableFileSystemLease:33–62](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseImpl.kt#L33)
  delegates to a supervisor child scope. Missing/replaced heartbeat makes renew
  return false, ending only the launched coroutine; enclosing supervisor Job is
  not explicitly completed/cancelled on this path.
- [SharedReadLeases:41–46,87–94](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemReadLeaseImpl.kt#L41)
  tests that Job's isActive before reusing owner/issuing a new handle. A live Job
  after heartbeat loss could falsely represent ownership.
- Real Home/Session owners retain the lease. Ordinary close/owner cancellation is
  implemented correctly in the traced path; this finding concerns loss/takeover.
- Confirmation: controlled filesystem/clock replaces/deletes heartbeat, waits a
  renewal interval, observes lease Job, retries read handle/conflicting writer.
  Clarify required failure signal. Minimal candidate: cancel owned lease on loss/
  failure and refuse stale sharing; keep approved memberless resource shape.
- Existing tests do not cover this; no stale takeover/renewal experiment was run.

#### B2-PRIVATE — Node inherits private-file protection as no-op

- **Inherited introduction `e13a3fa7`; relocation `6606efe9`.**
  [CoroutineFileSystem.kt:69–75,92–105](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineFileSystem.kt#L69)
  promises owner-only protection where supported, defaults protection to Unit,
  and calls it before private writes.
- [Node backend:39–45,187–212](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/NodeCoroutineFileSystem.kt#L39)
  has no protectPrivateFile override and opens by flags only. POSIX Node protection
  may therefore depend solely on creation defaults/umask. Documented Windows ACL
  reliance does not establish POSIX Node owner-only permissions.
- Common production
  [BackendFileSystemAuthStore.kt:226](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/shared/auth/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStore.kt#L226)
  calls writePrivateString on its filesystem. A deployed Node auth host/umask
  **was not verified**; native/JVM protection is separate and real.
- Confirmation: authorized Node/POSIX fixture under permissive umask, stat mode
  during/after private write and establish supported-host contract.
  Minimal candidate: implement Node protection or explicitly disallow/document that
  host contract; do not change auth wire shapes or create another file model.

#### B2-MUTEX-OWNER — rejected writer unlock may corrupt observable state

- **Inherited introduction `1728ae62`; extraction `6606efe9`.**
  [Impl:62–81](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/ReadWriteMutex.kt#L62)
  changes State to Free before underlying canWrite/canRead unlock(owner).
  Invalid-owner rejection may thus happen after state changes while locks remain held.
- [Spec:9–32](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/ReadWriteMutex.kt#L9)
  says observable State accompanies successful transitions but omits supported
  owner checks/deprecated select behavior. Impl reader holdsLock always returns
  false; both onLock properties throw UnsupportedOperationException.
- Actual SafeRw/Storage/MCP use normal lock/withLock patterns; no wrong-owner
  production call was found. This is not a normal-path failure claim.
- Confirmation: writer lock(A), invalid unlock(B), inspect State/locks, valid
  unlock(A), reacquire. Clarify reader-owner/deprecated API support before testing it.
  Minimal candidate: validate ownership before single-state mutation and document
  proxy restrictions; do not add a second state authority. Existing tests are ownerless.

### Per-domain original → spec → implementation → production → tests

#### coroutines — R: substantive spec-only functions

- Originals `7b935309` (cancellation helper), `5a9a02ca` (scopes); `6606efe9`
  unchanged move. Namespace `io.github.stream29.kodex.utils.coroutines`.
  [ChildScope.kt:13–54](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/coroutines/ChildScope.kt#L13)
  owns Job checks, regular/supervisor child creation and cancel-and-join.
  [RunCatchingCancellable.kt:5–12](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/coroutines/RunCatchingCancellable.kt#L5)
  captures ordinary failures and rethrows cancellation; bodies themselves are facts.
- Real production:
  [FileSystemKodexAgentSession.kt:33–35](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexAgentSession.kt#L33)
  owns supervisor scope; [McpClientImpl.kt:161,192,222–242](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt#L161)
  uses cancellable capture for catalog/connect/operations.
- Static tests:
  [ChildScopeTest.kt:18–71](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/coroutines/spec/src/commonTest/kotlin/io/github/stream29/kodex/utils/coroutines/ChildScopeTest.kt#L18)
  real parent cancellation/sibling failure/supervisor/context-Job helpers;
  [RunCatchingCancellableTest.kt:11–37](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/coroutines/spec/src/commonTest/kotlin/io/github/stream29/kodex/utils/coroutines/RunCatchingCancellableTest.kt#L11)
  suspending value/failure/cancellation identity.
- Keep helpers; do not manufacture impl. U: absent-parent test, operation @throws
  for illegal Job contexts and all downstream cleanup ordering.

#### external-url — real effect/result, B1-URL remains

- Sole OpenExternalUrlResult DTO is in spec, no old declaration/public alias.
  Internal expect host-launch operation has an operation representation, although
  added opener abstraction violates the specific URL decision.
- [ExternalUrlProcess.kt:9–33](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/externalurl/ExternalUrlProcess.kt#L9)
  owns ProcessClient/session, awaits exit, closes in finally, rethrows cancellation,
  returns URL-free failures; Started is handoff, not page success.
- [JVM actual:5–35](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/externalurl/ExternalUrl.jvm.kt#L5)
  and [native actual:9–35](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/impl/src/nativeMain/kotlin/io/github/stream29/kodex/utils/externalurl/ExternalUrl.native.kt#L9)
  use direct open/xdg-open/rundll32 arguments, no shell.
- Production/test chain is B1-URL above. U: actual launcher availability,
  process cancellation/close privacy behavior across excluded process internals;
  no real launcher invoked. Minimal ablation remains interface/singleton, not result.

#### filesystem-lease — R approved resource shape; B2-LEASE

- [FileSystemLease.kt:5–9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLease.kt#L5)
  owns `io.github.stream29.kodex.utils.filesystemlease.FileSystemLease` and
  FileSystemLeaseInUseException. RenewableFileSystemLease/SharedReadLease implement
  it; all factories return it; consumers close/join through resource/scope.
- Memberless contract/factory ownership explicitly approved:
  [lease-shape task:17–26](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-09-02-refine-filesystem-lease-shape.md#L17).
  It is not an unused marker; do not add acquire/state/release methods.
- [KodexHomeMigration.kt:21–33,63–101](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigration.kt#L21)
  retains acquired read lease, takes write lease and closes/joins NonCancellable.
  Session creates lease before returning.
  [Support:33–55,85–129](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseSupport.kt#L33)
  implements guard, atomic heartbeat temp replacement, identity release and
  fail-closed malformed decoding. Process-local reader reference counts are
  necessary ownership state, not parallel public model.
- [Lease tests:26–106](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseImplTest.kt#L26)
  exercise actual filesystem/factories for sharing/exclusion/release/cancel/malformed.
  [Process test:17–58](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/jvmTest/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseProcessTest.kt#L17)
  covers concurrent readers/writer rejection, not crashed-process takeover.
- U: B2 loss, stale quarantine TOCTOU, filesystem identity in shared-read cache,
  acquisition cancellation and spec error facts. Minimal documentation candidate:
  attach approved close-vs-join/ownership facts without changing member shape.

#### host-test-support — D-TEST-COORD, legitimate contractless test dependency

- Original `271300a6`; `6606efe9` moved dependency to spec/added source-free impl.
  [Spec build:7–9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/host-test-support/spec/build.gradle.kts#L7)
  exposes Ktor MockEngine;
  [impl build:7–9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/host-test-support/impl/build.gradle.kts#L7)
  re-exports spec. No production FQCN/implementation is needed.
- Explicit recorded exception:
  [migration:88,104–107](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-09-30-migrate-all-utils-to-spec-impl.md#L110).
  Impl compatibility coordinate is documented debt, not a dummy API.
- Consumer [OpenAI client build:20–24](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/build.gradle.kts#L20)
  is commonTest only;
  [OpenAiLoginClientTest.kt:7,23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonTest/kotlin/io/github/stream29/kodex/openai/client/OpenAiLoginClientTest.kt#L7)
  imports/constructs `io.ktor.client.engine.mock.MockEngine`.
- Optional minimal ablation: tests depend directly on spec, remove compatibility
  coordinate after target parity check; no marker interface. U: resolved variants.

#### images — R pure facts and genuinely consumed transformer

- Originals `c4ccf32a`/`d0b057a7`; unchanged move `6606efe9`.
  Namespace `io.github.stream29.kodex.utils.images`.
  [ImageInfo.kt:3–37](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/ImageInfo.kt#L3)
  owns MIME/dimensions;
  [PromptImageDimensions.kt:8–35,55–103](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/PromptImageDimensions.kt#L8)
  owns limits/resize policy. Data URL guards don't trust metadata MIME.
- [PromptImageProcessing.kt:22–50,79–105](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/PromptImageProcessing.kt#L22)
  copies byte ownership, passes through permitted source, calls
  PromptImageTransformer.transform when needed, checks output MIME/dimensions.
  Spec bodies/models are meaningful, no separate images impl required.
- [ViewImageToolClient.kt:11–35](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/view-image/src/commonMain/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageToolClient.kt#L11)
  stores interface transformer defaulted to HostPromptImageTransformer;
  [codec I/O:23–28](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/codec/PromptImageCodecIo.kt#L23)
  calls shared processing. Host codecs implement the same images contract,
  not parallel image DTOs (codec implementation review belongs to other lane).
- Local tests cover data URLs/guards, signatures, dimensions/limits and
  [processing:14–77](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonTest/kotlin/io/github/stream29/kodex/utils/images/PromptImageProcessingTest.kt#L14).
  Processing suite covers pass-through/transform-required, not successful supplied
  transformer checks. Other-lane codec contract tests invoke real host transformers.
- Keep spec/transformer. U: defensive-copy assertions, malicious/mismatching
  transformer/cancel, operation @throws, actual full-decoder behavior/platforms.
  Header fixtures are not decoder acceptance.

#### kotlinx-io-coroutines — R interfaces/close, B2-PRIVATE

- Original `bfd5bea9`, suspend upgrade `c0f07637`; `6606efe9` separated contracts
  from default/platform implementations. Namespace
  `io.github.stream29.kodex.utils.kotlinxiocoroutines`.
- [Spec filesystem:29–67](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineFileSystem.kt#L29)
  is implemented by [blocking backend:11–63](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/blockingMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/BlockingCoroutineFileSystem.kt#L11),
  NodeCoroutineFileSystem and MingwCoroutineFileSystem. Actual
  SystemCoroutineFileSystem returns this interface on JVM/JS/POSIX/Windows;
  production tool/lease clients default/consume it.
- Public expect selection is assembly, not unrepresented filesystem capability.
  Internal dispatcher/timestamp/chmod expect/actuals implement declared operations.
  Two internal 64KiB constants are mirrored mechanism constants, not copied public
  model or parallel resource authority.
- [RawIo.kt:14–58,78–98](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineRawIo.kt#L14)
  owns real source/sink/close. use closes NonCancellable, suppresses close failure
  behind original failure/cancel. Node keeps uncancellable open handoff before
  eventual use/finally; blocking/Windows open within dispatcher and enter use.
- [Filesystem tests:40–152,158–203](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineFileSystemTest.kt#L40)
  exercise actual filesystem and raw use. JS/JVM/Linux descriptor tests inspect
  cancellation closure; fake close-failure tests appropriately check suppression,
  not platform descriptor ownership.
- Keep genuine I/O/context adapters. U: actual native/Node execution, private mode
  tests, per-operation @throws, fingerprint races/directory sentinel parity,
  Windows ACLs; B2-PRIVATE is not a native/JVM permission-regression claim.

#### kotlinx-io-serialization — R generated serialization is real consumption

- Original `e92f12fa`; unchanged move `6606efe9`.
  [PathAsStringSerializer.kt:11–21](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-serialization/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxioserialization/PathAsStringSerializer.kt#L11)
  defines `io.github.stream29.kodex.utils.kotlinxioserialization.PathAsStringSerializer`
  implementing `KSerializer<Path>`, descriptor `kotlinx.io.files.Path`/STRING, Path.toString
  encoding and Path(string) decoding.
- [McpSettings.kt:143–150](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpSettings.kt#L143)
  and OpenAI CompactionModels use @Serializable(with=...). Generated consumption
  counts; no manual invocation is necessary.
- [McpServerConfigurationTest.kt:10–38](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonTest/kotlin/io/github/stream29/kodex/mcp/contract/McpServerConfigurationTest.kt#L10)
  round-trips stdio containing `/workspace`; utility has no own test.
- Keep spec serializer, no impl needed. U: Windows/relative/Unicode/malformed paths,
  persisted OpenAI roundtrips and cross-platform execution.

#### ktor-client-ext — R HTTP/SSE; B2-SSE-DEPS, explicit platform U

- Original `7052788b`; custom Curl `f4be9653`, lifecycle/timeout revisions
  `329d8dc8`/`3a03e3e6`/`9fe4cd12`, response-header callback `4de3591e`.
  `6606efe9` moved existing code, not a new engine algorithm.
- Spec namespace `io.github.stream29.kodex.utils.ktorclientext` owns real
  HeadersBuilder set/addAll, wire names, SseCompatibility and HttpClient SSE Flow
  functions. Shared pipeline adapter body is actually in **spec**, contrary to the
  migration table's shorthand; it isn't an unused project interface.
- [SseRequests.kt:52–80](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/SseRequests.kt#L52)
  keeps official SSE inside channelFlow lifecycle, checks plugin/positive socket
  inactivity timeout and sets POST/expectSuccess.
  [SseCompatibility.kt:26–76](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/SseCompatibility.kt#L26)
  changes missing content type only on marked successful responses; Ktor owns
  event parsing/session lifecycle. Keep this real context/response adapter.
- Production [OpenAiClient.kt:126–142,370–387](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L126)
  installs SSE/compatibility, uses headers and shared streaming functions;
  impl dependency supplies host engine inclusion.
- [KodexCurl.kt:24–41](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurl.kt#L24)
  eagerly registers KodexCurl : HttpClientEngineFactory, creating
  KodexCurlClientEngine : HttpClientEngineBase with SSE/WebSocket/timeout capability.
  Bare HttpClient construction is meaningful framework assembly, not missing host use.
- [MultiApiHandler:30–44,144–168,201–229,314–400,458–472](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt#L30)
  owns headers/stable refs until cleanup, queues identity-token cancellation,
  disposes error/completed/cancelled transfers, checks packet-idle timeout,
  disables multiplexing/forbids cancelled connection reuse.
  [Processor:45–54,106–127](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlProcessor.kt#L45)
  disposes unsent headers/drains close. Static trace is not native race proof.
- Tests: spec header functions; moved JVM suite checks data/comment inactivity,
  request-deadline distinction and unsuccessful/wrong/headerless responses.
  [Linux tests:15–37](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxX64Test/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlDefaultEngineTest.kt#L15)
  assert actual engine/capability and unreachable-port error, not native SSE,
  cancellation/resource stress or WebSocket behavior.
- U: linked default-engine selection, other host engines, pre-holder failure
  injection, native unpause/callback races and transfer error classification.
  No production WebSocket host call was established in this lane.
  Inherited [WebSocketSession.kt:54–59,93–101](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlWebSocketSession.kt#L54)
  completes parent Job on close while outgoing child may wait on receive, cleanup
  running on completion; authorized open/close/cancel test should determine cleanup
  delay. Do not count WebSocket lifecycle accepted.
- Minimal action: relocate test dependencies first, retain real Ktor adapters;
  no HTTP/SSE marker interface is warranted.

#### logging — R real context/file adapters; D-LOG-UPSTREAM

- Original `37c25878`, scoped facade `a0926b95`; unchanged extraction `6606efe9`.
  FQCN namespace `io.github.stream29.kodex.utils.logging`, including
  `initializeLogging`, private `ScopedKLogger` and internal `KermitAppender`.
  [ScopedLogging.kt:12–92](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/logging/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/logging/ScopedLogging.kt#L12)
  owns KLogger.global/session/agent/tool, one delegate/derived payload,
  marker/gating/event-block override. ScopedKLogger : KLogger is a meaningful
  context adapter, not migration forwarding.
- [ApplicationLogging.kt:16–47,61–77](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/logging/impl/src/fileLoggingMain/kotlin/io/github/stream29/kodex/utils/logging/ApplicationLogging.kt#L16)
  initializes directory/DirectLoggerFactory/appender;
  KermitAppender : Appender maps severity/payload/marker/cause to file writer.
  Connection to spec is real emitted scoped events, not direct extension calls.
- [CLI Main.kt:23–28](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L23)
  initializes before services;
  [RuntimeComposition.kt:45,115,198](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L45)
  emits scoped Session/Agent/Tool events.
- ScopedLoggingTest checks inheritance/override/laziness/marker; actual
  [ApplicationLoggingTest.kt:26–102](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/logging/impl/src/fileLoggingTest/kotlin/io/github/stream29/kodex/utils/logging/ApplicationLoggingTest.kt#L26)
  initializes global config and reads file with scoped fields; writer lifetime
  is intentionally process-wide, not fixture-closeable.
- D-LOG-UPSTREAM: recorded kermit-io failure/rotation println limitation and explicit
  no-reimplementation decision:
  [logging task Details](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-07-22-add-application-file-logging.md#L17).
  This debt can affect stdout; historical success does not erase it.
- U: repeated initialization, writer replacement/flush/close, Windows lifecycle,
  bootstrap I/O contract only on concrete initializer. Minimal fact cleanup if
  required, not deleting real Appender/duplicating logger state.

#### patch — R single models/parser/result; B1-PATCH

- [PatchModels.kt:15–93](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/PatchModels.kt#L15)
  owns sole Patch/Hunk/chunks/affected result/error/serial names.
  [Parser:14–38](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/PatchParser.kt#L14)
  owns normalized text/environment/hunks and syntax @throws; matcher is a real
  pure function called by filesystem algorithm, not an unused spec.
- [Impl:31–104,107–183](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/PatchFileSystem.kt#L31)
  preserves order, source regularity checks, matching tolerance/anchors,
  write/move/delete and single result. Earlier writes remain after later failure,
  matching spec nontransactionality.
- Parser/matcher tests moved to spec, real filesystem/Rust behavior tests to impl.
  Clean-model impl dependencies are **test** edges, not new spec-main impl leakage.
  Actual production client/compatibility chain is B1-PATCH above.
- U: complete filesystem IOException/cancellation @throws/runtime wire coverage.
  Keep facts/algorithm; minimal ablation is compatibility extension, not model/parser.

#### read-write-mutex — R existing real interface/state; B2-MUTEX-OWNER

- Original `1728ae62`; interface existed before extraction, not newly manufactured.
  FQCN `io.github.stream29.kodex.utils.ReadWriteMutex`, nested State defined only
  in spec; private ReadWriteMutexImpl owns one StateFlow and real exclusion Mutexes.
- [SafeRw.kt:8–32](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/SafeRw.kt#L8)
  stores contract and uses withLock sessions.
  Constructor-style ReadWriteMutex() is legitimate interface-returning assembly.
- [InMemoryKodexAgentStorage.kt:117](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/inmemory/InMemoryKodexAgentStorage.kt#L117)
  and [McpClientImpl.kt:95](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt#L95)
  create actual mutex; CachedAgentStorage uses SafeRw.
- [ReadWriteMutexTest.kt:20–150](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/ReadWriteMutexTest.kt#L20)
  real readers/writer/wait/cancel/state; SafeRwTest real sessions/suspend/release
  on exception. Ownerless deterministic tests don't prove owner error/concurrency.
- Migration table says SafeRw contract in spec; actual concrete SafeRw remains
  impl with no claimed separate interface. Correct inventory wording, preserve
  guarded-container abstraction, don't copy a parallel SafeRw.

#### rpc-exception — R serialization and real restoration assembly

- Original `14bb1ced`; unchanged move `6606efe9`.
  Namespace `io.github.stream29.kodex.utils.rpcexception`.
  [RemoteException.kt:13–16](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/rpc-exception/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/rpcexception/RemoteException.kt#L13)
  owns sealed known failures/computed JSON message; NoMatchException, SessionNotActive,
  SessionNotFound and CacheNonceMismatch are fieldless single definitions.
- [RestoreRemoteException.kt:6–28](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/rpc-exception/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/rpcexception/RestoreRemoteException.kt#L6)
  restores known leading-brace JSON, preserves unknown/malformed/cancellation;
  doesn't retry or collect Flow.
- Actual [RestoringRpcClient.kt:20–35](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/rpc/client/RestoringRpcClient.kt#L20)
  handles immediate/upstream streaming failures and isolates downstream exceptions.
  Production [Application.kt:85](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L85)
  builds services from RestoringRpcClient(rawClient); server throws these errors.
  Keep this actual RPC adapter, not a migration bridge.
- [RestoreRemoteExceptionTest.kt:36–245](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/rpc-exception/spec/src/commonTest/kotlin/io/github/stream29/kodex/utils/rpcexception/RestoreRemoteExceptionTest.kt#L36)
  covers discriminator/cancel/no retry/unknown JSON/upstream-downstream, explicitly
  rejects removed GenerationMismatch name as compat alias. Flow fixtures aren't
  real transport execution.
- Keep spec-only vocabulary/restoration; U: transport/platform execution, full
  frontend retry policy (other lanes), structured operation @throws beyond prose.

#### search-index — R real interface and runtime production use

- Original `2f08da51`; existing SearchDocument/SearchIndex moved in `6606efe9`,
  factory/platform code to impl. Namespace `io.github.stream29.kodex.utils.searchindex`.
- [Spec:3–16](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/searchindex/SearchIndex.kt#L3)
  owns snapshot/operation;
  [factory:6–9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/searchindex/SearchIndexFactory.kt#L6)
  returns actual interface from platform assembly.
  [Lucene:16–58](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/impl/src/luceneMain/kotlin/io/github/stream29/kodex/utils/searchindex/PlatformSearchIndex.kt#L16)
  and [JS fallback:3–37](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/searchindex/PlatformSearchIndex.kt#L3)
  implement SearchIndex<T>; build selects Lucene JVM/native, fallback JS.
- [ToolSearchEngine.kt:11–35](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/tool-search/src/commonMain/kotlin/io/github/stream29/kodex/tool/toolsearch/ToolSearchEngine.kt#L11)
  stores interface and calls search; production
  [KodexAgentTools.kt:111–124](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentTools.kt#L111)
  rebuilds it on MCP catalog changes, not only tests.
- [SearchIndexTest.kt:10–18](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/searchindex/SearchIndexTest.kt#L10)
  invokes actual factory/interface for one match; tool suites cover conversion/results.
- U: constructors retain supplied List; mutation/clear could violate fixed snapshot.
  Actual caller supplies a fresh map, so no production mutation found. Authorized
  check: MutableList mutation after create on JVM/JS; minimal candidate copy list
  ownership, not another model. Lucene reader/analyzer/directory not explicitly
  closed on replacement; memory directory/GC make native leak claim unproven.
  Resource accounting/nonpositive limits/ranking parity remain U. Keep actual
  interface/factory; no artificial wrapper ablation indicated.

#### terminal-text — R pure facts; preserve renderer-local composition

- Original `ae6a0c89`; unchanged move `6606efe9`.
  [TerminalText.kt:6–63](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/terminal-text/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/terminaltext/TerminalText.kt#L6)
  owns `io.github.stream29.kodex.utils.terminaltext.TerminalCellSegment`, UTF-16
  grapheme offsets/width/fitting prefix/suffix.
- Production [TextInput.kt:449,516](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TextInput.kt#L449)
  uses segments;
  [CLI TerminalText.kt:7–75](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TerminalText.kt#L7)
  wraps/ellipsizes using shared facts. These meaningful local visual algorithms
  aren't duplicate segment authority or migration forward wrappers.
- [TerminalTextTest.kt:7–38](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/terminal-text/spec/src/commonTest/kotlin/io/github/stream29/kodex/utils/terminaltext/TerminalTextTest.kt#L7)
  actual CJK/combining/zero-width/emoji/offset/fitting/zero-width-budget functions.
- Keep fact source/local composition; no meaningful impl/resource lifecycle to
  invent. U: full Unicode tables/large inputs/every emulator/platform execution.

### Error-contract and final acceptance gates

- No public typealias/second DTO or State definition found in scoped trees.
  Private Curl pointer aliases and necessary resource/context/data adapters aren't
  hidden public compatibility APIs. New forward additions are B1-URL/PATCH;
  test-coordinate forwarding has a recorded exception.
- **Spec-authoritative @throws completeness is not accepted.** ChildScope illegal
  Job contexts; image URL/detection/transformation failures; raw-I/O bounds/size/
  close and filesystem failures; SSE plugin/timeout/transport errors; Mutex owner/
  deprecated operations; restored RPC failures and search invalid limits lack
  complete structured operation @throws. Accurate prose/exception-class KDoc
  does not satisfy [boundary checklist:5–8](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/spec-impl-module-boundaries.md#L5).
  Patch documents domain failures, not all underlying I/O. Lease factories/logging
  bootstrap remain concrete entries. Decide normative placement without violating
  approved factory/resource shapes or manufacturing interfaces.
- **Two confirmed boundary blockers, four suspected blockers, two recorded debts.**
  Every domain has a traced substantive path and explicit U gaps.
  This is not fourteen passed migrations; no repair/resumption authorization.
- Suggested review gates, for coordinator/user decision only:
  - Independently recheck B1-URL against URL decision/all type references.
  - Recheck B1-PATCH and cut over actual Tool client/test chain.
  - Resolve B2-SSE-DEPS before trusting the SSE test inventory.
  - Validate lease-loss, Node modes and Mutex owners separately; these predate
    relocation and must not be called migration-introduced regressions.
  - Review operation facts and native/JS/Windows/Curl/WebSocket gaps before any
    whole-utils acceptance.
- Only this child report edited with apply_patch. No temporary files created,
  source fixes, commits/pushes/branch changes, Gradle/IDE or shared device use.
