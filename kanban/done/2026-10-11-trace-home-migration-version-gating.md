# Task Tree

- `Trace the first implementation and later refactors`()
- `Compare original user messages with agent planning additions`()
- `Record the origin and distinguish evidence from inference`()

# Details

- Scope: investigate the origin of no-migration version advancement and
  application-version equality gating. No runtime repair, real Home changes,
  builds, commits, releases or process termination are authorized in this task.
- Current user clarification: the Home version is meaningful for migration;
  no actual migration must not force an application-version-only update or
  prevent otherwise compatible instances from sharing Home.
- The earlier [error-message change](../done/2026-10-11-improve-home-version-update-lease-error.md)
  remains uncommitted. Its version-only-conflict fixture codifies the current
  defect, not the newly clarified desired behavior.

## Original conversation — Session 185

- [Initial user request](file:///home/stream/.kodex/sessions/185/index/1.json#L1):
  add Home versioning to check and migrate data before running.
- [Read/write lease discussion](file:///home/stream/.kodex/sessions/185/index/171.json#L1):
  the question offered ordinary multi-instance use with exclusivity during
  migration; the user proposed per-process read/write heartbeat files.
- [Initial agent plan](file:///home/stream/.kodex/sessions/185/index/259.json#L1):
  separate numeric `formatVersion`, with write access for data migration.
- [User correction](file:///home/stream/.kodex/sessions/185/index/261.json#L1):
  use the application's version rather than a `formatVersion` envelope and
  maintain a global version-to-migration table under `app/migration`.
- [Registry question and answer](file:///home/stream/.kodex/sessions/185/index/269.json#L1):
  the user selected “仅迁移版本”, rejecting a registry entry/no-op for every
  release. The option description still said the file uses the application
  version; it did not separately ask whether every release must advance it.
- [Agent response](file:///home/stream/.kodex/sessions/185/index/497.json#L1),
  2026-09-01 15:41:52 Asia/Singapore: added “即使当前版本没有 migration，也会在独占写租约下把
  `version.json` 推进到当前应用版本”.
- [Implementation approval](file:///home/stream/.kodex/sessions/185/index/1293.json#L1)
  followed a broader plan review. The archival conversation did expose this
  policy in the agent's summary, but contains no separate user instruction
  requiring no-migration version advancement. Do not misreport it as a
  specifically user-requested rule, or claim the user explicitly prohibited
  it in that historical conversation.

## Git chronology

- BuildKodex `8900c44f499e907996a541ad786ddb1f408ea791`,
  2026-09-01 18:16:30 +0800: first committed checklist and migration plan
  already contain no-migration advancement and same-application-version
  admission. At that commit the task was in `kanban/planning/`.
- Kodex `81d353affb040e9d36f9b61721826b882dcd4990`,
  2026-09-02 01:22:28 +0800: **first creation** of
  `app/migration/impl/.../KodexHomeMigration.kt`; its parent has no
  `app/migration` tree. The initial source already:
  - returns a shared Home handle only for `storedVersion == currentVersion`
    (historical lines 61–72);
  - acquires a write lease otherwise (line 84);
  - requires equality again after preparation (lines 96–101);
  - writes the application version after the migration loop, including
    when the loop executes no entries (lines 130–137).
- Kodex `125ee04c`, 2026-09-02: adds migration-start callback/progress reporting,
  not a different version-advancement condition.
- Kodex `fac1bdb6`, 2026-10-07: moves the real Home handle/exception contract
  into spec and retains those equality/write decisions.
- Kodex `13288e08`, 2026-10-07: repairs cancellation, handoff and observable
  cleanup failure; the version policy is unchanged.
- `git log --all -S 'storedVersion == currentVersion' -- app/migration`
  and `git log --all -S 'writeVersion(home, currentVersion' -- app/migration`
  identify the first-creation commit. The initial-to-HEAD source diff shows
  no subsequent change to those conditions.

## Conclusion and repair boundary

- Proven: this is **not a regression introduced by the recent refactors**.
  The implementation inherited the behavior from the initial planning.
- Inference, supported by the user correction and agent response: the planning
  conflated “use an application version as the migration watermark” with
  “advance it to every application's version and require exact equality”.
  The present user clarification makes the latter an unwanted behavior.
- A repair must change the preparation/admission conditions as well as the
  final version write; changing only the error wording does not fix it.
  Historical migrations and their fixtures remain frozen.
- Existing Homes already contain inflated no-migration release values such as
  `0.4.10`. Any implementation plan must explicitly handle that compatibility
  issue without automatically editing the real Home or pretending the stored
  value proves a data migration occurred.
- This task records the investigation only; no behavioral repair was made.
- Validation: original Session events, first-creation source, Git pickaxe/blame
  and initial-to-HEAD diff agree; document links and `git diff --check` pass.
  No builds are needed for this read-only code investigation.
