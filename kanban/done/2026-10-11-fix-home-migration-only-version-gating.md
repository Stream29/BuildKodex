# Task Tree

- `Confirm migration-only admission and legacy checkpoint compatibility`()
- `Add regression tests and repair Home preparation`()
- `Validate lifecycle and migration suites without changing frozen entries`()
- `Prepare the uncommitted patch for user review`()
- `Receive user review acceptance and publication authorization`()

# Details

- User accepted the reviewed patch and authorized commits, pushes and formal
  `0.4.12` publication. Publication is tracked separately; no process termination
  or real Home data mutation is authorized.
- [Historical investigation](../done/2026-10-11-trace-home-migration-version-gating.md)
  identifies the initial-planning mistake; include the earlier uncommitted
  friendly error change but remove its version-only-conflict branch/fixture.
- A versioned Home with no `stored < target <= currentApplication` entry keeps
  its shared lease and exact stored version, even when application versions
  differ. Do not scan Session data, create a write intent or rewrite version.
- Real migration acquires the existing exclusive lease, rereads the version,
  executes applicable frozen entries in order and publishes each completed
  target. Remove the application-version-only tail write.
- Unversioned Home still needs exclusive baseline validation/registration.
  Register the validated `0.3.2` baseline before running migrations so a failed
  first migration can restart without treating partial output as an unversioned
  baseline. New Home ends at the last activated migration, or baseline when none.
- `KodexHomeHandle.version` is the accepted persisted checkpoint, not the running
  application version. Recheck compatibility and absence of pending migrations
  after exclusive-to-shared handoff, not application-version equality.
- Preserve raw legacy inflated checkpoints such as `0.4.10`; do not normalize
  or downgrade them. Retain fail-closed rejection for `stored > currentApplication`
  because an older binary cannot prove a future release had no unknown migration.
- Keep registry, all frozen `v*` migration sources/codecs/fixtures, general lease
  implementation and application version untouched.

## Checks and review focus

- Red/green: `0.4.7` and legacy `0.4.10` checkpoints shared by simulated
  `0.4.10`/`0.4.11` instances in either startup order; no write intent/version
  write/scanning; accurate handle checkpoint and clean resource release.
- Exclusive migration/initialization conflict remains actionable; callbacks
  and data changes do not run before acquisition.
- Baseline-only initialization, first-migration failure/retry, ordered
  multi-version migration, future targets, last-target checkpoint, reread
  after writer acquisition and final handoff validation.
- Existing lifecycle cancellation, cleanup identity/suppressed failures,
  historical migrations, newer/malformed versions and registry ordering.
- Canonical checklist and spec KDoc must use the corrected semantics.
  Show review paths and test evidence; leave this task waiting for user review.
- IDEA is open for this project, but its MCP connection is unavailable.
  Do not manipulate the user's IDE windows or trigger synchronization.
- Red baseline: the updated impl suite compiled and ran 87 JVM tests against
  the old coordinator; 13 failed for the expected writer acquisition,
  application-version tail write, baseline registration or equality handoff.
- First repaired run passes the new invariants but exposes two obsolete
  coordinator expectations: current-store `0.4.6` tail write and frozen
  `v0_4_3` future-skill test's `0.4.2` tail write. Update the non-frozen assertion;
  precisely exclude only the frozen obsolete test across JVM/Native tasks and
  replace its activation/skill coverage in the new checkpoint suite. Retain
  the other 13 `v0_4_3` tests and the full frozen source bytes.

## Verification and review handoff

- JVM: `:app-migration-spec:jvmTest :app-migration-impl:jvmTest
  :utils-filesystem-lease-impl:jvmTest` passed: 2 spec + 87 Home impl +
  54 lease tests. The lease task was unchanged/up-to-date on the final rerun.
- Linux Native: `:app-migration-spec:linuxX64Test
  :app-migration-impl:linuxX64Test` passed: 2 spec + 86 Home impl tests.
  Both platforms run the 12 new checkpoint tests and 13 retained `v0_4_3`
  tests; exactly one obsolete assertion is replaced, not the whole suite.
- Builds reused existing Gradle 9.5.1 daemon PID `3383916` with its explicit
  Adoptium Java 25 home and `--max-workers=2`. Shared daemon remains untouched.
- Scoped `git diff --exit-code` verifies registry and all frozen migration
  source/codec/fixture directories unchanged. Both `git diff --check` pass.
  Isolated fixtures were cleaned; installed CLI checksum remains
  `dac5f65f273b28aa089d5e7ca55f5a97d6096ac9c2bf493feafde4a331eae070`.
- Review entry points:
  - [Admission and actual migration writes](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigration.kt#L83-L225).
  - [Checkpoint regressions](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigrationCheckpointTest.kt#L17-L204).
  - [Precise obsolete-test exclusion](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/build.gradle.kts#L53-L60).
  - [Corrected checkpoint contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeHandle.kt#L6-L65)
    and [canonical version protocol](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/kodex-home.md#L75-L151).
- Implementation/review ran no full CLI rebuild/installation, real Home mutation,
  commits or publication. The user subsequently accepted the patch; release
  execution belongs to the [0.4.12 release](2026-10-11-release-kodex-0-4-12.md).
- Release startup gates exposed a missed downstream equality check in
  `withKodexApplication`. Follow-up `a8b51e8a` removes it: the supplied prepared
  handle is already admitted by the coordinator. Two Application assertions
  now expect the actual `0.4.7` target; new real-host tests retain another reader
  while rendering with `0.4.7` and legacy `0.4.10` checkpoints. The 12-case
  Application checkpoint suite and 8 offline renderer boundary cases pass.
