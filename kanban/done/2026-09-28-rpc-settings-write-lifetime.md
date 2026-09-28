# Task Tree

- `Receive the first-message applicability decision`()
- `Trace popup command ownership and old write queue`()
- `Move accepted writes to the correct owner`()
- `Test close/write and cancellation races`()

# Details

- Second review finding; the [first-message risk](2026-09-28-rpc-uncertain-first-message.md) was deferred after review, without changing runtime behavior. Parent: [RPC review remediation](2026-09-28-remediate-rpc-review-findings.md).
- [RpcGlobalEditor.command](../../Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcGlobalEditor.kt#L76-L90) launches accepted edits under the popup-owned Job; [close](../../Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcGlobalEditor.kt#L258-L261) cancels it. A save followed immediately by close can abort a submitted write.
- Compare the original settings queue's lifetime before changing ownership. Separate disposable page observations from already accepted commands, and keep the [frozen contract](../../checklist/rpc-architecture.md#rpc-契约冻结) and CAS/unknown-outcome rules intact.
- Acceptance: deterministic close-after-save test; caller shutdown still releases resources; no silent loss or unintended replay.

## Result

- Reused the original SettingsUpdateQueue from the settings ViewModel module. Global and new-session defaults edits, local preference/Hook writes, and MCP configuration commits now drain in the application scope after popup close; preview, login, reset and page observations remain disposable. MCP's local source stays alive only until the accepted write queue drains.
- Session-bound editing retains its separate exact-binding cancellation policy. No RPC contract, backend operation or retry rule changed.
- Four new real in-memory RPC/frontend-file tests cover a stalled global CAS, stalled defaults CAS, stalled MCP CAS and Hook update immediately followed by popup close.
- `app-viewmodel-rpc` JVM/Linux x64 Native: 53 tests each, zero failures; `app-viewmodel-settings`: 7 each, zero failures. `app-cli:compileKotlinJvm` passed. Existing Gradle configuration-cache warnings remain unrelated.
