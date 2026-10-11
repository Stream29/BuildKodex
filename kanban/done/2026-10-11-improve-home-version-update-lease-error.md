# Task Tree

- `Investigate the live Home version and applicable migrations`()
- `Explain blocked initialization, data migration and version-only updates`()
- `Validate isolated Home fixtures and review the scoped changes`()

# Details

- Follow-up: the user clarified that Home versioning serves migrations only.
  [Historical investigation](2026-10-11-trace-home-migration-version-gating.md)
  identifies version-only exclusive updates as an unwanted initial-planning
  behavior. This task changed wording only; its conflict fixture is not the
  desired invariant for a future behavioral repair.
- User requested a friendlier startup error and investigation of why migration
  is needed. Preserve lease policy; do not stop running instances, change the
  real Home, install a binary, commit or release.
- Read-only investigation: real Home version is `0.4.10`; installed CLI embeds
  `0.4.11`; PID `2699438` embeds `0.4.10` and holds its live read lease.
  `/proc/2699438/exe` points to the replaced, deleted executable.
- Registry ends at `0.4.7`, so this startup needs only `version.json` advancement.
  The existing protocol requires exclusive access even without data migrations.
- Translate exclusive Home lease conflicts at the preparation boundary,
  preserving the cause and lock paths. Explain the Home path, old/target
  versions, applicable migration targets or version-only update, and retry
  after closing instances using that Home.
- Checks: isolated version-only, data-migration and unversioned fixtures;
  no migration callbacks or version changes on conflict; temporary write intent
  removed; existing reader retained; same-version sharing and retry after
  reader release still work; existing migration/lifecycle tests pass.
- IDEA is open for this repository, but its MCP connection is unavailable.
  Use native tools without manipulating user windows or IDE synchronization.

## Validation

- `:app-migration-spec:jvmTest :app-migration-impl:jvmTest` passed: 2 spec and
  77 impl tests, including all three new conflict fixtures and existing lifecycle
  and historical migration suites.
- Reused existing Gradle 9.5.1 daemon PID `3383916`, explicitly using its
  Adoptium Java 25 home, with `--max-workers=2`. The shared daemon is not stopped.
- Initial new-test assertions raced heartbeat atomic temporary files; corrected
  live snapshots to compare read/write owner files. After lease closure, the
  complete lock directory must still be empty. The rerun passed.
- Fixtures were cleaned; `git diff --check` passed in both repositories.
  Registry and frozen version-specific migration sources remain unchanged.
- Only exclusive preparation lease conflicts are translated; other failures
  and cancellation behavior remain unchanged. The CLI's existing error printer
  displays the contextual message without frontend changes.
- No native CLI rebuild, installation, real Home mutation, process termination,
  commit or publication was performed.
