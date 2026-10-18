# Task Tree

- `Confirm submission must preserve focus and release version 0.5.0`()
- `Keep submitting Composer focusable and validate regression tests`()
- `Commit the validated focus fix`()

# Details

- User authorized a minimal focus fix, commits, and publication as v0.5.0.
- Submitting previously disabled TextInput, removing its focus target and falling back to Sessions.
- Keep the target attached while read-only; block editing, paste, undo/redo and duplicate Enter.
- Preserve explicit disabled/request-input behavior and user-initiated focus navigation.
- Validate focus across submitting/success/failure, no forced focus recovery, blocked edits, and normal editing afterward.
- Preserve unrelated working changes, including Kodex/gradle.properties; build only clean committed product sources.
- Follow the release-kodex skill; keep historical Home migrations and protocol versions unchanged.
- Publication is recorded in [the completed release task](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-18-release-kodex-0-5-0-940.md#L1).

## Validation and commit

- Original production code with the final regression tests: 13 JVM cases, exactly two failures at `Submitting must not move focus to Sessions`; the navigation case and existing cases passed.
- Final candidate: Composer 13 and shared components 166 cases on both JVM and macOS Native, 358 total, zero failures/errors/skips.
- Tests wait for displayed submission transitions and drain queued terminal input through Mosaic frames; early test-driver attempts are not counted as regression evidence.
- The verified Mac patch and submitted local patch have identical full-index SHA-256 `a8eecb9c1bf8bf946d9cabaafae5cce5a04fe19889514f59966f8de03f1c07a9`.
- Signed product commit: `84f35bbe2` (`fix: preserve composer focus while submitting`), limited to three production files and two test files.
