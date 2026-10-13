# Task Tree

- `Read task boundaries and related experiment records`() // complete
- `Inventory current disk allocation and resource users read-only`() // complete
- `Ask the user to approve exact cleanup candidates`() // A1 and A2 approved separately
- `Recheck ownership and acquire the shared lock before approved cleanup`() // complete
- `Measure actual recovery and hand off the cleanup summary`() // complete

# Details

## Scope and authorization

- [Session 773](file:///home/stream/.kodex/sessions/773) independently handles
  Xiaoxin disk cleanup. Inventory first; deletion or migration requires the
  user's explicit approval of the exact scope. No builds or IDEA acceptance.
- Do not modify the active product, candidate worktree, forks, other task
  conclusions, versions, commits or remotes. Preserve existing resource gates:
  ordinary validation starts with at least 4GiB free and retains 2GiB;
  the full private IDEA experiment starts with at least 26GiB.
- Preserve user caches, source/worktrees, credentials, original failure
  evidence, test reports, release artifacts and private heaps unless separately
  approved. Inspect allocation and ownership, not private file contents.
- Before changing experiment caches, recheck active users and acquire
  `~/ACodeSpace/demo/kodex-gradle-research-445/device-heavy.lock` nonblocking;
  never kill unrelated processes or operate user windows.
- Protect the explicitly borrowed R36 `gradle-home` and `konan` under
  `~/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/gui-runs/baseline-kgp-v3-idle-gradle-initial-r36/`.
  Future toolchain verification may reuse them.
- Related records:
  [root-only validation](../done/2026-10-08-validate-kodex-toolchain-compatibility.md),
  [IDEA repair](../done/2026-10-08-repair-full-model-idea-import-oom.md),
  [KGP experiment](../done/2026-10-09-experiment-kgp-metadata-producer-reuse.md).

## Read-only checkpoint — 2026-10-12 11:12 +08:00

- SSH host confirmed `xiaoxin-ubuntu`. `/`, home and the experiment directory
  reside on `/dev/nvme0n1p2`, ext4: total 501,809,635,328 bytes,
  used 471,995,027,456 bytes, available 4,248,805,376 bytes (3.96GiB).
  Inodes are 39% used. The available value, not total minus used, governs gates.
- No Java/Gradle/Kotlin/IDEA/MAT process was visible in the process-name
  inventory. Existing desktop/container services remain untouched.
  `lslocks` and `fuser` showed no holder for the shared lock at this instant;
  this observation is not a reservation.
- Directory allocation inventory uses low-priority `du -x`; inaccessible
  system directories are not elevated with sudo. No file deletion, migration,
  archive creation or credential/heap-content inspection has occurred.
- Session 769's remote source/results/archive and local evidence/candidate
  are protected. Its results must not be treated as this session's cleanup.
- After cleanup, give the user actual recovered bytes, current free space,
  exact changed paths and retained evidence locations. The user must carry the
  summary back to Session 769; it receives no automatic result.

## Allocation and proposed candidates

- Low-priority `du -x` reports the following allocated directory sizes; parent
  and child totals overlap and must not be added. Inaccessible system paths
  make the readable root total incomplete.
- Experiment roots under `~/ACodeSpace/demo/`:
  `kodex-gradle-rollout-20261007` 128.15GiB;
  `kodex-kgp-producer-experiment-20261009` 104.81GiB;
  `kodex-gradle-research-445` 72.99GiB;
  `kodex-import-sources-repair-20261008` 44.52GiB;
  `kodex-build-cost-ablation-20261008` 28.49GiB.
- First proposed candidates, each relative to
  `~/ACodeSpace/demo/kodex-gradle-rollout-20261007/idea-current-runs/`:
  - A1: `baseline-5b41be52-r01/gradle-home/caches`
    (8,126,291,968 bytes) and `baseline-5b41be52-r01/konan`
    (8,549,601,280 bytes): total 15.53GiB.
  - A2: `baseline-5b41be52-r02/gradle-home/caches`
    (8,126,263,296 bytes) and `baseline-5b41be52-r02/konan`
    (8,549,601,280 bytes): total 15.53GiB.
  - Each candidate is a real mode0700 directory owned by `stream`.
    No multiply-linked regular files were found within these four directories.
    Estimates still require before/after filesystem measurement.
  - Preserve the entire remainder of both run roots: `evidence`, `private`,
    `project`, configuration, scripts, Gradle daemon logs and wrapper
    installations. Native caches include both 2.4.0 and 2.4.20 distributions.
    Regeneration needs downloads/recompilation; offline reruns and exact
    warm-cache state would be lost. These were proposals at inventory time.
- Additional candidates for separate consideration, not an approved batch:
  - KGP root `cinterop-gradle-home/caches` plus `cinterop-konan`: 15.51GiB.
  - Import-repair root `installed-source-seed/caches`: 8.74GiB.
  - Ablation root `gradle-home/caches`: 9.15GiB.
  - These may still support experiment replay; retain source, private
    diagnostics, reports, wrappers and sealed artifacts. Check their specific
    provenance and consumers before any later approval/execution.
- R36's protected cache allocation is now 12.84GiB (`gradle-home`) plus
  5.54GiB (`konan`). This is not a proposed deletion.
- Approving both A1 and A2 would nominally recover 33,351,757,824 bytes
  (31.06GiB), bringing the initial available space to about 35.02GiB.
  This is a projection, not achieved recovery or authorization to validate.
- At 11:16:46 +08:00, available space remained 4,248,768,512 bytes.
  The full root and eight targeted experiment inventories finished. A redundant
  home-level traversal was stopped by terminating only this session's exact
  verified `du` child; its partial output is not a complete home total.
  All SSH inventory commands have ended. No cleanup or migration occurred.

## Execution authorization

- The user explicitly selected “批准 A1 清理” and “批准 A2 清理” in the
  two separate confirmation questions. Delete only the four paths listed
  above; retain all other run contents and all other proposed candidates.
- Before deletion, capture allocation plus a metadata manifest of retained
  run contents, check active users and take the existing shared lock.
  Verify retained metadata and actual filesystem recovery afterward.

## Completed cleanup — 2026-10-12 11:20 +08:00

- Deleted exactly the four A1/A2 paths listed above, no additional candidate,
  source tree, private heap, report, archive or published artifact. No migration,
  build, GUI operation, commit, push or product/fork/version change.
- Initial process inspection stopped safely before changes because the SSH
  user could not inspect existing non-dumpable login services' cwd/fd metadata.
  Existing noninteractive sudo was then used only for a read-only process-name
  and cwd/fd-target check across all processes. Deletion stayed unprivileged.
  No process was killed for cleanup.
- Shared lock acquired nonblocking; active-user checks repeated before each
  deletion. The lock was released afterward and independently verified
  available. Final process-name check found no Java/Gradle/Kotlin/IDEA/Xvfb.
- Deleted allocation estimate: 33,351,757,824 bytes (31.06GiB). Actual available
  delta during cleanup: 33,348,608,000 bytes (31.06GiB), from 4,245,598,208
  to 37,594,206,208 bytes; net delta includes audit filesystem overhead and
  concurrent background filesystem activity, not just removed file allocation.
- Final independent `df` at 11:20:31: available 37,594,193,920 bytes
  (35.01GiB), used 438,649,638,912 bytes, 93%. This clears both stated startup
  disk thresholds at that instant; subsequent validation must recheck resources
  and obtain its own authorization/lock. No validation was started here.
- Both runs' remaining 15,759 metadata entries matched exactly before/after;
  all seven public evidence JSON files had identical SHA256 values.
  Private content was not read or hashed. R36 cache root identity/mode/owner
  metadata matched; both protected cache directories remain, as does Session
  769's remote evidence archive. No claim of full private-content hashing.
- The deleted Gradle dependency/transform caches and Native distributions/
  dependencies can be regenerated from retained run configuration and wrappers
  by later authorized runs, with network and compilation costs. Their exact
  former warm-cache state is not preserved; this is not a reversible backup.
- Cleanup audit on Xiaoxin:
  `~/ACodeSpace/demo/kodex-disk-cleanup-773/receipt.json`,
  `retained-metadata-before.json`, `retained-metadata-after.json`.
  Full copies and the exact executed cleanup script are retained locally at
  `~/ACodeSpace/local/xiaoxin-disk-cleanup-773/`.
- Three transferred JSON files have matching local/remote SHA256 values.
  Receipt SHA256:
  `c980baba6c8da846050b55825984378ab662d7ce4acf12ee4d15f0285dc31925`.
  Both metadata-manifest SHA256 values:
  `b72ad6044420ed37133750340756a4e5a63fefa5443aba8d16337ef9710e6a4c`.
- Original evidence remains inside both original run roots, under `evidence/`
  and `private/`; project/configuration, wrapper installations and daemon logs
  are retained in place. Other proposed caches and user home caches are untouched.
- All inspection/cleanup SSH commands finished. No temporary toolchain was
  installed: remote uv was absent, so the audit used system Python's standard
  library. No temporary working files remain; the audit is intentionally retained.
- User handoff to Session 769: A1/A2 cleanup complete, approximately31.06GiB
  recovered,35.01GiB currently available, R36 borrowed caches preserved.
  Recheck disk/lock before resuming; this session did not run validation.
