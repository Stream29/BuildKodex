# Task Tree

- `Read coordinator boundaries and retained-resolution findings`() // complete
- `Trace official resolver contracts and publication counterexamples`() // complete
- `Deliver bounded static candidate and equivalence conditions`() // complete

# Details

- **STATIC READY / RUNTIME BENEFIT UNKNOWN.** Investigation archived; repair gate remains open.
- Production: user-attested clean `5b41be52`, KGP 2.4.0 / Gradle 9.5.1; candidate
  2.4.20 / 9.6.1 paused. All targets, navigation and 4 GiB remain.
- Coordinator owns every importer/GUI/heavy execution. This investigation uses
  static source/cache reads only; no test, Gradle, IDE, process control, MAT,
  raw heap, secret properties, network write, Git operation or source edit.
- Parent: [repair gate](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L1).
  Prior: [retention findings](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-08-investigate-resolution-retention-cost.md#L153-L176).

## Concrete probes: do not check only old libraries

- **Confirmed static blocker:** modern resolver requires non-null `gradleArtifact`.
  Transformed metadata creates empty extras; original metadata also omits this
  extra. Thus modern publication alone does not imply modern source attachment.
  [Modern 2.4.0, lines 35–40](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeSourcesVariantsResolver.kt#L35-L40),
  [transformed 2.4.20, lines 50–60](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeTransformedMetadataDependencyResolver.kt#L50-L60),
  [original, lines 32–39](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeOriginalMetadataDependencyResolver.kt#L32-L39).
- Probe `:utils-images-codec-impl / jvmMain`:
  `com.twelvemonkeys.imageio:imageio-jpeg:3.13.1` has cached POM, binary and
  sources JAR, no cached `.module`; classical Java source fallback risk.
  Include transitive `imageio-core` / `imageio-metadata`: cached sources, POM-only directories.
  [declaration, lines 39–40](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/build.gradle.kts#L39-L40).
- Probe same owner `/ commonMain`, `com.soywiz.korlibs.korim:korim:4.0.10`:
  cached metadata has source variants, **not** a confirmed pre-1.8.20 counterexample; test transformed metadata
  coverage instead; cached archive contains `commonMain/korlibs/image/Quality.kt`.
  [declaration, lines 33–37](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/build.gradle.kts#L33-L37).
- Probe same owner `/ skikoNativeMain`, `org.jetbrains.skiko:skiko:0.9.18`:
  modern metadata source variant exists; shared transformed binaries can lack the required extra.
  [declaration, lines 16–29](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/build.gradle.kts#L16-L29).
- Probe `:utils-shell-client-impl / jvmMain`, `org.jetbrains.pty4j:pty4j:0.13.12`:
  cached `sourcesElements` exists: modern-JVM positive control, not proof its transitives have variants.
  [declaration, lines 44–45](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/build.gradle.kts#L44-L45).
- Probe shell-client owner `/ commonMain` and `/ processClientMain`:
  `org.jetbrains.kotlinx:kotlinx-coroutines-core:1.11.0` and
  `kotlinx-serialization-core:1.11.0`; inspect actual transformed binary extras
  and source entries, not just published sources.
  [hierarchy/declarations, lines 17–42](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/build.gradle.kts#L17-L42).

## Confirmed mechanism and exact disable conditions

- Legacy runs after modern at the same normal priority, querying **all** selected
  graph components without inspecting `sourcesClasspath`; even zero attachable
  binaries do not prevent graph access/query. The default flag is true.
  [factory, lines 135–149](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImportFactory.kt#L135-L149),
  [legacy, lines 35–62][legacy], [default, lines 399–400](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/PropertiesProvider.kt#L399-L400).
- Global false loses existing attachments whenever legacy adds an archive that
  modern does not: pre-1.8.20 KMP without source variants, classical publications,
  transformed/original metadata excluded by the extra filter, or modern
  variant/capability mismatch. This is a source-level condition, **not measured
  navigation loss**. Metadata `.module`/recent library version is insufficient.
- Modern requests all binary coordinate groups with the extra, including
  transitives; matches **owner GAV plus the entire capability set**, ignoring
  `sourceSetName`. It requests every capability, and uses the sole associated
  target or metadata target; JVM sources request `java-runtime`, other targets
  `kotlin-runtime`, plus documentation/sources/external/platform attributes.
  [matching, lines 67–123][modern], [attributes, lines 168–182](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/publishing/Publishing.kt#L168-L182).
- `jvmMain` is not an automatic legacy exemption: legacy chooses compile
  configuration only for **exactly one non-common compilation**, otherwise
  internal metadata configuration. Modern chooses by unique **target** instead.
  Probe `jvmTest`, shared test sets and intermediate sets too. [selection][legacy]
- Legacy's attachment key is own GAV for zero capabilities, the single
  capability's GAV for one, **unmatchable for multiple**. It never preserves
  arbitrary multi-capability variant selection; leave that to modern.
  [key, lines 71–84][legacy]

## Smallest credible guarded candidate, not an adopted fix

- Higher priority in SourcesAndDocumentation suppresses **both** built-ins, not just legacy.
- Candidate: disable only `kotlin.mpp.import.enableSlowSourcesJarResolver`,
  preserve built-in modern resolution, register a normal-priority
  `PostAdditionalArtifactResolution` fallback on the public
  `IdeMultiplatformImport` boundary; no modern copy or internal metadata configuration. Public but
  opt-in `ExternalKotlinTargetApi`, not a stable resolver-replacement guarantee.
  [API, lines 77–82, 163–172, 214–218][api]; [execution, lines 197–224](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImportImpl.kt#L197-L224).
- `IdeDependencyResolver.Companion.gradleArtifact` exposes the already-resolved
  artifact's public `ModuleComponentIdentifier`; use `forComponents` directly.
  Alternatively public **`forModule(group,name,version)`** accumulates exact
  coordinates, including metadata binaries without that extra: no detached
  configuration, dependency traversal, reflection or internal ID constructor.
  [extra, lines 94–98](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeDependencyResolver.kt#L94-L98),
  [Gradle API][query-api], [implementation, lines 100–104, 151–180][query].
  Gradle labels ARQ maintenance-only and discourages new code: narrow compatibility shim, not a new framework.
- Guard before query creation: compile binaries only; complete dependency set
  (not declarations); exact non-null selected version; legacy's zero/one-cap
  key; deduplicate locally; return when no eligible request. Use imported
  component IDs when their GAV equals that key. Coordinate-only requests require
  proof the key identifies a component in the baseline selected closure:
  a capability name is **not necessarily a separately published module**.
- Attach only resolved source artifacts matching that key to every corresponding
  binary; preserve modern files and all binary/source-set/cinterop results.
  ARQ does not recursively discover transitives: the imported set must already
  contain them. It resolves metadata/artifacts for requested components, still
  permitting HEAD/I/O; avoiding a full configuration graph is not allocation-free.
- **Missing-only is conditional, not universally safe:** nonempty modern
  sources can coexist with additional legacy archives. Require baseline archive
  identities and relevant entries to match before skipping these queries.
  If they differ, query those groups too; one attached JAR does not prove complete coverage.
- **No general supported safe selective replacement is established.** Unknown
  capability/component identity, null version, or lost baseline attachments
  rejects this candidate; do not fabricate navigation or silently omit cases.
  Upstream correction: return before graph access on no eligible binaries;
  intersect selected IDs with keys. Preserves selection but can still force the
  internal graph; not a proved OOM repair. No facade/cache/framework/ResolverManager.

## Upstream and comparison context

- Official 2.4.0 and 2.4.20 retain the mechanisms above; inspected current
  [2.4.21 legacy](https://github.com/JetBrains/kotlin/blob/v2.4.21/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeArtifactResolutionQuerySourcesResolver.kt#L35-L62)
  and [modern](https://github.com/JetBrains/kotlin/blob/v2.4.21/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeSourcesVariantsResolver.kt#L35-L65)
  still do too. [2.4.21 release](https://github.com/JetBrains/kotlin/releases/tag/v2.4.21)
  has no listed fix for this path. [Master retains it](https://github.com/JetBrains/kotlin/blob/master/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeArtifactResolutionQuerySourcesResolver.kt); no speculative upgrade.
- [KT-87790 in 2.4.20](https://github.com/JetBrains/kotlin/releases/tag/v2.4.20)
  concerns sync visibility computation, not this fallback removal.
  [IDEA-252320](https://youtrack.jetbrains.com/issue/IDEA-252320) records variant
  source lookup gaps; it is not proof of this KGP OOM's cause or a fixed version.
- If the public guarded route fails, upstream request should specifically cover
  unconditional legacy graph acquisition, metadata binaries missing modern's
  extra, and capability-safe selective IDs. Minimum-dependency strategy: change
  only a **proved legacy-only publication**, if a version supplies source variants
  and preserves all contracts; does not fix the metadata-extra exclusion. No justified bump identified.
- Serialization has 14 explicit settings includes plus included build logic;
  repositories include cache redirectors/dev/local, and its checked properties
  specify HeapDumpOnOOM but no explicit heap maximum. Kodex uses exclusive fork
  groups plus Google/Central: not an OOM excuse, equal-model benchmark or budget/repository permission.
  [settings](https://github.com/Kotlin/kotlinx.serialization/blob/master/settings.gradle.kts#L4-L78),
  [properties](https://github.com/Kotlin/kotlinx.serialization/blob/master/gradle.properties#L14-L24),
  [Kodex repositories](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L17-L48).
- Coordinator gate: compare `(owner, sourceSet, binary GAV, sourceSetName, capabilities,
  artifact identity, source archive/entries)` against `enableSlowSourcesJarResolver=true`; then
  full genuine 4 GiB import/navigation. Static evidence does not quantify saved
  memory, retained graphs, avoided HEADs or GUI success. No such tests run here.

- Remote cached metadata/source JARs were read on **Xiaoxin**, not generated:
  [korim module, lines 60–77](file:///home/stream/ACodeSpace/demo/kodex-gradle-rollout-20261007/idea-current-runs/candidate-private-heap-r01/gradle-home/caches/modules-2/files-2.1/com.soywiz.korlibs.korim/korim/4.0.10/fee705e286ea25b294f035b2303916942c0c395e/korim-4.0.10.module#L60-L77);
  [pty4j module, lines 110–126](file:///home/stream/ACodeSpace/demo/kodex-gradle-rollout-20261007/idea-current-runs/candidate-private-heap-r01/gradle-home/caches/modules-2/files-2.1/org.jetbrains.pty4j/pty4j/0.13.12/c55d0c968e87a6b25b1d5a3675c9f9f8c0e3d0e7/pty4j-0.13.12.module#L110-L126).
  Cache absence of `.module` is not a separate remote-publication existence test.

[legacy]: https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeArtifactResolutionQuerySourcesResolver.kt#L35-L84
[modern]: https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeSourcesVariantsResolver.kt#L24-L123
[api]: https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImport.kt#L77-L218
[query-api]: https://github.com/gradle/gradle/blob/v9.5.1/subprojects/core-api/src/main/java/org/gradle/api/artifacts/query/ArtifactResolutionQuery.java#L25-L83
[query]: https://github.com/gradle/gradle/blob/v9.5.1/platforms/software/dependency-management/src/main/java/org/gradle/api/internal/artifacts/query/DefaultArtifactResolutionQuery.java#L100-L180
