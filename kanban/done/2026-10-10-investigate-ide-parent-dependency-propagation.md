# Task Tree

- `Read parent scope and existing evidence`() // complete
- `Trace pinned KGP and installed IDEA consumers`() // complete
- `Check inheritance, exports and counterexamples`() // complete; no safe KGP-only deletion candidate
- `Return bounded findings to coordinator`() // complete; no implementation or runtime validation

# Details

## Verdict and boundary

- **NO SAFE CURRENT CANDIDATE:** matching a parent's entire encoded binary record is necessary, not sufficient to delete the child's library dependency.
- **Scoped investigation complete, benefit unknown:** a public, no-op KGP dependency-transformer diagnostic is available; this is not a deletion repair or an executed gate.
- Parent: [full-model OOM repair](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L1).
- Related: [private IDEA investigation](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-10-investigate-private-ide-receive-retention.md#L146).
- Preserve clean production `5b41be52`, KGP2.4.0 / Gradle9.5.1, all targets, source hierarchy, cinterop, API, tests and navigation; retain existing resource budgets.
- Only this new outer task is owned/edited. No Git, Gradle, tests, IDE/MAT launches, process inspection/control, private heaps/raw logs/secrets, network writes or local Python execution.
- Sources/JARs were read statically over SSH; official JetBrains sources were read online. No temporary files, staged credentials or persistent resources were created.

## Early exact call chain

- Producer FQCN: `org.jetbrains.kotlin.gradle.plugin.ide.dependencyResolvers.IdeDependsOnDependencyResolver`.
- `resolve(sourceSet)` emits `IdeaKotlinSourceDependency(Type.DependsOn, coordinates)` for **every member of `internal.dependsOnClosure`**, not just immediate parents.
- Primary: [pinned resolver16–26](https://github.com/JetBrains/kotlin/blob/add726ca8c82922b6ab4cb2a27ae738d6a780817/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeDependsOnDependencyResolver.kt#L16-L26).
- Actual local source: [resolver18](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeDependsOnDependencyResolver.kt#L18) on Xiaoxin.
- `org.jetbrains.kotlin.gradle.plugin.ide.IdeMultiplatformImportImpl`: phase resolvers → additional artifacts → transformers → effects; `resolveDependenciesSerialized` then serializes each result separately.
- Actual [pipeline170–176](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImportImpl.kt#L170-L176), [serialization42–57](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImportImpl.kt#L42-L57).
- IDE tooling: `org.jetbrains.kotlin.idea.gradleTooling.builders.IdeaKotlinDependencyContainerBuilderKt.buildIdeaKotlinDependenciesContainer` calls serialized resolution for each source set.
- Transport: `org.jetbrains.kotlin.idea.gradleTooling.IdeaKotlinSerializedDependenciesContainer` → `IdeaKotlinDependenciesContainerSurrogate` → `IdeaKotlinDeserializedDependenciesContainer`; decoding is per source set, not a parent-inheritance table.
- Primary: [builder](https://github.com/JetBrains/intellij-community/blob/idea/262.10968.63/plugins/kotlin/gradle/gradle-tooling/impl/src/org/jetbrains/kotlin/idea/gradleTooling/builders/IdeaKotlinDependencyContainerBuilder.kt), [container](https://github.com/JetBrains/intellij-community/blob/idea/262.10968.63/plugins/kotlin/gradle/gradle-tooling/impl/src/org/jetbrains/kotlin/idea/gradleTooling/IdeaKotlinDependenciesContainer.kt).
- `org.jetbrains.kotlin.idea.gradleJava.configuration.KotlinGradleProjectResolverExtension.populateModuleDependencies` **delegates when MPP exists**; its private legacy `addDependency` is not this model's MPP implementation.
- Primary: [MPP delegation282–290](https://github.com/JetBrains/intellij-community/blob/idea/262.10968.63/plugins/kotlin/gradle/gradle-java/src/org/jetbrains/kotlin/idea/gradleJava/configuration/KotlinGradleProjectResolverExtension.kt#L282-L290).
- Real consumer FQCN: `org.jetbrains.kotlin.idea.gradleJava.configuration.KotlinMppGradleProjectResolver`.
- `populateModuleDependencies` → `org.jetbrains.kotlin.idea.gradleJava.configuration.mpp.PopulateModuleDependenciesKt.populateModuleDependenciesWithDependenciesContainer`.
- That path substitutes project artifacts, dispatches through `AddIdeaKotlinDependencyKt.addDependency`, and assigns each created node the incoming dependency index as order.
- Primary: [MPP resolver105–114](https://github.com/JetBrains/intellij-community/blob/idea/262.10968.63/plugins/kotlin/gradle/gradle-java/src/org/jetbrains/kotlin/idea/gradleJava/configuration/KotlinMppGradleProjectResolver.kt#L105-L114), [population24–69](https://github.com/JetBrains/intellij-community/blob/idea/262.10968.63/plugins/kotlin/gradle/gradle-java/src/org/jetbrains/kotlin/idea/gradleJava/configuration/mpp/populateModuleDependencies.kt#L24-L69).

## Installed build, exports and facet

- Installed `IC2026.2.3`, build `262.10968.63`, confirmed from [product-info1–5](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/idea-oss/product-info.json#L1-L5) on Xiaoxin.
- Static `javap -c -p`, capped128MiB, checks agree with matching tag at the cited critical operations; this is not whole-JAR/source byte equivalence.
- Installed classes reside in [kotlin-plugin.jar](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/idea-oss/plugins/Kotlin/lib/kotlin-plugin.jar); producer tooling in sibling `kotlin-gradle-tooling.jar`.
- `org.jetbrains.kotlin.idea.gradleJava.configuration.mpp.AddIdeaKotlinSourceDependencyKt.addDependency` creates `ModuleDependencyData`, sets `COMPILE`, and records `DependsOn` in `KotlinSourceSetInfo.dependsOn`; it does not set export=true.
- `org.jetbrains.kotlin.idea.gradleJava.configuration.mpp.AddIdeaKotlinBinaryDependencyKt.addDependency` creates `LibraryDependencyData`, using MODULE level unless `isIdeaProjectLevel`; project-library sharing still requires a dependency edge per consumer.
- Primary: [source13–28](https://github.com/JetBrains/intellij-community/blob/idea/262.10968.63/plugins/kotlin/gradle/gradle-java/src/org/jetbrains/kotlin/idea/gradleJava/configuration/mpp/addIdeaKotlinSourceDependency.kt#L13-L28), [binary29–48](https://github.com/JetBrains/intellij-community/blob/idea/262.10968.63/plugins/kotlin/gradle/gradle-java/src/org/jetbrains/kotlin/idea/gradleJava/configuration/mpp/addIdeaKotlinBinaryDependency.kt#L29-L48).
- `com.intellij.openapi.externalSystem.model.project.AbstractDependencyData` defaults scope=COMPILE, exported=false; installed constructors confirm this for both new edge types.
- Primary: [defaults16–25](https://github.com/JetBrains/intellij-community/blob/idea/262.10968.63/platform/external-system-api/src/com/intellij/openapi/externalSystem/model/project/AbstractDependencyData.java#L16-L25).
- These are the installed default-path semantics, not a new live per-edge census; later extension/custom-import changes require Gate2 verification.
- `ModuleDependencyDataService` and `LibraryDependencyDataService` copy scope/exported into actual order entries; both installed implementations were checked.
- Primary: [module120–123](https://github.com/JetBrains/intellij-community/blob/idea/262.10968.63/platform/external-system-impl/src/com/intellij/openapi/externalSystem/service/project/manage/ModuleDependencyDataService.java#L120-L123), [library221–222](https://github.com/JetBrains/intellij-community/blob/idea/262.10968.63/platform/external-system-impl/src/com/intellij/openapi/externalSystem/service/project/manage/LibraryDependencyDataService.java#L221-L222).
- `org.jetbrains.kotlin.idea.gradleJava.configuration.KotlinSourceSetDataService` passes `dependsOnList` and friend visibility into the facet. For test modules it changes COMPILE order entries to TEST; it does not make them exported.
- Primary: [test116–122](https://github.com/JetBrains/intellij-community/blob/idea/262.10968.63/plugins/kotlin/gradle/gradle-java/src/org/jetbrains/kotlin/idea/gradleJava/configuration/KotlinSourceSetDataService.kt#L116-L122), [facet247–248](https://github.com/JetBrains/intellij-community/blob/idea/262.10968.63/plugins/kotlin/gradle/gradle-java/src/org/jetbrains/kotlin/idea/gradleJava/configuration/KotlinSourceSetDataService.kt#L247-L248).
- `org.jetbrains.kotlin.idea.base.fir.projectStructure.modules.source.KaSourceModuleDependenciesProvider` computes regular libraries from dependency entries and exported dependencies; facet refinement is a separate `getDirectDependsOnDependencies` result.
- `org.jetbrains.kotlin.idea.base.fir.projectStructure.KotlinExportedDependenciesCollector` includes only exported transitive library/module edges and ignores dependent SDK entries. Installed collector bytecode confirms the export checks.
- Primary: [provider52–65](https://github.com/JetBrains/intellij-community/blob/idea/262.10968.63/plugins/kotlin/base/fir/project-structure/src/org/jetbrains/kotlin/idea/base/fir/projectStructure/modules/source/KaSourceModuleDependenciesProvider.kt#L52-L65), [collector84–102](https://github.com/JetBrains/intellij-community/blob/idea/262.10968.63/plugins/kotlin/base/fir/project-structure/src/org/jetbrains/kotlin/idea/base/fir/projectStructure/KotlinExportedDependenciesCollector.kt#L84-L102).

## Strong counterexamples — static constructions, not executed tests

- **Exact-record failure:** C dependsOn P; C→L and P→L carry identical whole encoded binaries. Delete C→L while retaining P→L exported=false: L disappears from C's direct regular-library result/exported-root path. A facet refinement edge does not restore that original graph.
- Important nuance: **C→P exported=false alone does not block C from using P's exported libraries**. P→L must export; deeper traversal also needs exported intermediate edges. C→P controls re-export to consumers of C.
- KGP emits direct edges to the full ancestry closure; do not falsely attribute a missing ancestor to an intermediate non-exported edge when C already has a direct edge to that ancestor.
- **Exact-record priority failure after naive export:** C order `[P, B, L]`, P contains L. Initially P→L is not exported, so B precedes L. Exporting P→L to enable deletion exposes L at P's earlier position, yielding `[P, L, B]`; equal L records do not encode this surrounding order. B/L can provide the same symbol.
- The provider iterates the child's entries and collects exports at each module position; the importer explicitly preserves KGP's record index. These sources establish graph/order changes, not a measured navigation failure.
- **Native equal-file/GAV trap:** a shared `linuxMain` POSIX library and `linuxX64Main` leaf POSIX library need not be the same IDE dependency even if content hashes/GAV match.
- KGP encodes `coordinates.sourceSetName=target.identityString`, selected classpath and `KlibExtra`; shared resolution additionally marks `isCommonized`. Leaf/shared SDK and cinterop records must not be collapsed by basename/content.
- Primary: [Native coordinates39–64](https://github.com/JetBrains/kotlin/blob/add726ca8c82922b6ab4cb2a27ae738d6a780817/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/resolveNativeDistributionDependency.kt#L39-L64), [shared marker66–78](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeKonanDistributionLibsService.kt#L66-L78).
- This Native trap fails the proposed **whole-record** equality gate; the export and priority counterexamples above survive that equality gate.
- Native analysis also filters by the consuming target platform, with explicit stdlib/shared-Native exceptions; a parent's SDK/platform environment is not interchangeable with the child's.
- Primary: [HMPP filter](https://github.com/JetBrains/intellij-community/blob/idea/262.10968.63/plugins/kotlin/base/project-structure/src/org/jetbrains/kotlin/idea/base/projectStructure/kmp/SourceModuleDependenciesFilter.kt).
- Standard recursive exported-root enumeration skips non-exported entries beyond the first level and does not inherit a dependency's JDK as the child's SDK.
- Primary: [root enumeration](https://github.com/JetBrains/intellij-community/blob/idea/262.10968.63/platform/projectModel-impl/src/com/intellij/openapi/roots/impl/OrderEnumeratorBase.java).

## Existing filtering is not this repair

- No `redundantSourceDeps` parent-subtraction filter was found in the pinned KGP IDE resolver/transformer route; the actual built-in filter below is narrower.
- Its registered built-in transformer is `org.jetbrains.kotlin.gradle.plugin.ide.dependencyTransformers.IdePlatformStdlibCommonDependencyFilter`: selected platform constraints remove common stdlib/test binaries, not arbitrary ancestor binaries.
- Primary: [registration150–155](https://github.com/JetBrains/kotlin/blob/add726ca8c82922b6ab4cb2a27ae738d6a780817/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImportFactory.kt#L150-L155), [filter17–22](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyTransformers/IdePlatformStdlibCommonDependencyFilter.kt#L17-L22).
- Resolver `.toSet()` removes equal results within a source-set result. `IdeVisibleMultiplatformSourceDependencyResolver` excludes self-project loops, but maps all visible source-set names.
- `IdeTransformedMetadataDependencyResolver` also uses `allVisibleSourceSetNames`, including binary metadata already visible to an ancestor; no parent binary subtraction there.
- Actual [visible resolver31–58](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeVisibleMultiplatformSourceDependencyResolver.kt#L31-L58), [transformed resolver34–60](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeTransformedMetadataDependencyResolver.kt#L34-L60).
- Legacy IDE platform-propagation/source-visibility algorithms are the no-container branch, not the modern serialized-container branch. Hierarchy redundant-dependsOn tracking is not binary dependency inheritance.

## Concrete alternative and minimal gates

- Bounded alternative: factor only an exactly matching resolved child binary through its **original ancestor module**, while retaining all module/facet/source edges and nonmatching libraries. Current consumer lacks a proven semantics-preserving route.
- An IntelliJ Kotlin importer/analysis change may be a prerequisite, outside the currently authorized KGP experiment. No automatic version switch, fork or plugin patch is recommended/authorized.
- Blanket export changes expose dependencies to other consumers and change precedence; rebuilding child library entries restores semantics but defeats the proposed entity-count reduction.
- Public diagnostic entry: `org.jetbrains.kotlin.gradle.plugin.ide.IdeMultiplatformImport.instance(project).registerDependencyTransformer`, with `IdeDependencyTransformer` and `PostDependencyTransformationPhase`; requires `ExternalKotlinTargetApi` opt-in.
- Primary: [public method91–100](https://github.com/JetBrains/kotlin/blob/add726ca8c82922b6ab4cb2a27ae738d6a780817/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImport.kt#L91-L100), [phases244–267](https://github.com/JetBrains/kotlin/blob/add726ca8c82922b6ab4cb2a27ae738d6a780817/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImport.kt#L244-L267).
- Coordinator-only diagnostic would return the original dependency set unchanged; attest registration order, capture each source set once after attachments, compare parent/child records after collection, emit only numeric counts/lengths.
- Do not recursively resolve parents inside the transformer, alias mutable ByteArrays, export payloads/fingerprints, or reflect into workspace/entity storage. No diagnostic was implemented/run here.
- Gate1: project/build-scoped original ancestry; compare full type, coordinates/attributes/capabilities, ordered classpath, all encoded extras and source/doc attachments. No GAV/file-hash-only match.
- Gate2: prove effective export/scope/productionOnTest behavior for every consumer; include Main/Test, friend, diamonds and direct full-closure edges. No new visibility or loss.
- Gate3: compare ordered effective class/source/doc roots and symbol priority, not unordered root sets or direct-entry counts. Preserve Native target/commonizer identity, stdlib, SDK and original cinterop variants/tasks.
- Gate4: preserve every module/target/source-set identity and facet; verify declarations, expect/actual, completion and source navigation across common/shared/leaf Native, JVM/JS/Wasm and tests.
- Gate5: coordinator-serialized regressions and fresh-private-IDE initial/repeated imports at original budgets/reserve; measure aligned retained/peak receiver memory and actual entity reduction. Stop on any semantic mismatch.

## Benefit and handoff

- User-supplied full model:5,245 modules /673,278 ORDER entries; active heap338,876 `ModuleDependencyData` +323,916 `LibraryDependencyData`; zero exact duplicate ORDER within a module.
- Parent census PASSED:152 source sets /20,995 records /13,684,964 raw bytes versus1,346,524 globally unique bytes. This does not establish parent-local matches, safe byte-array identity sharing, wire savings or entity savings.
- Primary task evidence: [census164–170](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-10-investigate-private-ide-receive-retention.md#L164-L170).
- R34 active Java used2,681,422.4KiB; NMT Other201,589KiB. Java heap dominates observed growth; a large Native leak is not proved. Warm four-import runs pass twice; fresh-state repetition still fails acceptance.
- Binary factoring would not remove the338,876 module dependencies or guarantee importer/analysis caches shrink. Actual eligible population and memory benefit are **unknown**.
- Independent static investigation is complete. Parent main/other asynchronous reads remain independent; this record does not advance their state. All heavy diagnostics/implementation/gates remain serialized by the coordinator.
