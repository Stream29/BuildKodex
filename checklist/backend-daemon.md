# Backend Daemon — Confirmed Scope

- These requirements are implemented on product `main` at `082bfc66`.
  Source integration does not imply a published release or an installed binary update.
- Implementation and evidence are tracked in the
  [Daemon task](../kanban/done/2026-10-13-design-backend-daemon.md).
  Preserve previously accepted frontend/backend business boundaries unless a
  requirement below explicitly replaces them.

## Deployment and entrypoints

- First release: same machine, same OS user, multiple TUI clients. No remote
  listening, SSH integration, TLS setup or cross-device filesystem support.
- Each user has one production Home, fixed at `~/.kodex`, and one backend.
  The backend process cwd and business notification Hook cwd are that Home;
  Agent tools retain their explicitly configured project working directories.
- One executable: `kodex` is TUI, `kodex backend start` runs the backend in
  the current process, and other commands are CLI entrypoints.
- The bootstrap before TUI connects to an existing backend or starts one
  independently in the background, then waits for readiness. TUI itself does
  not stop, restart or replace the backend.
- A held instance lock with an unavailable/stopping backend is startup failure,
  not permission to steal the lock or wait and start a replacement.
- A TUI unable to establish its backend connection fails startup. No offline
  TUI or persisted settings-cache fallback is required.

## Discovery, authentication and compatibility

- Backend owns a filesystem single-instance lock throughout its resource
  lifetime. Its record contains the bound port and this instance's random
  password; clients read it without taking backend ownership.
- Bind loopback only, using an OS-assigned port. Generate a new random password
  on each backend start. Protect its file permissions and do not log it.
- Lock existence, lock ownership, readiness and successful authentication are
  distinct; stale metadata is not proof of a live or dead backend.
- CLI management and TUI business APIs have distinct responsibilities, not a
  new role-credential or same-user executable-identity security system.
- Business protocol versions are a discrete subset of application versions.
  TUI/backend protocol values must match, but build/application versions need
  not match. An incompatible TUI fails without disturbing the backend.
- Management RPC stays small and independent of business models and Home
  migration requirements. Its transport/codec compatibility must be tested.

## Lifetime and shutdown

- Daemon idle TTL defaults to five minutes. Configuration `-1` disables idle
  expiry; it does not prevent explicit stop or cancellation.
- Only successful frontend renewals and backend `runningTurn Job` renewals
  keep it alive. A TUI process merely existing is not a renewal.
- Host-interaction waiting, notification Hooks and auxiliary resources do not
  gain their own renewal reason. Ordinary suspension inside a running turn
  does not remove that turn's renewal.
- Daemon TTL, Session TTL and filesystem lock/lease validity are separate.
- `backend stop` asks the backend to perform cooperative coroutine
  cancellation and cleanup. CLI returns after acceptance, not after exit.
  Accepted shutdown is irreversible; renewals cannot resurrect it.
- Keep the single-instance lock until resource cleanup completes. A disconnected
  management reply does not prove acceptance or completed shutdown. Do not
  automatically force-kill a slow cleanup.
- Core graph exit must not leave an empty Starting/Ready host. Close admission
  and request the existing shutdown path, including independent cancellation;
  ordinary child-operation cancellation does not acquire this fatal meaning.
- Only the request that first accepts management stop completes its reply barrier.
  Concurrent core exit/signals must not prematurely release that request's reply wait.

## TUI and accepted work

- A failed connection is local to that client, not cancellation of accepted
  backend work. Recover a connection only to the same backend instance.
- A restarted backend is an error for an old TUI, which must be re-established;
  do not transparently connect it to the replacement.
- Tabs, navigation, editing drafts and presentation remain frontend-owned.
  Rebinding RPC resources is not closing those local owners or erasing input.
- Recreate required proxies/subscriptions/state after reconnect; reactivate
  Session owners and validate cache nonces. Do not replay result-unknown
  mutations or manufacture missing Sessions.
- Frontend renewal runs in its own frontend-owned task, independent of transport
  replacement and business recovery. It is periodic renew, not a separate health
  state machine; failed delivery grants no lifetime extension.
- Preserve the existing exact-owner, CAS, pending-tool and OAuth cleanup
  semantics; do not ask the user to reselect these settled boundaries.

## Settings and notification ownership

- Backend manages one settings YAML with `backend` and `frontend` sections.
  TUI subscribes to its frontend projection and submits edits to the backend.
- Manual disk edits enter runtime via an explicit management reload command,
  not a file watcher. Runtime settings are the source for saves, including
  background MCP token saves; un-reloaded disk edits may be overwritten.
- Reload must validate before publishing and follow existing MCP mutation
  coordination. File acceptance is not proof every external service connected
  or that all external-resource effects can be rolled back.
- Business notification Hooks execute once through the backend owner, even
  without a TUI. TUI-local error/warning Hooks continue to execute locally.
  Keep those configuration sections and execution paths distinct.
- Automatically preserve existing Hook commands during settings merge and
  split rules by execution purpose. A rule formerly serving both kinds of
  error remains in both applicable sections; migration never executes it.
- Add a new migration through the existing Home registry. Preserve frozen
  historical migrations and codecs; verify direct upgrades, partial cleanup,
  missing legacy files, conflicting targets and interruption recovery.

## Integrated implementation

- The isolated `feat/backend-daemon` worktree is
  [local source](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787).
  Its verified tree is merged into `main`; this does not authorize a release.
- `app/backend-daemon` owns the state machine, platform kernel lock and
  detached process primitives; `app/backend-host` owns the backend graph,
  readiness, shutdown and bootstrap.
- `rpc/local` carries only TUI business connectivity and renewals.
  `rpc/backend-control-client` is for management CLI/bootstrap;
  `rpc/local-server` serves both authenticated routes on the backend.
  `rpc/websocket` is their generic message transport, with no settings,
  Session, management or application graph ownership.
- Management/business HTTP clients use Ktor's dependency-selected default engine
  with their own common plugin/timeout configuration. Keep platform engine
  dependencies and module isolation; no per-platform configuration wrappers.
- Lifecycle observations publish only the phase. The first accepted stop reason
  is retained by the existing stop signal, not copied into a second state DTO.
- An established TUI never calls management status/stop/reload. Readiness is
  confirmed through its business heartbeat/handshake. Frontend assembly borrows
  `FrontendRpcConnection`, not a concrete transport or an optional local store.
- Session recovery failures on a live replacement transport reach the captured
  Session owner; network loss does not become a Session business failure.
  Neither case replays result-unknown mutations.
- Daemon Stopping closes the shared backend operation/observation admission,
  including requests from existing connections, before resource teardown.
- `.locks/backend/instance.lock` is the stable kernel guard inode.
  `connection.json` is its private atomic sibling record, not an atomic
  replacement of the locked inode. Never unlink/replace the guard to recover.
- `settings.yml` is the unified runtime authority's persistence file.
  `getFrontendSettings`, its Flow and full CAS carry the frontend projection;
  TUI-local field edits preserve unrelated latest values. Global MCP changes
  and explicit reload retain the existing backend validation coordination.
- `withFrontendApplication` is the production frontend assembly. The old
  `withKodexApplication` and its startup checkpoints exist only in
  `app/test-support/rpc`; production modules must not depend on that module.
  In-memory tests consume the same backend-managed settings/RPC paths.
- Unified storage owns one whole-file snapshot under its mutex. Library
  `MutableStateFlow`/`asStateFlow` values are private publication projections,
  not independently writable authorities or hand-implemented StateFlows.
  Hook execution reads a list snapshot, not a fabricated frontend settings DTO.
- Unified YAML reads reuse the section-node decoder directly. Text callers
  delegate to it; no AST-to-text-to-AST round trip or custom node serializer.
- Per-process rolling logs use `Backend-<pid>` / `Tui-<pid>` names. Bounded
  inactive history cleanup excludes live process groups and foreign/legacy names.
  Include the pinned [Kermit 2.1.0 rolled filenames](https://github.com/touchlab/Kermit/blob/2.1.0/kermit-io/src/commonMain/kotlin/co/touchlab/kermit/io/RollingFileLogWriter.kt#L155)
  (`<base>-<index>.log`) when cleaning a retired process group.
- Home/business checkpoints remain 0.4.15 independently of subsequent app releases;
  management protocol is 1. Business compatibility compares protocol values,
  not the whole application version. Historical migration entries stay frozen.
- Terminal failure text is an optional field in the existing private instance record.
  Read it only for the expected instance (and a fresh launched PID at bootstrap);
  a retained record never implies liveness or durable error history. CLI/TUI show
  the concrete reason without a new redaction layer; original causes and cleanup
  suppression remain available. Startup log fallback is limited to new output tagged
  with the launched PID, not an arbitrary previous failure.
- Real-runtime acceptance covers Mac arm64 and Linux x64. Linux arm64 and
  Windows x64 additionally compile, but their runtime is not claimed tested.
  Models used by running-turn/Hook tests are injected offline fixtures; no
  paid model call or real user Home is part of this checkpoint.
