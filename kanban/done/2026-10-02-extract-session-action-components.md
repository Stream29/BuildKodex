# Task Tree

- `Inventory all session rename and delete entrypoints`()
- `Extract dependency ports, component contracts and ViewModels`()
- `Move renderers and retarget Application, Settings and Catalog adapters`()
- `Run component and downstream JVM regression suites`()
- `Record validation and obtain user acceptance`()

# Details

## Authorization and scope

- The user selected the session-action family and authorized executable tracking
  and implementation. Cover both complete interaction types, not only one popup.
- Preserve the uncommitted Path Picker, Catalog and Login component batches,
  unrelated user work, branch and commit state.
- Follow [Frontend Application Boundary](../../checklist/frontend-application-boundary.md)
  and [Spec/Impl Module Boundaries](../../checklist/spec-impl-module-boundaries.md).

## Component mapping

| Component | Projects | All existing entrypoints |
| --- | --- | --- |
| Session Rename | `app/component/session-rename/{spec,impl/viewmodel,impl/view}` | Application tab menu and Settings current-session rename. |
| Session Delete | `app/component/session-delete/{spec,impl/viewmodel,impl/view}` | Application tab deletion and Session Catalog deletion confirmation. |

## Contract and adapter decisions

- Each spec owns a typed operation dependency, component ViewModel, state and
  rendering/lifecycle/exception KDoc. No dependency on Application, Settings,
  Catalog, RPC or concrete Session implementation is needed.
- Rename captures an initial title, owns draft edits, trims the submitted name
  and uses the bound dependency. The input submits on plain Enter with no action
  buttons. Empty input is not submitted by the renderer. Start the operation from
  Enter handling so subsequent input cannot change the submitted snapshot;
  Settings' synchronous command handoff remains before later input events.
- Application retains its captured `SessionViewModel` as an ownership handle
  in a compatibility adapter, delegating interaction behavior to the component.
  Settings binds the exact source and `expectedRevision` from its effect.
  A normal return can mean queued dispatch or a stale/closed no-op, not durable
  persistence; stale revisions remain handled by the existing Settings command.
- Delete captures index and display title at opening, returns the existing
  Boolean outcome and leaves parent navigation to the host. Application closes
  after either Boolean result; Catalog closes only on true. Preserve this
  observable difference, including exception/cancellation propagation.
- Components do not close the captured Session, Settings or Catalog owner.
  Closing a child is idempotent, ignores later draft edits and rejects new
  operation calls. Closing does not roll back or cancel already-started commands.
- Keep per-host appearance through explicit renderer presentation parameters;
  no component dependency on Settings styles or Application View.
- Reuse existing operation and wire semantics. Do not add concurrency queues,
  change error handling, persist drafts or replace RPC protocols during extraction.

## Validation gates

- Exercise trimming, captured dependency, blank-input gating, false deletion,
  failures/cancellation and close/late-call behavior with dependency-only tests.
- Render both titles and input/confirmation interactions without backend I/O.
- Check exact popup identity and Settings revision/Catalog callback wiring.
- Run component JVM tests and Application/Settings ViewModel/View regression
  suites; compile direct consumers including RPC adapters.
- Scan for duplicate old interaction implementations and improper spec/view
  dependencies; run inner/outer `git diff --check`.
- Record Native/JS/CLI as unverified unless actually run. Await review without
  committing or switching branches.

## Checkpoint

- All six projects are implemented on `refactor/spec`. Both specs declare real
  dependency ports and ViewModel contracts with draft/target, rendering,
  submission, cancellation, closure and `@throws` semantics.
- Application delegates rename interactions to the component while retaining
  its exact Session ownership handle. The old delete interface name is a
  compatibility typealias to the component contract. The component projects
  depend on neither parent ViewModels nor RPC/Settings/Catalog implementations.
- Settings' `SessionRenameAdapter` binds its exact source and effect revision.
  Both rename entrypoints share one renderer with compact/labeled presentation;
  both deletion entrypoints share the captured-target confirmation renderer.
  All former renderer/interaction implementations are removed from their hosts.
- Component code leaves registry updates, revision rejection, queueing, tab
  cleanup and Catalog refresh in existing host commands. The Application
  dismiss-on-either-result and Catalog dismiss-on-true policies remain distinct.
- 25 component JVM tests passed: Rename ViewModel 6, Delete ViewModel 6,
  Rename View 8, Delete View 5. These cover submission snapshots, draft edits,
  blank/modified Enter, target/title capture, false results, failure/cancellation,
  idempotent close, in-flight operations, disposal and replacement/late callbacks.
- 216 downstream JVM tests passed: Application ViewModel 27, Settings View 25,
  Application View 100, Settings ViewModel 5, RPC ViewModel 59. New adapter and
  ownership tests exercise source/revision binding, captured Session selection,
  popup replacement and missing-session deletion. All 241 tests passed without
  failures, errors or skips.
- Direct JVM consumers compiled with the existing Daemon JVM and one worker.
  Initial renderer test assertions needed explicit Mosaic frame advancement and
  waiting for actual disposal instead of a potentially earlier pending snapshot.
  The final rerun passed after adding the late-callback and submission-order tests.
- Static checks verified all six projects, their spec/impl pairing, project
  coordinates, dependency directions, absence of duplicate old implementations
  and obsolete compiled classes, and inner/outer `git diff --check`.
- Native/JS/CLI validation was not run. The remainder of Application, Settings
  and the Catalog renderer stays in its existing host and is not claimed as
  migrated by this bounded family.
- The user accepted this batch and authorized submission followed by the next
  similar interaction family. Inner commit `fc9f1936` contains this batch and
  the previously accepted Path Picker/Catalog/Login batches as one buildable
  component migration. No pushes or branch switches were performed; unrelated
  user tasks and files were left untouched.
