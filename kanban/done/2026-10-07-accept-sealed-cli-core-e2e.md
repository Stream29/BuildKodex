# Task Tree

- `Verify the unchanged sealed executable and isolated resources`()
- `Exercise actual Settings, authenticated Luna and tool operations`()
- `Exercise persistence, compaction, History, forks and deletion`()
- `Exercise startup rejection, lease conflicts and write failures`()
- `Release owned resources and retain exact reproduction evidence`()
- `Resolve the confirmed CLI operation-error exit before acceptance closure`()
- `Record acceptance of scoped E2E and submission/publication authorization`()

# Details

## Baseline and authorization

- User requested immediate E2E because other main-branch work is blocked, then
  explicitly authorized existing credentials and the inexpensive 6LUNA model.
- Used the sealed Linux x64 executable at commit `7b23c481`, SHA-256
  `5d9276ecd5781d028a2b71bcc170f4fb5b3924f5e111aaaae08baf34e0d84b87`.
  Initial E2E used that unchanged binary. The distinct restoration delta below
  does not replace its sealed evidence or the installed binary.
- [Preparation and full matrix](../executable/2026-10-07-seal-repair-baseline-and-build-e2e-cli.md#full-e2e-matrix-to-execute-on-this-binary)
  remains the parent. This record is actual native CLI acceptance, not another
  JVM test count or a claim that every fault/platform matrix passed.
- All model operations selected `gpt-6-luna`, low effort, including the title
  configuration. Credentials were copied only into the private acceptance Home;
  both copies were removed through the application's actual logout flow.
  Original credentials remain present, were not removed or explicitly refreshed.
  No reset credit, remote revoke or browser sign-in was triggered.
- Initial copied Codex credentials belonged to a different account with exhausted
  quota. A credential-safe curl diagnostic returned HTTP 429
  `usage_limit_reached`; repeated CLI requests were stopped. Switching through
  Settings to the user's existing Kodex source enabled actual model responses.
  This quota limitation is not a migration failure.

## Actual native results

| Scope | Result and evidence |
| --- | --- |
| Fresh Home and lazy repository | Initial draft and Settings did not create `sessions/`; first submitted draft created Session 0. |
| Preferences | Input preference persisted and survived process restart; restored Shift+Enter/newline and Enter/submit. |
| Sources and authentication | Both sources loaded safe summaries; selected Kodex source completed actual responses. Removing one isolated source did not remove the other; final both copies absent and original files retained. |
| Luna and error recovery | Actual assistant output `E2E-LUNA-OK`; after logout, Resume rendered one operation failure and left Agent/CLI usable with retained history. |
| Shell | Actual command tool executed `printf 'SHELL-OK\n'`; workgroup persisted and rendered after reopening. |
| Patch | Actual apply_patch created `acceptance-workspace/e2e-patch.txt` with `PATCH-OK`. |
| MCP | Actual stdio process initialized, listed one tool, received `tools/call`, and Agent finished `TOOLS-OK`. Disable/enable started another actual SDK connection. |
| Ambiguous MCP configuration | Adding `e2e_mcp` against existing `e2e-mcp` reported Settings failure; rejected entry did not persist and existing server remained healthy. |
| Hook | Locally configured assistant notification command produced `hook-observed.txt`. |
| Pending question | Actual `request_user_input` displayed PASS/FAIL; selected PASS survived tab round-trip; Submit resumed the model and produced PASS. |
| Draft and tab ownership | Sibling draft text survived pending interaction, fork deletion and tab switching. |
| Automatic title | Session 1 acquired an automatic title; catalog observed it and reopened its `BACKGROUND-OK` history. This does not prove title completion happened after its view closed. |
| Compaction | Actual remote compaction displayed in-flight status. Name write at settings index 26/window 0 survived compaction index 27/window 1 as `E2E Compaction Name`. |
| Fork/revert/delete | Catalog fork created index 2 from index 1; opened actual inherited history, renamed, reverted-and-edited into Composer, then deleted only index 2. Index 1 and Session 0 retained. |
| Archive | Archived index 1 disappeared from normal catalog, reappeared with Show archived, and archive marker survived restart. |
| History | Real index Check out and wheel input; before/after tab round-trip all 45 screen rows were equal. Persisted history, question answer and compaction marker rehydrated on process restart. |
| Home upgrade and rejection | Real 0.4.6 Home upgraded to 0.4.8 and split legacy settings; sentinel remained. Future version and malformed YAML rejected startup without overwriting fixtures. |
| Settings I/O failure | Read-only isolated settings parent caused real write failure, old preference remained, CLI stayed alive; restoring permissions and explicit retry succeeded on the same queue. |
| Shutdown | Normal owned runs exited 0; no lease files or own MCP/CLI child processes remained. |
| Cross-process Session conflict | Original binary exited on the occupied-Session error, reproduced twice. Distinct restored-host binary repeated both attempts without exit, delivered exactly two local Hook inputs, and completed a subsequent real Luna request. |

## Confirmed failure: recoverable catalog error exits the entire CLI

- Runs `secondary` and `lease-repro` each started a separate real native CLI on
  the same acceptance Home. Opening the Session held by the first process
  printed the lease-owner diagnostic and exited 0 before any Ctrl+C.
- The original process remained usable and retained its lease. No takeover or
  data corruption occurred; the defect is the failure outlet, not exclusivity.
- The sealed binary's CLI Main did not supply a nonfatal `onOperationFailure`.
  [Root renderer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L119)
  defaults it to throwing and passes it to Catalog.
  The Catalog catch reports once, but that default escapes the composition
  coroutine and closes Mosaic; Main prints `Unable to run Kodex` and returns.
- User confirmed that before this refactor the Application reported unhandled
  errors to Hook. Historical Application source confirms that host outlet;
  restore it rather than inventing a new failure UI or changing lease behavior.
  The actual Root callback receives the reporter, cancellation and exact targets
  remain unchanged, and local errors do not fabricate backend Agent Stops.
- [Restoration delta](2026-10-07-restore-cli-unhandled-error-hook.md) tracks the
  independent new artifact and two-process recheck.
- Recheck with the same two-process reproduction, a subsequent successful
  operation in the second CLI, first-process lease unchanged, and clean shutdown.
  Preserve this binary/results; any repaired build receives a distinct baseline.
- That recheck now passed on the distinct restored-host binary: two local Hook
  reports, subsequent **RESTORE-OK** from real Luna, unchanged incumbent lease
  and durable data, three native exits 0, and persisted response reopened.
  The original failure traces remain valid evidence for the old binary.
- User subsequently requested commit, push, merge into main and release
  **v0.4.10**, accepting this scoped closure rather than declaring every remaining
  platform/fault matrix complete.

## Evidence, cleanup and limits

- Mutable E2E evidence is under
  [acceptance-results](file:///home/stream/ACodeSpace/local/kodex-e2e-20261007-7b23c481/acceptance-results/):
  `events.json`, per-run native screens, `mcp-observed.jsonl`, provider diagnostic
  and the E2E receipt. Sealed baseline evidence is not overwritten.
- Harness mistakes are retained: an initial expectation incorrectly assumed
  Shift+Enter submit instead of actual Ctrl+Enter; two attempted Settings clicks
  while Catalog still owned its overlay were rejected. These are corrected
  driver assumptions, not successful assertions or product failures.
- Full browser OAuth, deliberate token refresh, MCP OAuth/reconnect fault races,
  background-title-after-view-close ordering, disk partial-publication injection,
  long-history saturation and other OS/PTY matrices are not closed by this run.
  Existing B2/U remain separate from this confirmed interactive failure.
- The original E2E required no source/build/IDE changes. The restoration delta
  is now separate and uncommitted; no original credential deletion,
  quota-reset consumption or user process stop occurred.
- Core E2E and the confirmed restoration recheck passed. This supports integration
  without treating unexecuted browser/platform/fault matrices as migration blockers
  or falsely marking those matrices complete.
