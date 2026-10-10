# REVIEW READY — archived guarded-source / installation diagnostic review

- **Allow bounded controlled full-model GUI diagnosis; production adoption BLOCKED.**
- Canonical source-download booleans preserve KGP's paired opt-out; noncanonical parsing caveat is in B2.
- Independent reviewer, not coordinator/581/584; only this report is writable.
- No tests, Python units, Gradle, GUI, MAT, heap reads, process control, Git or secret/config reads.
- Coordinator owns the third genuine GUI run, heavy lock, deadline and cleanup; reviewer does not wait.

# Task Tree

- `Read scoped context and verify both fixed hashes`() // complete
- `Inspect guards, official source mapping and receipt metadata`() // complete
- `Report diagnostic readiness and remaining acceptance boundaries`() // complete

# Details

## R — checked scope and provenance

- User-attested clean production baseline `5b41be52`; KGP 2.4.0 / Gradle 9.5.1.
- Retain all targets, hierarchy, cinterop, binaries, existing navigation and 4GiB; no target profile.
- Context: [parent repair](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L9-L33),
  [584 v1 review](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-08-recheck-selective-source-fallback-diagnostic.md#L63-L122).
- Independently hashed [V2 adapter](file:///tmp/kodex-selective-idea-sources-v2.init.gradle#L1-L79):
  `72913637819b5899a054c705d4f937ccb5b88b3442bc234b2690c8a0d6e6fdbd`.
- Independently hashed [installer](file:///tmp/kodex-install-official-gradle-sources.py#L1-L87):
  `2926a3e42140e7c6f8ac0e406aa8edc95e4936fe390f300bf16e73ec6d2cb722`.
- Read cached KGP 2.4.0 runtime/IDE source archives and Gradle 9.5.1 source ZIP; no installer execution.
- Local source ZIP is 75,062,830 bytes; independently computed SHA-256:
  `7196ad0b60c1cd37d16a99e0282c643773876920783503236d1259fe8901d79c`.
  Official-checksum provenance is [coordinator evidence](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L118-L121), not a new reviewer download.
- Read only remote receipt counts/hash metadata: 683 installation files, 10,942 sources, 210 roots.
  [Receipt](file:///home/stream/ACodeSpace/demo/kodex-import-sources-repair-20261008/installed-sources-receipt.json#L1) is on **Xiaoxin**, not this host.
- Receipt hashes for `SourcePathProvider.kt` and `FindGradleSources.kt` match local ZIP contents.
  This verifies those recorded contents, not an independent rehash of the whole installed tree.

## B1 — confirmed corrections and concrete control

- V1 actually requested CoreNFC as Maven; V2 reads public `PrimitiveExtrasKt.isNativeDistribution`
  before both selected-GAV collection and requests. KGP's local Native producer sets that marker.
  It excludes synthetic distribution coordinates without removing binaries or existing attachments.
  [Guard](file:///tmp/kodex-selective-idea-sources-v2.init.gradle#L20-L38),
  [actual GUI witness](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L90-L99).
- Paired opt-out uses the original Gradle-property=true AND IDE-system-property=false condition;
  absent IDE property does not disable sources. Both canonical booleans match KGP's condition.
  [Pair](file:///tmp/kodex-selective-idea-sources-v2.init.gradle#L31-L33).
- Modern resolver stays registered; V2 supplements normal-priority PostAdditional only.
  Compile binaries, missing sources, exact non-null version, original zero/one-capability key,
  membership in imported own-GAVs, deduplication and exact returned module GAV remain.
  [Selection/attachment](file:///tmp/kodex-selective-idea-sources-v2.init.gradle#L34-L77).
- ARQ requests only those components; no new configuration graph, binary transform, source-set mutation,
  fake provider/model cache or resource-owning job. ARQ still permits repository I/O/allocation.
- Blind false was genuinely source-RED for common/native stdlib/serialization.
  Coordinator's V2 real-importer comparison against separate original legacy-enabled baseline records
  equal binary/source contents for Images JVM and rpc-exception commonMain/nativeMain/linuxX64Main,
  now with explicit GAV/sourceSetName/capabilities/Native markers. Not rerun by reviewer.
  [Control and V2 result](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L47-L54),
  [expanded comparison](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L97-L100).

## B2 — specific installation and adoption boundaries

- Installer copies previous **private** wrapper tree, preserves same-version `bin` distribution layout,
  verifies source SHA, writes only real `src` content there, then checks every recorded binary-file hash.
  No binary upgrade, global installation mutation or relabelled official `all` package.
  [Copy/hash checks](file:///tmp/kodex-install-official-gradle-sources.py#L12-L36),
  [verification/receipt](file:///tmp/kodex-install-official-gradle-sources.py#L65-L87).
- Main-source mapping agrees with actual Gradle transform: `subprojects/<project>` or
  `platforms/<platform>/<project>`, consume `src/main/<language>`, retain project + remaining path.
  Absolute/parent paths, symlink file entries, wrong prefix and differing-content collisions reject;
  length/slice guards prevent truncated-entry indexing. Identical duplicates merge as Gradle does.
  This is a fixed trusted-hash archive assumption, not a generic untrusted extractor.
  [Mapping](file:///tmp/kodex-install-official-gradle-sources.py#L38-L64),
  [matching Gradle transform](file:///home/stream/ACodeSpace/demo/kodex-import-sources-repair-20261008/installed-source-seed/wrapper/dists/gradle-9.5.1-bin/iq79hdu3mqx29lgffhp8bfmx/gradle-9.5.1/src/kotlin-dsl/org/gradle/kotlin/dsl/resolver/FindGradleSources.kt#L76-L120).
- Official `SourcePathProvider` first loads installation `src` subdirectories; only absence invokes
  downloaded sources. These populated roots can bypass detached source acquisition while retaining
  actual script-source contents. Empty/fake `src` would suppress fallback and is not acceptable.
  [Official source path](file:///home/stream/ACodeSpace/demo/kodex-import-sources-repair-20261008/installed-source-seed/wrapper/dists/gradle-9.5.1-bin/iq79hdu3mqx29lgffhp8bfmx/gradle-9.5.1/src/kotlin-dsl/org/gradle/kotlin/dsl/resolver/SourcePathProvider.kt#L73-L89).
- Only `caches`/`jdks` are additionally copied, not daemon, root properties or IDE state.
  Reused dependencies/Native caches prohibit empty-cold-cache claims; timing comparisons need matched seeds.
  Failure may leave partial private seed; quarantine it unless completed checks/receipt succeed.
  [Cache scope](file:///tmp/kodex-install-official-gradle-sources.py#L70-L84).
- Wrapper URL stays `9.5.1-bin`, not `all`; URL basename/hash selects the copied `wrapper/dists` tree,
  and existing `.ok`/valid-root checks reuse it. The installer itself does not select GUI Gradle home.
  [Unchanged wrapper](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/gradle/wrapper/gradle-wrapper.properties#L1-L9).
- Coordinator must establish effective private home: wrapper `-g` wins home selection; otherwise
  `gradle.user.home` system property (including project `systemProp.gradle.user.home`) precedes environment.
  Environment-only selection is insufficient proof; no private property file was inspected here.
- Narrow parser mismatch: Groovy `String.toBoolean()` trims and accepts `y`/`1`; KGP's Kotlin parser
  accepts only case-insensitive `true`. Static jar disassembly confirms this; pair is exact only for
  canonical inputs. Noncanonical settings equivalence remains an adoption check, not observed GUI loss.
  [Affected adapter](file:///tmp/kodex-selective-idea-sources-v2.init.gradle#L31-L33).
- Missing-only still skips modern A plus distinct-content legacy B. No current-corpus witness is
  established; byte-identical Lucene duplicates do not prove one. Do not claim solved, or block this
  source-enabled diagnosis solely on an unsupported counterexample. Whole-corpus adoption must compare
  complete attachments/entries, selected-component identities and real navigation, including capability keys.
- Reflection remains quarantined in the public-API diagnostic adapter; not a production extension/framework.

## U — genuine acceptance and minimum ablation

- Second actual GUI ended at 2400s without captured external SUCCESS or recorded OOM;
  sampled source resolution was `SourceDistributionResolver -> SourcePathProvider -> Kotlin DSL model`
  with `gradle-9.5.1-src.zip`. No raw heap inspected; timeout is not accepted import or a proved OOM fix.
  [Observed gate](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L101-L121).
- Official `all` checksum availability is not an acquired/verified `all` ZIP. Coordinator reports bounded
  slow GitHub download stopped using proven own-curl ownership; this seed is private bin + official sources.
- Minimum source ablation already has blind-false source-RED versus supplemented coverage.
  Next gate is coordinator-owned r03 versus guarded r02: same V2/full model/4GiB/old GUI harness,
  installed real source roots are the intended intervention; different cache histories prevent a strict
  single-factor speedup claim. No extra Gradle model probe or reviewer-run experiment is required here.
- Require actual external GUI success, no missing/error suppression, full source/script/navigation coverage,
  binary/target/hierarchy/cinterop equality and functional regressions before adoption.
- No all-203-module import pass, speedup, peak-memory reduction, tests or benchmarks claimed by reviewer.
