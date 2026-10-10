# REVIEW READY — independent private streaming-observer review

- Scoped static review complete; the findings below preserve their original
  review-time evidence. Later runtime outcomes belong to the active parent,
  not a retrospective revision of this review.

- **B1: no confirmed static blocker in successful snapshot serialization.** Minimal observer-only ablation is justified; this is not an accepted Gradle/IDE OOM repair or GC fix.
- **B2: failure-file visibility changes and incomplete-model acceptance require explicit boundaries.** Existing positive-snapshot validation is not full model/navigation acceptance.
- **U: measured memory benefit, exact final R27/R28 bytes, cold/full GUI, navigation and fresh product regressions remain pending.** Review is complete independently of R28; do not wait for heavy work.

# Task Tree

- `Read scope, active experiment and parent repair`() // complete
- `Verify fixed source and map original callback boundaries`() // complete
- `Record static findings and outstanding acceptance gates`() // complete

# Details

## Scope and fixed identities

- Parent: [full-model OOM repair](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L1); active [receive-retention investigation](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-10-investigate-private-ide-receive-retention.md#L1).
- Reviewer writes only this new outer task file. Read-only source/metadata/receipt SSH; no source/test/other-document/Git/network writes, builds/tests (including Python), IDE/MAT/heaps/credentials, process or device control.
- Coordinator alone owns Xiaoxin heavy work under `device-heavy.lock`; no parallel heavy operation or interference with local gaming. No reviewer-created temporary files/resources require cleanup.
- Fixed [changed source](file:///tmp/kodex-streaming-observer-20261010/src/research/SyncObserver.java#L1): SHA256 `00a18e60c79d0ac84aa8032af108929344f27657674db69248f1ae8ecda4a17a`, verified twice locally and against remote build input.
- Immutable [original source, on Xiaoxin](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/observer/src/research/SyncObserver.java#L1): SHA256 `f31bdf7002f15c70472ddf3d677b41008c2ca3d2f9db626a95dfb8b7310b7d0a`, verified read-only. Remote `file://` links below identify Xiaoxin, not local copies.
- Actual private JAR SHA256 `647f4458c522f2fb01f54b9bcaf755f0a191554a883e71ab0178019df0a92a0f`, independently hash-read; [build receipt](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/streaming-observer/build-receipt.json#L2) records release21 and installed-IDE API compilation.
- Receipt preserves the missing-JBR-`jar` packaging failure and JDK25 packaging correction. All IDE/Java/Gradle API jars are a tiny observer's compile classpath, not a product-toolchain/source change.

## R — actual responsibility and original callchain

- FQCN stays `research.SyncObserver`, implementing `StartupActivity.DumbAware`; [private metadata](file:///tmp/kodex-streaming-observer-20261010/META-INF/plugin.xml#L1) is text-identical to the original remote descriptor, including passive `postStartupActivity`, dependencies/version/build range.
- Original `runActivity` → project-disposable dumb-mode/editor listeners and external notification adapter → `snapshotWhenSmart` through startup, dumb exit and external end: [original55–110](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/observer/src/research/SyncObserver.java#L55) maps to [changed55–112](file:///tmp/kodex-streaming-observer-20261010/src/research/SyncObserver.java#L55).
- Original sorted module/root traversal → `add(StringBuilder,...)` → whole `Files.writeString` → SMART event: [original111–144](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/observer/src/research/SyncObserver.java#L111) maps to [changed113–146](file:///tmp/kodex-streaming-observer-20261010/src/research/SyncObserver.java#L113).
- Diff is only two imports, whole-snapshot builder/write replaced by try-with-resources writer, and private helper parameter/checked IOException. No Gradle action/model request, target/test/API/source-input change or installed IDE/JAR/cache surgery; no invented ObserverManager/facade/platform cache.
- Actual full snapshot is202,842,210 bytes: [verified archive receipt18–21](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/completed-r27-model-archive-receipt.json#L18). Removing observer whole-snapshot assembly/copying is the minimal ablation; exact allocation multiplier and memory savings are not measured.

## B1 — successful-content and lifecycle review

- Header, sorted module names, SDK/content/source/order/library-source/library-class traversal and nested API calls are unchanged at [changed111–137](file:///tmp/kodex-streaming-observer-20261010/src/research/SyncObserver.java#L111). Ordering within returned arrays is preserved, not newly normalized.
- Both no-option file APIs use UTF-8 and CREATE/TRUNCATE_EXISTING/WRITE; explicit LF is retained, not OS-dependent `newLine()`. Both report encoding/I/O errors: [JDK21 Files contract](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/file/Files.html#newBufferedWriter(java.nio.file.Path,java.nio.file.OpenOption...)).
- [Helper144–146](file:///tmp/kodex-streaming-observer-20261010/src/research/SyncObserver.java#L144) retains value tab/LF→space; CR remains unchanged. Module/kind fields remain unescaped. These are pre-existing TSV limitations, not a new escaping bug; the event logger still strips tab/LF/CR separately.
- The six actual compiled-helper cases pass in [check receipt1](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/streaming-observer/check.log#L1). [Check source23–38](file:///tmp/kodex-streaming-observer-20261010/ObserverEncodingCheck.java#L23) covers empty/ordinary/control/Chinese/supplementary Unicode/JAR URL values via real private helpers; it compares characters through StringWriter, not on-disk bytes, malformed UTF-16 or full traversal.
- `runWhenSmart` → pre-read-action `project.isDisposed()` → `runReadAction` is unchanged at [106–109](file:///tmp/kodex-streaming-observer-20261010/src/research/SyncObserver.java#L106). No new post-lock disposal check is added; the prior potential disposal interval is neither enlarged by new scheduling nor repaired.
- All module/root reads and writing still occur inside that read action. Writer closes on normal/exceptional exit; open/write/flush/close IOException becomes IllegalStateException. SMART follows successful close at [138–139](file:///tmp/kodex-streaming-observer-20261010/src/research/SyncObserver.java#L138); failed snapshot writes cannot emit that completion marker. Event-write failure still throws through `log`.
- Snapshot text retention is bounded by writer/encoder buffers plus current field/row strings, not total TSV. Module sorting arrays, root/order/library API arrays, SDK strings and per-value replacement allocations remain; no zero-allocation/constant-total-IDE-memory claim. Event logging is unchanged.

## B2 — specific caveats and reproducers, not blanket rejection

- **Failure visibility differs:** new code opens/truncates before traversing roots; old code assembles first. Concrete unexecuted reproducer: make a later root read fail, or fail writer flush/close after several rows. New output may contain a prefix and no SMART; old traversal failure occurs before opening. IOException type/wrapping and “no SMART on write failure” are preserved, not identical file side effects.
- Files may be visible while writing; neither implementation promises atomic publication. Consumers must pair filename/sequence with successful SMART and validate the complete model, not treat existence or a partial exported TSV as completion. Try-with-resources also preserves a primary traversal failure with a close failure suppressed.
- **Known weak gate:** [collector99–103](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/idea-gate-private-recovery/collect.py#L99) accepts any positive smart count after each actual success. Concrete existing control: R26 restored5245 → refresh0/217/625;625 has no library roots. With required success/playback markers, that intermediate meets this count predicate but is not a complete model.
- Reviewed [R28 launcher140–152](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/kodex-private-ide-streaming-observer.py#L140) adds final5245 and positive SOURCE/LIBRARY_SOURCE/LIBRARY_CLASS counts. That rejects the known625/no-library state, but5245 duplicate/wrong module identities or one substituted library URL could still meet counts. Stronger final gate needs exact modelIds and actual per-module roots/attachments against the sealed full baseline, then interactive navigation separately.
- Same launcher's `model.read_text().splitlines()` loads the full TSV in the coordinator, so the complete harness is not bounded-memory merely because the IDEA writer is. Potential host-reserve effect is unmeasured; do not misattribute collector allocation to IDEA Java heap.

## U — measurements and whole acceptance still required

- R26 `-all` MAT reachable≈796.4MiB versus passive Java used≈2GiB/RSS≈4.1GiB is an active-refresh capture, not steady-state live retention or a GC-fix proof; preserve that phase distinction in [active evidence](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-10-investigate-private-ide-receive-retention.md#L47).
- R27 [result](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/gui-runs/baseline-kgp-v3-compact-idle-gc-r24/private-ide-normal-reclamation-r27/evidence/result.json#L2) is warm-only895.235s; [counts](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/gui-runs/baseline-kgp-v3-compact-idle-gc-r24/private-ide-normal-reclamation-r27/evidence/final-model-counts.json#L4) confirm5245 modules/418 sources/86434 library-source/328720 library-class roots. Four native imports are not four full source-navigation passes.
- Coordinator reports R28 serial genuine GUI with new observer/original IDEA GC defaults. Reviewed launcher keeps Gradle4GiB/IDE3GiB/compiler2GiB,4096MiB reserve and2400s deadline; it changes no infinite/full-capture budget. Final R28 receipt was absent at the reviewed launcher path when checked; no completion or memory benefit is inferred.
- R27→R28 also changes IDEA GC policy and uses successive recovered cache state, so their memory difference alone is not an observer-only causal estimate. Any coordinator-run large-assembly control must freeze source/model/GC/cache/budgets, measure observer allocations/IDE heap separately from RSS/host reserve, preserve old controls, and serialize heavy work.
- Exact final on-disk R27/R28 comparison remains pending; equal counts or six helper cases cannot substitute for it. Retain source/JAR hashes, original artifacts/tags and raw-equivalent evidence; explain any actual content difference before acceptance.
- Whole repair still requires complete genuine initial/repeated GUI import, exact targets/source ancestry/modelIds/actual binary+source roots, interactive navigation including dependency/Gradle sources, meaningful fresh tests and unchanged public API at the fixed budget. No target reduction, timeout extension, failure masking or production adoption follows from this review.
- Static instrumentation cost is an IDEA measurement contaminant, **not the demonstrated independent Gradle OOM cause**. Reduce that diagnostic cost before assessing further IDEA tuning; do not rename it the accepted4GiB repair.
