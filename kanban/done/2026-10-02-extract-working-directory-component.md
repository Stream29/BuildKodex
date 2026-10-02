# Task Tree

- `Commit previously accepted component batches`()
- `Inventory all working-directory picker owners and callbacks`()
- `Extract working-directory spec and implementation topics`()
- `Retarget Application and Settings ownership adapters`()
- `Run component and owner regression tests`()
- `Record user acceptance and commit the validated component batch`()

# Details

## Authorization and scope

- The user authorized committing the accepted batch followed by the next similar
  component family. Choose working-directory selection as one complete family.
- Previous submissions: inner `fc9f1936`, outer documentation `4a2b827` and
  submodule pointer `6e5258b`. The user accepted the accumulated component work
  and authorized commit/push; inner commit `7310a445` includes Working Directory
  with the four-component batch, whose overlapping hosts form one buildable slice.
  Remain on `refactor/spec`; unrelated tasks/IDE windows remain outside scope.
- Follow [Frontend Application Boundary](../../checklist/frontend-application-boundary.md)
  and [Path Picker](../../checklist/path-picker.md).

## Mapping and complete entrypoint coverage

- New projects: `app/component/working-directory/{spec,impl/viewmodel,impl/view}`.
- Reuse `app/component/path-picker` for directory navigation, validation,
  filtering, state/effect publication and Mosaic browsing. Do not duplicate
  filesystem logic or add directory RPC.
- Application: virtual New Session cwd, materialized Agent cwd, and a pending
  Suggest Subagent Task configuration's batch cwd.
- Settings: the exact `SessionWorkingDirectoryPicker` handle and captured
  revision, for either materialized or virtual target.
- MCP's textual stdio cwd field is not a directory-browser interaction and stays
  with its editor family. Do not migrate whole Settings or pending-tool workflows.

## Contract and ownership decisions

- Spec defines one owned picker child, the bound cwd-selection dependency,
  interaction/rendering semantics, lifecycle and observable exceptions.
  It depends only on picker spec, Kotlin/coroutines and path values.
- Keep Application target ownership in an adapter. The normal dependency updates
  its captured target; the suggested dependency rechecks its captured callId and
  submitting guard, and never changes the source Session cwd.
- Settings retains handle identity and revision-based admission/queue semantics.
  It removes and closes the exact picker before attempting to enqueue the update;
  normal return does not guarantee writable revision or durable persistence.
- The component closes its picker after normal dependency return and on explicit
  close. Settings may consume/close it earlier within its dependency. Failures
  propagate; they do not close an otherwise still-owned child automatically.
- Close is idempotent, never updates cwd, and does not close/cancel the target
  or undo commands already dispatched. The renderer owns its caller jobs.
- A picker renderer used by this owning component borrows its child; preserve
  standalone picker rendering's default disposal ownership. There must be only
  one effect collector and no duplicate state authority.
- Preserve selection/validation, terminal focus/filter/navigation and all
  existing Settings queues, revision guards and RPC shapes.

## Validation gates

- Dependency-only tests: bound path handoff, close timing, failure/cancellation,
  idempotent disposal, closed calls, and in-flight selection.
- Rendering: reuse picker loading/ready/failure state, selection and cancellation,
  correct disposal ownership, and suppress stale callbacks after replacement.
- Owner tests: exact Application target, suggestion admission guards, Settings
  replacement/stale handle, consume-before-write, stale revision and target loss.
- Compile/test new projects, existing Path Picker, Application/Settings and RPC
  consumers with the available idle shared Daemon, one worker.
- Check spec/impl direction, old implementations, task links and inner/outer
  `git diff --check`. Record Native/JS/CLI as outstanding unless actually run.

## Checkpoint

- All three working-directory projects are implemented. Spec owns the bound
  selection capability, component interface/factory, browser rendering contract,
  handoff/close order, cancellation and `@throws` semantics.
- Application retains a target-only ownership adapter and delegates selection
  and child disposal. Its suggestion adapter retains captured-call admission
  and merges cwd into the latest pending configuration, never Session settings.
- Settings' exact handle now contains `selection: WorkingDirectoryViewModel`;
  the existing `viewModel` getter still exposes its browser for read compatibility.
  Handle consumption/CAS, captured revision, writability checks, queue and
  error reporting remain in Session Settings. Replacing/disposing a handle
  closes the whole selection component instead of only its browser.
- Both hosts use the same component renderer. Standalone Path Picker retains
  its original three-argument API; the owning component uses an explicit
  borrowed-disposal overload. Only the component closes that browser, while
  the browser renderer remains the sole effect collector.
- 13 new component JVM tests passed (7 ViewModel, 6 View), covering handoff,
  close-before/after-dispatch, failure/cancellation, in-flight close, the three
  browser state variants, confirmation, filter-first Escape, disposal ownership
  and late completion after replacement.
- 15 existing Path Picker JVM tests passed (10 ViewModel, 5 View), including
  filesystem browsing and terminal focus/filter/navigation behavior.
- 229 owner/downstream JVM tests passed: Settings ViewModel 11, Settings View
  25, Application ViewModel 34, Application View 100, RPC ViewModel 59.
  The new owner/adapter tests cover stale handles/revisions, target loss,
  consume-before-write, popup replacement and all suggestion admission states.
  All 257 scoped tests passed without failures, errors or skips.
- Direct consumers compiled using the existing JDK 26 Daemon and one worker,
  with both shared Daemons idle before starting. The initial Escape test needed
  to await the rendered cleared-filter snapshot, not only the updated StateFlow,
  before sending the second Escape; the corrected test and final full rerun passed.
- Static checks passed for all three component projects and six host project
  coordinates, dependency direction, absence of the former implementation/
  direct host picker callbacks and obsolete compiled classes, task links and
  inner/outer `git diff --check`.
- Native/JS/CLI validation was not run. No backend, RPC wire or whole Settings
  migration is claimed. The user accepted this component and the following
  four-component batch; both are committed as `7310a445`, with 392 final scoped
  JVM regressions passing in the combined tree.
- Unrelated user files and IDE windows were left untouched. No pushes or branch
  switches were performed.
