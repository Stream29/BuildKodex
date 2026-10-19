# Task Tree

- **`Align on Kotlin/Native Antigravity integration architecture`()**
  - `Confirm zero-dependency native delivery constraint (reject libpython cinterop and Python subprocess bridges)`()
  - `Adopt Google Antigravity SDK Layer 3 (LocalConnection) protocol as the reference specification`()
- `Design the native Kotlin connection client for local Antigravity harness`()
  - `Specify process supervisor for /home/stream/.local/bin/agy / localharness daemon`()
  - `Define WebSocket / Connect-RPC transport client using Ktor in Kotlin/Native`()
  - `Map localharness protobuf / JSON events (InputEvent, OutputEvent, StepUpdate) to Clean models`()
- `Design the Continuation model and cross-provider switching matrix`()
  - `Define AgyContinuationAnchor (cascadeId, stepIndex, syncedKodexIndex) storage timeline`()
  - `Specify lazy revert (RevertToCascadeStep) and lazy fork (ForkConversation) semantics`()
  - `Specify cross-provider handoff: agy -> OpenAI (zero-cost direct projection) and OpenAI -> agy (delta bridge)`()
- `Define AgentRuntime and AgentState abstraction boundaries`()
  - `Route execution in ResumableAgentLayer.resume() based on settings.model provider`()
  - `Keep AgentRuntimeRpc, TimelineRpc, and TUI contracts 100% untouched`()
- `Review and promote discussion task to implementation planning`()

# Details

## 定位与状态 — 2026-10-19 / Session945

- 用户明确本线是纯探索性讨论，与大部分工作正交，不作为 Codex 对齐、Session 层级、subagent 或 Tool as CLI 的前置。
- [连接层候选方案](2026-10-19-plan-antigravity-native-connection-layer.md)也保留在 discussion；下方架构、传输选择和实验描述是探索材料，不构成已接受的生产实施规划或实现授权。

- Context:
  - The user requested discussing integrating Antigravity (`agy`) into Kodex as an alternative agent implementation while keeping upper layers unchanged, modeling each storage `index` as a continuation.
  - After discovering the official [Antigravity Python SDK](https://github.com/google-antigravity/antigravity-sdk-python) on PyPI (`google-antigravity`), inspection of its binary wheel revealed that Python SDK's `LocalConnection` is a thin client communicating over WebSocket/JSON with an embedded Go executable (`google/antigravity/bin/localharness`), exactly matching the codebase of `/home/stream/.local/bin/agy`.
  - The user confirmed adopting **Option 3: Directly implement the Antigravity connection layer in Kotlin/Native (KN)**, preserving Kodex's single-binary, zero-external-dependency native distribution.
- Architecture Decisions:
  - **Transport Protocol Selection & Empirical Verification**:
    - Compared two local transport mechanisms:
      1. Connect-RPC over HTTP (`LanguageServerService` on `localhost:<port>`).
      2. Headless Stdio `stream-json` (`agy --input-format stream-json --output-format stream-json -p=`).
    - **Empirical winner on 性价比 (ROI): Headless Stdio `stream-json`**:
      - Zero port management, zero CSRF token extraction, zero network firewall/socket flakiness.
      - Pure Unix pipe (stdin/stdout) using standard POSIX `posix_spawnp` / pipes in Kotlin/Native.
      - Zero new dependencies: parses NDJSON lines with existing `kotlinx.serialization`.
      - Empirically verified: full real-time `step_update.text_delta` streaming, full tool call/result events, multi-turn state preservation on persistent stdin, and full prompt cache hit (`cache_read_tokens`).
  - **Account Safety & Anti-Ban**:
    - Never call `daily-cloudcode-pa.googleapis.com` directly from third-party HTTP clients with extracted OAuth tokens (Google enforces Go TLS fingerprints, telemetry, and policy guardian, banning third-party spoofed clients).
    - All cloud communication, OAuth keyring authentication, and prompt caching are strictly handled by the official local binary.
  - **Continuation Model & Storage Abstraction**:
    - [`KodexAgentStorage.kt`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/core/src/specMain/kotlin/io/github/stream29/kodex/agentstorage/contract/KodexAgentStorage.kt#L16-L25) maintains its 6 public timelines (`index`, `work`, `settings`, `timestamp`, `tokenCount`, `unstable`) as the single source of truth for conversation history and UI rendering.
    - An internal sparse timeline `agyContinuation: IndexVersioned<AgyContinuationAnchor>` records `(cascadeId, stepIndex, syncedKodexIndex)` at each Antigravity-synced index.
    - Because `AgyContinuationAnchor` lives in an `IndexVersioned` timeline, [`MutableKodexAgentStorage.revert`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/core/src/specMain/kotlin/io/github/stream29/kodex/agentstorage/contract/KodexAgentStorage.kt#L71-L78) and `forkUntilInclusive` automatically handle anchor rollback and cloning.
  - **Cross-Provider Switching Matrix (OpenAI Codex <-> Antigravity)**:
    - **`agy` -> OpenAI**: Zero-cost direct continuation. Every `agy` step (reasoning, tool calls, tool results, assistant responses) is persisted as [`StableCleanEvent.kt`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/clean-models/src/specMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableCleanEvent.kt), which already implements [`CleanOpenAiEvent.kt`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/clean-models/src/specMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/CleanOpenAiEvent.kt). OpenAI builds requests directly from clean storage.
    - **OpenAI -> `agy` (Delta Bridge)**:
      - If consecutive `agy` turns (`anchor.syncedKodexIndex == prevTurnEndIndex`): native continuation on `anchor.cascadeId` (100% prompt cache hit).
      - If first entry to `agy` (`anchor == null`): calls `StartCascade` and prefixes prior clean history as a `<conversation_history_handoff>` block.
      - If switching back (`OpenAI -> agy -> OpenAI -> agy`, where `anchor.syncedKodexIndex < prevTurnEndIndex`): reuses `anchor.cascadeId` (preserving prior prompt cache) and only bridges the delta `(anchor.syncedKodexIndex .. prevTurnEndIndex]` as a `<conversation_history_delta>` block.
  - **Runtime & Execution Layer Alignment**:
    - Branching occurs in [`ResumableAgentLayer.kt`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/contract/src/specMain/kotlin/io/github/stream29/kodex/agentruntime/contract/ResumableAgentLayer.kt#L13-L24):
      - When `settings.model` is OpenAI: routes to `KodexToolRuntime(KodexAgentCompactionRuntime(openAiState))`.
      - When `settings.model` is Antigravity: routes to `AgyResumableAgentLayer`, which drives the local harness, streams step updates into clean events, forwards `pendingSteer` during the turn, and maps `ask_question` to `ToolPending(RequestUserInput)` for `completeToolCall` handling.
    - Contracts for [`AgentRuntimeRpc.kt`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/client/src/specMain/kotlin/io/github/stream29/kodex/rpc/contract/AgentRuntimeRpc.kt), [`AgentRuntime.kt`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/contract/src/specMain/kotlin/io/github/stream29/kodex/agentruntime/contract/AgentRuntime.kt), and TUI components remain completely unchanged.
