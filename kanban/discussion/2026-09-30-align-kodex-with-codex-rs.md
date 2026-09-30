# Task Tree

- `Review the refreshed codex-rs range against Kodex's current contracts`()
- `Inventory model, Responses transport, retry, MCP, and session-state drift`()
- `Separate compatible protocol updates from architecture-gated features`()
- `Specify implementation batches, fixtures, and compatibility checks`()
- `Wait for explicit implementation authorization`()

# Details

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
