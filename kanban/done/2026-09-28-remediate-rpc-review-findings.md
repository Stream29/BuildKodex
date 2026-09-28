# Task Tree

- `Review the four frontend findings and frozen contract`()
- `Review uncertain first-message submission`([first-message task](2026-09-28-rpc-uncertain-first-message.md))
- `Keep accepted settings writes alive`([settings-lifetime task](2026-09-28-rpc-settings-write-lifetime.md))
- `Allow retry after initial Session view failure`([view-retry task](2026-09-28-rpc-initial-view-retry.md))
- `Surface settings write failures`([settings-error task](2026-09-28-rpc-settings-failure-visibility.md))
- `Run focused and integrated frontend regression checks`()

# Details

## Scope and order

- This task follows the P08/P09/P11 implementation and a subsequent **static** review; the four findings are not claims of runtime reproduction.
- Handle one finding at a time in the order above. Do not mark the next child executable before the prior child is verified; keep its task-local evidence in that child.
- Preserve the [frozen RPC contract](../../checklist/rpc-architecture.md#rpc-契约冻结). None of these findings currently requires a contract change. If that changes, stop the affected fix for review rather than editing the contract.
- The large P01–P12 working tree and the separate native-segfault finding are existing work. Do not stage, revert, or reformat them as part of this task.
- No implementation is authorized merely by linking to an older Done task; the four child tasks govern these review fixes.

## Acceptance

- The synthetic lost-reply case is recorded as a deferred defensive risk, not counted as a reproduced CLI regression or silently changed.
- Accepted settings edits are not canceled just because the popup closes, and a failed edit is visible to the user.
- A failed first Session-view activation does not permanently poison retries for that index.
- Tests cover the affected race/failure paths, plus relevant JVM/Native and CLI compilation checks.

## Outcome

- First item was downgraded with the user's agreement: the lost-reply test injects a fault after a successful RPC call, and no retryable surviving-draft path has been demonstrated for the current in-memory CLI. No speculative first-message UX was added.
- Remaining three frontend defects were fixed sequentially with real RPC interleaving tests and safe rendering feedback. The frozen RPC contract, backend and the separate native-segfault investigation were not changed by this remediation.
- JVM and Linux x64 Native focused suites passed; CLI compiled for both. Changes remain uncommitted on `refactor/rpc` together with the earlier large refactor; no real Home, release or upstream fork was touched.

## Post-fix review and tmux smoke

- Rechecked settings write ownership, popup disposal, first Session-view failure cleanup, and error presentation against the frozen RPC contract. No deterministic blocking regression was found in these paths; the artificial lost-reply case remains deferred.
- Re-ran focused tests after the fixes: RPC frontend JVM/Linux x64 Native 59 each, settings view JVM 24, and application ViewModel JVM 24; all passed. Re-linked the current Linux x64 Native CLI.
- Exercised the Native CLI in a separate tmux socket with an isolated temporary `HOME`: changed the frontend newline key and a backend context-source checkbox, closed/reopened Settings, restarted the CLI, and confirmed both values persisted. Both CLI exits returned zero. No credentials or model request were used.
- The first independent tmux server loaded the user's tmux configuration and restored copies of named sessions inside that **separate** socket; it was terminated. The restart check used `tmux -f /dev/null -L ...`; the default server and its original sessions were untouched.
- This manual smoke does not inject a write failure or a first-activation failure; those are covered by focused automated tests, not claimed as manually reproduced.
