# Task Tree

- `Receive settings write lifetime result`()
- `Trace first-open failure and cached view state`()
- `Remove only terminal failed initial views`()
- `Test retry and concurrent callers`()

# Details

- Third review finding; [settings write lifetime](2026-09-28-rpc-settings-write-lifetime.md) is verified. Parent: [RPC review remediation](2026-09-28-remediate-rpc-review-findings.md).
- [RpcSessionViews.open](../../Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionViews.kt#L49-L55) caches a view before its first activation; [terminal initial failure](../../Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionViews.kt#L162-L173) leaves that same failed object in the map. [Registry.open](../../Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt#L24-L30) gets no handle on failure, so ordinary retry cannot release it.
- Do not evict a successfully opened view during transient `SessionNotActive` recovery; do not activate directory-only reads. Preserve missing-Session handling and shared tab ownership.
- Acceptance: initial failure then corrected backend and retry succeeds; concurrent callers do not receive a closed or leaked view; release/join checks pass.

## Result

- A terminal first-open failure now closes and joins the failed view before removing its exact map entry; a canceled waiter alone does not remove a view still initializing for another caller. No behavior change to an already-ready view's recovery loop.
- Four new tests cover unknown initial error, initial SessionNotFound, concurrent waiters, and cancellation of one waiter. Real RPC frontend module JVM/Linux x64 Native each passed 57 tests; Session ViewModel JVM passed 8 tests.
- No RPC contract or backend change. Original application navigation remains responsible for explicitly requesting a retry.
