# Task Tree

- `Locate the existing steer delivery and interruption boundaries`()
- `Define earlier steer delivery with existing runtime capabilities`()
- `Specify focused changes and regression checks`()
- `Wait for explicit implementation authorization`()

# Details

- User-selected follow-up from the Codex update review: faster Steer is the only additional candidate to pursue. The separately queued Ultrafast task remains unchanged.
- User assessment: This is straightforward work that Kodex could already do but had not implemented; treat it as an improvement to existing steer responsiveness, not a new protocol integration.
- Status: Queued in discussion; implementation has not started.
- Goal: Let follow-up instructions affect a running task sooner while retaining current-turn Steer semantics.
- Existing behavior: `SteerRuntimeImpl.resume()` checks pending input before and after `delegate.resume()`, and delivery requires `canAppendUserMessage`. See [runtime, lines 17–27](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/steer/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/steer/SteerRuntimeImpl.kt#L17).
- Scope to settle: Identify where earlier delivery is safe during model generation and tool execution, reusing the existing runtime and interruption capabilities. Do not make WebSocket adoption or a new OpenAI endpoint a prerequisite.
- Planning checks:
  - Follow-up input is delivered earlier in the agreed running states, with a deterministic test of the delivery timing.
  - Multiple inputs preserve order and are neither lost nor delivered twice.
  - Partial model output and tool-call/output history remain valid; interruption does not duplicate executed side effects.
  - Queue-versus-Steer behavior, cancellation, and Session persistence remain coherent.
- Context: [October 8 official faster-steering announcement](https://help.openai.com/en/articles/6825453-chatgpt-release-notes) and [local candidate review](../../shared-context/findings/2026-10-09-codex-support-candidates.md).
