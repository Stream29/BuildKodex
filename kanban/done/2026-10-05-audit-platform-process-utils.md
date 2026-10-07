# REVIEW READY — platform/process utility reacceptance

# Task Tree

- `Trace real contracts, implementations, consumers and historical extraction`()
- `Review behavior, tests and meaningful ablation candidates`()
- `Publish evidence and unreviewed coverage`()

# Details

- Parent: [Spec/impl reacceptance](2026-10-05-reaccept-spec-impl-refactor.md).
- Own review scope: `utils/{shell-client,process-client,os-environment,kodex-home,images-codec}`.
- Report only in this file; source, other tasks and shared resources are read-only.
- Apply parent acceptance rules and pinned baseline; no independent fixes or builds.
- Status: REVIEW READY; static audit complete, not runtime/platform acceptance.
- Baseline verified at start and before report: `refactor/spec`,
  `821b66275aae2f0a9541fa2e9e03023d875c30b5`; inner worktree clean.
- Only this outer child task was edited. No source changes, builds, new tests,
  commits, branch changes, device operations, or IDE sync.

## Review report

### Verdict and method

- **Do not accept this five-domain batch as a completed spec cutover.**
  Four confirmed migration blockers, two pre-existing lifecycle suspicions,
  and two explicitly bounded debt items are listed below.
- Do not infer that all five production implementations are broken:
  the process command/session extraction and codec exception/I/O extraction
  are real; Kodex Home has a live implementation-to-caller chain.
- Reviewed tracked main/test source, build/source-set declarations, platform
  actuals and cinterop definitions, `git show`/`git diff`, and production
  assembly. Symbol searches were corroborated by declarations and callers.
- All five pairs and the added interfaces were introduced by
  **`6606efe93bfbb70d63bd93424f2f91385a0e08b4`** (`6606efe9`).
  `git log 6606efe9..HEAD --` the five domains contains no later scoped changes.
  `f1a90006` concerns the preceding Runtime/OpenAI/Unified Exec extraction,
  not implementation of these new utility interfaces.
- History was compared against `6606efe9^`, not the earlier task's
  “completed” labels. The [utility migration mapping][migration-map] and
  [normative boundary rules][boundary] define the promised result.
- URI links below refer to current pinned source with relevant line ranges.
  Historical evidence is identified as `git show <revision>:<old path>`;
  those removed paths are not represented as existing files.

### Full domain coverage

| Domain | Original declaration → current spec | Actual implementation/assembly → real caller | Existing tests reviewed | Classification |
| --- | --- | --- | --- | --- |
| `shell-client` | Old `ShellClient`, `ShellProcessCommand`, shell `ProcessSession`, `Shell`/serializer, `ShellSettings`, `StdoutBuffer` remain in impl. Added spec is a separate command/client/session trio. | Scope factory → JVM/Node/POSIX/MinGW `ShellClient`; pipes → direct ProcessClient, PTY → target backend. Unified Exec, Hook, notification and settings use the original API. | ProcessSession, UnicodeShell, StdinChannel, StdoutBuffer tests; real client paths, not new spec trio. | **B1-PU-01**, **D-PU-01**; production paths R; acquisition race B2-PU-01. |
| `process-client` | Real `ProcessCommand`, raw `ProcessSession`, `ProcessException` moved to spec. Added `ProcessClientSpec` has no implementation. | Scope factory returns expect/actual concrete `ProcessClient`; all four sessions implement spec `ProcessSession`. Shell, external URL and MCP use concrete launching API. | Three common I/O cases and target command fixtures; MCP JVM service and Node stdio tests. | **B1-PU-02**; moved models/session R; **B2-PU-01/02**. |
| `os-environment` | Old top-level query API remains impl-owned; new spec `OsEnvironment` duplicates its operations. | `DefaultOsEnvironment : OsEnvironment` delegates to actual functions, but production calls those functions directly. | One common test calls the old function for an absent variable. | **B1-PU-03**; platform query implementations R. |
| `kodex-home` | Old process-wide `KodexHome: Path` becomes getter → `DefaultKodexHomeProvider.path`; new spec `KodexHomeProvider.path`. | Default object really implements provider and initializes `Path(requireUserHomeDirectory(), ".kodex")`; CLI migration/logging and runtime artifacts reach it through getter. | No direct test of provider/getter in domain; migration tests belong to separate home-handle API, not provider integration proof. | **R-PU-04 + D-PU-02**, **not dead**; not a blanket spec-closure pass. |
| `images-codec` | Existing exceptions and `PromptImageCodecIo` move unchanged to spec; real transformer remains in `utils/images/spec`. Added `PromptImageCodec` is unused duplicate. | Host actual → ImageIO/Sharp/Skiko/Windows native; all implement real `PromptImageTransformer`; tool clients invoke moved `readPromptImage`. | Common host/KorIM capability tests and filesystem I/O tests. | **B1-PU-04**; real model/transformer/exception/I/O chain R. |

FQCN prefixes used below:

- `S = io.github.stream29.kodex.utils.shellclient`
- `P = io.github.stream29.kodex.utils.processclient`
- `E = io.github.stream29.kodex.utils.osenvironment`
- `H = io.github.stream29.kodex.utils.kodexhome`
- `I = io.github.stream29.kodex.utils.images`
- `C = io.github.stream29.kodex.utils.images.codec`

### B1 — confirmed migration blockers

#### B1-PU-01 — Shell spec is a parallel, disconnected, weaker API

- **Introducing commit:** `6606efe9`. Its diff moves every original Shell
  source/test to impl with 100% similarity and adds [ShellClientSpec.kt][s-spec].
  Earlier ownership history includes `a83e8040` (Shell execution),
  `b4ecfe8f` (scope-managed ShellClient), `5a9a02ca` (child-scope factory).
- **Added FQCNs:** `S.ShellCommandSpec`, `S.ShellSessionSpec`,
  `S.ShellClientSpec`. All tracked occurrences are in that definition file.
  None is `@Serializable`; no source generator/factory binds them.
  The [impl's Gradle api dependency][s-build] is not an implementation relation.
- **Production authority remains:** [S.ShellClient and scope factory][s-client]
  return the concrete expect/actual class; `start` accepts
  [S.ShellProcessCommand and returns S.ProcessSession][s-session].
  JVM, Node, POSIX and MinGW actual supertypes are `CoroutineScope` and
  `AutoCloseable`, not `ShellClientSpec`/`ShellSessionSpec`.
- **Concrete callers:** [UnifiedExecToolClient][unified-client], lines 35–56,
  owns `S.ShellClient`; its [argument conversion][unified-command] constructs
  `ShellProcessCommand(login = true, tty = tty, shell = explicit ?: settings)`.
  [ShellClientHook][hook-shell] launches the original command and reads
  `standardOutput`/`standardError`; [notification owner][notification] also
  uses the original client. Neither a DTO from Unified Exec nor a direct
  `P.ProcessCommand` substitutes for the shell contract.
- **Lost from the purported spec, still live in impl:**
  - `Shell` syntax/type selection, resolved default versus supplied path,
    `login`, `tty`, environment-key/NUL validation.
  - Merged output plus separately destructive stdout/stderr buffers;
    PTY has one stream, stderr buffer empty.
  - Session owner scope, cancellable child ownership, asynchronous
    tree-termination request versus external scope cancellation, preserved
    exit observation after `close`, and send acknowledgement after write.
  - Bounded head/tail snapshots, omission counts, cancellation-safe destructive
    reads and terminal-buffer behavior in [StdoutBuffer][s-buffer].
  - Observable failures: blank command/closed client → `S.ProcessException`;
    invalid environment → `IllegalArgumentException`; invalid shell JSON →
    `SerializationException`; closed stdin → `ClosedSendChannelException`;
    cancellation stays distinct from final platform exit status.
  - `stdin.onSend` actually throws `UnsupportedOperationException`
    ([StdinChannel, 71–84][s-stdin]). The sealed-select-clause constraint is an
    implementation mechanism; the unavailable public capability must still
    be stated on the real contract.
- **Pure model/serializer evidence:** [S.Shell and ShellType][s-model] encode
  the executable path as one JSON string, recognize a syntax from that path,
  and preserve explicit paths. [ExecCommandArguments.shell][unified-model]
  uses this model under generated Kotlin serialization, not `shellPath` from
  `ShellCommandSpec`. Settings use [ShellSettings/Shell][settings-shell].
  These real values/serializer can constitute spec without another interface.
  `Shell.default`/`resolve` are host-policy surfaces mixed into the same class,
  requiring an explicit separation decision, not a silent wire change.
- **Test limitation:** [ProcessSessionTest][s-tests] creates concrete clients
  (32–47), verifies shell string encoding (85–112), pipes/separate streams,
  environment/PTY (165–252), close versus cancel and owner Job behavior
  (291–375), blocked sends/EOF/concurrent sessions (407–466).
  [UnicodeShellTest][s-unicode] checks bytes and working directories for both
  pipe and PTY, plus preferred-path resolution. [Buffer tests][s-buffer-tests]
  and [channel tests][s-stdin-tests] exercise actual retention/write mechanics.
  None takes a new spec type, so those tests
  could all pass with the new spec file deleted.
- **Minimal candidate, not authorized work:** delete the three disconnected
  additions; establish one spec owning the real command/session/output/model
  API and normative KDoc. If retaining `ShellClientSpec`, its operation must
  use the real moved `ShellProcessCommand`/`ProcessSession`, and every actual
  client and consumer must use that boundary. Keep concrete PTY, resolution,
  cinterop and I/O mechanics in impl. Stop for a decision on `Shell.default`
  API placement while preserving serializer/FQCN and wire behavior; do not
  add a mutable global resolver or DTO bridge.

#### B1-PU-02 — direct process models are real, launching spec is not adopted

- **Introducing commit:** `6606efe9`. Compare
  `git show 6606efe9^:utils/process-client/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClient.kt`.
  Old lines 38–72 contain the command, session and exception actually moved.
  Origin is `ee358d20`; later `1fa926af` supplies Node support/current I/O shape.
- **Real extraction:** [P.ProcessCommand, P.ProcessSession and
  P.ProcessException][p-spec] retain actual names/types used by all target
  code. JVM/POSIX/Windows/Node session classes really implement this spec
  `ProcessSession`; command defaults/fields are not parallel DTOs.
- **Disconnected addition:** `P.ProcessClientSpec` has no implementing class.
  [Common factory][p-client] returns `P.ProcessClient`, and
  [JVM][p-jvm], [Node][p-node], [POSIX][p-posix], [Windows][p-windows]
  actual declarations implement only scope/close. Similar method signatures
  are not Kotlin nominal inheritance; expect/actual does not synthesize it.
- **Corroborated assembly:** [McpServiceImpl][mcp-service] owns a concrete
  `scope.ProcessClient()` (49) and [calls its original extension][mcp-service-transport]
  (213).
  [McpStdioTransport.kt][mcp-stdio] has original concrete extension (21–46)
  **and** new `DefaultMcpStdioTransportFactory`/private spec extension
  (48–80), duplicating command construction and transport setup.
  No production call reaches the new factory; its parameter cannot accept any
  current concrete client. This is a reference to an unimplemented contract,
  not proof it is used.
- **Other real paths:** [Shell pipe assembly][s-pipe], lines 28–47,
  converts shell invocation to the real `P.ProcessCommand` and wraps its raw
  spec session for text. [ExternalUrlProcess][url-process] owns a concrete
  client, awaits status, and closes session/client in `finally`.
- **Contract regression in the actual moved declarations:** old command KDoc
  specified `Path(".")` inheritance and environment overlay; new spec removes
  both property guarantees. Old `ProcessSession` documented reads/writes/
  flushes/closes not blocking the caller's dispatcher, tree termination on
  `close`, and eventual resulting status. New spec reduces this to
  “suspendable” operations/final status and no close contract. Client
  ownership/cascading close remains documented only on impl.
  “Transfers ownership” on the new `start` cannot alone explain that client
  cancellation still invalidates sessions/streams.
- **API errors versus mechanism:** closed client → `P.ProcessException`;
  Node blank executable → that exception; JVM rejected start wraps
  `IOException`; POSIX fork/pipe errors and Windows API errors use
  `ProcessException`. POSIX child `chdir`/exec failure can instead produce
  exit **127**, not a synchronous `start` exception
  ([spawn definition, 225–259][p-spawn]).
  Stream failures are not universally the same exception: Node/POSIX
  raw stream code uses `IOException`; Windows checks can throw
  `ProcessException`. Document conditions on the real operations, not a
  universal “all failures throw at start” promise.
- The moved exception also changes `public class` to `public open class`.
  That is an additional public extensibility delta, not needed for relocation;
  no runtime regression is inferred from it, but it needs deliberate retention
  or removal rather than being described as an unchanged declaration.
- **Test limitation:** [ProcessClientIoTest][p-tests] covers byte exchange with
  independent stdout/stderr, client-close producing nonzero status, and one
  environment override. Target fixtures explicitly launch `/bin/sh`,
  PowerShell, or Node; the process layer does not insert that shell.
  Tests never type a client as `ProcessClientSpec`, assert descendant death,
  or cancel the acquisition handoff.
  [JVM MCP test][mcp-jvm-test] exercises assembled service; [Node MCP test][mcp-node-test]
  invokes original concrete extension. Neither tests new factory integration.
- **Minimal candidate:** have the actual clients implement the meaningful
  launching spec, migrate factory/consumer static types, and consolidate MCP's
  duplicate paths around one live factory. Preserve the real moved model/
  raw-session API, restore guarantees/@throws, and retain host scope ownership.
  Alternatively delete the unimplemented interface and revisit the launching
  boundary; do not delete valid command/session spec to “fix” this.
  MCP source is outside this child's repair ownership.

#### B1-PU-03 — OsEnvironment wrapper exists, but production bypasses it

- **Introducing commit:** `6606efe9`; earlier query history includes
  `d0c6985e` and package rename `c59de796`. Compare old
  `utils/os-environment/src/commonMain/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironment.kt`
  at `6606efe9^`: all original top-level operations remain in
  [current impl][e-impl], with a delegating object added at 20–27.
- **Spec/implementation FQCNs:** [E.OsEnvironment][e-spec] really is implemented
  by `E.DefaultOsEnvironment`. It is **not unimplemented**.
  But no production query invokes that object or accepts `OsEnvironment`.
  The old public `environmentVariable`, `userHomeDirectory`, `processId`,
  and `requireUserHomeDirectory` remain the caller-facing authority in impl.
- **Positive caller evidence:** [ShellResolution][s-resolution] calls
  `environmentVariable` for SHELL/PATH/COMSPEC/PATHEXT;
  [FileSystemLeaseImpl][lease-pid] calls `processId` for heartbeat;
  [SystemDirectoryPickerBrowser][picker-home] defaults its nullable path from
  `userHomeDirectory`; [AgentContextPrefixResolver][context-home] expands `~`
  via `requireUserHomeDirectory`; Kodex Home also calls that impl function.
  Thus deleting wrapper/spec would leave the current host-query call chain
  unchanged. No serializer or platform actual binds these calls to the object.
- **Real capability:** [JVM actual][e-jvm] uses `System.getenv`,
  nonblank `user.home` then environment fallback, and `ProcessHandle.pid`.
  [Node actual][e-node] uses runtime globals `process.env`/`process.pid`.
  [Native actual][e-native] uses `getenv`/`getpid` including MinGW source set.
  Common home fallback chooses nonblank USERPROFILE, then HOME, then
  HOMEDRIVE+HOMEPATH. This detects a path, not existence/readability/creation.
  The require operation throws `IllegalStateException` if unavailable; it has
  no spec counterpart or spec `@throws`. OS-specific discovery precedence is
  mechanism; nullability and required-home failure are caller-visible contract.
- **Test limitation:** [OsEnvironmentTest][e-test] only asserts an absent
  variable returns null through original function. It proves neither wrapper
  adoption, home fallback/exception, nor PID/Windows Unicode behavior.
- **Minimal candidate:** wire real callers to one `OsEnvironment` instance
  and keep platform query functions as internal backend mechanism; place a
  meaningful required-home helper and its failure contract on that boundary.
  Preserve fallback and observable results. Or explicitly reconsider a
  spec-only functional API grouping. Do not keep a public duplicate API just
  to give the new interface an implementation count.

#### B1-PU-04 — PromptImageCodec is redundant; actual image contracts are valid

- **Introducing commit:** `6606efe9`; original codec/host-transformer and
  prompt-image model history includes `d0b057a7`.
  The diff moves exceptions/I/O code and implementations unchanged, adds
  [C.PromptImageCodec][c-added], and leaves the real transformer in
  [I.PromptImageTransformer][i-models].
- `PromptImageCodec.transform` duplicates the real transform operation over
  the **same** `PromptImageTransformRequest`/`EncodedImage`. No platform object,
  host factory, test, or caller implements/uses it; no annotations/code
  generation turn it into the real interface.
- **Real assembly:** [C.HostPromptImageTransformer][c-host] has type
  `I.PromptImageTransformer`; actual values select
  `C.JvmImageIoPromptImageTransformer`, `C.SharpPromptImageTransformer`,
  `C.SkikoPromptImageTransformer`, or `C.WindowsNativePromptImageTransformer`.
  Each implements that actual spec interface; Windows delegates to GDI+,
  falls back to KorIM, and preserves the native failure as suppressed.
- **Real production:** runtime [tool assembly][runtime-tools] creates
  ViewImage/ImageGeneration clients using defaults;
  [ViewImageToolClient][view-image] and [ImageGenerationToolClient][image-gen]
  pass `HostPromptImageTransformer` to [moved readPromptImage][c-io].
  `I.ByteArray.toPromptImage` invokes `transformer.transform`, checks output
  MIME/dimensions, or preserves original bytes without invoking a transformer.
- **Meaningful spec retained:** `C.ImageCodecException` and
  `C.UnsupportedImageCodecException` are thrown by real codecs and moved to
  [codec spec][c-exceptions]. `readPromptImage`/`writeEncodedImage` are reusable
  algorithms over the filesystem spec, with no default filesystem/host codec
  imported. `EncodedImage`/request defensively copy bytes; plan/mode and data
  URL behavior are real pure contracts. Do **not** invent interfaces for them
  or call the entire images-codec spec empty.
- **Normative KDoc failure:** the added interface has explicit codec `@throws`,
  but the implemented `PromptImageTransformer.transform` has none.
  The attractive documentation is attached to a dead substitute.
  Actual exceptions must be documented on the live boundary without
  creating an `images-spec ↔ images-codec-spec` dependency cycle.
- **Capabilities/errors/ownership:** see platform matrix below.
  Business metadata detection/policy failures and a required transformation
  without a transformer are not codec backend failures.
  Output MIME/dimension mismatch throws `IllegalArgumentException` in the
  pure algorithm. Unsupported encode/decode and failed codec operation are
  distinguished by real exception classes; implementation-specific native
  messages, JPEG quality, interpolation, plugin discovery and COM/Skia object
  cleanup are mechanisms, not replacements for transform semantics.
- **Test limitation:** [transformer contract tests][c-tests] actually exercise
  host and KorIM: PNG resize, GIF→PNG, host JPEG decode/resize/encode with
  detected MIME/dimensions. KorIM is deliberately not in JPEG suite.
  [codec I/O tests][c-io-test] cover unchanged-byte read and a fake **real**
  `PromptImageTransformer`; this latter case proves algorithm delegation, not
  target codec support. Neither test references added `PromptImageCodec`;
  corruption, platform resource-release failure and cancellation cases are
  not covered here. No new test execution or native-host validation occurred.
- **Minimal deletion candidate:** delete only `ImageCodecSpec.kt`'s unused
  interface; keep the actual transformer, pure models, used exceptions and
  filesystem extension spec. Consolidate normative transform/error docs onto
  the real boundary with an explicit domain-placement decision. No adapter
  between the two interfaces is warranted.

### Platform and lifecycle evidence matrix — reviewed paths (R)

| Path | True capability and ownership at pinned source | Evidence / migration consequence |
| --- | --- | --- |
| Shell pipes, all host targets | Explicit shell invocation → direct executable raw process; shared text session wraps raw streams, owns a child Job, buffers merged/separate output. Close requests termination, observer settles output and status; external scope cancellation aborts. | [PipeProcessSession][s-pipe], 28–115, 132–157, 215–230. Raw and shell `ProcessSession` are different FQCNs and valid layer boundaries, not duplicated DTO authority. |
| Shell JVM PTY | pty4j with environment overlay, login syntax, 80×24 initialization and ConPTY option. Session child Job; close requests tree termination; completion waits output readers, flushes buffers and finishes resources in NonCancellable cleanup. | [JvmShellClient][s-jvm], 26–73, 76–188, 219–240. Added Shell spec exposes none of PTY/output/owner semantics. |
| Shell Node PTY | Lazy external `require("node-pty")`; subscriptions pause/resume output and report exit. NodeProcessSession owns pending callback Jobs, ordered writes, termination/release. Stdin close does not close PTY master; explicit terminal EOF input differs from pipe EOF. | [NodeShellClient][s-node], 35–45, 79–188; [NodeProcessSession][s-node-session], 21–74, 109–200. POSIX group kill or Windows taskkill/fallback is mechanism. |
| Shell POSIX PTY | `forkpty`/`execvp`; cwd injected into shell invocation, environment inherited/overlaid, nonblocking master and polling/waitpid lifecycle. Group termination, output flush, NonCancellable cleanup. | [PosixShellClient][s-posix], 58–139, 240–340; [Shell.posix][s-posix-invocation], 27–44; [PTY cinterop][s-spawn], 66–108. `forkpty` passes null winsize: do not claim all backends explicitly initialize 80×24 merely from constants' comment. |
| Shell MinGW PTY | Dynamically checks Windows Create/ClosePseudoConsole availability, borrows input/output handles until pseudo-console close, creates suspended child, attaches Job Object before resume. Unicode command/env/cwd APIs; one combined PTY output. | [MingwShellClient][s-windows], 65–166, 253–349; [ConPTY definition][s-conpty], 84–87, 109–131, 173–235. Unavailable OS capability becomes ProcessException at start; no silent pipe fallback. |
| Direct process JVM | ProcessBuilder, ordinary independent pipes, blocking I/O moved to IO dispatcher. Session guard tied to client. Close is idempotent, can synchronously wait/force descendants, then closes streams and settles status. | [JvmProcessClient][p-jvm], 20–97. Do not falsely promise nonblocking **session** close because stream close is suspendable. |
| Direct process Node | `spawn(shell = false)`, detached POSIX group; raw sources one-chunk backpressure, mutex-serialized sink callbacks; cancellable start has explicit cancellation handler. Close schedules termination in independent cleanup scope, observer settles final status. | [NodeProcessClient][p-node], 57–191, 219–297, 299–379. Cleanup ownership concern is B2-PU-02, not newly introduced by spec move. |
| Direct process POSIX | Pipes with close-on-exec, environment merge and executable lookup in cinterop, fork/process group/execve, IO-dispatched raw streams, group/direct SIGKILL and waitpid status. | [PosixProcessClient][p-posix], 53–120, 164–237; [spawn definition][p-spawn], 32–63, 112–163, 225–259. Child exec/cwd failure is status 127. |
| Direct process MinGW | CreateProcessW with Unicode environment/cwd, quoted args; parent handles noninheritable; suspended child attached to Job Object before resume. Guard closes streams and terminates job; observer closes process/job handles. | [WindowsProcessClient][p-windows], 69–191, 224–290, 380–438. Stream ownership is shared with session teardown, not perpetual independent ownership after client close. |
| Image JVM | ImageIO (+ JPEG plugin), PNG/JPEG output, GIF decoding for PNG conversion. Graphics/writer disposed in finally; IO-dispatched work. GIF output unsupported. | [ImageIO transformer][c-jvm], 19–45, 47–91. IOException decode/encode → ImageCodecException; missing encoder → UnsupportedImageCodecException. |
| Image Node | Sharp resize, PNG/JPEG output, GIF→PNG input business path. Promise await preserves coroutine cancellation; transform rejection wraps ImageCodecException. | [Sharp transformer][c-node], 18–57. No native-close handle escapes; no claim cancellation kills underlying Sharp work. |
| Image Linux/macOS native | Skiko decode/raster/draw/encode PNG/JPEG; source, paint, image snapshot, encoded data, surface closed in nested finally. | [Skiko transformer][c-skiko], 19–89, 108–113. GIF output unsupported; supported input depends on decoder, exercised by host GIF test. |
| Image MinGW | GDI+ PNG/JPEG decode+encode preferred; GIF conversion uses KorIM fallback. Native codec failures can also trigger fallback; failures retain suppressed native cause. COM streams/global-memory ownership and GDI+ lifecycle released in finally. | [GDI+/WindowsNative][c-windows], 71–154, 168–195, 204–262. KorIM only PNG/GIF decode, PNG encode ([KorIM][c-korim], 19–41, 53–67); JPEG fallback is not magically supported. |
| Kodex Home | Path provider only, no create/scan/migrate/close resource. Object computes one default home path; getter reads that implemented property. CLI prepares Home before logging/application; artifacts still use default process path. | [H.DefaultKodexHomeProvider/KodexHome][h-impl], [CLI main][cli-home], [runtime tools][runtime-tools]. No replacement lifecycle handle or second root is introduced. |

Host implementations target JVM, Node.js, Linux x64/arm64, macOS arm64 and
MinGW x64 ([host convention][host-convention], [target configuration][host-targets]).
Shared spec convention also publishes browser/wasm/iOS targets
([shared convention][shared-targets]); publishing a pure spec there is not
evidence a process/codec host backend exists there. No such backend is claimed.
Project discovery is real recursive Gradle inclusion ([settings][gradle-settings],
34–48, 63), but discovery/dependency presence alone does not establish adoption.

### B2 — suspected lifecycle blockers; no runtime claim

#### B2-PU-01 — cancelled acquisition may retain an unreturned process/session

- JVM/POSIX/MinGW direct `start` creates and returns an owned session inside
  `withContext(IO)` ([p-jvm][p-jvm], [p-posix][p-posix], [p-windows][p-windows]).
  JVM/POSIX/MinGW PTY start similarly returns resources across a dispatcher
  handoff ([s-jvm][s-jvm], [s-posix][s-posix], [s-windows][s-windows]).
- Session ownership is attached to the longer-lived client, not necessarily
  the calling operation Job. Source has no explicit close-on-cancelled-return
  handoff. If acquisition finishes but cancellation prevents receipt, caller
  cannot close the handle while the client can remain active.
- Node direct start differs: its continuation has `invokeOnCancellation`
  (114–116). Do not label it the same missing-handler path.
- **History:** risk predates `6606efe9`; platform files moved unchanged.
  Direct clients originated in `ee358d20`, not in the new interface.
- **Needed reproducer, not run:** independently owned live client; cancel only
  the launching operation at controlled post-spawn/pre-return dispatcher
  points; check no unreturned child, descendants or pipe/PTY handles remain
  and owner completion includes cleanup. Test each native OS/JVM backend
  without closing the client first. Existing “client.close terminates child”
  tests would mask this risk.

#### B2-PU-02 — Node direct close has fire-and-forget cleanup ownership

- `NodeProcessSession` creates `CoroutineScope(SupervisorJob() +
  Dispatchers.Default)` without parent (133), and `close` cancels the client
  guard then launches termination there (150–155, 171–190).
  No `cleanupScope.cancel`/join or owner-child cleanup registration appears.
- Current process spec lacks a waitable client/session close guarantee.
  A parent/client Job may finish before that cleanup Job or child tree exits;
  a post-close observer event is not proof that parent completion waits.
  This is a concrete lifecycle risk, not proof of a leaked OS child.
- **History:** Node implementation introduced in `1fa926af`, moved unchanged
  in `6606efe9`. Existing resource lifecycle checklist requires structured,
  waitable release ([lifecycle policy][lifecycle]); no exemption found.
- **Needed reproducer, not run:** Node POSIX group and Windows taskkill
  branches; start a child with descendants, close/cancel client, await its
  owner Job, immediately verify process tree and pending cleanup Jobs;
  instrument delayed termination/failure and require cleanup completion before
  owner completes. Existing raw-process test only awaits nonzero exit status.

### D — documented debt with bounded authorization

#### D-PU-01 — legacy Shell mixed-project edges remain explicitly deferred

- [Unified Exec spec build][unified-build] imports `utils-shell-client-impl`
  because its real `Shell` type is still there. Agent Context/session/settings
  also expose `Shell`/`ShellSettings`.
- [Parent planning boundary][legacy-shell] explicitly records preservation
  of this mixed edge and serializer shape. [Subsequent component record][legacy-closure]
  records `clean-models → unified-exec spec → shell impl → os-environment impl`
  as existing closure debt, not new component failure.
- Authorization is to **preserve/report**, not claim final graph purity or
  treat Shell's unused spec as solving it. B1-PU-01 remains a blocker against
  the “Shell boundary completed” claim.

#### R-PU-04 + D-PU-02 — Kodex Home is live, residual default-path facade

- `git show 6606efe9^:utils/kodex-home/src/commonMain/kotlin/io/github/stream29/kodex/utils/kodexhome/KodexHome.kt`
  contains original `KodexHome = Path(requireUserHomeDirectory(), ".kodex")`.
  The migration retains that computation in `H.DefaultKodexHomeProvider`,
  implements [H.KodexHomeProvider.path][h-spec], and makes `H.KodexHome`
  read the object property ([h-impl][h-impl], 7–13).
- [CLI main][cli-home] (14, 24) and [runtime image artifacts][runtime-tools]
  (91) reach that actual spec property through the getter. Therefore
  absence of interface-typed injection is **not** sufficient to call it dead,
  and there is no alternate DTO/root/state store.
- The old impl-facing accessor remains, explicitly labelled a compatibility
  projection. Its default-path role is authorized by [Kodex Home policy][home-policy]
  (7–10): default process root for logging/artifacts, distinct from a prepared
  per-application handle. The [migration mapping][migration-map] also calls
  for a stable path contract plus host-derived implementation.
- That authority supports default composition, **not** a general permission
  for arbitrary compatibility wrappers. Do not count this as proof consumers
  depend only on spec or that the provider is replaceable throughout the app.
  Its spec does not fully state stable identity/no-directory-effects or the
  default unavailable-home failure (impl initializer calls throwing helper).
  These code-level documentation/boundary gaps remain to resolve.
- **Test limitation:** no direct provider/getter tests found, and no new
  initialization-failure/platform tests run. [Home migration tests][home-tests]
  operate on supplied temporary paths/handles; they cannot establish adoption or failure
  semantics of this process-wide provider.
- **Minimal candidate:** after a separate public-API decision, cut the few
  callers over to one spec-facing provider/path access and remove the old
  accessor, or reconsider whether a provider is necessary for this stable
  value domain. Merely renaming the facade would not establish a boundary.
  Preserve default logging/artifact policy and separate prepared Home handle
  ownership. Deleting the current default object alone is **not** a no-effect
  ablation: it is on the live path.

### R / U and coordinator handoff

- **R:** all five domains have original→spec→implementation→caller→test
  traces; host actual selection, concrete process/codec mechanisms, Shell
  serialization and actual pure image/raw-I/O contracts reviewed as above.
- **U:** runtime execution on all platforms; generated build outputs/binaries
  (may be stale, not trusted as pinned evidence); stress/failure-injection
  cleanup cases; full native handle correctness and external codec dependency
  behavior; downstream RPC/storage round trips and UI behaviors beyond the
  scoped caller evidence. No U is counted as passing.
- **Deletion priority:** Shell's disconnected trio; image-codec duplicate
  interface; process-client/MCP shadow launching path after selecting real
  boundary; OsEnvironment wrapper/bypass after choosing single authority.
  These are proposals, not permission to modify or resume migration.
- Keep meaningful process models/session spec, codec exception/I/O spec, real
  image transformer/pure models, and the live Kodex Home default chain.
- Coordinator can read immediately; this report does not wait for
  consolidation. Checkpoint findings have been superseded by this final report.

### Evidence links

[boundary]: file:///home/stream/ACodeSpace/push/BuildKodex/checklist/spec-impl-module-boundaries.md#L3-L11
[migration-map]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-09-30-migrate-all-utils-to-spec-impl.md#L104-L124
[legacy-shell]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-09-28-plan-domain-spec-impl-boundaries.md#L212-L219
[legacy-closure]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-02-extract-pending-tool-and-resource-components.md#L101-L108
[home-policy]: file:///home/stream/ACodeSpace/push/BuildKodex/checklist/kodex-home.md#L5-L13
[lifecycle]: file:///home/stream/ACodeSpace/push/BuildKodex/checklist/coroutine-resource-lifecycle.md#L3-L11
[s-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ShellClientSpec.kt#L7-L26
[s-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/build.gradle.kts#L32-L78
[s-client]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ShellClient.kt#L8-L33
[s-session]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ProcessSession.kt#L8-L99
[s-model]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/Shell.kt#L13-L84
[s-buffer]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/StdoutBuffer.kt#L25-L123
[s-stdin]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/StdinChannel.kt#L58-L84
[s-resolution]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ShellResolution.kt#L6-L38
[s-pipe]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/processClientMain/kotlin/io/github/stream29/kodex/utils/shellclient/PipeProcessSession.kt#L28-L230
[s-jvm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/shellclient/JvmShellClient.kt#L26-L240
[s-node]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/shellclient/NodeShellClient.kt#L35-L188
[s-node-session]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/shellclient/NodeProcessSession.kt#L21-L200
[s-posix]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/posixMain/kotlin/io/github/stream29/kodex/utils/shellclient/PosixShellClient.kt#L58-L340
[s-posix-invocation]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/posixMain/kotlin/io/github/stream29/kodex/utils/shellclient/Shell.posix.kt#L27-L44
[s-spawn]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/nativeInterop/cinterop/shell_client_spawn.def#L66-L108
[s-windows]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/mingwX64Main/kotlin/io/github/stream29/kodex/utils/shellclient/MingwShellClient.kt#L65-L349
[s-conpty]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/nativeInterop/cinterop/shell_client_conpty.def#L84-L235
[s-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/shellclient/ProcessSessionTest.kt#L32-L466
[s-unicode]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/shellclient/UnicodeShellTest.kt#L48-L103
[s-buffer-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/shellclient/StdoutBufferTest.kt#L15-L126
[s-stdin-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/shellclient/StdinChannelTest.kt#L15-L104
[unified-client]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolClient.kt#L35-L107
[unified-command]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolClient.kt#L248-L264
[unified-model]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecModels.kt#L16-L42
[unified-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/unified-exec/build.gradle.kts#L5-L12
[hook-shell]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/hooks/src/commonMain/kotlin/io/github/stream29/kodex/hook/impl/ShellClientHook.kt#L25-L83
[notification]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/shared/notification/src/commonMain/kotlin/io/github/stream29/kodex/cli/notification/NotificationHooks.kt#L34-L70
[settings-shell]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/shared/settings/contract/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/KodexGlobalSettings.kt#L12-L43
[p-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClientSpec.kt#L8-L43
[p-client]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClient.kt#L12-L46
[p-jvm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/processclient/JvmProcessClient.kt#L20-L97
[p-node]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/processclient/NodeProcessClient.kt#L57-L409
[p-posix]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/posixMain/kotlin/io/github/stream29/kodex/utils/processclient/PosixProcessClient.kt#L53-L237
[p-windows]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/mingwX64Main/kotlin/io/github/stream29/kodex/utils/processclient/WindowsProcessClient.kt#L69-L438
[p-spawn]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/nativeInterop/cinterop/process_client_spawn.def#L32-L259
[p-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClientIoTest.kt#L18-L74
[url-process]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/externalurl/ExternalUrlProcess.kt#L9-L34
[mcp-service]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt#L41-L66
[mcp-service-transport]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt#L206-L214
[mcp-stdio]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/stdio/src/commonMain/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransport.kt#L21-L80
[mcp-jvm-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/stdio/src/jvmTest/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransportIoTest.kt#L29-L85
[mcp-node-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/stdio/src/jsTest/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransportIoTest.js.kt#L23-L53
[e-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironmentSpec.kt#L5-L20
[e-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironment.kt#L5-L39
[e-jvm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironment.jvm.kt#L5-L15
[e-node]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironment.js.kt#L5-L12
[e-native]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/impl/src/nativeMain/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironment.native.kt#L9-L17
[e-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironmentTest.kt#L9-L13
[lease-pid]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseImpl.kt#L83-L94
[picker-home]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/pathpicker/DirectoryPickerBrowser.kt#L10-L24
[context-home]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/impl/AgentContextPrefixResolver.kt#L92-L99
[h-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kodex-home/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kodexhome/KodexHomeSpec.kt#L5-L14
[h-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kodex-home/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/kodexhome/KodexHome.kt#L6-L13
[home-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigrationTest.kt#L14-L41
[cli-home]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L11-L31
[runtime-tools]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentTools.kt#L77-L95
[i-models]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/PromptImageProcessing.kt#L19-L105
[c-added]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/codec/ImageCodecSpec.kt#L6-L20
[c-exceptions]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/codec/ImageCodecExceptions.kt#L5-L13
[c-io]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/codec/PromptImageCodecIo.kt#L11-L39
[c-host]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/codec/HostPromptImageTransformer.kt#L3-L8
[c-korim]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/codec/KorimPromptImageTransformer.kt#L14-L67
[c-jvm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/images/codec/JvmImageIoPromptImageTransformer.kt#L19-L109
[c-node]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/images/codec/SharpPromptImageTransformer.kt#L18-L57
[c-skiko]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/skikoNativeMain/kotlin/io/github/stream29/kodex/utils/images/codec/SkikoPromptImageTransformer.kt#L19-L113
[c-windows]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/mingwX64Main/kotlin/io/github/stream29/kodex/utils/images/codec/GdiPlusPromptImageTransformer.kt#L71-L310
[c-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/images/codec/PromptImageTransformerContract.kt#L14-L66
[c-io-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/images/codec/PromptImageCodecIoTest.kt#L34-L74
[view-image]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/view-image/src/commonMain/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageToolClient.kt#L11-L39
[image-gen]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationToolClient.kt#L20-L106
[host-convention]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-host.gradle.kts#L9-L23
[host-targets]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt#L22-L33
[shared-targets]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-shared.gradle.kts#L22-L41
[gradle-settings]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L34-L63
