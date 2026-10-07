REVIEW READY — Runtime/OpenAI independent static reacceptance at `6b7129fa0e719dde8f88a0034b9d572679984a03`

# Task Tree

- `Verify the pinned source baseline`() — complete
- `Trace all twenty projects through production assembly`() — complete
- `Review history, cancellation, serialization and test doubles`() — complete
- `Publish the independently evidenced report`() — complete

# Details

## Verdict and checkpoints

- **Physical migration and actual contract takeover: substantially verified for all
  20 projects, not a directory/compile-based pass.** Eight Runtime and twelve
  OpenAI tracked build projects; principal implementations, factories, production
  callers, original declarations and regression-test bodies were read.
- **Not an unconditional acceptance:** two confirmed existing behavior/policy
  blockers, one unexecuted account-isolation schedule, and five contract-document
  debts in the coverage table. These are not twenty independent failures.
- Checkpoint 1: HEAD/clean verification and Runtime order recorded in this report.
  Checkpoint 2: production/default assembly traced; retry mismatch and cancellation
  cleanup identified; independent mock construction verified.
- Final scope is read-only inner repository. Only this outer child report was
  written, using `apply_patch`. No builds/tests, network, IDE, devices, live
  providers, shared resources, commits, pushes or branch switches.
- `R` means reviewed critical paths, **not runtime/platform certification**.
  `D` is specific boundary/document debt; `B1` has a concrete failing path;
  `B2` needs the stated minimal experiment. Every row also has platform **U**.
- Final handoff recheck: `Kodex` remains on `refactor/spec`, exact pinned HEAD;
  `git status --porcelain=v1` is empty. No source baseline was mixed.
- Source findings below are independent of earlier green test records. The
  parent report owns repository-wide deduplication and acceptance.

## Per-project coverage — exactly 20

Each project name is its actual Gradle project path. Evidence keys below resolve
to exact file URIs/lines, FQCNs, implementation/factory → actual caller → tests.
History keys name introducing/cutover commits, not build output directories.

| # | Project | Result | Contract, implementation and actual caller | History | Platform |
| --- | --- | --- | --- | --- | --- |
| 1 | `:agent-runtime-spec-contract` | D | R1: `AgentRuntime`, `ResumableAgentLayer` → private `AgentRuntimeImpl` → Session/RPC; D1 | H1, H2 | U |
| 2 | `:agent-runtime-impl-composition` | B1 | R1/R2: real owner and compact→steer→tool assembly, borrowed shared services; B1-CANCEL | H1, H2, H5 | U |
| 3 | `:agent-runtime-spec-decorator-compact` | R | R3: `KodexAgentCompactionRuntime` plus observable retry exception → actual implementing class → R2 | H1, H2 | U |
| 4 | `:agent-runtime-impl-decorator-compact` | R | R3: `KodexAgentCompactionRuntimeImpl`/`compactionRuntime` → production R2 → real-state tests; separate Agent sampling budget | H1, H2 | U |
| 5 | `:agent-runtime-spec-decorator-steer` | R | R4: `SteerRuntime`, linearizable `SteerProvider.take` → implementing class → R2 | H1, H6 | U |
| 6 | `:agent-runtime-impl-decorator-steer` | R | R4: `SteerRuntimeImpl`/`steerRuntime` → composition CAS lambda → tests of actual injection | H1, H6 | U |
| 7 | `:agent-runtime-spec-decorator-tool` | D | R5: `KodexToolRuntime` → real routed execution, borrowed ownership → R2; D2 exception mismatch | H1, H2 | U |
| 8 | `:agent-runtime-impl-decorator-tool` | R | R5: `KodexToolRuntimeImpl`/`toolRuntime`, internal `ToolRouting` → R2 → handler/storage tests | H1, H2 | U |
| 9 | `:openai-spec-models` | D | O1: pure DTOs/serializers, budget functions, auth/result facts → real transport/AgentState/storage; D3 | H1, H6, H9 | U |
| 10 | `:openai-spec-json-codec` | R | O2: public `OpenAiJsonCodec: Json` is itself authoritative wire policy → real bearer/OAuth clients and storage | H1 | U |
| 11 | `:openai-spec-client` | D | O3: client/auth/login/header interfaces and compaction exceptions → bearer class/OAuth factory → backend/State; D4 | H1, H6 | U |
| 12 | `:openai-impl-client` | B1 | O3/O4: real private HTTP clients, metadata/SSE/retry → backend defaults → State/RPC; B1-RETRY | H1, H3, H6 | U |
| 13 | `:openai-spec-account-usage` | R | O5: domain snapshot/state/attempt semantics → actual store → backend credential-free projection, not copied auth state | H1, H4, H7 | U |
| 14 | `:openai-impl-account-usage` | B2 | O5: `CodexAccountUsageStoreImpl`/owned factory → `BackendAccountState` → Global RPC; B2-ACCOUNT | H1, H4, H7 | U |
| 15 | `:openai-spec-model-catalog` | R | O6: `OpenAiModelCatalogStore` snapshot/resolve/borrow contract → actual store → backend/Runtime | H1, H4 | U |
| 16 | `:openai-impl-model-catalog` | R | O6: bundled list, actual refresh/resolve/store/owned factory → backend and compact budget consumers | H1, H4 | U |
| 17 | `:openai-spec-codex-cli-storage` | D | O7: actual read-only interface, auth/MCP wire models and serializer → real filesystem decoder → backend; D5 | H1 | U |
| 18 | `:openai-impl-codex-cli-storage` | R | O7: `CodexCliStorageImpl`/same-name factory → backend auth and explicit MCP import; no Hook/cache/thread route | H1 | U |
| 19 | `:openai-spec-client-test` | R | O8: nonempty complete handler-builder contract → independent actual double → real State/decorator test consumers | H1, H8 | U |
| 20 | `:openai-impl-client-test` | R | O8: `MockOpenAiClientBuilderImpl`, private `MockOpenAiClient`, `InMemoryOpenAiAuthStore` → test fixture defaults | H1, H8 | U |

- Totals: **12 R / 5 D / 2 B1 / 1 B2 / 0 wholly-unread projects**.
  Platform/real-provider execution remains U for every row.
- All twenty build scripts use `kodex.kmp-host`. Inspected main dependency edges
  have no spec→implementation shortcut in these two roots; each impl has a
  meaningful corresponding spec. Composition’s impl dependencies are assembly,
  not a parallel spec. Pure models/serializers and configured codec need no
  artificial implementation pair.
- Actual inclusion uses [module-tree registration][gradle-includes].
  Stale old directories contain generated output but no tracked build/source
  projects; these were not counted or treated as executable hidden routes.

## Runtime production and test evidence

### R1 — real core contract, Session factories and RPC caller

- FQCNs:
  `io.github.stream29.kodex.agentruntime.contract.AgentRuntime`,
  `io.github.stream29.kodex.agentruntime.contract.ResumableAgentLayer`;
  private implementation
  `io.github.stream29.kodex.agentruntime.impl.AgentRuntimeImpl`.
- [AgentRuntime contract][runtime-spec]: inherited atomic State operations,
  Unit-returning `resume`, same caller-Job slot for explicit `compact`, pending
  clean steer and shared `UnifiedExecClient`. Concurrent admission exceptions
  are on the actual spec operations, not only the exception class.
- [Implementation][runtime-core] delegates the same State; slot CAS surrounds
  full outer execution and cleanup. It does not construct a detached turn Job.
  Explicit compaction does not run resume cleanup or claim steer.
- [Filesystem Session factory][session-fs] creates concrete State and calls
  `buildMasterAgentRuntime`; [in-memory Session factory][session-memory] does
  the same. Public Session holders receive `AgentRuntime`, not an old owner
  plus forwarding model.
- Actual RPC chain:
  [BackendAgentRuntimeRpc.command/resume/forcedCompact][runtime-rpc] →
  [BackendSessionHost.inSession][host-operation] → backend-owned operation
  `async` → runtime caller-Job slot. Cancelling the RPC waiter does not own the
  accepted backend operation; explicit Stop cancels the current slot.
  RPC’s history-operation admission mutex is not a second Runtime decorator or
  an injection/admission callback added to `resume`.
- [Composed Session tests][runtime-tests] construct real in-memory sessions,
  persist real user/assistant items, verify mixed host calls remain pending
  without automatic completion, and test shared service cleanup.
  [Compaction cancellation/auto-slot tests][runtime-cancel-tests] verify
  occupied slot during noncancellable cleanup and preserved queued steer.
  They do **not** exercise failure of normal-resume pending cleanup.

### R2 — sole production assembly and resource distinction

- [Root factory/order][runtime-composition]: State → compaction → steer →
  tool → private `AgentRuntimeImpl`. Tool handling runs existing pending calls
  first; only when the boundary is clear does it re-enter steer/compact.
- Composition owns [fixed tools, providers and dynamic projections][runtime-tools].
  Dynamic cwd/model/settings are read on operation start; shared OpenAI client,
  catalog and MCP service are borrowed from Session dependencies. Decorator
  tools do not create another auth store or mutable settings authority.
- Composition exposes the **same** concrete unified-exec client behind
  `AgentRuntime.unifiedExecToolClient`; tools and shell observers share it.
  Fixed tools close on owner completion; construction failure cleanup uses
  suppressed failures. MCP tools are not closed by the borrowing decorator.
- H2 physically deletes turn-hook specs/impl/tests and removes compact/tool
  hook calls. Tracked source searches found no active `KodexHooks`,
  `TurnHookRuntime`, control-Hook port, fixed TurnRunner or continuation
  injection path in Runtime/Session. Notification publishing is in the RPC
  caller, not a fourth control decorator. D1 is stale KDoc, not a hidden route.
- [Actual tool tests][tool-tests] verify JSON that looks like a legacy
  `{"action":"block"}` remains ordinary output; [composed tests][runtime-tests]
  likewise verify continuation-shaped assistant text does not run a control
  route. These are actual handler/state paths, not a removed protocol emulator.

### R3 — compact decorator

- FQCNs:
  `io.github.stream29.kodex.agentruntime.decorator.compact.KodexAgentCompactionRuntime`,
  `io.github.stream29.kodex.agentruntime.decorator.compact.KodexAgentCompactionRuntimeImpl`,
  `io.github.stream29.kodex.agentruntime.decorator.compact.AgentResponseRetryLimitExceededException`.
- [Spec][compact-spec] is implemented by the [production class/factory][compact-impl],
  used at R2; this is not an interface beside an unrelated concrete runtime.
  Unit resume samples State; pending tools return to outer layers; Continue
  resets consecutive retry count; Retryable is bounded; failures/cancellation
  propagate. PreTurn/MidTurn auto compaction uses the shared State budget.
- [Retry/reset tests][compact-tests] use actual State/storage and client double,
  assert 21 samples/20 retries and reset on Continue. This **ordinary Agent
  sampling budget** is implementation policy and is not B1-RETRY.
- [Manual-compaction reset test][compact-reset-test] verifies committed synthetic
  token count zero and no immediate recompaction. The known concurrent-settings
  overwrite remains in State, not this decorator:
  [current checkpoint code][state-compact-commit] and
  [explicit known-risk test][state-known-risk]. It is an existing cross-line B1,
  not repaired by the move or this report; backend/State reviewer should own it.

### R4 — steer decorator

- FQCNs:
  `io.github.stream29.kodex.agentruntime.decorator.steer.SteerRuntime`,
  `io.github.stream29.kodex.agentruntime.decorator.steer.SteerProvider`,
  `io.github.stream29.kodex.agentruntime.decorator.steer.SteerRuntimeImpl`.
- [Spec and claim ownership][steer-spec] →
  [actual implementation/factory][steer-impl] → R2’s
  `pendingSteer.getAndUpdate { emptyList() }` lambda. Checks appendability
  before `take`, injects ordered clean events, and repeats after a late claim.
  No decorator-owned duplicate queue, CAS algorithm or lock.
- [Tests][steer-tests] persist actual clean user/agent-message events, preserve
  turn identity, do not consume in ToolPending, and race runtime claim against
  interrupt claim. Most isolate a simple delegated runtime rather than full
  production tool/compact assembly; R1 supplies complementary composed tests.
- Claim transfers ownership **before persistence**, as explicitly documented.
  A subsequent storage failure is not evidence of copied pending state or a
  mandate to invent an acknowledgment/requeue owner.

### R5 — tool decorator

- FQCNs:
  `io.github.stream29.kodex.agentruntime.decorator.tool.KodexToolRuntime`,
  `io.github.stream29.kodex.agentruntime.decorator.tool.KodexToolRuntimeImpl`,
  internal `io.github.stream29.kodex.agentruntime.decorator.tool.ToolRouting`.
- [Borrowing/routing spec][tool-spec] →
  [real handler execution/factory][tool-impl] +
  [internal index/merge algorithm][tool-routing] → R2.
- Function/freeform/namespace routes are indexed; duplicates fail, missing MCP
  routes produce explicit failed clean completion, unowned host calls remain
  pending, invalid calls use invalid completions. `Tool.handle` yields one clean
  event; the runtime does not create another raw-output authority.
- Dynamic tool snapshot is per pending batch; search samples the current engine.
  Clean completion is submitted only if the call remains pending (state-bound
  plan tool may already have completed it). Handler exceptions propagate.
- [Tests][tool-tests] verify direct successful input/output, missing MCP,
  invalid calls, updated dynamic tools, latest cwd and ordinary plan dispatch.
  Their `RequestOnlyRuntime` is a useful isolated delegate, **not** the full
  production compact→steer→tool graph or live shell/platform acceptance.

## OpenAI production and test evidence

### O1/O2 — pure models, serializers and shared JSON policy

- FQCNs under `io.github.stream29.kodex.openai` include
  `ResponsesApiRequest`, `ResponseItem`, `ResponsesStreamEvent`,
  `CodexResponsesMetadata`, `CodexResponsesClientMetadata`, `OpenAiAuthState`,
  `OpenAiResult`, `ModelInfo`; configured value is
  `io.github.stream29.kodex.openai.jsoncodec.OpenAiJsonCodec`.
- [Responses DTO][responses-dto] is the actual JSON shape, not the operation
  argument type. `stream` is fixed true; transport `turnState` is absent.
  Identity/request-purpose [models][metadata-models] and
  [flattening serializer][metadata-serializer] retain wire names and detached
  memory omission. Pure serializer logic legitimately belongs in the spec.
- [Other protocol serializers][model-serializers] preserve tagged shapes, raw
  unknown stream events, client/server tool-search distinction and legacy
  `ultra`→`max`; unknown reasoning remains represented. This is one protocol
  algorithm source, not a runtime or filesystem implementation disguised as spec.
- [Pure result getOrThrow contract][result-models] documents the observable
  result exception. Account wire DTOs, auth unavailable enum, image/search/plan
  models stay alongside serializers, without application UI framework leakage.
- [Model-only budget functions][model-budget] are reused by State context-window
  projection and compact runtime; catalog owns lookup, State owns the live
  snapshot. No need for a generic Aggregate/provider layer.
- [Codec][codec] is one `Json` value: ignores unknown keys, omits explicit
  nulls, encodes defaults subject to per-field overrides. Both real clients and
  Codex decoding consume it; tests explicitly depend on this spec.
- H1 mostly moves the original model/serializer declarations unchanged; the
  deliberate flattened operation and transport-field removal are visible in
  that commit, not silently equated with a directory rename. Model serialization
  tests (e.g. [metadata fixtures][metadata-tests]) cover metadata, compaction, image, plan, reasoning, response items,
  hosted search and results. These are fixture compatibility evidence, not
  external-provider certification. D3 identifies a specific KDoc mismatch.

### O3/O4 — actual bearer/OAuth transport, metadata, cancellation and default graph

- Interface FQCNs:
  `io.github.stream29.kodex.openai.client.contract.OpenAiClient`,
  `.OpenAiLoginClient`, `.OpenAiAuthStore`, `.OpenAiResponseHeaders`.
  Concrete bearer class:
  `io.github.stream29.kodex.openai.client.OpenAiClient`.
  OAuth factory:
  `io.github.stream29.kodex.openai.client.OpenAiLoginClient()`,
  private concrete `io.github.stream29.kodex.openai.client.OpenAiLoginClientImpl`.
- [Actual contract][client-spec] → [bearer implementation][client-impl] and
  [private-client OAuth factory][login-impl] →
  [withBackendServices default factories/dependencies/cleanup][backend-services].
  API client observes read-only auth; backend filesystem store owns credential
  loading/refresh/persistence. No application-auth capability supplied to client.
- Bearer `HttpClient` is private and closed by concrete `close`. JSON operations
  use structured provider-error values; Responses returns event Flow, not
  synthesized `Response`. Remote compaction intentionally collects its stream
  internally and checks exactly one compaction item before terminal completion.
- [Ordinary request projection][client-projection] copies flattened controls,
  computes `promptCacheKey ?: threadId`, validates required paired identity,
  sends request metadata headers and **argument-owned** turn-state header.
  Response headers are a separate callback result, not outgoing cache state.
- [Actual State caller][state-response] projects persisted settings snapshot,
  context/history/tools, and persists the first returned turn state for the
  matching turn in [settings timeline][state-turn-state]. Client recreation
  does not lose this routing truth; there is no client-local turn-state cache.
  Empty explicit cache override is retained by nullable fallback, not replaced.
- [Streaming/retry collection][client-stream] decodes events in Flow, discards
  `[DONE]` for ordinary requests and converts retryable transport exhaustion to
  flow completion; [State][state-response] returns Retryable if no terminal,
  not false success. Cancellation is excluded from retry classification;
  wrapped SSE/HTTP 429 uses the rate-limit category.
- [SSE helper][sse-helper] holds Ktor SSE session inside `channelFlow`, calls
  header observer before body forwarding, and sets packet-idle socket timeout.
  Ktor’s installed SSE path and engine decide timeout integration; the helper
  does not create an independent retained stream registry. Static source alone
  does not certify the absence of a total timeout on every actual engine.
- [Remote retry operation][client-retry] shares a retry-count object between
  HTTP retries and outer stream retries, with diagnostics but no total wall-clock
  deadline. External cancellation exits. The implementation default is wrong
  against the current checklist: see B1-RETRY.
- [OAuth spec][login-spec]/implementation return `OpenAiLoginResult.Failure`
  for HTTP errors, successful decoding remains exceptional, refresh override
  applies only to refresh. Backend auth refresh and OAuth completion branch
  before persistence; [backend refresh caller][auth-refresh] is not an old
  exception-only contract. No old `OpenAiLoginException` route remains.
- [Loopback transport tests][client-http-tests] exercise real bearer class,
  actual body/header projection, paired identity rejection, metadata bytes,
  response-header fields, reasoning and beta distinction. [429 tests][client-429-tests]
  use real local HTTP/SSE and prove retries/empty-flow handoff/4xx behavior.
  They are JVM-only and were **not run here**.
- Common tests use internal pure decoding/retry/compaction collectors and an
  OAuth Ktor test engine. They do not establish live provider compatibility,
  actual network cancellation cleanup, refresh-override deployment or Native SSE.

### O5 — account usage: real domain owner, not parallel auth state

- FQCNs:
  `io.github.stream29.kodex.openai.accountusage.CodexAccountUsageStore`,
  `.CodexAccountUsageStoreImpl`, `.CodexAccountUsageSnapshot`,
  `.CodexAccountUsageState`; factory `.ownedCodexAccountUsageStore`.
- [Spec][usage-spec] → [actual store/factory][usage-impl] →
  [BackendAccountState][account-caller]. Backend projects the original domain
  snapshot into credential-free Settings states, hides attempt/idempotency
  data and requires a nonblank exact credit. This binding is meaningful, not
  a second generic aggregate or copied auth DTO.
- Store owns auth observation/background refresh, borrows auth/client; optional
  reset-detail/token-profile failures do not destroy mandatory usage.
  Same-account fallback is conditional; switch clears it. Reset checks attempt
  mapping and expected credentials at actual HTTP submission, keeps definitive
  business outcomes through follow-up refresh failure in backend-owned variant.
- Scoped backend factories attach child Jobs; backend close joins owners before
  closing shared transports. Unscoped legacy factories have explicit
  AutoCloseable owners but are not the production default used above.
- [Local snapshot tests][usage-tests] only test mapping/fallback/null-versus-zero
  values; they do not instantiate and race the live store.
  [Actual backend account tests][account-tests] cover exact credit, all outcomes,
  one-way switch and observer cancellation. They do not test the A→B→A
  multi-endpoint schedule in B2-ACCOUNT.

### O6 — model catalog: actual resolution and borrowed client

- FQCNs:
  `io.github.stream29.kodex.openai.modelcatalog.OpenAiModelCatalogStore`,
  `.OpenAiModelCatalogStoreImpl`; factories `.OpenAiModelCatalog` and
  `.ownedOpenAiModelCatalog`.
- [Spec][catalog-spec] → [actual store, longest-prefix resolution and factory][catalog-impl]
  → [BackendGlobalState default assembly][catalog-caller] → Runtime compact
  and configuration consumers. Bundled catalog initializes synchronously;
  owned startup refresh keeps order and first duplicate reasoning preset.
  Failed refresh leaves existing snapshot; longest prefix permits only one
  provider namespace fallback; unknown model uses 272000/95% fallback.
- `close` cancels owned bootstrap work, **not** the injected client. Explicit
  suspending refresh is executed in its operation caller; do not describe
  `close()` as joining every possible external explicit call.
- No models-cache/Codex filesystem route was found here. Native/live model
  freshness is U, not a claim based on bundled names.
- [Catalog tests][catalog-tests] cover startup retention, ordering/normalization,
  resolve/fallback and cancellation of pending startup. Backend global tests
  additionally exercise parent-owner cancellation/join without frontend close.

### O7 — Codex read-only storage: factories and genuine consumers

- FQCNs:
  `io.github.stream29.kodex.openai.codexclistorage.CodexCliStorage`,
  `.CodexCliStorageImpl`, `.CodexCliMcpServer`, `.CodexCliMcpServerSerializer`,
  `.CodexAuthJson`; implementation-side factory `.CodexCliStorage(...)`.
- [Spec operations][codex-spec] and [wire model/untagged serializer][codex-models]
  → [actual filesystem/TOML decoder][codex-impl] →
  [backend auth source caller][codex-auth-caller] and
  [explicit MCP import caller][codex-mcp-caller].
- [Application default][application-default] supplies fixed user-home `.codex`;
  no `CODEX_HOME` read, thread import, Hooks, model cache or general settings
  ownership in this pair. Backend credential persistence is a separately
  authorized backend function, not a write operation secretly added to this
  read-only storage interface.
- Missing files return absent values; invalid records/filesystem errors
  propagate; MCP candidates are sorted and classified using supported fields,
  unsupported diagnostics expose names/generic failure rather than secret values.
- [Storage tests][codex-tests] cover authentication fixtures, missing files,
  supported/unsupported transports, unsupported nested fields and preview
  diagnostics. They are synthetic filesystem
  fixtures, not the user’s actual Codex directory or all platform permission
  semantics. D5 concerns operation-level exception documentation only.

### O8 — complete independent mock builder, with explicit limits

- FQCNs:
  `io.github.stream29.kodex.openai.client.test.MockOpenAiClientBuilder`,
  internal `.MockOpenAiClientBuilderImpl`, private `.MockOpenAiClient`,
  public `.InMemoryOpenAiAuthStore`; factory `.mockOpenAiClient`.
- [Builder spec][mock-spec] configures every OpenAiClient operation (ten,
  counting ordinary and remote Responses separately), both Responses handler
  forms, and independent build. [Actual implementation][mock-impl] snapshots
  each handler reference into a newly constructed double.
- Crucially `build` captures `val simpleResponseHandler = createResponseHandler`;
  the fallback lambda references that immutable local, **not mutable builder
  state**. Full handler overrides also retain their own reference. Later builder
  mutation cannot change already-built clients; [tests][mock-tests] assert both.
  Caller-provided handler closures may intentionally share their own state;
  that is not builder ownership duplication.
- Mock is a real independent test double, not a forwarding production-client
  wrapper or a copied cache/Session owner. [Test dependencies default][mock-caller]
  and compact/tool test chains consume it.
- Limits: `turnState` influences mock identity validation but is not observable
  by DTO handlers; mock turn metadata is the sentinel `"mock-turn-metadata"`,
  and header callbacks are controlled by configured handlers, not an SSE
  engine. Thus mock tests cannot prove outgoing turn-state/header bytes or
  callback collection timing. Keep complementary loopback tests, rather than
  inventing a second generic provider. Only unconfigured `listModels` is
  directly asserted in its own tests, though all missing-handler branches were read.

## Findings, history, impact and minimal ablation

### B1-RETRY — existing remote-compaction default violates two-retry SOP

- Evidence: [current checklist line 16][openai-checklist] says at most two
  protocol/transport retries. [OpenAiClientConfig line 13][client-config] sets
  **20**; [actual remote operation][client-projection] supplies that value to
  `retryOpenAiStreamingTransportWithBudget`. [Backend default][backend-services]
  constructs uncustomized client, and State/RPC manual or automatic compaction
  reaches that same implementation. No default composition override fixes it.
- Impact: outer protocol/incomplete-stream retries can permit 21 attempts
  rather than 3. Actual HTTP retry plugin has its own cap, but that does not
  bound repeated stream failures to two. Packet-idle waits can be long; there
  is correctly no shared total wall-clock cutoff.
- History **H3**: `fa436ce8` (2026-08-25) explicitly changed 2→20 and 300000→600000
  before the migration. `f1a90006` moved config unchanged. This is **old policy
  inconsistency, not a new spec/impl migration regression**; coordinator should
  not count ordinary Agent response `CompactionRetryPolicy.maxRetries = 20`
  as the same error.
- Test limit: config test asserts 20. [“two retries” test][remote-tests] explicitly
  passes `maxRetries = 2` to an internal helper, so its green result does not
  certify the real default.
- Minimal correction, subject to authorization: change only remote-compaction
  default to 2 and its default assertion, then add real-client loopback failures
  that exhaust the default combined HTTP/stream budget. Keep ordinary Agent
  sampling retries, socket-idle timeout, cancellation and no wall-clock budget.
  If 20 was deliberately re-approved, reconcile the SOP explicitly instead;
  this review found no such reconciliation. No retry wrapper/new provider needed.

### B1-CANCEL — resume cleanup can erase the original cancellation

- Evidence: [AgentRuntimeImpl.resume lines 153–158][runtime-core] catches the
  cancellation, calls `delegate.clearPending()` inside NonCancellable without
  a catch, and only then rethrows the original. [clearPending][clear-pending]
  performs real individual `completeToolCall` writes, which can fail; its spec
  lists invalid-transition/argument/cancellation failures, and underlying
  storage also has real I/O failure paths. This is not an impossible no-op.
- Actual path: composed tool handler is cancelled while a durable local tool
  remains pending → clearPending completion/storage fails → cleanup exception
  leaves resume instead of the caught cancellation. Outer finally still clears
  the slot, but [RPC exception branch][runtime-rpc] can see a generic failure
  and publish UnhandledError instead of treating the stop as cancellation.
  Violates [failure-preserving cleanup requirement][lifecycle-checklist].
- History **H5**: `3b21e5b6` introduced this unguarded pending-cleanup catch in
  the predecessor composition (then named Codex); H1 move and H2 retirement
  retain it. Existing failure mode, not newly introduced by refactoring.
- Test limit: current composed tests cover successful cleanup/compaction
  cancellation and service-close suppression, not failing normal-resume pending
  storage writes. No new failure-injection run was performed.
- Minimal correction: catch cleanup failure around only clearPending, attach it
  as suppressed to the original cancellation, and rethrow that cancellation;
  keep NonCancellable and slot finally. Test a composed session with a pending
  tool, cancelled handler and failing completion storage; assert primary
  cancellation, suppressed cleanup error, retained legal history prefix and
  released slot. Do not remove cleanup, swallow its diagnostics or introduce a
  second cancellation/turn owner.

### B2-ACCOUNT — account snapshot can straddle an untested A→B→A transition

- Evidence: [fetchSnapshot lines 221–248][usage-impl] starts three independent
  client requests; [actual bearer authenticate][client-auth] reads live auth
  separately on each submission. [refresh lines 199–202 and matches][usage-impl]
  check final current key/access token against the initial account, not which
  account submitted each endpoint. Explicit refresh is operation-caller work,
  so the auth observer’s `collectLatest` cancellation does not cancel it.
- Suspected schedule: explicit refresh starts under A; one request submits with
  A; auth changes to B before another endpoint submits; auth returns to the
  same A credentials before final publication. The final equality check can
  accept an aggregate containing B’s optional data under A. One-way-switch
  tests are not sufficient. This is a **specific static inference**, not a
  reproduced live cross-account disclosure.
- History: endpoint aggregation/identity checks came from **H7** `67a54770`,
  moved unchanged in H1; H4 added backend-owned factories. No claim that the
  migration newly introduced the schedule.
- Minimal next experiment: barrier-controlled real client on loopback plus
  mutable auth; capture credential/account headers for all three endpoints,
  drive A→B→A during an explicit refresh, and assert no mixed aggregate is
  published. Include optional cancellation and final account change controls.
- If reproduced, bind all three submissions to the same captured account and
  invalidate an in-flight aggregate on account transition. Keep this localized
  to account fetch/transport ownership; no new generic provider or alternative
  Aggregate. No source change recommended as “proven necessary” before that
  experiment.

### D1–D5 — specific contract debts, not fake implementation findings

- **D1 / Runtime core:** [ResumableAgentLayer KDoc lines 8–9][resumable-spec]
  still lists “hooks” as a runtime delegation extension. H2 deliberately removed
  control Hooks; current notification-only [Hook SOP][hooks-checklist] excludes
  Runtime/Session injection. Minimal ablation: remove that obsolete capability
  from the normative sentence. Actual routing is already retired.
- **D2 / Tool spec:** [resume @throws][tool-spec] documents
  IllegalArgumentException for bad sampled routes, but [ToolRouting lines 60–64][tool-routing]
  uses `error(...)` (IllegalStateException) for a supplied hosted/noncallable tool
  spec. Duplicate/empty/collision/name conditions use `require` as documented.
  Minimal: document the separate unsupported-spec IllegalStateException, or
  normalize only that guard with an explicitly reviewed compatibility choice.
  No evidence that normal production local-tool catalog supplies hosted tools.
  H1 extracted this guard and its incomplete exception contract.
- **D3 / Models:** [ResponsesApiRequest reasoning KDoc lines 20–21][responses-dto]
  says default reasoning is omitted, but line 44 is `@EncodeDefault(ALWAYS)`;
  current tests/code deliberately send explicit reasoning. Minimal: update that
  KDoc only, not wire bytes or a duplicate DTO. The omission description dates
  to `9c506941`; **H9** `2b7a63e9` changed the annotation to ALWAYS before H1,
  which retained both. This mismatch predates the module move.
- **D4 / Client operations:** [OpenAiClient operation KDoc][client-spec] describes
  propagating transport/decoding/cancellation in class prose, but lists only
  auth/argument/specific compaction exceptions on operations. The project’s
  operation-level `@throws` rule is not fully met. Add applicable documented
  exception families/conditions to the actual spec methods without pretending
  every engine throws exactly one subtype. OAuth documents these families
  already; its exchange blank-field condition is enforced by the parameter
  constructor, so [value constructor][login-values] should carry that validation
  contract rather than implying exchange can receive such a value.
- **D5 / Codex storage:** [readAuthOrNull/readMcpImportCandidates][codex-spec]
  mention filesystem/invalid-TOML propagation only in prose; auth lists only
  SerializationException. Add accurate filesystem/decode `@throws` on both
  spec operations with missing-file absence distinguished. Interface extraction
  H1 introduced this partial contract. Pure TOML polymorphic serializer is valid;
  do not move it to impl or introduce an empty storage proxy.

## Introducing and corrective history

- **H1 — `f1a90006`**, `refactor: split runtime and OpenAI spec modules`:
  original `agent-runtime/contract`, concrete decorators and root `impl` →
  current specs/impl composition; `openai/client-contract`, client, models,
  json-codec, codex-cli-storage, client-test and earlier account/catalog pairs →
  current layout. Same Kotlin packages retained; meaningful decorator/read-only/
  builder interfaces extracted. Ordinary Responses flattened into operation
  arguments; OAuth provider errors became values. `git show f1a90006^:<old-path>`
  and rename-aware diff were checked, not only current implements clauses.
- **H2 — `206266c8`**, `refactor: hard-cut over shared behavior to its real owners`:
  removes actual turn-hook projects/call sites, compact/tool control interception;
  later `19bad603` retargets real State owner. Three decorators remain by design.
- **H3 — `fa436ce8`**, remote-compaction retry default 2→20, before H1.
- **H4 — `8397a155`**, backend-owned account/catalog factories;
  later backend lifecycle cutovers make them the real defaults.
- **H5 — `3b21e5b6`**, original cancellation pending-cleanup addition.
- **H6 — `4de3591e` / `8ce9d9c7`**, persisted Responses turn state /
  unified running-turn slot before H1. Current State hard cutover `19bad603`
  retargets persisted clean steer/State contracts rather than reviving old DTOs.
- **H7 — `67a54770`**, original account-usage reset/aggregation behavior.
- **H8 — `e89c5094`**, fixes builder independence by snapshotting the simple
  handler, adds both builder-mutation regression cases. Present HEAD has this
  correction; the old mutable-builder-capture issue is not reported as remaining.
- **H9 — `2b7a63e9`**, explicit reasoning serialization, including ALWAYS on
  Responses reasoning, retained through H1. Original pure model budget
  calculation was introduced by `c099ef12`, then moved as pure spec code.

## Tests and actual-platform U

- No tests were run in this audit. Earlier task records report focused JVM
  successes; those are historical evidence, not the audit’s own execution.
- Host plugin defines JVM, linuxX64, linuxArm64, macosArm64 and mingwX64
  [actual target configuration][host-targets], plus JS/Node in
  [host plugin][host-plugin]. Every row’s Native/JS/JVM runtime result is U here.
- No OpenAI per-platform `actual` exists inside these roots; concrete network,
  environment and filesystem actuals are transitive infrastructure. Engine
  selection, libcurl/Windows/macOS resource cleanup, JS transport behavior and
  live auth/SSE are not independently run or exhaustively re-reviewed here.
- Read loopback tests establish what they would assert, not a fresh passing run.
  OAuth engine mocks, pure serializers and DTO handler doubles cannot prove
  live provider schema/retry/metadata acceptance. Live protocol unknowns remain
  **separate from migration takeover and B1-RETRY**.
- Minimal follow-up gates after authorization: the two failure-path tests above,
  account multi-endpoint schedule, default retry-budget loopback, cancellation
  during header callback/SSE body on actual engines, and relevant supported
  targets. Do not compete with current shared builds or run them from this child.
- Final clean/HEAD/branch recheck completed above. This report changes only
  the outer child file, not inner source or tests.

## Evidence URI index

[gradle-includes]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L55
[host-targets]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt#L22
[host-plugin]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-host.gradle.kts#L1
[runtime-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/contract/AgentRuntime.kt#L17
[resumable-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/contract/ResumableAgentLayer.kt#L5
[runtime-core]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L137
[runtime-composition]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L40
[runtime-tools]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentTools.kt#L39
[session-fs]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexAgentSession.kt#L44
[session-memory]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/inmemory/InMemoryKodexSessionRepository.kt#L174
[runtime-rpc]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendAgentRuntimeRpc.kt#L64
[host-operation]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L115
[runtime-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/in-memory/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/inmemory/AgentRuntimeRunningTurnTest.kt#L64
[runtime-cancel-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/in-memory/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/inmemory/AgentRuntimeRunningTurnTest.kt#L305
[compact-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/decorator/compact/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/compact/KodexAgentCompactionRuntime.kt#L8
[compact-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/compact/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/compact/KodexAgentCompactionRuntimeImpl.kt#L19
[compact-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/compact/src/commonTest/kotlin/io/github/stream29/kodex/agentruntime/decorator/compact/KodexAgentCompactionRuntimeTest.kt#L200
[compact-reset-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/compact/src/commonTest/kotlin/io/github/stream29/kodex/agentruntime/decorator/compact/KodexAgentCompactionRuntimeTest.kt#L375
[steer-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/decorator/steer/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/steer/SteerProvider.kt#L5
[steer-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/steer/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/steer/SteerRuntimeImpl.kt#L12
[steer-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/steer/src/commonTest/kotlin/io/github/stream29/kodex/agentruntime/decorator/steer/SteerRuntimeTest.kt#L117
[tool-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/decorator/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/KodexToolRuntime.kt#L6
[tool-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/KodexToolRuntimeImpl.kt#L26
[tool-routing]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/ToolRouting.kt#L16
[tool-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/src/commonTest/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/KodexToolRuntimeTest.kt#L376
[state-known-risk]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonTest/kotlin/io/github/stream29/kodex/agentstate/impl/AgentStateHardCutoverTest.kt#L112
[state-compact-commit]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L632
[clear-pending]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentStateExtensions.kt#L67
[responses-dto]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ResponsesApiModels.kt#L9
[metadata-models]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/CodexResponsesMetadataModels.kt#L8
[metadata-serializer]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/CodexResponsesMetadataSerializer.kt#L13
[metadata-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonTest/kotlin/io/github/stream29/kodex/openai/CodexResponsesMetadataSerializationTest.kt#L1
[model-serializers]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/OpenAiJson.kt#L20
[result-models]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/OpenAiResult.kt#L28
[model-budget]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ModelCatalogModels.kt#L117
[codec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/json-codec/src/commonMain/kotlin/io/github/stream29/kodex/openai/jsoncodec/OpenAiJsonCodec.kt#L5
[client-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/contract/OpenAiClient.kt#L32
[login-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/contract/OpenAiLoginClient.kt#L12
[client-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L90
[client-projection]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L196
[client-auth]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L329
[client-stream]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L370
[client-retry]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L651
[client-config]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClientConfig.kt#L13
[login-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiLoginClient.kt#L34
[login-values]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/OpenAiLoginModels.kt#L42
[backend-services]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L36
[state-response]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L183
[state-turn-state]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L545
[sse-helper]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/SseRequests.kt#L52
[auth-refresh]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStore.kt#L144
[client-http-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/jvmTest/kotlin/io/github/stream29/kodex/openai/client/ExplicitReasoningHttpTest.kt#L21
[client-429-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/jvmTest/kotlin/io/github/stream29/kodex/openai/client/OpenAiRateLimitHttpTest.kt#L18
[remote-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonTest/kotlin/io/github/stream29/kodex/openai/client/OpenAiRemoteCompactionV2Test.kt#L171
[usage-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/account-usage/src/commonMain/kotlin/io/github/stream29/kodex/openai/accountusage/CodexAccountUsageStore.kt#L7
[usage-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/account-usage/src/commonMain/kotlin/io/github/stream29/kodex/openai/accountusage/CodexAccountUsageStoreImpl.kt#L32
[account-caller]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendAccountState.kt#L23
[usage-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/account-usage/src/commonTest/kotlin/io/github/stream29/kodex/openai/accountusage/CodexAccountUsageSnapshotTest.kt#L18
[account-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendAccountStateTest.kt#L22
[catalog-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/model-catalog/src/commonMain/kotlin/io/github/stream29/kodex/openai/modelcatalog/OpenAiModelCatalogStore.kt#L8
[catalog-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/model-catalog/src/commonMain/kotlin/io/github/stream29/kodex/openai/modelcatalog/OpenAiModelCatalogStoreImpl.kt#L25
[catalog-caller]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalState.kt#L65
[catalog-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/model-catalog/src/commonTest/kotlin/io/github/stream29/kodex/openai/modelcatalog/OpenAiModelCatalogTest.kt#L106
[codex-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/codex-cli-storage/src/commonMain/kotlin/io/github/stream29/kodex/openai/codexclistorage/CodexCliStorage.kt#L5
[codex-models]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/codex-cli-storage/src/commonMain/kotlin/io/github/stream29/kodex/openai/codexclistorage/CodexCliStorageModels.kt#L12
[codex-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/codex-cli-storage/src/commonMain/kotlin/io/github/stream29/kodex/openai/codexclistorage/CodexCliStorageImpl.kt#L10
[codex-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/codex-cli-storage/src/commonTest/kotlin/io/github/stream29/kodex/openai/codexclistorage/CodexCliStorageTest.kt#L17
[codex-auth-caller]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStore.kt#L186
[codex-mcp-caller]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendCodexMcpImport.kt#L17
[application-default]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L58
[mock-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/client-test/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/test/MockOpenAiClientBuilder.kt#L23
[mock-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client-test/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/test/MockOpenAiClient.kt#L31
[mock-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client-test/src/commonTest/kotlin/io/github/stream29/kodex/openai/client/test/MockOpenAiClientTest.kt#L61
[mock-caller]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/test/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/test/TestKodexAgentDependencies.kt#L16
[openai-checklist]: file:///home/stream/ACodeSpace/push/BuildKodex/checklist/openai-module-boundaries.md#L16
[lifecycle-checklist]: file:///home/stream/ACodeSpace/push/BuildKodex/checklist/coroutine-resource-lifecycle.md#L10
[hooks-checklist]: file:///home/stream/ACodeSpace/push/BuildKodex/checklist/hooks.md#L15
