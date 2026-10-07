# Task Tree

- `Confirm the historical Application reporter and RPC Stop boundary`()
- `Restore the actual CLI reporter and reuse the local Hook executor`()
- `Test cancellation, real commands and backend/local input separation`()
- `Build a distinct binary without replacing the sealed or installed executable`()
- `Repeat the two-process lease conflict and subsequent operation`()
- `Record the precise delta and remaining acceptance limits`()
- `Record user acceptance and commit/push/release authorization`()

# Details

- Parent: [sealed CLI core E2E](2026-10-07-accept-sealed-cli-core-e2e.md).
- User accepted the scoped result and explicitly requested commit, push, merge
  into main, and publication; selected release version **v0.4.10**.
- User confirmed the original behavior: unhandled operations were reported and
  observable through Hook. Historical `8397a155^` Application supplied that
  reporter; the RPC CLI Main omitted the replacement host callback.
- Restore the existing typed Root callback with an Application-owned reporter,
  not a new error center or a change to lease exclusivity.
- Local failures select the existing Unhandled error Hook and provide local JSON
  `{"type":"unhandled_error","message":...}`. They are not backend Agent Stops:
  no Session index is fabricated and no RPC notification union changes.
- Backend and frontend-local inputs share the one actual serial command executor,
  configuration snapshots and timeout/cleanup rules. Cancellation is not reported;
  failed Hook commands remain diagnostic and cannot recursively trigger Hook.
- Local input is live, replay-free and best-effort, using 64 buffered messages
  with oldest-drop rather than an unbounded launch per error.
- Targeted regression must use actual application/RPC/command execution, preserve
  backend Stop JSON, and prove local errors do not end the frontend owner.
- New native output has a distinct immutable source-patch hash and artifact hash.
  Original sealed output/evidence and the user's installed binary remain unchanged.
- The original repair run did not authorize commits; the subsequent explicit
  submission/publication request now authorizes this delta.

## Validation and distinct artifact

- Four targeted JVM tasks passed with 124 reported tests and zero failures:
  notification 14, Application 52, Root View 54, CLI 4. The new actual
  Application test executes local commands through the same consumer as real
  backend Stops, ignores cancellation, and proves failed Hooks do not recurse or
  end the host. It is not a replacement for native interaction.
- The independent detached `7b23c481` checkout applied only six changed files.
  Source patch SHA-256:
  `68c44d25e494144c80b7844f38b5112bff65c539855b36929b0a9dad941222b2`.
  Fixed submodule commits matched; source hashes matched before and after build.
- Complete Linux x64 release build passed in **6m1s**. The
  [distinct executable](file:///home/stream/ACodeSpace/local/kodex-e2e-20261007-unhandled-restore/artifacts/kodex)
  is 73,409,096 bytes, SHA-256
  `a47e439da30f60080092394393f3a6cda69e95a9ad455e0919c1463d129d2ea1`.
  Provenance records the base plus **uncommitted patch**, not a fictitious new commit.
- [Actual CLI outlet](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L34)
  supplies the
  [Application reporter](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L98).
  No new VM, error center, RPC union or control Hook was introduced.

## Actual native recheck

- Two CLI processes used a fresh private copy of the generated acceptance Home.
  The first held Session 0; the second attempted to open it **twice**.
  Each failure produced exactly one local `unhandled_error` Hook JSON, with
  only `type`/`message`, while the second CLI and Catalog remained usable.
- The same second CLI then created Session 2 and completed one real
  `gpt-6-luna`/low request with output **RESTORE-OK**. Automatic title generation
  was disabled for this fixture to avoid an extra model request.
- Hook received one unchanged backend `stop_assistant_message` containing the
  actual Session 2 index and response. Thus local errors did not masquerade as
  Agent Stops and the single executor still handled genuine backend notifications.
- The original lease stayed with the first PID; all durable Session 0 bytes
  compared equal to the copied original. Both processes exited 0; a third run
  reopened Session 2 and rendered the persisted response, then also exited 0.
- The isolated credential copy was removed through actual logout. Original
  credentials remain; no remote revoke/reset/forced refresh was requested.
  No Session leases or owned CLI/PTY children remain.
- [Native receipt](file:///home/stream/ACodeSpace/local/kodex-e2e-20261007-unhandled-restore/acceptance-results/e2e-receipt.json#L1),
  events, per-run screens, exact Hook input and lease identity evidence are retained.
  The temporary build checkout and this run's build/controller processes are gone.
- All **5303 original sealed-file checksums** still pass. The installed CLI target
  retains its original checksum and was not overwritten.

## Scoped closure

- The confirmed two-process host-exit blocker is closed on this distinct binary.
  Core E2E plus this restoration recheck support proceeding with integration;
  they do not claim every OS, browser OAuth, fault-injection or History saturation
  scenario passed.
- This fix restores the existing typed History/Catalog failure callback at the
  actual CLI host. It does not rewrite every raw coroutine callback in the UI or
  claim all possible unhandled failures now share a tested path.
- At the end of the repair run, six product files and scoped documentation were
  uncommitted; no push or installed-binary replacement occurred. Subsequent
  submission is tracked in the [completed release task](2026-10-07-release-kodex-0-4-10.md).
- User is taking the local machine for gaming. No further local build/runtime
  validation is permitted in this work; subsequent validation uses
  `xiaoxin-ubuntu` over SSH in an independent workspace, not its existing user
  research checkout. This run's local processes were already released.
