# Task Tree

- `Hard-cut over real Shell, Process and OS contracts`()
- `Consolidate actual MCP transport composition`()
- `Write regression fixtures and publish integration handoff`()
- `Record user acceptance and commit scoped repair`()

# Details

- Parent: [Reacceptance and authorized repairs](../executable/2026-10-05-reaccept-spec-impl-refactor.md).
- 用户接受当前修复并授权分批提交；内层提交 `eff7b10e`，本轮未推送。
- User authorized repairs; initial baseline `821b6627`, inner `refactor/spec`.
- Own `utils/{shell-client,process-client,os-environment}` and `mcp/{spec,impl}/{stdio,streamable-http,composition}` source/tests/build files.
- Other consumer Gradle files may be changed only for these exact spec/impl dependency retargets; do not edit their Kotlin behavior.
- User's lane prompt also permits consumer Kotlin import/default-shell/factory
  spelling changes required by this cutover; the concrete paths are recorded below.
- Preserve real names, serializers, output/channel/session/owner behavior and platform process/PTY algorithms.
- Delete unused substitute types and parallel startup; specs own actual caller contract and normative exceptions/lifecycle.
- OS may retain one functional API when meaningful; do not invent injection just to make the dead interface used.
- MCP factories are retained only if they are the real necessary composition boundary. No third wrapper; actual production path must prove use.
- Do not run Gradle, commit, push, switch branches, operate IDE/resources or modify other work lines.
- Handoff must list exact API/consumer changes, platform actuals and tests; central validation pending.

## API handoff (published before cutover; now implemented)

- Original FQCNs remain authoritative:
  `io.github.stream29.kodex.utils.shellclient.ShellClient` and
  `io.github.stream29.kodex.utils.processclient.ProcessClient`
  become `CoroutineScope, AutoCloseable`
  interfaces with `suspend fun start(command: ShellProcessCommand): ProcessSession`
  and `suspend fun start(command: ProcessCommand): ProcessSession`, respectively.
  Existing `CoroutineScope.ShellClient()` / `CoroutineScope.ProcessClient()`
  factories remain in impl and return these spec static types; platform owners
  become internal `PlatformShellClient` / `PlatformProcessClient`.
- `Shell`, `ShellType`, `Shell.Serializer`, `ShellSettings`, `ShellProcessCommand`,
  shell `ProcessSession`, `ProcessException`, `StdoutBuffer` and
  `StdoutBufferSnapshot` move to shell spec without FQCN/wire changes.
  `ShellProcessCommand.shell` becomes explicitly required: spec cannot choose a
  host executable. Calls previously omitting it pass `shell = Shell.default`.
  All other defaults are retained.
- Host discovery keeps `Shell.default` / `Shell.resolve(type, preferredPath = null)`
  syntax via impl extensions `val Shell.Companion.default: Shell` and
  `fun Shell.Companion.resolve(type: ShellType, preferredPath: Path? = null): Shell?`.
  Outside the shell package, import `io.github.stream29.kodex.utils.shellclient.default`
  / `.resolve` where used. No resolver service, registration or adapter.
- OS retains exactly the original top-level `environmentVariable(String): String?`,
  `userHomeDirectory(): Path?`, `processId(): Long`, `requireUserHomeDirectory(): Path`.
  Its actuals and tests move into host-targeted spec; empty impl project is removed.
  Consumers retarget `utils-os-environment-impl` → `utils-os-environment-spec`.
- MCP retains the functional APIs actually used:
  `suspend fun ProcessClient.openMcpStdioTransport(Stdio): Transport`,
  `fun CoroutineScope.McpStreamableHttpClient(): HttpClient`,
  `fun HttpClient.withMcpAuthorization(suspend (Boolean) -> String?): HttpClient`,
  `fun HttpClient.openMcpStreamableHttpTransport(StreamableHttp): Transport`.
  Portable transport algorithms/ownership and tests consolidate into existing
  host-targeted transport specs; redundant factories/default objects and empty
  transport impl projects are removed. Composition dependencies retarget to spec.
  No auth, catalog, routing or protocol algorithm changes.
- Pure-model consumer dependencies in context/session/unified-exec specs retarget
  shell impl → shell spec. Settings still needs impl for its host-discovered default.
- This is the integration contract, not a compilation claim. Central validation
  remains with the coordinator.

## IMPLEMENTATION READY — centralized validation pending

- Removed the unused substitute contracts, OS wrapper and both MCP transport
  factory/default-object pairs. One real stdio startup extension remains, now on
  the real `ProcessClient` spec interface; service defaults use its scope factory.
- Transport projects consolidate into functional specs because their operations
  already have meaningful configuration, raw-stream and ownership behavior with
  no platform declaration needing a separate implementation project. Keeping
  empty forwarding projects or inventing factories would not add a contract.
  Their existing production algorithms and real I/O fixtures were moved, not replaced.
- `public fun String.shellTypeOrNull(): ShellType?` is the shared pure path/syntax
  operation in shell spec. Invocation arguments, host-platform detection,
  executable discovery, pipe/PTY readers and native algorithms remain internal impl.
- Shell `ShellProcessCommand.shell` is required at spec construction; every
  previously implicit host-default call now passes `shell = Shell.default` at
  that same construction site. Working directory/login/TTY/environment defaults,
  model field order, `Shell` string encoding and descriptor name `"Shell"` remain.
- Process `ProcessException` is final again, matching the original pre-extraction
  declaration; scoped source scan found no subclass. Command/session/client
  ownership and failure KDoc is attached to the real spec.
- MCP service's internal constructor accepts trailing assembly dependencies:
  `httpClient: HttpClient = scope.McpStreamableHttpClient()` and
  `processClient: ProcessClient = scope.ProcessClient()`. Production scope-factory
  signature is unchanged. Fake composition tests enter the same service
  reconciliation/openTransport chain; there is no test-only transport router.

### Moved source and platform paths

- Shell common declarations moved from `utils/shell-client/impl/src/commonMain`
  to `utils/shell-client/spec/src/commonMain`, preserving package paths:
  `Shell.kt`, `ShellSettings.kt`, `ProcessSession.kt`; public output declarations
  extracted to spec `StdoutBuffer.kt`, real client interface to spec `ShellClient.kt`.
  Writable/destructive buffering stays in impl `StdoutBuffer.kt`; invocation
  declarations/constants stay in impl `ShellInvocation.kt`.
- Process real models/session/exception/interface now occupy spec
  `ProcessClient.kt` instead of `ProcessClientSpec.kt`.
- Internal actual owners retain their original implementation files:
  Shell `JvmShellClient.kt`, `NodeShellClient.kt`, `PosixShellClient.kt`,
  `MingwShellClient.kt`; Process `JvmProcessClient.kt`, `NodeProcessClient.kt`,
  `PosixProcessClient.kt`, `WindowsProcessClient.kt`.
- OS `OsEnvironment.kt`, JVM/JS/native actuals and `OsEnvironmentTest.kt` moved
  impl → spec with the same source sets/package paths. Spec uses `kodex.kmp-host`
  to retain original query backends: JVM, Node, Linux x64/arm64, macOS arm64,
  MinGW x64. Unsupported targets formerly exposed only by the unused interface
  are not claimed as query backends.
- MCP stdio `McpStdioTransport.kt`, JS I/O test, JVM server fixture and JVM service
  I/O test moved `mcp/impl/stdio/src` → `mcp/spec/stdio/src`.
  HTTP `McpStreamableHttp.kt` and `McpStreamableHttpTest.kt` moved correspondingly
  `mcp/impl/streamable-http/src` → `mcp/spec/streamable-http/src`.
- Removed build files for empty OS impl and MCP transport impl projects;
  recursive settings discovery therefore no longer includes those leaves.
  Existing shared build artifacts were not deleted.

### Exact dependency retargets

- `utils-os-environment-impl` → `utils-os-environment-spec`, preserving dependency
  configuration, in `utils/shell-client/impl`, `utils/filesystem-lease/impl`,
  `utils/kodex-home/impl`, `agent-context/impl/prefix`, `openai/impl/client`,
  `integration-test`, `app/cli`, `app/viewmodel/application`, `app/viewmodel/rpc`,
  `app/component/path-picker/impl/viewmodel` build files.
- `utils-shell-client-impl` → `utils-shell-client-spec`, preserving `api`, in
  `agent-context/spec/{contract,prefix}`, `agent-session/spec/contract`,
  `tool/spec/unified-exec` build files. Host-discovery consumers keep impl.
- `mcp/impl/composition`: stdio/HTTP impl dependencies retarget to existing specs;
  add `ktor.client.mock` only to its owned common tests.
- Shell spec adds `api(utils-process-client-spec)` for the documented live pipe
  startup exception and `kotlinx.serialization.json` in common tests.
- MCP stdio spec retains its real process-spec dependency; moves portable I/O
  dependency and real I/O fixture dependencies from removed impl build.
  Process impl is **test-only**, including Node's scope factory; composition
  and serialization-json dependencies remain JVM-test-only.
- HTTP spec moves `ktor.sse` and common mock test dependency from removed impl.
  Its JVM/Linux/macOS/MinGW engine bundles are declared directly with the same
  selections formerly supplied through `utils-ktor-client-ext-impl`; it does not
  depend on that impl just to acquire engines.

### Consumer Kotlin changes (imports/default shell only)

- Import `default` only:
  - [BackendServices.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L3)
  - [KodexGlobalSettings.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/shared/settings/contract/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/KodexGlobalSettings.kt#L3)
  - [ShellClientHook.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/hooks/src/commonMain/kotlin/io/github/stream29/kodex/hook/impl/ShellClientHook.kt#L3)
  - [ConfiguredHooksTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/hooks/src/commonTest/kotlin/io/github/stream29/kodex/hook/impl/ConfiguredHooksTest.kt#L3)
  - [HooksIntegrationTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/commonTest/kotlin/io/github/stream29/kodex/integrationtest/HooksIntegrationTest.kt#L3)
  - [MinimalAgentConversationTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/commonTest/kotlin/io/github/stream29/kodex/integrationtest/MinimalAgentConversationTest.kt#L3)
  - [KodexToolRuntimeTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/src/commonTest/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/KodexToolRuntimeTest.kt#L3)
- Import `resolve` only:
  - [UnifiedExecTestShell.jvm.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/jvmTest/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecTestShell.jvm.kt#L3)
  - [UnifiedExecTestShell.js.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/jsTest/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecTestShell.js.kt#L5)
  - [UnifiedExecTestShell.posix.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/posixTest/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecTestShell.posix.kt#L3)
  - [UnifiedExecTestShell.mingw.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/mingwX64Test/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecTestShell.mingw.kt#L3)
- [NotificationHooks.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/shared/notification/src/commonMain/kotlin/io/github/stream29/kodex/cli/notification/NotificationHooks.kt#L45)
  and [NotificationHooksTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/shared/notification/src/commonTest/kotlin/io/github/stream29/kodex/cli/notification/NotificationHooksTest.kt#L85):
  explicit `shell = Shell.default` at formerly defaulted command construction,
  plus imports. No notification business state, cleanup, dispatch or effect
  behavior was changed.
- Owned Shell `ProcessSessionTest.kt` likewise passes the host default explicitly.
  Owned Process I/O tests and Node MCP I/O test now visibly type clients as the
  real spec interfaces; original real-I/O assertions remain.

### Tests and static evidence

- Added [ShellModelContractTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/spec/src/commonTest/kotlin/io/github/stream29/kodex/utils/shellclient/ShellModelContractTest.kt#L13):
  pure path encoding/descriptor, explicit model defaults/environment validation,
  and snapshot omission/accounting; no host discovery dependency.
- Added [ShellContractFactoryTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/shellclient/ShellContractFactoryTest.kt#L13)
  and [ProcessContractFactoryTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/processclient/ProcessContractFactoryTest.kt#L12):
  actual scope factories assigned to spec types, independent child Job,
  client-close without parent cancellation and closed-start sentinel rejection.
- Added [McpStdioContractTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/commonTest/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioContractTest.kt#L29):
  real extension → fake real ProcessClient/ProcessSession → SDK raw framed write
  and flush, unchanged cwd/args/secrets projection, exactly-once session close,
  delegate close callback, no process-client close, same startup failure instance
  and acquired-session cleanup on transport construction failure.
- Added [McpTransportCompositionTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonTest/kotlin/io/github/stream29/kodex/mcp/impl/McpTransportCompositionTest.kt#L33):
  actual service stdio route reaches spec fake with exact ProcessCommand;
  actual service HTTP route reaches MockEngine with configured URL/header,
  not a process launcher. Sentinel failures become existing failed-client state.
- Extended moved [McpStreamableHttpTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/streamable-http/src/commonTest/kotlin/io/github/stream29/kodex/mcp/streamablehttp/McpStreamableHttpTest.kt#L23):
  configured notification goes through the actual transport; closing transport
  leaves borrowed HttpClient active. Existing cancellation, one-401 refresh and
  same-endpoint authorization-isolation cases remain.
- Read-only comparison to pinned `821b6627`: eight owner actual files are exact
  after only class/supertype/override/receiver-label cutover; cinterop, OS actuals,
  shell invocation/path/dispatcher/session/channel algorithms unchanged.
  Thirty files checked for unchanged/mechanically retargeted content.
- Compared remaining stdio startup and owned transport to the baseline production
  extension; its algorithm is exact, and the added duplicate is absent.
  HTTP client/auth/transport algorithm bodies also match the baseline.
- All other MCP composition main files match the baseline; service matches after
  only the two internal dependency parameters replace the original property
  initializers, in their original HTTP/process creation order.
- Whole Kotlin/Gradle source scan found no removed contract or removed-project
  references; all outside-package discovery calls import the impl extensions.
- Static checks confirm every owned dependency project path exists and every real
  shell command construction selects its shell. New owned source files were also
  checked for trailing whitespace without staging them.
- Scoped `git diff --check` found no whitespace errors; this is static evidence.
- **No compilation or test execution in this lane.** Central coordinator must
  compile affected main/tests and run the new fixtures plus retained real I/O
  tests before acceptance. No commits, pushes, branch mutations, IDE/device
  operations or Gradle invocations. Temporary scan file removed.

### Remaining platform checks

- Actual type checking/linking and real pipe/PTY tests on JVM, Node, Linux
  x64/arm64, macOS arm64 and MinGW x64 remain centralized/platform work.
- Preserve explicit outstanding acquisition-cancellation handoff and Node cleanup
  ownership experiments from the audit; neither was silently fixed or tested.
- Re-run retained shell close/exit/send/output/environment/Unicode tests, direct
  process I/O, stdio service/Node fixtures, HTTP auth/isolation and service I/O.
  Native query/home/PID semantics still need actual target validation.
- New/moved suites now live under `:utils-shell-client-spec`,
  `:utils-shell-client-impl`, `:utils-process-client-impl`,
  `:utils-os-environment-spec`, `:mcp-spec-stdio`, `:mcp-spec-streamable-http`,
  `:mcp-impl-composition`; do not target the three removed impl projects.
- Static evidence does not close descendant termination, dispatcher handoff,
  native resource release, real HTTP/server protocol or broader concurrency risks.

## Central validation (supersedes unrun lane checkpoint)

- JVM targets passed: Shell spec **3**, Shell impl **33**, Process impl **4**,
  OS spec **1**, MCP stdio **4**, Streamable HTTP **4**, composition **20** tests.
  Process spec has no test source; its real factory interface is exercised by impl tests.
- Actual MCP service transport sentinel and retained real pipe/PTY/local I/O fixtures passed.
  Composition has one stdio startup route; obsolete transport factories/projects are absent.
- Shell, Process and OS JS/Linux x64 main compilation passed; this verifies actual declarations
  beyond JVM, not linked binaries or runtime ownership on those platforms.
- Coordinator corrected the empty Shell companion body and a nullable Pair assertion in the
  new HTTP test; no serializer descriptor or production transport algorithm was changed.
- Unified Exec, RPC/Application downstream JVM tests and Integration test-source compilation
  also passed. Notification, Runtime Tool, Prefix, Path Picker and filesystem lease regressions
  plus CLI JVM main compilation passed in a supplemental consumer batch.
  The common parent lists the combined 43-target matrix and remaining platform checks.
- No commit/push; original outstanding process acquisition/Node experiments remain open.
