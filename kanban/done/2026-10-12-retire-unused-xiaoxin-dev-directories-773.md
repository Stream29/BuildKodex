# Task Tree

- `Recheck available disk and current heavy users`() // complete
- `Inventory old development roots and separate cache from evidence`() // complete
- `Confirm the exact twenty-four-cache deletion scope`() // explicitly approved
- `Wait for the active validation to release the shared heavy lock`() // released; Session787 acquired and released its cleanup lock
- `Recheck users and preserve retained metadata under the shared lock`() // original before snapshot retained
- `Delete only the approved twenty-four cache directories`() // complete: all24 absent
- `Verify retained data and report actual recovery`() // complete: retained metadata and evidence equal

# Details

## Completed — 2026-10-13 / Session 787

- 用户授权本卡收尾；原 Session773 精确 24 处缓存授权不扩大到实验根。
- 新审计根为 Xiaoxin
  `~/ACodeSpace/demo/kodex-disk-cleanup-787-second-batch`；原四路径回执不覆盖。
- 24 处预检通过，39 个 Native 工具链多链接 inode 的所有别名均在删除集合内。
  同一 PID/birth 的锁 FD 和缓存目录 FD 单独识别；外部进程、保留根 FD、
  cwd/exe/maps 仍阻止执行。只读 fixture 已验证其他 PID 的占用必定被拒绝。
- 原首轮在首个缓存内中止，未完成任何整目录；停止回执、脚本、日志保留。
  续跑前重新比较原保留元信息、公有证据哈希与保护路径身份，原比较通过后续跑。
- 最终回执 `status=complete`，原精确 24 目录全部删除且无残留；
  3,931,647 条保留元信息前后逐项相同，两份原始快照 SHA-256 均为
  `eafd82e0ad93010d2bd5fae6f3b6439124f43e7c71150557434b089d0989575e`。
  公有证据哈希、R36/用户缓存/共享锁身份及 Session769 全目录元信息未变。
- 当前批次获准缓存占用 186,659,045,376 bytes（173.84 GiB）。
  执行回执实际可用空间增量 185,433,346,048 bytes（172.70 GiB），
  含本轮大型审计快照及同期文件系统写入的净影响，不把估算直接当回收量。
- [本机保存的最终回执](file:///home/stream/ACodeSpace/demo/kodex-closeout-787/legacy-closeout/receipt.json#L1)；
  [审计保存及资源释放](file:///home/stream/ACodeSpace/demo/kodex-closeout-787/legacy-closeout/README.md#L28)
  已通过：无损压缩字节与远端/本机 hash 一致，冗余 raw/复制 uv/cache 已清理，
  原共享锁可重取并释放，不覆盖原回执。清理结束可用空间 205.33 GiB，
  相对本批预检净增加 173.47 GiB（含审计和同期文件系统活动）。
- [本轮协调记录](2026-10-13-closeout-legacy-development-tasks.md#L11)；
  下文保留 2026-10-12 的原清点、授权与资源等待记录。

## Scope

- Follow-up in [Session 773](file:///home/stream/.kodex/sessions/773):
  the user asks what else can free substantial space and recalls several
  unused development directories.
- The [first four-cache cleanup](../done/2026-10-12-clean-xiaoxin-disk-session-773.md)
  is complete. Its A1/A2 approvals do not authorize any additional deletion.
- Initial inventory was read-only. Distinguish current activity, historical task
  completion and the user's confirmation that an environment is retired.
  A quiet process list does not establish that a directory is disposable.
- Preserve R36's borrowed Gradle/Native caches, Session 769 inputs/evidence,
  user source/worktrees, original failure evidence, private diagnostics,
  reports and publication artifacts. Full-root retirement requires an
  explicit preservation plan and approval, not just a cache-size estimate.

## Live checkpoint — 2026-10-12 11:23:56 +08:00

- Root filesystem available: 37,594,169,344 bytes (35.01GiB), 93% used.
  No Java/Gradle/Kotlin/IDEA/Xvfb process or shared-heavy-lock holder observed.
- Xiaoxin's `~/ACodeSpace` has `demo`, `fork`, `local`, `push`.
  The shallow directory inventory identifies eleven `demo` roots, one
  `fork/kotlinx-rpc`, and `push/PersonalSkills`; no directory has been removed
  by this follow-up inventory.
- Relevant dispositions: the ablation experiment is a completed diagnostic,
  not an adopted memory fix; the KGP maintenance route was rejected, but its
  evidence and the R36 cache borrowed by current validation are still retained.
  `kodex-gradle-research-445` also owns the current shared-lock path and must
  not be deleted wholesale.

## Read-only allocation results

- `~/ACodeSpace` occupies 348.40GiB; `demo/` occupies 348.11GiB.
  `fork/kotlinx-rpc` occupies only 0.29GiB. Remote `local/` is effectively
  empty and `push/PersonalSkills` occupies less than 1MiB.
- The five large experiment roots account for almost all workspace allocation.
  These are environment retirement candidates, not permission to delete
  whole roots or proof that their evidence is obsolete.

| Experiment under `~/ACodeSpace/demo/` | Root allocation | Selected cache candidates |
| --- | ---: | ---: |
| `kodex-build-cost-ablation-20261008` | 28.49GiB | 25.94GiB |
| `kodex-gradle-rollout-20261007` | 97.09GiB | 64.71GiB |
| `kodex-import-sources-repair-20261008` | 44.52GiB | 24.60GiB |
| `kodex-kgp-producer-experiment-20261009` | 104.81GiB | 32.14GiB |
| `kodex-gradle-research-445` | 72.99GiB | 26.45GiB |

- Cache candidates total 186,659,045,376 allocated bytes (173.84GiB).
  Low-priority `du -x -B1 -s` measured the selected directories in one
  invocation; no parent totals are added to their child totals. Actual
  filesystem recovery still needs execution-time ownership/link checks and
  before/after measurement.
- All these cache classes can be regenerated or repopulated, but network
  availability, compilation cost, old dependency availability and original
  warm-cache conditions are not guaranteed. Retained experiment inputs and
  sealed artifacts must remain available; do not promise exact rollback.

## Exact selected cache sets

All paths below are relative to their experiment root. No selection includes
source, `private/`, `evidence/`, results, daemon logs, wrapper installations,
local Maven publication repositories or the shared heavy lock.

- Ablation, 27,853,299,712 bytes:
  - `gradle-home/caches`
  - `gui-runs/baseline-local-rule-r01/gradle-home/caches`
  - `gui-runs/baseline-local-rule-r01/konan`
- Rollout, 69,478,543,360 bytes: only `gradle-home/caches` and `konan`
  below each of these four `idea-current-runs/` labels:
  - `baseline-5b41be52-r03`
  - `candidate-koin121-961-r01`
  - `candidate-no-model-probe-r01`
  - `candidate-private-heap-r01`
- Import repair, 26,416,173,056 bytes:
  - `installed-source-seed/caches`
  - `gui-runs/baseline-installed-gradle-sources-r04/gradle-home/caches`
  - `gui-runs/baseline-installed-gradle-sources-r04/konan`
- KGP experiment, 34,508,910,592 bytes:
  - `cinterop-gradle-home/caches`
  - `cinterop-konan`
  - `gui-runs/baseline-kgp-reuse-r09/gradle-home/caches`
  - `gui-runs/baseline-kgp-reuse-r09/konan`
  - R36 is explicitly excluded; upstream source and sealed plugin artifacts
    remain outside this selection.
- Research 445, 28,402,118,656 bytes:
  - `gradle-home/caches`
  - `consumer-gradle-home/caches`
  - `idea-sync/gradle-home/caches`
  - `idea-sync/konan`
  - `fork-publication/gradle-home/caches`
  - `fork-publication/konan`
  - Keep the root and `device-heavy.lock`, publication artifacts and records.

## Smaller roots and confirmation boundary

- `kodex-gradle-remaining-20261011` is about164MiB and
  `kodex-gradle-adoption-20261011` about42MiB. Session 769's retained root is
  under5MiB. Removing these evidence-heavy roots is not a material space win.
- Both remote inventory commands completed. No additional deletion, migration,
  build, IDE operation or process termination occurred in this follow-up.
- Prioritize discussing the completed ablation environment and old rollout
  caches: the selected caches alone total90.65GiB. This is not authorization
  to delete their full roots; a whole-root request requires a separate
  evidence/source preservation decision.

## Explicit authorization and resource hold — 2026-10-12 11:35

- After the user said “删”, the assistant clarified that the proposed scope
  means the 24 selected cache directories, not the five full development
  roots. The user selected “仅删24处缓存”.
- This authorizes only the exact selected cache sets above. Source, private
  heaps, reports, publication artifacts, the shared lock, R36's borrowed
  caches and all Session 769 data remain protected. No whole-root deletion.
- Execution-time check at 11:35:20 +08:00 found the shared heavy lock held
  by python3 PID3508790, with Java PIDs3508792,3508837,3509064 active.
  Available space was35,852,865,536 bytes. No deletion has started.
- The main toolchain task now records resumed Node/Native validation after
  the first cleanup. Do not compete with its active workload, terminate its
  processes or try to reclaim its lock. No background cleanup is scheduled.
- Resume only after the existing owner releases the lock: recheck live
  processes/open-file usage, acquire the shared lock nonblocking, validate
  all 24 paths and preserve retained metadata before deletion. Abort if
  any owner is still active or any protected path overlaps the selection.
- Keep the first cleanup's audit intact. Use a separate second-batch audit;
  record each successful deletion and verify retained evidence afterward.
