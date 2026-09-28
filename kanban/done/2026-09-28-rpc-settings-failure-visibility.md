# Task Tree

- `Receive initial Session view retry result`()
- `Trace settings error projection and consumers`()
- `Connect failure to an existing user-visible channel`()
- `Test remote failures and popup disposal`()

# Details

- Fourth review finding; [initial view retry](2026-09-28-rpc-initial-view-retry.md) is verified. Parent: [RPC review remediation](2026-09-28-remediate-rpc-review-findings.md).
- [RpcGlobalEditor.failure](../../Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcGlobalEditor.kt#L31-L32) and the draft-settings failure projection currently have no observed UI/log consumer, while commands catch exceptions. A failed edit may disappear without feedback.
- Coordinate with the earlier settings-write lifetime fix; page close must not discard feedback for an already accepted write. Prefer existing frontend feedback facilities; do not add a notification RPC or change the [frozen contract](../../checklist/rpc-architecture.md#rpc-契约冻结).
- Acceptance: injected remote write failure is visible, cancellation is not displayed as a business failure, and successful edits do not leave stale failure UI.

## Result

- Unknown global/default settings failures now reach one application-owned local StateFlow, so a write finishing after popup close is shown on the next open. The Settings popup renders a generic dismissible warning; only the exception class is logged, never the remote message or raw configuration.
- Existing page-owned operations still propagate cancellation normally. Successful accepted edits or explicit dismissal clear the warning. This changes only the frontend app contract, not the frozen RPC contract or the backend Notification flow.
- Two new real-RPC failure tests and two renderer snapshot tests passed. RPC frontend, Settings view and Application ViewModel each passed JVM/Native test suites (59/24/24 tests respectively); CLI JVM and Linux x64 compilation passed.
