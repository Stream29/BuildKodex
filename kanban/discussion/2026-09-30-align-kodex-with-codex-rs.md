# Task Tree

- **`Maintain the shared alignment direction and coordinate scoped downstream tasks`()**
- `Review the refreshed codex-rs range against Kodex's current contracts`()
- `Inventory model, Responses transport, retry, MCP, and session-state drift`()
- `Separate compatible protocol updates from architecture-gated features`()
- `Specify implementation batches, fixtures, and compatibility checks`()
- `Wait for explicit implementation authorization`()

# Details

## 共同上游与任务关系 — 2026-10-19 / Session945

- 用户明确将 Codex 对齐作为多条工作的共同上游；具体能力拆到下游卡讨论、规划和交付，本卡不是一张必须整体完成才能推进下游的串行门禁。
- [Backend Daemon](../done/2026-10-13-design-backend-daemon.md)：已完成的架构下游；[生命周期治理](../done/2026-10-14-discuss-daemon-coroutine-lifecycle.md)也已按用户确认结项，未来遇到具体问题再优化。
- 活跃下游完整保留：[Session 内 Agent 层级](2026-10-19-restore-session-agent-hierarchy.md) → [层级 UI](2026-10-19-design-session-agent-hierarchy-ui.md)与[subagent 工具及统一上下文](2026-10-19-redesign-subagent-tools-and-execution-context.md) → [Tool as CLI](2026-10-03-discuss-integrate-tool-cli-into-kodex.md)。中间两项协调设计，对应工具 CLI 化等待前置交付。
- [Responses 用途归因](2026-10-13-complete-responses-attribution-metadata.md)是限定范围的协议对齐下游，不因共同上游而扩大其授权。
- [Antigravity 接入](2026-10-19-discuss-antigravity-integration-via-kotlin-native.md)是与大部分工作正交的纯探索性讨论，不设为上述交付链的前置或完成条件。

## 原始对齐范围

- User requested a follow-up task for aligning Kodex with the refreshed `codex-rs` reference.
- Upstream reference baseline: `d42056091aded7feb1d88ac7e83972108b2aa478`; refresh evidence is recorded in [Codex upstream refresh](../../shared-context/findings/2026-09-30-codex-upstream-refresh.md).
- Known candidate areas:
  - model catalog capabilities currently absent from `ModelInfo`, including visibility, Responses Lite, WebSocket preference, search-tool support, verbosity, and multi-agent metadata;
  - server retry advice and `Retry-After` propagation through Responses retries and transport fallback;
  - request and response tool-metadata budgets and omission semantics;
  - MCP authorization discovery, OAuth lifecycle, credential boundaries, and connection reuse;
  - history-aware WebSocket prewarming and continuation state;
  - app-server protocol additions, subject to the frozen Kodex RPC contract.
- Current boundary: this discussion records alignment work only. It does not authorize changing the RPC wire contract, adding WebSocket or Responses Lite transport, changing MCP credential ownership, or copying upstream orchestration and prompt policy.
- Planning checks must compare Rust protocol/source models with Kotlin DTOs, preserve existing storage and Session semantics, add fixtures for each wire change, and validate HTTP/SSE behavior before considering transport expansion.
