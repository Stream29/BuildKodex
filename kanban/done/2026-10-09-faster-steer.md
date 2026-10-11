# Task Tree

- `Locate the existing steer delivery and interruption boundaries`()
- `Define owner-local steer response and safe delivery points`()
- `Specify focused changes and regression checks`()
- `Implement the approved minimal runtime changes`()
- `Migrate and extend focused regression tests`()
- `Run targeted validation on remote MacBook`()
- `Wait for user code review`()

# Details

- User-selected follow-up from the Codex update review: improve existing Steer responsiveness.
- User assessment: This is straightforward work that Kodex could already do but had not implemented; treat it as an improvement to existing steer responsiveness, not a new protocol integration.
- Status: User accepted the code review on 2026-10-12 and authorized separate commits and pushes in both repositories. Implementation and targeted remote validation are complete; no release was requested.
- Goal: Let follow-up instructions affect a running task sooner while retaining current-turn Steer semantics.
- Previous behavior: the independent `SteerRuntimeImpl.resume()` checked pending input before and after its inner runtime returned. The replacement retains those natural boundaries inside the model runtime and adds request-local preemption and next-sampling checks.
- Confirmed architecture, 2026-10-12: Steer must not remain an independent runtime decorator. A runtime capable of responding to Steer owns its response and chooses its own safe insertion points. The external decorator cannot infer safety inside a model request, continuation loop, or tool execution.
- Keep the existing backend pending queue as the sole input source and preserve atomic, ordered ownership transfer. Shared input access does not decide another runtime's cancellation, continuation, or delivery timing.
- Accepted implementation removes `SteerRuntime` and composes Compaction Runtime directly inside Tool Runtime; the previous independent-decorator design is superseded. See the [canonical runtime decision](../../checklist/agent-state-and-runtime.md).
- Reviewed implementation:
  - The compaction runtime owns individual model requests, continuation, retry, and automatic compaction. See [request loop](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/compact/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/compact/KodexAgentCompactionRuntimeImpl.kt#L35).
  - The tool runtime owns a sequential pending-call batch. Completing one call may leave `ToolPending`, which is not an appendable state. See [tool loop, lines 35–113](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/KodexToolRuntimeImpl.kt#L35).
  - Cancelling `exec_command` or `write_stdin` currently closes the process session. Ending an output wait while preserving the process requires a normal-return path, not generic coroutine cancellation. See [command waits, lines 78–107](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolClient.kt#L78).
  - Cancelling the outer runtime triggers pending-tool cleanup and releases the running-turn slot. Steer must not be implemented as Stop followed by resume. See [turn cancellation, lines 147–185](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L147).
- Approved implementation scope:
  - Remove the independent SteerRuntime and relocate the shared provider contract/adapter to `agent-runtime/{spec,impl}/steer`.
  - Add `waitForSteer(): Unit` without consumption or reservation.
  - Compaction Runtime preempts only its active model request, retains completed output items, and discards uncommitted deltas. After request cleanup, pending tool calls keep Steer queued.
  - Recheck queued input inside the sampling loop and after automatic compaction; preserve the old initial/final and post-tool delivery behavior.
  - Keep tools, command waiting, unstarted calls and manual compaction behavior unchanged. Tool Runtime uses the existing handoff to the inner model Runtime; no duplicate claim point is needed.
  - Serialize a claimed batch through cancellation-resistant delivery so Stop does not abandon it between claim and persistence. Storage failures still propagate without replaying already committed input.
  - Preserve retry limits and real failures; Steer preemption itself does not count as a retry. Do not add final-turn auto-resume or a new handoff protocol.
  - Modify locally; run heavy validation in an isolated remote MacBook checkout. Preserve parallel UI changes and do not manipulate IDEA windows or trigger local Sync/builds.
- Reuse existing runtime and transport capabilities; WebSocket adoption or a new OpenAI endpoint is not a prerequisite.
- Context: [local candidate review](../../shared-context/findings/2026-10-09-codex-support-candidates.md).

## Proposed response and insertion strategy

- Reassessed at the user's request on 2026-10-12 using the confirmed non-consuming wait contract. The approved minimal subset above takes precedence over optional strategies below.
- Count semantic sites, not duplicated calls at every wrapper boundary. The current executing layers are `KodexAgentCompactionRuntimeImpl` and `KodexToolRuntimeImpl`; `AgentRuntimeImpl` owns the turn slot and cancellation, while the unified-exec client is a tool implementation, not another runtime.
- User proposal, 2026-10-12: do not require a multi-item model response to finish before Steer can take effect; investigate preemptive insertion at a legal history boundary.
- At a legal idle boundary, try `take()` without waiting; an empty batch must not delay normal work. During a responsive operation, `waitForSteer()` observes input without claiming it. After owner-local cleanup, assess the actual history/state before any claim. Persist a claimed batch before subsequent sampling.
- The current append predicate governs ordinary writes, not whether active sampling can be preempted. Wake-up, safe request termination, and legal history insertion are distinct boundaries.

| ID | Owner / situation | Wait and response | Actual insertion |
| --- | --- | --- | --- |
| C1/C2 | Model runtime: before preparing context and immediately before each request | No suspended wait; try to claim already queued input without blocking. Reassess context preparation after delivery. | In appendable state, persist before compaction/request snapshot. Covers first sampling, continuation and permitted retries. |
| P1 | Model runtime: active sampling | Race only this operation against a non-consuming wait. On wake, request termination at a recoverable boundary; do not cancel the outer turn. Cancel/join an unused waiter when sampling finishes. | After old request ownership ends and history is legal. Keep committed output; do not accept late old-stream commits after insertion. |
| C3 | Model runtime: normal response has restored state | Cancel/join the unused waiter, retain the real response outcome, and check for queued input before continuation or normal return. | `Continue` and permitted `Retryable`: before the next preparation/request. `Finish`: a nonempty delivery continues sampling. `ToolPending`: no claim; hand off. |
| T0 | Tool runtime: calls remain pending, before starting another handler | Not an insertion point. Respond here only if an explicit policy for terminating unstarted calls has been agreed. | Retain input until pending calls are truthfully resolved; never clear them merely because the waiter won. |
| T1 | Tool runtime: result commit has left no pending calls | No suspended wait; try to claim before re-entering sampling. Ordinary handlers need no Steer waiter if they cannot safely yield. | Check actual state after completion handling, including self-persisting handlers; not the handled flag or original batch's last index. Adjacent T1 and C1 coverage can share placement. |
| E1 | Tool runtime / command client: waiting for command output | With a cooperative yielding capability, end only the output wait normally; preserve the process and record real output/session identity. Do not generically cancel `Tool.handle`. | Only through T1 after the real completion is committed and all pending calls are resolved. The command client does not claim input. |
| A1 | Model runtime: automatic compaction | Do not preempt compaction by default or arm an unusable waiter. Preserve incoming queue entries while compaction owns state. | Before starting compaction when legal, and after successful compaction in the owning resume path, before the next request. |
| M1/O1 | Manual compact / outer turn ownership | No new Steer wait or generic preemption; preserve operation results, Stop cleanup and running-slot ownership. | No consumption in manual compact, generic exception cleanup or slot-release `finally`. |

- P1 is the primary request-internal response candidate; natural boundaries retain non-blocking claims. Completion variants such as tool search, invalid calls and missing-MCP-route failures share T1 instead of receiving their own waiting/claim mechanisms.
- Request-internal findings:
  - A request captures its input once. Writing Steer into history cannot change the existing request's input; continuing its output would still use the old instructions. See [request snapshot and input, lines 160–200](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L160).
  - Completed history items are committed individually, before the response terminates. Cancellation retains those commits and restores state from storage. The current item's uncommitted deltas must not be presented as a completed historical item. See [item commits, lines 233–285](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L233) and [state recovery, lines 701–707](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L701).
  - A completed local tool-call item can leave pending calls, but does not mean the tool has started executing. The tool runtime invokes handlers only after its inner model runtime returns. See [tool handoff, lines 35–43](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/KodexToolRuntimeImpl.kt#L35).
  - Function/custom-tool input can stream incrementally; this is call generation, not local execution progress. Kodex commits calls only on `OutputItemDone`; an uncommitted partial call remains transient. The application executes local functions and submits their outputs separately. See [OpenAI function-calling guide](https://developers.openai.com/api/docs/guides/function-calling) and [item commits, lines 233–243](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L233).
  - The approved minimal implementation discards uncommitted deltas when preempting, and uses the actual restored state; it does not wait for another item commit or infer every item boundary from StateFlow.
- Scope correction after checking current tool composition: the main request exposes client-executed tools, including Tool Search with `execution = "client"`, not hosted Tool Search. See [visible tool assembly, lines 24–63](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/tool/KodexVisibleToolSpecs.kt#L24) and [Tool Search mode, lines 5–17](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/tool-search/src/commonMain/kotlin/io/github/stream29/kodex/tool/toolsearch/ToolSearchTools.kt#L5). The earlier hosted-pairing concern came from supported protocol/storage types, not an enabled main-request tool; it is not a prerequisite for this optimization.
- Hosted tools are a separate API capability: built-in web search and code interpreter can emit execution-progress events, and hosted Tool Search produces distinct call/output items. Keep their existing compatibility handling, but do not infer partially executed local tools from those protocol types or add a new hosted recovery design to this task. See [streaming events](https://developers.openai.com/api/reference/resources/responses/streaming-events) and [Tool Search modes](https://developers.openai.com/api/docs/guides/tools-tool-search).
- Proposed race rules:
  - The waiter winner determines a response attempt, not input ownership or insertion permission. After request cleanup, a local pending-call commit sends execution to tool handling without claiming.
  - Normal completion racing with wake-up uses actual committed state. A real failure or explicit Stop must not be disguised as successful Steer preemption; do not increment the response retry budget solely for Steer.
  - After wake-up, if `take()` is empty because input was retracted or claimed elsewhere, report no delivery and continue from actual history; do not repeatedly restart on the stale wake-up.
  - A pending queue is not permission to cancel an arbitrary side-effecting handler. Proposed conservative behavior is to finish an already running handler once; skipping unstarted calls needs separate confirmation and truthful completions.
- Excluded sites:
  - Direct history insertion while the old stream still owns `RequestResponse` and can append more output. `OutputItemDone` may trigger P1, but insertion requires finishing the old local operation and resolving history obligations first.
  - `Compacting`, tool handling before its completion is committed, and any remaining `ToolPending`.
  - Direct/manual `compact()` entry or exit: preserve its existing no-steer-consumption/no-resume contract. Automatic compaction is covered only in the owning `resume` path.
  - Generic failure, explicit Stop cleanup, and turn-slot release `finally`: propagate the existing operation result instead of consuming input during cleanup.
- Remaining limitations: persistence failures do not roll back earlier committed input; the in-memory queue is not crash-durable. Input arriving after the final checkpoint but before the running slot is released retains the existing queue behavior; this change does not introduce a new final-turn handoff protocol.

## Confirmed non-consuming wait contract

- User decision, 2026-10-12: provide `suspend fun waitForSteer(): Unit` to wait without consuming; if no Steer arrives, the operation remains cancellable. This changes the access contract, not the pending queue's data model.
- Keep one queue and separate observation from ownership:
  - `waitForSteer()` observes an already nonempty queue or waits for a later observed nonempty state. It does not require a new enqueue after subscription.
  - Waiting neither removes nor reserves input. Cancellation releases the waiter and propagates cancellation without changing the queue.
  - `take()` remains the existing non-waiting atomic ordered claim; it may legitimately return empty even after a wait completed.
- Host-side implementation can observe the same `pendingSteer` StateFlow with a nonempty predicate; no additional event Channel, queue mirror, or stored "waiting window" is needed. This is state-based waiting, not an event log: an input retracted or claimed before observation need not wake a waiter.
- `SteerProvider` is now a two-method interface in `agent-runtime/spec/steer`; composition uses `StateFlowSteerProvider` from `agent-runtime/impl/steer`, not a SteerRuntime layer.
- Request/tool-call race:
  - Waiting observes input but does not claim it.
  - If the owning runtime preempts, end and join only its current request operation so that actual committed history and pending state are known.
  - If a pending tool call won the commit race, do not claim Steer; retain it in the queue and follow the owning tool runtime's handling policy.
  - If history permits insertion, atomically claim and persist input before subsequent sampling. A concurrent retraction or competing claim can leave the batch empty; do not report delivery in that case.
- Waiter lifecycle belongs to the operation that can respond: when that operation finishes, cancel and join its unused waiter. After waking into a non-steerable state, advance tool handling or another real state boundary instead of repeatedly calling an immediately returning wait.
- This separation addresses premature consumption but does not by itself solve cancellation after claim, persistence failure, tool-call resolution, or final-turn handoff races.

## Planning checks

- Follow-up input is delivered earlier in the agreed running states, with a deterministic test of the delivery timing.
- Multiple inputs preserve order and are neither lost nor delivered twice.
- Partial model output and tool-call/output history remain valid; interruption does not duplicate executed side effects.
- Preemption occurs before the original multi-item response normally completes, with no late old-stream output committed after Steer insertion and no false outer-turn cancellation or retry accounting.
- Wait covers input present before subscription and input appended during waiting; neither wait completion nor cancellation changes the queue. Request completion leaves no waiter behind.
- A deterministic race between waiter completion and tool-call commit preserves queued input when history is not insertable; retraction/competing claims after wake can produce an empty claim without duplicate delivery or a busy loop.
- Partial local tool-call input does not become durable pending work or invoke a handler; a committed call remains pending and unexecuted at the model/runtime handoff. Concurrent normal failure or explicit Stop retains its real outcome.
- Queue-versus-Steer behavior, cancellation, and Session persistence remain coherent.

## Implementation and validation — 2026-10-12

- Implemented the approved subset without modifying Tool Runtime or unified-exec handlers:
  - Shared [wait/take contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/steer/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/steer/SteerProvider.kt#L13) and StateFlow adapter replace the independent Steer Runtime modules.
  - [Request-local race](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/compact/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/compact/KodexAgentCompactionRuntimeImpl.kt#L106) joins request cleanup before state inspection; committed pending calls leave input queued.
  - [Delivery](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/compact/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/compact/KodexAgentCompactionRuntimeImpl.kt#L132) persists claimed input before further sampling or propagating Stop.
  - Preemption does not spend/reset retries; normal Finish followed by new input retains the old fresh-budget behavior.
- Regression coverage: 7 provider tests and 23 model-runtime Steer tests, including multi-item streaming, discarded partial calls, tool-commit race, real two-layer tool execution once, queue retraction, cleanup failures, Stop after claim, automatic PreTurn/MidTurn compaction, and retry limits. Existing compaction/tool/session/Stop tests remain covered.
- The existing outer cancellation regression now checks cancellation type/message and the original suppressed cleanup failure, rather than exception identity: crossing a request child allows JVM coroutine stack recovery to copy the exception.
- Remote MacBook results, all zero failures/errors/skips:
  - JVM: **91 tests** — provider 7, compaction/Steer 34, tool 12, in-memory session/runtime 24, Runtime RPC 13, minimal conversation integration 1.
  - macOS ARM64 Native: **77 tests** — provider 7, compaction/Steer 34, tool 12, in-memory session/runtime 24.
  - Composition compiled for JVM and macOS ARM64; the integration test sources compiled for JVM.
- Validation used Java 25 and the detected shared daemon, one worker, no parallel build, and the device heavy-task lock. The source snapshot started at `03a719f30ebd91b4936dad31c0cf7ffecc9b24c9` and overlaid only this task's changes; parallel UI modifications and live provider probes were excluded. Local/remote hashes of the changed/new source files matched.
- Scoped diff, source hashes, final logs and XML reports are retained under [local evidence](file:///home/stream/ACodeSpace/push/BuildKodex/out/steer-747/). Temporary remote source/resources are cleaned after evidence transfer.
- Remaining limitations above are unchanged. User review is accepted and this implementation task is complete; no version bump or release was performed.

## Accepted product commits

- `cc927736028896a2aee5debc29a625f96aea5391` — `feat: respond to steer within the model runtime`: provider relocation, request-local response, composition and existing-call-site migration.
- `fda13a79493c70aaff857fc593d8d0e770d498c3` — `test: cover runtime-local steer preemption`: focused provider and model-runtime regression suites.
- Both commits are signed and pushed to `Kodex/main`. The outer repository records the product pointer separately from this task archive and confirmed design updates.
- Parallel UI changes, the release skill and unrelated task records are excluded from these submissions.
