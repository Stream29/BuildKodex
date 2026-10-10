# REVIEW READY — independent fixed-overlay review

# Task Tree

- `Review fixed source overlay against pinned upstream parser and consumers`() // complete
- `Identify correctness blockers and exact runtime gates`() // complete
- `Deliver scoped experimental verdict without adopting artifacts`() // complete

# Details

- Parent: [isolated experiment](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-09-experiment-kgp-metadata-producer-reuse.md#L1); main [full GUI OOM repair](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L1).
- Independent READONLY review: only this outer report changed; no source edits, tests/builds, IDE/MAT/process/resource control, Git/network writes, private heaps, credentials or raw-property dumps.
- Loaded project/workflow, Gradle, abstraction/lifecycle and upstream root/KGP/.ai instructions. Isolated source has no IDE/MCP project; used the explicitly authorized SSH-read fallback.
- Supplied official v2.4.0 pin: commit `add726ca8c82922b6ab4cb2a27ae738d6a780817`, tree `e94052d76d53e57f3dc6592f89a245657d0906e4`; original source read on Xiaoxin, not modified.
- Independently verified fixed overlay SHA-256:
  - [Factory][OF]: `afb62a4addff1a0713c6447d67fd2e5ba0a72005da3c167ccd719aaa44ef450f`.
  - [Test][OT]: `6785fd586f2eac4cf40eeba8d29bb7557a17b458c3c3592ed2174602ee93982d`.
  - [Extractor][OE]: `47f439673abaa347e6df5c387e1cf80567375d511adb67008752b1e623ea06a8`.

## Verdict and B1: deterministic test-oracle blocker

- **Controlled build-route investigation may continue; this exact overlay is not ready to pass its regression gate.** One source-proven B1 is in the test fixture, not a demonstrated cache corruption. No runtime execution was performed by this reviewer.
- FQCN `org.jetbrains.kotlin.gradle.unitTests.KotlinProjectStructureMetadataReuseTest`: [snapshot21–29][OT21] supplies only `nativeMain -> {commonMain}` in `sourceSetsDependsOnRelation` and only `commonMain -> {group:module}` in `sourceSetModuleDependencies`.
- Call chain: `withFile` writes `sample.toJson()` → factory `readProjectStructureMetadata` → `org.jetbrains.kotlin.gradle.plugin.mpp.parseKotlinSourceSetMetadataFromJson` → `parseKotlinSourceSetMetadata`.
- Original [serializer223–230][PS223] emits **both** source sets, including empty `dependsOn` for commonMain and empty `moduleDependency` for nativeMain. Original [parser329–363][PS329] inserts both map entries unconditionally.
- Parsed/frozen metadata therefore additionally contains `commonMain -> emptySet()` in dependsOn and `nativeMain -> emptySet()` in module dependencies. Map equality distinguishes missing from empty entries; `assertEquals(sample, …)` at [snapshot49][OT49], [111][OT111] and [131][OT131] must fail after successful parsing.
- Minimal correction for implementer: preserve normalization, either complete those two fixture entries or compare against the unchanged parser's result for the same JSON. Do not remove assertions or change parser semantics. Revised files require new hashes and actual regressions.

## Static cache/consumer findings

- Original FQCN `org.jetbrains.kotlin.gradle.plugin.mpp.KotlinProjectStructureMetadataExtractorFactory.getOrCreate` [58–61][UF58] creates a factory per call. `GranularMetadataTransformation.Params` takes the consuming Project factory at [174][GT174]; extraction calls `create` then `getProjectStructureMetadata` at [513–520][GT513].
- Overlay [42–66][OF42] reads `file.readText()` on **every** call before comparing exact decoded String content. Path/mtime/length alone never establishes a hit. `absoluteFile` aliases can duplicate entries, not return another file's stale metadata. No GAV or cross-project cache key appears.
- Current read/parse failure propagates without catching or serving an old success; old snapshot stays stored until successful replacement. IOException behavior and JSON parser exception path are preserved. Missing selected artifacts still follow original warnings/null.
- Overlay [70–97][OF70] preserves selected-component lookup, PSM Usage, empty-marker exclusion and `singleOrNull`. Cache contains no resolved component/configuration, visibility result or provider graph.
- All seven collection fields are copied into ordered `LinkedHashMap`/`LinkedHashSet` wrappers, including nested sets; `.copy` retains `formatVersion` and `isPublishedAsRoot`. Original [typed fields99–149][PS99] define nine fields total.
- Original parser [334–337][PS329] constructs **base** `ModuleDependencyIdentifier` instances. Despite public class/open vals allowing subclasses, these parser values have immutable String fields; layout values are the two singleton objects at [72–90][PS72]. No mutable parser element remains shared.
- Original [serialize/toJson201–269][PS201] iterates variant maps, sourceSetNames, dependsOn sets and dependency sets. Freezing preserves insertion iteration; Map/Set equality alone would not prove this.
- Actual mutation trace: `GranularMetadataTransformation.processPSMDependency` [397–447][GT397] reads metadata and adds dependencies into a **new** mutable set; each visibility decision and metadata provider is newly computed for its consumer.
- `SourceSetVisibilityProvider.getVisibleSourceSets` [135–211][SV135] uses union/intersection/filter/associate and separate sorting state, without mutating PSM. Host-specific `.keys.first()` at166–169 makes preserved map order significant.
- `CompositeMetadataArtifactImpl` [42–47][CA42], [80–93][CA80], [106–108][CA106], [174–182][CA174] reads names/layout/cinterop paths and creates separate content/file resources. These providers/resources are not cached.
- Native `ChooseVisibleSourceSets.visibleSourceSetProvidingCInterops` [78–92][CI78] reads relation/variant maps and constructs new sets. Producer `GenerateProjectStructureMetadata.generateMetadataXml` [94–109][GP94] uses `.copy` with a new dependency map, not in-place mutation (and is not the cached parser path).
- No required mutation of shared parsed collections was found in these actual KGP consumers. A public metadata type alone does not establish a mutable-map contract; external mutation needs a concrete caller/countertest, not a default B1.
- Overlay [extractor36–45][OE36] keeps uncached parsing as default; only actual factory passes its method reference. Jar JSON/XML and included-build/provider extractor behavior remains unchanged; no Node or cinterop bypass appears.

## Ownership, publication and configuration-cache boundary

- Overlay [99–105][OF99] uses `projectStoredProperty` and `synchronized(project)` around this factory's get/init. Original [StoredLazyProperty74–75, storage104–118][SP74] stores values in each Project's extra storage, keyed by delegate identity; [ExtraPropertiesExtension.getOrPut11–18][EP11] separately guards storage initialization.
- Companion stores delegate/initializer, **not** Project models. Static delegate lifetime keeps its key alive within each Project storage, not a global map of Projects. Snapshot values live with reachable Project/factory consumers.
- Actual source-set Project is the final constructor property at [DefaultKotlinSourceSet28–30][DS28]; [source-set transformation17–30][ST17] and task constructors use consuming Project → Params → factory. No new ownerProject decorator, producer lookup or rootProject substitution appears.
- Snapshot read/compare/parse/freeze/publication share the private map monitor. No Gradle resolution, provider query or project-monitor acquisition occurs inside it. Factory acquisition completes before file reading; no newly visible reverse lock cycle was found.
- This guards callers of **this** getter, not all unrelated KGP WeakHashMap entries. Concurrent-reader test already obtains factory at117; cold first-access identity is an additional targeted test, not demonstrated failure.
- Method reference retains factory(logger + snapshots), not a Gradle Project/Job/global model. `readText` retains no file handle. Frozen containers retain values only, not original parsed metadata/maps/JSON tree; temporary parse/freeze containers become collectible, not immediately measured as reclaimed.
- **Configuration-cache compatibility is unproven, not rejected merely for lacking Serializable.** Original factory is already non-Serializable and already in Params. Concrete task reachability: [MetadataDependencyTransformationTask.transformationParameters71–72][MT71] and [CInteropMetadataDependencyTransformationTask.parameters119–120][CT119] → [Params.factory130–131][GT130].
- New state is snapshot map/objects; extractor callback is local to extraction, not added to Params/task fields. No new direct Project-valued field was found. Compare original/patched cache store/load on those actual tasks, including populated-before-store state and changed/missing PSM on reload; an actual new diagnostic, stale result or codec failure would block adoption.

## B2/runtime gates and cost limits

- Six regressions exist, **not passed**: repeat/independent projects+files/same-path same-timestamp change/malformed+missing/immutable casts/concurrent readers. Fix B1 before interpreting these failures as reuse defects.
- Use unchanged parser as all-nine-field oracle. Add multi-entry ordered comparisons (nested dependency order and JSON output), both layouts, old 0.3.1 fixture, alternate format/root flag, and mutation attempts for remaining outer maps/nested sets. Current assertions exercise only some ordered/frozen shapes.
- Actual selector integration needs a real `create` route test, not only constructor callback at test55; compare missing/ambiguous/empty-marker selection and malformed exception types against original extractor. Current `assertFails` cannot prove malformed type parity.
- Cold concurrent factory access, real metadata/cinterop tasks and configuration-cache store/reuse are targeted gaps, not invented architectural B1s. Kotlin `internal` extractor constructor changed; actual plugin build/ABI checks are needed, not an assumed stable public constructor contract.
- Cardinality is latest **successful content per absolute file per consuming Project**, not transformations or one global version. Failed/currently unreferenced paths stay cached for factory lifetime; old metadata may legitimately remain in earlier consumer decisions.
- Entries additionally retain full decoded content String + File/map/wrapper overhead. Hits still allocate/read text and serialize file reads per factory; first parse temporarily has original and frozen containers. Distinct files/projects cannot benefit; memory/time benefit is **unknown**.
- Minimal ablation must retain actual selection/read/error/cardinality ownership, comparing unchanged parse-per-call against reuse, without providers, ModelManager or visibility sharing. Neither 39k PSM objects nor the 817MB source-set dominator group is established recoverable saving.
- Official prerequisites: [wrapper 9.4.1][WR3] and [bootstrap 2.4.0-dev-8449][PR42]. Coordinator's stated `embeddableGradle813Jar --dry-run` is route configuration only, not patch compilation/test/artifact evidence. Report infrastructure failure; no fake JAR/classpath/reflection workaround is authorized.
- After reproducible real build/regressions: compare typed metadata, selected components, binaries/full model/source attachments/navigation and representative JVM/Node/Native/cinterop behavior; require actual complete unchanged 4GiB full GUI acceptance before adoption.
- Parent observations are not patch results: test-owner 140/63, 353 representative passes and 216/1344/5026 model counts coexist with r06/r07 host-memory-guard stops and no EXTERNAL_SUCCESS. Warm r07 `BUILD SUCCESSFUL in 10m 30s` is not GUI pass.
- Production remains clean `5b41be52`; no candidate is adopted. No fork/publish/production switch, target change or budget increase follows from review.

## Evidence URIs (SSH links refer to Xiaoxin)

[OF]: file:///tmp/kodex-kgp-metadata-source-overlay-20261009/KotlinProjectStructureMetadataExtractorFactory.kt#L1
[OT]: file:///tmp/kodex-kgp-metadata-source-overlay-20261009/KotlinProjectStructureMetadataReuseTest.kt#L1
[OE]: file:///tmp/kodex-kgp-metadata-source-overlay-20261009/MppDependencyProjectStructureMetadataExtractor.kt#L1
[OT21]: file:///tmp/kodex-kgp-metadata-source-overlay-20261009/KotlinProjectStructureMetadataReuseTest.kt#L21
[OT49]: file:///tmp/kodex-kgp-metadata-source-overlay-20261009/KotlinProjectStructureMetadataReuseTest.kt#L49
[OT111]: file:///tmp/kodex-kgp-metadata-source-overlay-20261009/KotlinProjectStructureMetadataReuseTest.kt#L111
[OT131]: file:///tmp/kodex-kgp-metadata-source-overlay-20261009/KotlinProjectStructureMetadataReuseTest.kt#L131
[OF42]: file:///tmp/kodex-kgp-metadata-source-overlay-20261009/KotlinProjectStructureMetadataExtractorFactory.kt#L42
[OF70]: file:///tmp/kodex-kgp-metadata-source-overlay-20261009/KotlinProjectStructureMetadataExtractorFactory.kt#L70
[OF99]: file:///tmp/kodex-kgp-metadata-source-overlay-20261009/KotlinProjectStructureMetadataExtractorFactory.kt#L99
[OE36]: file:///tmp/kodex-kgp-metadata-source-overlay-20261009/MppDependencyProjectStructureMetadataExtractor.kt#L36
[UF58]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/KotlinProjectStructureMetadataExtractorFactory.kt#L58
[PS72]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/KotlinProjectStructureMetadata.kt#L72
[PS99]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/KotlinProjectStructureMetadata.kt#L99
[PS201]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/KotlinProjectStructureMetadata.kt#L201
[PS223]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/KotlinProjectStructureMetadata.kt#L223
[PS329]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/KotlinProjectStructureMetadata.kt#L329
[GT130]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/GranularMetadataTransformation.kt#L130
[GT174]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/GranularMetadataTransformation.kt#L174
[GT397]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/GranularMetadataTransformation.kt#L397
[GT513]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/GranularMetadataTransformation.kt#L513
[SV135]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/SourceSetVisibilityProvider.kt#L135
[CA42]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/CompositeMetadataArtifactImpl.kt#L42
[CA80]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/CompositeMetadataArtifactImpl.kt#L80
[CA106]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/CompositeMetadataArtifactImpl.kt#L106
[CA174]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/CompositeMetadataArtifactImpl.kt#L174
[CI78]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/targets/native/internal/CInteropMetadataDependencyClasspath.kt#L78
[GP94]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/GenerateProjectStructureMetadata.kt#L94
[SP74]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/utils/storedProperty.kt#L74
[EP11]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/utils/extraProperties.kt#L11
[DS28]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/sources/DefaultKotlinSourceSet.kt#L28
[ST17]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/sources/KotlinSourceSetMetadataTransformation.kt#L17
[MT71]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/MetadataDependencyTransformationTask.kt#L71
[CT119]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/targets/native/internal/CInteropMetadataDependencyTransformationTask.kt#L119
[WR3]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/gradle/wrapper/gradle-wrapper.properties#L3
[PR42]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/gradle.properties#L42
