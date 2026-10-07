# Task Tree

- `Clean exactly the reserved directory on failed creation`()
- `Preserve primary write/fork failure with suppressed cleanup`()
- `Add deterministic injected filesystem failure tests`()
- `Publish integration handoff and delegate central validation`()
- `Record user acceptance and commit scoped repair`()

# Details

- Parent: [Reacceptance and authorized repairs](2026-10-05-reaccept-spec-impl-refactor.md).
- 用户接受当前修复并授权分批提交；内层提交 `88d26812`，本轮未推送。
- Own `agent-session/impl/filesystem` and `agent-storage/impl/filesystem` sources/tests only.
- Fix create failure before returned index and direct temporary/fork cleanup masking primary error.
- Cleanup is NonCancellable and acts only on exact owned reserved path/temporary. Preserve source data, pointer/serializer/layout and primary cancellation.
- A successful operation's cleanup failure remains observable; do not blanket swallow I/O errors.
- Fault-inject init/write/move/delete and cancellation, verify partial target absent/reusable and original error identity plus suppressed cleanup.
- Do not modify filesystem-layout/frozen migrations, lazy repository policy, RPC assembly, process/lease code or other lanes.
- No Gradle/commit/push/branch/IDE/resources; exact signatures, tests and limitations in handoff.

## Integration handoff

- **IMPLEMENTATION HANDOFF COMPLETE — centralcompilepending.** Session 506
  finishes without waiting for central tests; coordinator owns compilation,
  discovery, execution and final acceptance. Keep this task executable until
  that validation is recorded.
- Read-only baseline check confirmed inner `refactor/spec`,
  `821b66275aae2f0a9541fa2e9e03023d875c30b5`. Other lanes' concurrent changes
  were left untouched. Only the four source/test files below and this task
  were edited, using `apply_patch`.

### Production signatures and behavior

- `io.github.stream29.kodex.agentsession.filesystem.FileSystemKodexSessionRepository`:
  unchanged `override suspend fun create(): Int` and
  `override suspend fun createFork(sourceEntryIndex: Int): Int`.
  Both catch post-reservation failure, recursively delete **only** the returned
  reserved directory in `NonCancellable`, suppress a distinct cleanup failure
  onto the original throwable, and rethrow that same object. Entries publish
  only after successful initialization/materialization.
  [create/fork implementation](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt#L142-L185).
- `io.github.stream29.kodex.agentstorage.filesystem.FileSystemIndexVersioned<T>`:
  unchanged `public suspend fun setUnsafe(index: Int, value: T)` and private
  `suspend fun writeAtomically(destination: Path, content: String)`.
  Each locally tracks its primary throwable and deletes its exact temporary
  in `NonCancellable`; delete failure is suppressed when a primary exists,
  otherwise propagated. Cleanup results cross the `NonCancellable` boundary
  as values before rethrow, preserving identity; same-object suppression is
  guarded.
  [setUnsafe](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemIndexVersioned.kt#L221-L253),
  [writeAtomically](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemIndexVersioned.kt#L322-L338).
- Serializer choices, six timelines, latest-before-record publication,
  pointer compensation, raw fork, staging/revert and reservation algorithms
  remain unchanged. No journal, recovery policy, API or dependency changes.
- Existing `CoroutineCloseable.use` has the desired failure semantics for
  handles, not reserved directories/temporary paths; wrapping paths as
  closeable adapters would add an unnecessary abstraction. Local handling
  follows the same pattern; no global cleanup helper was added.

### Exact declared tests

- **66 declared cases**, suite
  `io.github.stream29.kodex.agentsession.filesystem.fileSystemSessionCleanupTest`,
  in [FileSystemSessionCleanupTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemSessionCleanupTest.kt#L30-L149):
  - `create <boundary> <IOException|cancellation> cleans exact reservation`:
    27 IOException + 33 cancellation cases. Boundaries: `storage-create`,
    `resolve-1`, `resolve-2`; `mkdir-`, `write-`, `move-`, `delete-` for each of
    `index`, `work`, `settings`, `timestamp`, `token-count`, `unstable`;
    cancellation additionally covers each `read-<timeline>`.
  - `create preserves <IOException|cancellation> when target delete fails`:
    two cases, initialization `write-settings` plus recursive target deletion.
  - `fork copy <IOException|cancellation> with delete failure=<false|true>`:
    four cases, real raw settings-record copy then injected operation failure;
    failed deletion preserves primary identity and suppresses the exact error.
  - Successful cleanup asserts target absence, exact-path deletion, unchanged
    source/unknown/legacy bytes, unchanged entry snapshot, fresh scan before
    retry, reusable index and six clean `-1` pointers. Fork retry compares raw
    copied bytes. Deliberately failed cleanup does not claim target absence.
- **20 declared cases**, suite
  `io.github.stream29.kodex.agentstorage.filesystem.fileSystemWriteCleanupTest`,
  in [FileSystemWriteCleanupTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemWriteCleanupTest.kt#L24-L141):
  - `<EntryWrite|EntryMove|PointerWrite|PointerMove> preserves <IOException|cancellation> with failed delete`: eight cases.
  - Same four faults, `removes temporary after <IOException|cancellation>`:
    eight cases, absence of owned temporaries and successful retry.
  - `<EntryWrite|PointerWrite> successful publication exposes cleanup failure`:
    two cases, exact cleanup exception identity and published content checked.
  - `nested pointer move preserves <IOException|cancellation> with both deletes failing`:
    two cases, pointer and entry cleanup exceptions suppressed in order,
    original identity retained and previous pointer restored.
- Cancellation cases cancel the actual operation child Job with the injected
  exception. Cleanup wrappers call `ensureActive()`; caught throwables are
  captured before coroutine await exception recovery can copy them.
- Fixtures delegate to `SystemCoroutineFileSystem` under uniquely reserved
  local temporary directories. One-shot faults use fixed operation/path
  boundaries, not sleeps or races. Teardown bypasses fault wrappers.
  Session source initialization is direct storage with inert dependencies:
  no `Session.open`, runtime, lease, process, IDE or running service.

### Validation and limits

- Scoped tracked `git diff --check` and untracked test-file
  `git diff --no-index --check /dev/null <file>` passed; manually checked
  signatures, boundary order, wrapper delegation and task consistency.
- **86 is a declared matrix count, not a discovered/executed pass count.**
  No Gradle, compiler, test runner, commit, push, branch mutation or shared
  resource operation was run. Coordinator validation targets:
  `:agent-session-impl-filesystem:jvmTest` and
  `:agent-storage-impl-filesystem:jvmTest`; discovery/classpath and downstream
  compilation remain pending.
- Tests model filesystem failures at controlled operation boundaries,
  including failure after real bytes are written/copied. They do not establish
  kernel fault behavior, process-crash recovery, cross-process races, symlink
  adversaries or Native/JS/Windows/macOS execution.
- Missing-pointer IOException remains intentionally tolerated by the existing
  reader; those read boundaries inject cancellation, not a falsely fatal I/O
  error. Reservation acquisition/cancellation before `reserveSessionDirectory`
  returns its owned path is not covered or changed by this post-reservation fix.
- Frozen filesystem-layout/migrations, lease implementation, RPC/Root lazy
  repository policy, audit report and parent/master task were not edited.

## Central validation (supersedes declared-only counts)

- Actual test discovery/execution: new storage cleanup suite **20/20** and new
  Session cleanup suite **66/66** passed. The stated **86** is now an executed count.
- Full filesystem storage **36** and filesystem Session **103** tests passed.
  Existing pointer/layout/fork regressions and the new failure/cancel/suppression cases are green.
- Selected RPC/server/Session/Application JVM regressions and Integration test-source compilation
  passed. No additional wire, migration, lease or lazy-admission policy was introduced.
- Kernel faults, process crashes, reservation handoff cancellation and other-platform
  runtime verification remain outside these deterministic injected tests; no commit/push.
