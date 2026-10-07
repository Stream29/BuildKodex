REVIEW READY

# Task Tree

- `Verify the pinned source baseline`()
- `Trace every assigned platform contract, actual and production consumer`()
- `Review lifecycle, signature compatibility and test evidence`()
- `Publish the independently evidenced report`()

# Details

## Baseline and verdict

- Independently reviewed `6b7129fa0e719dde8f88a0034b9d572679984a03`.
  Start and pre-publication HEAD checks matched; inner `git status --porcelain=v1`
  was empty on both checks. No evidence from another HEAD is merged.
- Scope: exactly nine actual build projects below; tracked source, native cinterop,
  history, tests and relevant production consumers read. Existing ignored build
  artifacts are **not** treated as fresh test results.
- Only this outer report changed. No source changes, builds, runtime experiments,
  IDE/device/network operations, commits or resource management were performed.
- **Physical split and real Shell/Process/OS cutover are confirmed statically.
  “All migration fully accepted” is not supported:** P1/P2 contain current
  contract/lifecycle conflicts; P3 is a definite documentation gate failure,
  not a demonstrated image-processing regression.
- Findings below are inherited behavior/documentation or explicit boundary debt,
  **not newly proved migration regressions**. Moving an old guarantee into spec
  does not make its implementation conform. Nine project grades are 2 R, 2 D,
  5 B1; shared findings are not five independent defects.
- R means reviewed static paths, not execution certification. B1 documentation
  conflicts are explicitly separated from B1 lifecycle implementation defects.
  B2 experiments remain unrun. No U project remains, but runtime/platform paths
  remain unvalidated as listed below.

## Per-project coverage

| Actual Gradle project | Grade | Original → present authority / real implementation / production types | History and limits |
| --- | --- | --- | --- |
| `:utils-shell-client-spec` | **B1** P1 | Original `shellclient.Shell`, `ShellType`, `ShellSettings`, `ShellProcessCommand`, `ProcessSession`, `ProcessException`, `StdoutBuffer`, `StdoutBufferSnapshot`, and concrete expect `ShellClient` now have their real model/operation declarations here. [Shell model:19–69][shell-model], [command/session:16–111][shell-session], [client:13–29][shell-contract], [buffer:17–84][buffer-spec]. `UnifiedExecToolClient.shellClient` and its managed session have these real static types, not replacement `*Spec` types ([38–57, 201–219][unified]). | Initial mechanical split `6606efe9`; real cutover `eff7b10e`. P1 inherited guarantee remains. `ShellModelContractTest:13–44` tests real serializer, explicit command/defaults/environment validation and snapshot accounting; no running platform in this spec test. |
| `:utils-shell-client-impl` | **B1** P1; B2 Q1/Q2 | [Scope factory:8–26][shell-factory] returns `io.github.stream29.kodex.utils.shellclient.ShellClient`; internal `PlatformShellClient` actuals implement it on JVM/Node/POSIX/MinGW. Pipe path genuinely composes direct `ProcessClient` ([28–59][pipe]); PTY paths use pty4j, node-pty, forkpty or ConPTY, not a forwarding host. [Production factory:190–199][unified], agent tools [51–68][agent-tools], notification [52–68][notification]. `default`/`resolve` are host-selection extensions, not spec serializers. | `eff7b10e` replaces concrete expect API with real spec and internal actuals; algorithms largely predate migration. Factory test verifies child Job and closed-client rejection; real shell tests cover pipes, PTY, input, EOF, Unicode, cancellation and final status, but not P1/Q1/Q2 failure matrices. |
| `:utils-process-client-spec` | **B1** P2 | Original `io.github.stream29.kodex.utils.processclient.ProcessCommand`, raw `ProcessSession`, `ProcessException`, concrete expect `ProcessClient` become [real contract:10–69][process-spec]. Distinct from shell's same-simple-name session/exception. `McpServiceImpl.processClient: ProcessClient` ([41–48][mcp-service]) → `ProcessClient.openMcpStdioTransport` → raw spec session/streams ([30–49][mcp-stdio]); shell pipe layer uses these same types. | `6606efe9` initially added unused `ProcessClientSpec`; `eff7b10e` removed it and moved actual declarations. New ownership prose at 22–26 is not met by Node close. Factory/real-child tests reside in impl; no direct spec test source in this project. |
| `:utils-process-client-impl` | **B1** P2; B2 Q1/Q3 | [Scope factory:12–40][process-factory] returns real spec type. Internal `PlatformProcessClient` actuals: [JVM:20–97][process-jvm], [POSIX:53–237][process-posix], [MinGW:69–290][process-win], [Node:57–175][process-node]. Blocking raw adapters shift operations to IO; Node uses callbacks/backpressure. Actual production: MCP service, shell pipe, external URL [12–32][external-url]. | `eff7b10e` cutover; `e89c5094` repaired cancellation-guard installation order, **not** detached Node cleanup. `ProcessClientIoTest:25–68` checks bytes, separate streams, environment and close → nonzero exit; factory test checks independent Job. No owner-join/resource-release or startup-cancel race assertion. |
| `:utils-os-environment-spec` | **R** | Original `io.github.stream29.kodex.utils.osenvironment.environmentVariable`, `userHomeDirectory`, `processId`, `requireUserHomeDirectory` are the [actual expect/common API:9–35][os-spec]. JVM actual uses `System.getenv`, `user.home` then env, `ProcessHandle`; JS actual uses Node globals; native actual uses `getenv/getpid`. Production Home [7–13][home-impl], shell resolution [50–55, 116–139][shell-resolution], filesystem-lease owner IDs consume these functions directly. No OS service wrapper remains. | `eff7b10e` removed fake `OsEnvironment`/`DefaultOsEnvironment` and moved actual sources into spec. Legitimate spec-only platform functions. Only local test asserts missing env → null; home precedence, absent-home exception, Unicode and PID are not runtime-verified. |
| `:utils-kodex-home-spec` | **D** D1 | `io.github.stream29.kodex.utils.kodexhome.KodexHomeProvider.path` is a meaningful Path contract ([11–14][home-spec]), but original `KodexHome: Path` did **not** move here. The provider is referenced only by its impl object; production consumers accept `kotlinx.io.files.Path`, not a provider. This is boundary debt, not a reason to invent DI. | Provider introduced by `6606efe9`, unchanged thereafter. No local tests. Root policy and original declaration remain in impl; D1 explains why this differs from real Shell/Process operation cutover. |
| `:utils-kodex-home-impl` | **D** D1 | Original process-wide `KodexHome` initializer is now [DefaultKodexHomeProvider.path plus compatibility getter:7–13][home-impl]. Actual CLI [14, 24][cli-main] consumes the getter; agent image-output assembly [91][agent-tools] also consumes it. One singleton path, not two competing home owners or a hidden lease implementation. | `6606efe9` changed top-level initialized val to object-backed getter. OS dependency updated by `eff7b10e`. No new root-path/local initialization tests; downstream Home-handle tests do not test this default-host discovery chain. |
| `:utils-images-codec-spec` | **B1** P3 (documentation only) | Original `CoroutineFileSystem.readPromptImage` overloads and `writeEncodedImage` moved intact [14–39][codec-io]; original `ImageCodecException` and `UnsupportedImageCodecException` also moved intact [5–13][codec-errors]. Real transformer operation authority is **already** `io.github.stream29.kodex.utils.images.PromptImageTransformer` in images spec ([49–69][image-contract]), not an interface manufactured for this pair. Production ViewImage [14, 35][view-image] and image-generation [24, 105][image-generation]. | `6606efe9` moves original IO/exceptions; `e89c5094` deletes unused `PromptImageCodec` and documents the real transformer. P3 missing `@throws` inherited. Pure IO glue/spec functions are legitimate; no extra CodecService required. |
| `:utils-images-codec-impl` | **R** (negative-path tests limited) | `HostPromptImageTransformer: PromptImageTransformer` expect/actual is host binding; actual objects `JvmImageIoPromptImageTransformer`, `SharpPromptImageTransformer`, `SkikoPromptImageTransformer`, `WindowsNativePromptImageTransformer` plus GDI+/KorIM implement the **real** spec. Production tool fields are typed `PromptImageTransformer`, assembled in [agent tools:79–91][agent-tools]. No parallel codec contract/current alias remains. | `6606efe9` moves platform implementations byte-for-byte; `e89c5094` removes fake interface. PNG resize/GIF transcode/host JPEG tests use the actual host object. IO transformer test uses a fake; no corrupt-image, failure, resource-leak, cancellation or complete capability matrix. R is not all-codec certification. |

## Declaration, history and assembly checks

### Shell and direct-process responsibilities

- `eff7b10e` is a real repair, not another adapter: `ShellClientSpec`,
  `ProcessClientSpec`, fake OS interfaces/objects are absent from current tracked
  source. Internal platform classes explicitly implement the real interfaces;
  source factory return types and production fields agree.
- Client factories call `supervisorChildScope`; helper requires a parent Job and
  creates `SupervisorJob(parentJob)` ([41–48][child-scope]). Independent cancel
  does not cancel the parent. Factory tests genuinely assert that relationship,
  rather than asserting only an interface's existence.
- Factory KDoc omits the `IllegalArgumentException` for a missing parent Job,
  although the helper and shell test expose that precondition. This is a
  concrete impl-factory documentation debt, not a reason to create another
  factory interface. Direct startup parameter validation is also platform-specific:
  JVM only wraps `IOException`; environment/argument rejection is not exhaustively
  described by `ProcessClient.start`'s current `@throws`.
- `ShellSettings` moved unchanged and is used by `AgentContextSettings` and
  `KodexAgentDependencies.shellSettings`; unified-exec captures settings for each
  new command, preserving session shell selection thereafter.
- `Shell.Serializer` retains primitive descriptor name `Shell`, path-string
  encoding and filename-based type recognition. Compared `eff7b10e^` source,
  not merely the new tests: no host discovery was removed from deserialization;
  **it already preserved supplied paths**. No proven wire regression.
- Two source API changes are intentional cutover, not wire equivalence:
  `ShellProcessCommand.shell` no longer has an implicit host default;
  `Shell.default` / `Shell.resolve` changed companion members to impl extensions
  ([7–22][shell-resolution]). Notification explicitly passes `Shell.default`;
  backend defaults [119–126][backend-defaults] and unified-exec defaults are
  selected in production assembly. No model adapter was inserted.
- Resolution retains host order: Windows PowerShell then cmd; macOS `$SHELL`,
  zsh, bash, sh; Linux `$SHELL`, bash, zsh, sh. Preferred path, matching `$SHELL`,
  COMSPEC, PATH/PATHEXT and fallback candidates are resolved in impl. Final
  fallback may be absent; existence/executability is not a spec guarantee.
  `resolve` KDoc and its regular-file predicate have the Q2 limitation below.
- Pipe sessions intentionally add merged/destructive bounded text buffers and
  acknowledged text writes over distinct raw stdin/stdout/stderr. That layer
  owns real session resources; removing it would remove responsibilities needed
  by unified-exec. It is **not** an obsolete compatibility bridge.
- `StdoutBuffer` mutable producer interface, chunk budget, writer/flush actor,
  head/tail retention and `StdinChannel` claimed-write acknowledgements are
  impl-local. Public snapshot/rendering/accounting functions are in spec.
  `send` waits for platform write; `trySend` only reports rendezvous handoff;
  `onSend` is explicitly unsupported in spec. Public consumers reviewed use
  `send`, not `trySend` as write acknowledgement.
- PTY uses one terminal output, standardError empty in native/Node construction;
  JVM requests merged stderr from pty4j. Pipe EOF closes raw stdin; PTY input
  close does not hang up terminal master. Native command invocation/cwd,
  environment merge, UTF-16 Windows arguments and POSIX C spawn code were read.
  Shell MinGW Job fallback and exceptional startup remain Q1, not certified.
- Native direct POSIX C setup resolves executable/environment in the parent,
  forks process group, duplicates independent pipes, chdirs/execves in child;
  child rejection is exit 127 as spec says. MinGW creates suspended child then
  Job binding/resume. JVM and Node use direct executable APIs, not an injected
  shell. Choosing `/bin/sh -c` in test fixtures does not turn ProcessClient itself
  into a shell implementation.
- Main dependencies of these nine builds have no spec → impl edge. Impl → impl
  composition is real (shell → process; codec → IO implementation), not grounds
  for an empty interface. Host bindings/factories may legitimately reside in
  impl. OS actuals residing in spec are an explicitly verified exception to
  mechanical pairing.

### OS/Home and codec ownership

- Host targets: JVM, Node JS, Linux x64/arm64, macOS arm64, MinGW x64
  ([host target configuration:22–33][host-targets] and JS host plugin:14–23).
  Shared Shell/Process/Home spec also declares browser, WasmJS and iOS arm64
  targets ([shared plugin:22–41][shared-targets]); this is portable declaration
  availability, **not** an implementation for those extra targets.
- OS actual discovery differs intentionally: JVM `user.home` first; native/Node
  USERPROFILE → HOME → HOMEDRIVE+HOMEPATH, skipping blanks. OS expect KDoc does
  not promise a uniform host precedence. `requireUserHomeDirectory` accurately
  documents absent-home `IllegalStateException`.
- Actual OS files read: [JVM:5–15][os-jvm], [Node:5–12][os-js],
  [native:9–17][os-native]. The latter is the source actually shared by Linux,
  macOS and MinGW, not a conjectured `OsEnvironment` implementation object.
- Home utility owns a default path value, **not a scope/lease**. Scope ownership
  belongs to `app.migration.KodexHomeHandle` and application infrastructure.
  The separate UI/Home audit owns reproduced Home lease cleanup failures;
  this report does not count those as new utility migration bugs.
- Codec objects are stateless per-call transformations; `EncodedImage` and
  `PromptImageTransformRequest` copy bytes in the true images spec. Host transform
  results are checked for requested MIME/dimensions by `toPromptImage:117–124`.
  JVM owns/disposes graphics and image writer; Skiko owns/closes image/surface/
  paint/data; GDI+ balances startup, images, streams and graphics. Sharp awaits
  promise and preserves cancellation; native/KorIM synchronous calls do not
  promise prompt cancellation. The real spec expressly limits cancellation to
  suspending codec boundaries, so no invented cancellation guarantee is applied.
- Windows fallback is intentional capability composition, not a duplicate spec:
  native supports PNG/JPEG; unsupported/failing GIF decode may use KorIM PNG/GIF.
  No new `PromptImageCodec` is needed to describe it. Exception subclasses remain
  in codec spec while the real transformer contract documents their broad
  `UnsupportedOperationException`/`IllegalStateException` families.
- Concrete codec chain read under package
  `io.github.stream29.kodex.utils.images.codec`:
  [Host expect:8][codec-host] → [JVM actual/object:19–37][codec-jvm],
  [Node actual/Sharp:18–45][codec-js], [Skiko actual/object:19–89][codec-native],
  [MinGW actual:5][codec-win-binding] → [WindowsNative/GDI+:71–154][codec-win];
  [Korim object:19–68][codec-korim] is the genuine fallback/alternative.
  All implement `io.github.stream29.kodex.utils.images.PromptImageTransformer`.

## Findings, impact, history and smallest ablation

### P1 — B1: Shell exit-status and nonwaiting-close guarantees exceed implementation

- Contract: `io.github.stream29.kodex.utils.shellclient.ProcessSession.exitCode`
  [88–93][shell-session] says it fails only when final status cannot be observed.
  Concrete `PipeProcessSession.consumeStdin` [160–184][pipe] calls `fail` for
  write/close I/O failure; `fail:154–157` completes **exitCode exceptionally**
  before cancelling the session. The JVM/POSIX/MinGW/Node PTY sessions use the
  same failure policy (e.g. [JVM:186–205][shell-jvm],
  [Node:149–179][shell-node-session]).
- Closing child input while keeping the child alive can cause a write failure
  while a platform exit observer still exists. The implementation deliberately
  prevents a subsequently observable numeric exit from winning. Therefore the
  narrow published guarantee is false; this is not merely missing interface DI.
- Production impact: unified-exec [98–102, 211–214, 228–233][unified] uses both
  `stdin.send` and `exitCode.await`; it follows exceptional/failure handling
  rather than always receiving final process status after an I/O failure.
- A second independently visible conflict is `ProcessSession.close`'s
  “returns without waiting” at [95–100][shell-session]. Pipe close
  [103–115][pipe] launches **UNDISPATCHED**, then calls non-suspending raw
  `process.close()` before any suspension. Raw JVM close [75–93][process-jvm]
  waits for grace/termination and then calls `process.waitFor()` synchronously.
  Spec explicitly permits raw close to wait, but the shell wrapper does not
  dispatch that blocking call despite selecting an IO dispatcher. Therefore
  shell pipe close on JVM can wait on the caller; no timing experiment is
  required to establish that control flow. Exact delay remains unmeasured.
- History: guarantee originates `a83e8040`; pipe failure behavior originates
  `ee358d20`, before spec/impl migration. `eff7b10e` moved the guarantee intact.
  **Inherited contract conflict; no new runtime regression claimed.**
- Smallest ablation: remove the false exclusivity sentence and replace it with
  the actual I/O/observer failure semantics on the spec operation; document send
  failure families there as well. Do **not** remove exit observation, actor or
  session owner. If numeric status must survive I/O failure, that requires a
  separately authorized behavior design, not a migration-only wording fix.
- For the nonwaiting-close promise, replace undispatched execution of blocking
  raw close with an actually dispatched owner-child cleanup (retaining exit
  observation and joined cleanup), or explicitly review a contract relaxation.
  Do not merely rename the dispatcher. Pipe undispatched close originates
  `ee358d20`, before migration. Current tests await close/exit afterward and do
  not detect caller blocking; a controlled JVM child which ignores graceful
  termination plus caller progress barrier is the minimal future check.
- Test limit: shell tests cover normal exit/close/cancel but no child-stdin-close
  while alive. Minimal future experiment: child announces ready, closes stdin,
  remains alive; send sufficiently large data; capture send failure and
  `exitCode.await`, then wait for owner cleanup. Test pipes and each PTY backend
  separately. This experiment was not run.

### P2 — B1: Node direct-process cleanup escapes its owner and close leaves raw outputs usable

- Real contract `processclient.ProcessSession` [20–26, 34–39][process-spec]:
  client cancellation invalidates sessions/streams; close releases streams.
- `NodeProcessSession.cleanupScope` [133][process-node] creates a **parentless**
  `SupervisorJob`. Its close [150–155] closes stdin but launches termination into
  that detached scope [171–175]; no owner relationship/join or scope completion
  is provided. Windows termination may await taskkill for three seconds.
  This violates the project's structured resource cleanup rule independently
  of whether a platform happens to terminate quickly.
- Output streams are not closed by normal session close. `processStdout/Stderr`
  only receive `finishFromProcess`; `NodeProcessRawSource.readAtMostTo:245–259`
  checks its own `closed` flag, not client activity. After natural exit,
  `close:151` returns immediately, and guard cancellation cannot release these
  streams either. Retained bytes can still be read through the old raw source.
  This contradicts invalidation/release prose; preserving final buffered output
  might be desirable but must be explicitly specified, not silently assumed.
- Production: `McpServiceImpl.processClient` [47, 213, 312][mcp-service] →
  [MCP raw bindings:33–48][mcp-stdio]; shell pipe [28–41][pipe] composes the same
  Node process session. MCP closes its own stream adapters too, mitigating that
  particular consumer, but not repairing client ownership or generic close.
- History: detached scope, early return and output-close omission all blame to
  `1fa926af`; `eff7b10e` added stronger real spec ownership prose.
  `e89c5094` changed lazy-guard registration/start ordering only.
  **Inherited behavior now explicitly contradicted by spec; not a newly
  introduced process implementation regression.**
- Smallest ablation: remove detached cleanup scope; replace its required
  asynchronous termination with a finite `NonCancellable` cleanup in an
  owner-child/session Job which is joined when owner ends. Reconcile output
  drain-after-exit versus invalidation on owner close on the existing raw session
  contract, and close/destroy streams at the agreed boundary. No new manager,
  transport adapter, second session state or global cleanup registry is needed.
- Test limit: process tests assert nonzero exit after client close, not that
  owner Job completion waits for process-tree cleanup or streams reject reads.
  Future minimal checks: close → join client Job → verify child/grandchild gone;
  natural exit with retained bytes → close client → attempt raw read; repeat
  on Node Linux/macOS/Windows with controlled delayed termination. Not run.

### P3 — B1 documentation gate: codec IO spec omits known observable exceptions

- `io.github.stream29.kodex.utils.images.codec.readPromptImage` first overload
  [14–18][codec-io] delegates to no-transform `ByteArray.toPromptImage`, which
  throws `ImageTransformRequiredException` when resize/transcode is required
  ([98–103][image-contract]). Neither this throwing spec operation nor its short
  KDoc says so. This is a confirmed missing `@throws`, not an inferred codec bug.
- Transformer overload [23–28][codec-io] propagates decoding/transform capability
  failures/cancellation; both reads enforce `PromptImages.MaxInputBytes`, whose
  raw reader throws `kotlinx.io.IOException` for oversized input
  ([78–95][raw-read]). `writeEncodedImage:33–38` also propagates filesystem failure
  but has no operation exception semantics. Generic filesystem errors need not
  be fabricated into one exact exception across hosts; known bounded-input and
  transform requirements should still be recorded.
- Production uses transformer overload in ViewImage and ImageGeneration; the
  first overload is currently tested as a bytes-preserving utility. It is still
  a public original spec operation. No runtime corruption or wire regression
  follows merely from missing KDoc.
- History: IO source moved 100% unchanged by `6606efe9`; `e89c5094` fixes the real
  transformer KDoc in images spec but not these IO operations. **Inherited
  documentation debt left unresolved by migration.**
- Smallest ablation: retain functions, exceptions and real transformer; replace
  generic one-line KDoc with conditions/`@throws` for transform-required,
  oversized/invalid input and forwarded transform/cancellation failures.
  Do not recreate deleted `PromptImageCodec`.
- Tests: codec IO tests [36–71][codec-io-test] cover no-op and fake-transform
  success only; no oversized, transform-required, invalid input, propagated
  failure/cancel, or write/append test. Minimal future negative tests should
  exercise the actual public IO overloads, not merely exception constructors.

### D1 — Home's single path projection is boundary debt, not a second resource owner

- Original `io.github.stream29.kodex.utils.kodexhome.KodexHome: Path` became
  object-backed compatibility getter in `6606efe9`; `KodexHomeProvider.path`
  advertises a configurable provider shape but no production provider parameter
  or alternate provider exists. Global tracked-source search found provider
  references only in its declaration and impl file.
- Production CLI and generated-image assembly use the original Path property.
  The property executes the provider implementation internally; thus it would
  be inaccurate to say the spec is *entirely disconnected*, or to demand an
  interface for every Path consumer. There is one frozen default path, not
  parallel root state. Path-valued functions/bindings are legitimate.
- Remaining gate: decide whether this utility is a simple host-selected path
  contract or a genuinely substitutable provider. The current two-type
  projection does not demonstrate the latter. Canonical process-wide root /
  `.kodex` selection lives in impl; no provider-consumer test closes that claim.
- Smallest ablation if retaining original semantics: remove provider wrapper
  and place the **original Path-valued declaration** at an appropriate
  host expect/actual spec boundary; existing Path callers remain unchanged,
  actual initialization still uses `requireUserHomeDirectory`. Alternatively
  retain the binding as an explicitly accepted exception. Do not add provider
  DI to logging, every storage API or migration solely to make the table green.
- Missing-host/Unicode/precedence/cached-path tests are needed if this boundary is
  revised. Home lease cancellation tests are unrelated to default-path discovery.

## B2 / residual experiments (not asserted migration regressions)

- **Q1, startup handoff and exceptional native PTY paths:** JVM/POSIX/MinGW
  starts return acquired sessions through cancellable `withContext`; sessions
  are owner-registered but caller cancellation need not cancel that long-lived
  owner. Need a controlled cancel-at-return test to prove no unreachable child
  survives until owner shutdown. Node resumes acquired session without a
  resource-return cancellation hook; its startup cancellation handler covers
  some races, not a proved exhaustive handoff. Native PTY spawn
  [124–139][shell-posix] closes the master descriptor on subsequent setup failure
  but does not explicitly kill/reap the spawned child. MinGW PTY Job setup
  [576–583][shell-win] can return null and falls back to root-only termination.
  Test descriptor failure, cancellation, and Job-bind failure with descendants.
  These are inherited paths (`a83e8040`/later pre-migration native updates);
  no claimed reproduction on a live device.
- **Q2, shell resolver readiness:** [resolve KDoc:11–17][shell-resolution] says
  “installed executable,” yet resolution checks only regular-file metadata.
  [Unicode test:92–102][unicode-test] creates a single zero byte `cmd.exe` and
  accepts it as resolved. This proves file discovery, **not executability**.
  Smallest clarification: document candidate discovery versus successful launch;
  a true executable-ready guarantee would require target-specific checks/tests.
  Include invalid/absent default, `$SHELL`/COMSPEC/PATH/PATHEXT precedence and
  permissions in future tests. Algorithm predates migration (`a83e8040`).
- **Q3, exit-status normalization and reaping:** Node direct close callback
  [141–145][process-node] drops signal and substitutes `1` when code is null;
  POSIX direct waitpid [223–231][process-posix] substitutes `0` on ECHILD.
  POSIX PTY observer [430–439][shell-posix] treats ECHILD as “not exited” and
  keeps polling. Real status/unknown-status behavior across these cases is not
  specified or tested. No invented cross-platform exact numeric convention is
  imposed. Minimal review/experiment: signal termination and externally reaped
  child under isolation, then agree whether unknown status is failure or an
  explicit documented convention. Inherited Node `1fa926af` / shell `a83e8040`;
  direct POSIX normalization predates the split.
- Native PTY `readAvailableOutput` collects a list of all currently readable
  chunks before bounded retention; buffer unit tests do not certify whole-pipeline
  peak memory or fairness for continuously producing children. Performance
  experiment deferred; no memory-bound guarantee stronger than spec is inferred.

## Test evidence and unreviewed execution paths

- **Static source evidence only:** no fresh pass/fail result is reported.
  Factory tests are meaningful interface/ownership checks, but closed-client
  rejection never starts a real resource. Real-child tests do exercise actual
  implementations when run; their existence is not proof that they ran at HEAD.
- Shell `ProcessSessionTest` [143–158, 165–252, 291–354, 378–467][shell-test]:
  separated pipes, environment, PTY/interactive close, cancellation, blocked
  send, pipe EOF and descriptor isolation. PTY merged-output distinction,
  onSend/trySend, platform-write failure, cleanup failure, cancellation during
  acquisition and close-with-descendants are not completely covered.
- `StdoutBufferTest:16–121` covers bounded retention, accounting, consumption,
  waiting and terminal empty read. `StdinChannelTest:16–98` uses a controlled
  pending write; it does not test pty4j/node-pty OS write acknowledgement.
  Unicode tests check command/cwd/output bytes and regular-file preferred path,
  not valid executable discovery for every shell syntax.
- Direct process [IO tests:25–68][process-io-test] have no invalid executable/cwd,
  large stdout/stderr backpressure, pending raw read cancellation, cleanup-failure,
  acquired-resource return cancellation or owner-end resource assertion.
  MCP transport source/contract tests establish the spec integration boundary;
  they do not make detached process cleanup structured.
- Codec [actual host tests:14–65][codec-contract-test] check PNG resize, GIF→PNG,
  host JPEG format/dimensions. There is no complete image capability/failure
  matrix, corrupted data, cancellation/resource-disposal experiment, direct
  arbitrary transform request, metadata/orientation/alpha equivalence or
  writeEncodedImage append test. No UI acceptance is implied.
- **Unrun platforms:** all JVM OS variants, Node Linux/macOS/Windows, Linux x64/
  arm64 native, macOS arm64 native, MinGW x64 native. Platform actual source was
  reviewed, but system library/npm behavior and runtime cleanup were not
  independently executed. Browser/Wasm/iOS spec declaration targets do not
  supply missing host implementations. No Mac x64/other targets are invented.
- Findings may cross Tool/MCP/UI/Home reports; coordinator should deduplicate by
  P1/P2/P3 root cause, not add up project-row counts. This line is ready for
  asynchronous consumption without waiting for centralized builds.

## File evidence

[shell-model]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/Shell.kt#L19
[shell-session]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ProcessSession.kt#L16
[shell-contract]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ShellClient.kt#L13
[buffer-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/StdoutBuffer.kt#L17
[shell-factory]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ShellClient.kt#L8
[shell-resolution]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ShellResolution.kt#L7
[pipe]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/processClientMain/kotlin/io/github/stream29/kodex/utils/shellclient/PipeProcessSession.kt#L28
[shell-jvm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/shellclient/JvmShellClient.kt#L26
[shell-posix]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/posixMain/kotlin/io/github/stream29/kodex/utils/shellclient/PosixShellClient.kt#L58
[shell-win]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/mingwX64Main/kotlin/io/github/stream29/kodex/utils/shellclient/MingwShellClient.kt#L65
[shell-node-session]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/shellclient/NodeProcessSession.kt#L21
[process-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClient.kt#L10
[process-factory]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClient.kt#L12
[process-jvm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/processclient/JvmProcessClient.kt#L20
[process-posix]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/posixMain/kotlin/io/github/stream29/kodex/utils/processclient/PosixProcessClient.kt#L53
[process-win]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/mingwX64Main/kotlin/io/github/stream29/kodex/utils/processclient/WindowsProcessClient.kt#L69
[process-node]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/processclient/NodeProcessClient.kt#L57
[os-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironment.kt#L9
[os-jvm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/spec/src/jvmMain/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironment.jvm.kt#L5
[os-js]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/spec/src/jsMain/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironment.js.kt#L5
[os-native]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/spec/src/nativeMain/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironment.native.kt#L9
[home-spec]: file:///home/stream/ACodeSpace/local/kodex-e2e-20261007-7b23c481/evidence/source-baseline-6b7129fa/utils/kodex-home/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kodexhome/KodexHomeSpec.kt#L11
[home-impl]: file:///home/stream/ACodeSpace/local/kodex-e2e-20261007-7b23c481/evidence/source-baseline-6b7129fa/utils/kodex-home/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/kodexhome/KodexHome.kt#L7
[codec-io]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/codec/PromptImageCodecIo.kt#L14
[codec-errors]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/codec/ImageCodecExceptions.kt#L5
[codec-host]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/codec/HostPromptImageTransformer.kt#L8
[codec-jvm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/images/codec/JvmImageIoPromptImageTransformer.kt#L19
[codec-js]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/images/codec/SharpPromptImageTransformer.kt#L18
[codec-native]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/skikoNativeMain/kotlin/io/github/stream29/kodex/utils/images/codec/SkikoPromptImageTransformer.kt#L19
[codec-win-binding]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/mingwX64Main/kotlin/io/github/stream29/kodex/utils/images/codec/MingwPromptImageTransformer.kt#L5
[codec-win]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/mingwX64Main/kotlin/io/github/stream29/kodex/utils/images/codec/GdiPlusPromptImageTransformer.kt#L71
[codec-korim]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/codec/KorimPromptImageTransformer.kt#L19
[image-contract]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/PromptImageProcessing.kt#L49
[child-scope]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/coroutines/ChildScope.kt#L41
[raw-read]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineRawIo.kt#L78
[unified]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolClient.kt#L35
[agent-tools]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentTools.kt#L51
[notification]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/notification/src/commonMain/kotlin/io/github/stream29/kodex/cli/notification/NotificationHooks.kt#L52
[mcp-service]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt#L41
[mcp-stdio]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/commonMain/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransport.kt#L30
[external-url]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/externalurl/ExternalUrlProcess.kt#L12
[backend-defaults]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L119
[cli-main]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L14
[view-image]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/view-image/src/commonMain/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageToolClient.kt#L14
[image-generation]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationToolClient.kt#L24
[shell-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/shellclient/ProcessSessionTest.kt#L143
[unicode-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/shellclient/UnicodeShellTest.kt#L92
[process-io-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClientIoTest.kt#L25
[codec-io-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/images/codec/PromptImageCodecIoTest.kt#L36
[codec-contract-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/images/codec/PromptImageTransformerContract.kt#L14
[host-targets]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt#L22
[shared-targets]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-shared.gradle.kts#L22
