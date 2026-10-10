# REVIEW READY — archived independent source-fallback diagnostic review

- **Allow bounded same-full-model GUI diagnosis; production adoption BLOCKED.**
- **Attention:** compile-type filtering admits synthetic Native-distribution GAVs; smallest guard is in B2.
- Independent reviewer; not coordinator/581 author. Only this new outer report is owned.
- No tests, Gradle, IDE, MAT, heap reads, process control, Git, remote writes or heavy wait.
- Main's legacy=true full GUI run and HeavyLock remain exclusively coordinator-owned.

# Task Tree

- `Read scope, repair task and independent investigation`() // complete
- `Check candidate hashes, source contracts and retained comparisons`() // complete
- `Deliver diagnostic permission and adoption blockers`() // complete

# Details

## R — scope and provenance actually checked

- Production remains user-attested `5b41be52d2f64a12ea3671bcecb9151fd2d1fa60`, KGP 2.4.0 / Gradle 9.5.1.
- Keep 4 GiB, every target, hierarchy, cinterop, binaries and existing navigation; no target profile.
- Context: [repair gate](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L1-L79),
  [581 investigation](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-08-investigate-legacy-source-fallback-fix.md#L55-L112).
- Reviewed candidate: [77-line acceptance adapter](file:///tmp/kodex-selective-idea-sources.init.gradle#L1-L77).
- SHA-256: `bab52e3c1d22c672bdb1956b2d5370012766ab0a11fc03aa20aacdb94c618231`.
- Local candidate, remote `selective.init.gradle`, and staged helper have that same hash.
- All remote file URIs below identify **Xiaoxin**, not this host. Receipt root:
  [source-repair evidence](file:///home/stream/ACodeSpace/demo/kodex-import-sources-repair-20261008/).
- Control/selective identity maps differ at exactly three paths: root `build.gradle.kts`,
  `gradle.properties`, `gradle/idea-missing-sources.gradle`; checked current hashes match the delta.
- Stage script verifies control-file hashes before copying; adds root apply, import flag=false and helper only.
  [stage source](file:///tmp/kodex-selective-sources-stage.py#L14-L41),
  [retained delta](file:///home/stream/ACodeSpace/demo/kodex-import-sources-repair-20261008/gui-source-delta.json#L1-L20).
- Selective manifest records 216 registered projects, full_targets=true, accepted commit and production versions.
  This checks manifest provenance, not a new complete on-disk census or completed GUI import.
- Cached generic and gradle813 KGP 2.4.0 source archives were inspected; Native producer agrees in both.
- Runtime-variant source SHA-256: `9c620eb9dc218e0106cca714804589a5612e55ebc2db112feafc2126dbe2d193`.
- Earlier probe revisions were mutable; current hashes do not retrospectively seal their first execution.

## Mechanism — public supplement, not a replacement model

- Flag=false disables only the legacy query registration; original `IdeSourcesVariantsResolver` remains.
  [KGP factory](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImportFactory.kt#L134-L148).
- Public opt-in `IdeAdditionalArtifactResolver` permits attaching extras to prior dependencies, not removing data
  or modifying source sets. Normal `PostAdditionalArtifactResolution` follows modern sources without overriding them.
  [contract](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeAdditionalArtifactResolver.kt#L29-L46),
  [phase/priority implementation](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImportImpl.kt#L170-L224).
- Candidate considers resolved `KOTLIN_COMPILE` binaries with empty sources; project-source dependencies are excluded.
- Zero capabilities use own GAV; one uses capability GAV; multiple cannot match original legacy either.
  Requests must occur among imported compile-coordinate GAVs; no guessed separately published capability module.
  [original key](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeArtifactResolutionQuerySourcesResolver.kt#L35-L84).
- Deduplicated exact-version `forModule` requests use public `JvmLibrary` / `SourcesArtifact`; original legacy
  also uses JvmLibrary for Native source sets. Only resolved artifacts with actual module GAV matching the key attach.
- Gradle's query accumulates component IDs and resolves their metadata/artifacts, not a transitive dependency graph.
  `withDefault` creating an empty unmatched result bucket cannot issue another query or resolve transitives.
  [Gradle 9.5.1 implementation](https://github.com/gradle/gradle/blob/v9.5.1/platforms/software/dependency-management/src/main/java/org/gradle/api/internal/artifacts/query/DefaultArtifactResolutionQuery.java#L100-L180).
- No candidate-created configuration, binary transform, source-set mutation, replacement cache/model, owner job or timer.
  ARQ can still perform repository I/O and allocate; modern's original detached configuration remains.
- ARQ is public but maintenance-only; not a recommended new framework.
  [Gradle API](https://github.com/gradle/gradle/blob/v9.5.1/subprojects/core-api/src/main/java/org/gradle/api/artifacts/query/ArtifactResolutionQuery.java#L24-L75).
- Proxy/reflection is the classloader-crossing experimental adapter to public opt-in APIs, not shipping business API.
  A typed extension would be required **after** real benefit; no generic ResolverManager/interface extraction is needed.

## B1 — concrete source-loss control, repaired only within measured scope

- Blind false without supplement loses commonMain stdlib/core/json serialization sources and nativeMain
  serialization sources: actual RED, not a conjecture based on publication age.
  [recorded changes](file:///home/stream/ACodeSpace/demo/kodex-import-sources-repair-20261008/metadata-native-comparison.json#L1-L92).
- Modern excludes binaries lacking `gradleArtifact`; transformed metadata uses empty extras and original metadata
  omits that extra. Recent Gradle Module Metadata alone therefore cannot justify a global disable.
  [modern filter](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeSourcesVariantsResolver.kt#L35-L40),
  [transformed producer](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeTransformedMetadataDependencyResolver.kt#L50-L60),
  [original producer](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeOriginalMetadataDependencyResolver.kt#L32-L38).
- Independently normalized retained JSON by removing only `seconds`: original v2 legacy=true equals selective false
  for Images JVM and rpc-exception commonMain/nativeMain/linuxX64Main, including recorded binary/source SHA and entries.
  [comparison](file:///home/stream/ACodeSpace/demo/kodex-import-sources-repair-20261008/selective-metadata-comparison.json#L1-L22).
- Original baseline JSON SHA: `d751311e955f0868e7e803828820c62e17bff9ae97921d62ee63fa5a6759c820`.
- Selective false JSON SHA: `14e377d2ba5df731264de586cbecd520f8cf86da4eeb48f067f39d5f506db031`.
- Both selective flag contexts included the helper; comparison above deliberately uses the separate original baseline.
- Minimum useful ablation is already source-RED without fallback; no new ablation was executed by this reviewer.
- JVM/JS four-owner/five-source-set original flag probes: 57.972s false / 111.265s true, exit=0.
  Non-randomized cache-sensitive first passes do not establish repeatable wall-time, memory or GUI benefit.
- Lucene true adds a second filename, but both archives share SHA
  `bca36b8de971b16467743a1631bd277f606429cf2cea739621d39d11a93f5e41` and 1,373 entries: not extra source content.
  [original comparison](file:///home/stream/ACodeSpace/demo/kodex-import-sources-repair-20261008/jvm-js-comparison.json#L1-L4195).

## B2 — adoption boundaries and source-counterproducer gates

- **Concrete correction:** Native platform/distribution binaries are also `KOTLIN_COMPILE`, not excluded by that filter.
  Their producer derives coordinates from local KLIB names and marks `isNativeDistribution=true`.
  [actual producer](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/resolveNativeDistributionDependency.kt#L25-L49).
- Retained selective rows contain source-empty builtin/iconv/posix/zlib, plus linux on linuxX64Main.
  These pass the candidate's own-GAV/imported-set guard; those coordinates are not proof of selected repository modules.
  Static consequence: the adapter requests these synthetic GAVs. No observed source loss or measured overhead is claimed.
  [retained Native rows](file:///home/stream/ACodeSpace/demo/kodex-import-sources-repair-20261008/selective-metadata-native-legacy-false.json#L3078-L3130).
- Smallest guard is the existing Native-distribution marker before module requests, preserving binary/local attachments;
  do not require `gradleArtifact` globally, which would undo the proved metadata repair. Candidate was not edited here.
  [public marker](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin-idea/src/main/kotlin/org/jetbrains/kotlin/gradle/idea/tcs/extras/primitiveExtras.kt#L21-L27).
- Nonempty modern sources can coexist with an additional **distinct-content** legacy archive. Candidate skips that case.
  Lucene's byte-identical duplicate is not such a counterexample; no distinct-content current-project witness is established.
- Required counterproducer/comparison gate: modern attaches A, original legacy adds B with unique SHA/entries;
  missing-only must preserve that coverage or reject adoption. Compare complete attachment identities and navigation,
  not just “at least one source JAR”. Also prove imported GAV coverage against original selected components.
- Null/unsupported coordinate or capability compositions remain scoped unproved cases, not blanket wrapper guilt.
- Additional concrete settings difference: KGP's download-sources suppression covers SourcesAndDocumentation only,
  not PostAdditional; this supplement can query when modern/legacy are suppressed. Guard that opt-out for adoption;
  current source-enabled diagnostic is not rejected for this separate setting.
  [suppression condition](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImportImpl.kt#L200-L209).
- Both query paths ignore unresolved components/artifacts. Unexpected proxy/query throws reach KGP's logged resolver
  error collection; non-strict import may continue, strict import may throw. Never treat such errors as a successful repair.
  [catch/report behavior](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImportImpl.kt#L158-L168),
  [resolver wrapper](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImportImpl.kt#L308-L320).
- ARQ resolution alone does not validate archive contents/corruption; the retained probe hashes and opens sampled JARs.

## U — unresolved full acceptance, not permission to ship

- Known fallback behavior differences block adoption, not an honestly labelled same-full-GUI diagnostic.
  Keep any observed loss/error reported; a completed import alone does not prove navigation preservation.
- No claim of selective coverage for all 203 KMP modules, complete capabilities, complete attachments or all-target tests.
- Require full binary/source/identity equality, source hierarchy/cinterop and real navigation comparisons,
  representative compile/functional regressions, and runtime resource benchmarks before production adoption.
- Four retained samples omit separate capability/attribute/artifact-ID fields; they cannot close those broader gates.
- Full GUI/OOM outcome remains coordinator-owned and pending in this review; no waiting, window takeover or lock acquisition.
