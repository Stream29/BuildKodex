# Task Tree

- `Check concurrent CLI use and credential freshness`()
- `Run one bounded live conversation in isolated tmux`()
- `Exercise one user-input interaction and inspect rendering`()
- `Exit, remove copied credentials, and record evidence`()

# Details

## Scope

- User explicitly authorized a real-credential test of streaming, rendering, and interaction after the isolated smoke in [the review remediation](../done/2026-09-28-remediate-rpc-review-findings.md#post-fix-review-and-tmux-smoke).
- Use the current `refactor/rpc` Native CLI in a separate `tmux -f /dev/null -L ...` server. Do not send input to the user's existing CLI or default tmux server.
- Existing real Home is occupied by another CLI and must not be migrated or written. Use an empty temporary Home and working directory; copy only the required authentication file with mode 0600, configure the matching source and disable automatic title generation. Do not display tokens or preserve a transcript containing secrets.
- Send a short benign prompt and answer one safe user-input request. Do not run shell tools, spawn subagents, read project files, perform resets, or invoke Hooks. Limit model use to this test and stop if credentials, server, or terminal misbehave.

## Acceptance

- Confirm at least one real assistant response arrives incrementally and renders in the terminal.
- Confirm an interactive request can be answered and the conversation resumes to a final answer.
- Confirm clean exit, no change to the real Home or default tmux server, and no copied credential left behind.
- If the model does not issue the requested interaction, report the limitation rather than fabricating a pass.

## Outcome

- An existing CLI was using the real Home, so the live test used a separate 0700 temporary Home and work directory. A still-valid Kodex authentication file was copied with mode 0600; no credential value was printed. The test selected the Kodex source, disabled automatic title generation and external context sources, and used `gpt-6-sol` with low reasoning effort.
- A first fixture that placed only one split settings file before Home migration was correctly rejected as ambiguous and removed. A fresh isolated Home was then initialized to 0.4.7 before adding the copied credential and test settings. The real Home was neither migrated nor written.
- In `tmux -f /dev/null -L ...`, the live assistant rendered an initial response, issued `request_user_input` with two color choices, accepted the selected blue option, and rendered the correct final reply. A second response rendered incrementally: repeated 0.25-second captures observed 1 through 9 lines before all 20 lines appeared.
- The CLI exited with status zero, then reopened the isolated Session and rendered the saved text and interaction; the second exit also returned zero. Its test log had no `ERROR`, `FATAL`, or `Exception` lines.
- Both isolated tmux servers and the entire temporary Home (including the copied credential and test Session) were removed. The default `ACodeSpace`, `game`, and `home` tmux sessions and the existing user CLI process were not touched. No source or frozen RPC contract was changed for this smoke test.
