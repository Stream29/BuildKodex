# Task Tree

- **`Design Antigravity Native Connection Layer for Kotlin/Native`()**
  - `Specify dual-transport architecture (LocalHarness WebSocket + Agy Connect-RPC)`()
  - `Define process supervisor and lifecycle manager for local Go daemon`()
  - `Specify stdio handshake protocol (4-byte length-prefixed InputConfig/OutputConfig)`()
  - `Define WebSocket event loop and frame serialization using Ktor in Kotlin/Native`()
  - `Define Connect-RPC HTTP client for agy CLI binary fallback`()
- `Design Protobuf and Clean Event Bidirectional Mapping`()
  - `Map localharness InputEvent (Prompt, ToolResult, Confirmations) from Kodex state`()
  - `Map localharness OutputEvent / StepUpdate (Thinking, Text, ToolCall) to StableCleanEvent`()
  - `Map ask_question interactions to ToolPending(RequestUserInput) and completeToolCall`()
- `Design Storage and Continuation Integration`()
  - `Introduce internal sparse timeline agyContinuation: IndexVersioned<AgyContinuationAnchor>`()
  - `Implement lazy revert and lazy fork synchronization`()
  - `Implement cross-provider Delta Handoff (OpenAI <-> Antigravity)`()
- `Design ResumableAgentLayer Runtime Dispatcher`()
  - `Create AgyResumableAgentLayer composing state and connection transport`()
  - `Wire multi-provider branching in AgentRuntime.resume() by settings.model`()
  - `Verify zero changes to AgentRuntimeRpc, TimelineRpc, and TUI contracts`()
- `Draft checklist and validation suite`()
  - `Record SOP and design decisions under checklist/antigravity-native-runtime.md`()
  - `Define test cases for handshake, streaming, tool mapping, revert, and switching`()

# Details

## 定位与状态 — 2026-10-19 / Session945

- 按用户确认，本卡从 planning 移回 discussion，保留文件名和候选方案；与[接入探索](2026-10-19-discuss-antigravity-integration-via-kotlin-native.md)同属纯探索性讨论。
- 本线与大部分主线工作正交，不阻塞 Codex 对齐、Session 层级、subagent 或 Tool as CLI。
- 下方 Task Tree、模块分解和传输方案均为探索候选，尚有 stdio 与双传输方案未收敛；不作为已批准实施计划，不在本次状态整理中替用户选择方案。

- Context:
  - The user requested identifying the most cost-effective / highest ROI (性价比最高) solution under the premise that the user already has `/home/stream/.local/bin/agy` installed locally.
  - Reverse-engineering and empirical testing of `/home/stream/.local/bin/agy` (v1.4.0) reveals three potential local integration paths:
    1. **Tier 1 (Highest ROI, Zero Extra Dependencies): Headless Stdio `stream-json` (`AgyStdioStreamJsonTransport`)**:
       - Command: `agy --input-format stream-json --output-format stream-json --dangerously-skip-permissions -p=`
       - Transport: Pure Unix pipes (stdin/stdout) via POSIX `posix_spawnp` / standard pipes.
       - Zero port extraction, zero CSRF tokens, zero HTTP/WebSocket connection management.
       - Live verified: multi-turn conversation memory, real-time token streaming (`step_update.text_delta`), tool execution telemetry, and prompt cache hits (`cache_read_tokens`).
    2. **Tier 2: Connect-RPC over HTTP (`AgyConnectRpcTransport`)**:
       - Embedded `LanguageServerService` on `http://127.0.0.1:{port}/exa.language_server_pb.LanguageServerService/<Method>`.
       - High complexity: dynamic port sniffing, CSRF extraction, Protobuf framing.
    3. **Tier 3: Standalone `localharness` WebSocket (`LocalHarnessWebSocketTransport`)**:
       - Protocol extracted from Python SDK wheel; requires bundling or downloading the standalone `localharness` Go binary.
  - **Decision**: Adopt **Tier 1 (`AgyStdioStreamJsonTransport`)** as the primary implementation for Kodex. It delivers maximum developer ROI, minimum code footprint, zero new dependencies, and rock-solid reliability.
- Module Decomposition:
  - **Transport Interface (`AntigravityTransport`)**:
    - Abstraction defining `start(workspaceUris, env)`, `send(inputEvent)`, `receiveEvents(): Flow<AntigravityEvent>`, `cancel()`, `revert(stepIndex)`, `fork(stepIndex)`, and `stop()`.
    - Implemented by `LocalHarnessWebSocketTransport` and `AgyConnectRpcTransport`.
  - **Process Supervisor (`AntigravityProcessSupervisor`)**:
    - Discovers binary candidate in order:
      1. Explicit config `antigravity.harnessPath` / `ANTIGRAVITY_HARNESS_PATH`.
      2. System `PATH` (`localharness` or `agy`).
      3. User home cache `~/.gemini/antigravity/bin/localharness` or `~/.local/bin/agy`.
    - Spawns process with pipes: `stdin`, `stdout`, `stderr`.
    - Handles graceful shutdown (`SIGTERM`, 3s timeout, `SIGKILL`) attached to Kodex Session lifecycle.
  - **Stdio Handshake Protocol (`LocalHarnessWebSocketTransport`)**:
    - Step 1: Serialize `InputConfig(storageDirectory = ..., clientInfo = ClientInfo("kotlin", "kodex-1.0", ...), env = ...)` to Protobuf bytes `bytes`.
    - Step 2: Write 4-byte little-endian length prefix `[len(bytes)]` + `bytes` to process `stdin` and flush.
    - Step 3: Read 4-byte little-endian length prefix from process `stdout`, then read payload bytes.
    - Step 4: Parse `OutputConfig`, extracting `outputConfig.port` (uint32) and `outputConfig.apiKey` (string).
    - Step 5: Establish Ktor WebSocket client connection to `ws://127.0.0.1:$port/` with header `x-goog-api-key: $apiKey`.
    - Step 6: Send `InitializeConversationEvent` frame containing workspace paths, model target, and tool capabilities.
  - **CLI Connect-RPC Protocol (`AgyConnectRpcTransport`)**:
    - Spawns `agy --input-format stream-json --output-format stream-json --log-file <path> -p=`.
    - Extracts `port` from log line `Language server listening on random port at <port> for HTTP` and `csrfToken` via bootstrap handshake.
    - Sends HTTP POST Connect-RPC requests to `/exa.language_server_pb.LanguageServerService/<Method>`.
- Event Mapping Plan:
  - `InputEvent` (Client -> Harness):
    - `user_message` / `prompt`: Encodes user text and image content items.
    - `tool_result`: Delivers completed tool output (e.g. for `ask_question`).
    - `cancel`: Aborts active generation turn.
  - `OutputEvent` / `StepUpdate` (Harness -> Client):
    - `thought_delta` -> mapped to `KodexAgentStateValue.RequestResponse.Reasoning` and [`StableWorkEvent`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/clean-models/src/specMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableWorkEvent.kt).
    - `text_delta` -> mapped to `KodexAgentStateValue.RequestResponse.Message` and [`StableIndexEvent`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/clean-models/src/specMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableIndexEvent.kt).
    - Built-in tool executions (file edits, bash commands, searches) completed by the harness -> mapped to corresponding [`StableCleanEvent.CompletedTool`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/clean-models/src/specMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableCleanEvent.kt) records.
    - `user_question` interaction -> pauses in `ToolPending(RequestUserInput)` and waits for `completeToolCall`.
- Storage & Continuation Architecture:
  - Add internal timeline `agyContinuation: IndexVersioned<AgyContinuationAnchor>` to [`KodexAgentStorage.kt`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/core/src/specMain/kotlin/io/github/stream29/kodex/agentstorage/contract/KodexAgentStorage.kt).
  - Lazy Revert: When `revertHistory(targetIndex)` is called, storage truncates `agyContinuation` to `<= targetIndex`. When the Antigravity runtime resumes, it checks the latest anchor and sends `RevertToCascadeStep(stepIndex = anchor.stepIndex)`.
  - Lazy Fork: When `forkSessionHistory(targetIndex)` creates a new session, it calls `ForkConversation` on first resume to spawn a distinct `newCascadeId`.
  - Delta Handoff: When switching from OpenAI to Antigravity, clean events in `(anchor.syncedKodexIndex .. prevTurnEndIndex]` are rendered into a structured history delta block.
- Boundary Invariant Guarantees:
  - Zero modification to [`AgentRuntimeRpc.kt`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/client/src/specMain/kotlin/io/github/stream29/kodex/rpc/contract/AgentRuntimeRpc.kt), [`AgentRuntime.kt`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/contract/src/specMain/kotlin/io/github/stream29/kodex/agentruntime/contract/AgentRuntime.kt), or TUI view models.
  - Zero external Python dependency; Kodex remains a standalone native binary.
  - Full adherence to Google CloudCode anti-ban constraints by delegating cloud networking to the official binary.
