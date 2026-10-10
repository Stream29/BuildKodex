# STATIC READY — producer opportunity identified; runtime repair unproved

# Task Tree

- `Read sanitized current heap groups and exact KGP parser sources`() // complete
- `Trace repeated metadata ownership and immutable sharing conditions`() // complete
- `Deliver bounded producer candidate and private evidence requests`() // complete

# Details

## Authority and evidence

- Parent: [full import repair](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L157).
- Product `5b41be52`, KGP 2.4.0 / Gradle 9.5.1, all 203 projects, targets/navigation/cinterop and 4GiB unchanged.
- Coordinator's r05: 300 graph comparisons and 241 tests pass; genuine GUI OOM at ~678.570s. Neither gate proves memory repair.
- Only this report changed. No Git, credentials, raw heap/HTML strings, MAT/query, runtime/build/test/IDE, process/device control or network writes.
- Read exact cached sources, sanitized summary over read-only SSH, and official source GETs. No temporary files or owned heavy resources.
- Remote Xiaoxin evidence: [sanitized summary:39–68](file:///home/stream/ACodeSpace/demo/kodex-import-sources-repair-20261008/current-heap-class-summary.txt#L39).
- Source-set dominator group: 5,026 / 817,618,472 retained bytes; 39,360 PSM + 39,360 ChooseVisibleSourceSets.
- Group also contains 5,461,865 LinkedHashMap entries and 4,880,987 Strings: **not all proved to belong to PSM**, nor to duplicate content.
- Whole-heap PSM count is 39,560; group count is 39,360. Neither is a distinct-library count.
- [Thread group:70–96](file:///home/stream/ACodeSpace/demo/kodex-import-sources-repair-20261008/current-heap-class-summary.txt#L70): 849,507,520 bytes includes one 385,903,112-byte StreamByteBuffer and live tooling/model state.
- Configuration diagnostic group 712,343,928 bytes + configuration-container group 237,226,056 bytes remain separate candidates.
- Smaller diagnostic retention than old KGP 2.4.20 heap is **different toolchain/phase**, not controlled benefit; that candidate GUI also failed at 4GiB.

## Exact KGP 2.4.0 sources and call chain

- Source archive [KGP sources](file:///home/stream/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-gradle-plugin/2.4.0/8cc8a31f5f34bbace6166c7ad0187724c345932f/kotlin-gradle-plugin-2.4.0-sources.jar).
- Entries below use `org/jetbrains/kotlin/gradle/` prefix. Four mpp parser/factory/transformation entries are byte-identical in cached `2.4.0-gradle813-sources.jar`.
- `plugin/sources/KotlinSourceSetMetadataTransformation.kt:17–33`: source-set `extrasStoredProperty` owns metadataTransformation; parents' transformations supply visibility.
- `plugin/mpp/GranularMetadataTransformation.kt:169–187,199–205`: per-source-set Params; lazy resolutions and lazy visibility map retain their computed results.
- `doTransform:208–268` visits selected component once **per transformation**, not once across consumers/source sets.
- `processDependency:294–309 → processPSMDependency:397 → extractProjectStructureMetadata:506–531`.
- [Factory:15–61](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/KotlinProjectStructureMetadataExtractorFactory.kt#L15-L61): selected component → resolved PSM artifacts → kotlin-psm-metadata usage → non-empty single file → new file extractor.
- Despite `getOrCreate` naming, factory getter **constructs a new factory**; it is not shared per project.
- [Extractor:36–63](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/MppDependencyProjectStructureMetadataExtractor.kt#L36-L63): file `readText → parseKotlinSourceSetMetadataFromJson`, no memoization.
- Other JSON call site is JAR extractor: entry stream `reader.readText`; XML fallback exists. Current factory returns the **file** extractor, not that JAR path.
- [Parser:274–364](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/KotlinProjectStructureMetadata.kt#L274-L364): Gson tree → new maps/sets/ModuleDependencyIdentifiers → typed PSM.
- Thus the **same unchanged PSM file**, if selected by two transformations, is parsed twice and creates separately owned typed structures. Source proves this conditional mechanism, not current per-object duplication.
- [Transformation:402–447](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/GranularMetadataTransformation.kt#L402-L447): PSM → platform visibility → requested transitive dependencies → parent subtraction → ChooseVisibleSourceSets.
- `ChooseVisibleSourceSets:82–98` owns PSM, selected ResolvedComponentResult, visible sets/transitive edges and metadataProvider.
- External `CompositeMetadataArtifactImpl:19–25` also references that PSM; provider owns version, primary artifact and host-specific files.
- Static owner hypothesis: DefaultKotlinSourceSet.extras → StoredPropertyStorage.values → transformation → metadataDependencyResolutions$delegate → lazy value → ChooseVisibleSourceSets.projectStructureMetadata.
- `utils/storedProperty.kt:74–123`: weak **delegate keys**, strong values; top-level delegate remains live. Not a weak-value cache or proof of a daemon leak.
- Coordinator must confirm actual owner fields; no heap owner path was queried here.

## Typed metadata: sharing contract, not “val means immutable”

- `KotlinProjectStructureMetadata.kt:99–126`: val Map/Set fields, but parser uses mutableMapOf/mutableSetOf/toSet. **Not deeply immutable by construction**.
- Audited KGP source consumers read fields; no in-place mutation of parsed maps/sets found in inspected archive.
- [GenerateProjectStructureMetadata:90–110](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/GenerateProjectStructureMetadata.kt#L90-L110) uses `copy(sourceSetModuleDependencies = ...)`, not mutation of the original object.
- Do not replace parsed file metadata with the extension's producer object: producer initially has empty module dependencies; generated file contains resolved dependency aliases.
- `SourceSetVisibilityProvider.kt:64–212,245–266` derives union/intersection, host-file mapping and topological ordering without modifying PSM.
- Relative cinterop directories: parser `339–340` stores unchanged JSON values; `CompositeMetadataArtifactImpl.kt:80–93,174–182` reads them, normalizes a local ZIP path, lists/copies entries. **No rewrite to an absolute consumer path**.
- Binary output relative filenames `117–127,160–167` separately incorporate module identity/version/source set/checksum; they are not PSM fields.
- Typed PSM includes formatVersion, publication-root flag, variant/source-set topology, dependencies, layout, host-specific sets and cinterop directories.
- ModuleDependencyIdentifier stores group/module, not selected version or project/build identity. Group/module alone is an unsafe sharing key.
- Project dependency aliases come from GenerateProjectStructureMetadata's coordinates data; selected project/build IDs and consumer resolution stay outside PSM.
- Same typed content can plausibly be shared; require immutable ordered snapshots and exact equality, preserving iteration/topological tie order, paths, layouts and format.
- Keep every ChooseVisibleSourceSets/provider graph-local: platform compilations, ancestry, selected components, host artifacts/version and transitive edges can differ even with equal PSM.
- Public typed class/collections are exposed; inspected KGP reads do not prove all third-party users never cast/mutate them.
- Preserve private cinterop header ownership/commonization. No NativeOnly profile, target deletion or disabled commonization.

## Smallest bounded correction / prerequisites

- **No supported parser/factory replacement API found**: factory and transformation are internal; sealed factory interface and concrete cast prevent public replacement.
- IDE dependency-resolver extensions run at another boundary; adding/replacing one does not release already-owned transformations. No generic ModelManager/cache/resolver/SerializedViewAdapter or reflection mutation.
- Conditional KGP-internal experiment: make existing Project factory genuinely project-owned using its existing `projectStoredProperty` mechanism; reuse only immutable parsed snapshots of the same selected PSM file.
- First scope is **one consuming project**, across its source-set transformations. Do not promise dedup across all 203 projects or introduce cross-build/static state.
- Factory/extractor memo must preserve selected-artifact checks/errors; key by file snapshot identity/content, not GAV/path/mtime alone. Generated files can change.
- Require stable generation/read boundary, exact content validation, ordered deep immutability and safe concurrent publication. A `lazy` on each freshly created extractor alone does not solve repetition.
- Cache retains unique snapshots too: no saving estimate until multiplicity/retained-size proof; never clear graph results while parents/resolvers still need them.
- **Not an authorized product edit or supported init-script experiment.** This entails a fourth maintenance line: a patched KGP artifact/publication, beyond current three maintained framework lines.
- Prerequisites: pinned Kotlin monorepo KGP build/dependency toolchain, correct Gradle variant/plugin publication and classloader selection, isolated distribution/rollback, then coordinator-owned equivalent model/source/function/GUI gates.
- It need not rewrite/fork compiler or Native runtime implementations, but publishing a patched KGP is not a small product build-script change; build cost is unmeasured here.
- Without separate authorization for that prerequisite, hand off an upstream producer issue with bounded proof, not a pretend supported production fix.
- Official [PR 6887](https://github.com/JetBrains/kotlin/pull/6887) is verified merged 2026-08-07. It caches platform dependency lookup/visibility computations; **not verified PSM parse interning**.
- Official [2.4.20 extractor:36–41](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/MppDependencyProjectStructureMetadataExtractor.kt#L36-L41) and factory still show fresh file parsing. No unverified upgrade recommendation.

## Model transport / serializer: what is actually proved

- KGP `plugin/ide/IdeMultiplatformImportImpl.kt:42–59`: resolveDependenciesSerialized → resolveDependencies → map each dependency to protobuf ByteArray.
- Transformed resolver emits IdeaKotlinResolvedBinaryDependency with classpath/coordinates/extras, not a raw PSM field. Producer sharing need not dedup wire models.
- Exact official Gradle 9.5.1 installed source on Xiaoxin: [PayloadSerializer:44–78](file:///home/stream/ACodeSpace/demo/kodex-import-sources-repair-20261008/installed-source-seed/wrapper/dists/gradle-9.5.1-bin/iq79hdu3mqx29lgffhp8bfmx/gradle-9.5.1/src/serialization/org/gradle/tooling/internal/provider/serialization/PayloadSerializer.java#L44).
- [Official source:44–62](https://github.com/gradle/gradle/blob/v9.5.1/platforms/core-runtime/serialization/src/main/java/org/gradle/tooling/internal/provider/serialization/PayloadSerializer.java#L44-L62): ObjectOutputStream writes the whole supplied payload to StreamByteBuffer, then returns a chunk list.
- Exact call sites: `tooling-api-builders/.../AbstractClientProvidedBuildActionRunner.java:89–98` serializes action result; `DefaultBuildController.java:164–169` serializes dispatched streamed value. Neither automatically establishes which one owns this buffer.
- `build-state/.../BuildSessionLifecycleBuildActionExecutor.java:113–119` can serialize client result **or failure**. Check frame/payload classes before labeling the buffer a model or attributing the OOM allocation site.
- [StreamByteBuffer:121–134,268–297,372–381](https://github.com/gradle/gradle/blob/v9.5.1/platforms/core-runtime/io/src/main/java/org/gradle/internal/io/StreamByteBuffer.java#L121-L134): list extraction reuses full unread chunk arrays; partial chunk copied. `readAsByteArray` allocates a full copy, but PayloadSerializer does **not** use it.
- `SerializedPayloadSerializer.java:31–36` forwards each chunk with encoder.writeBinary; no second complete model-byte buffer allocated at that layer.
- Object graph plus encoded bytes coexist during serialization; ObjectOutputStream identity sharing is within that stream, not content interning across distinct objects/calls.
- Official IntelliJ [GradleDaemonModelHolder:105–120](https://github.com/JetBrains/intellij-community/blob/master/plugins/gradle/tooling-extension-impl/src/com/intellij/gradle/toolingExtension/impl/modelAction/GradleDaemonModelHolder.java#L105-L120) converts/queues models then returns phase state.
- Its [ToolingSerializer:68–95](https://github.com/JetBrains/intellij-community/blob/master/plugins/gradle/tooling-extension-impl/src/com/intellij/gradle/toolingExtension/impl/modelSerialization/ToolingSerializer.java#L68-L95) selects serialization service; converter can fall back to original object.
- [DefaultSerializationService:21–31](https://github.com/JetBrains/intellij-community/blob/master/plugins/gradle/tooling-extension-impl/src/com/intellij/gradle/toolingExtension/impl/modelSerialization/DefaultSerializationService.java#L21-L31) uses ByteArrayOutputStream then toByteArray: a copy there, **if that service is selected**.
- This IntelliJ master chain is contextual primary code, not matched installed-IDE bytecode. Extra serialization of its byte arrays by Gradle is possible; universal “every actual model is double-buffered” is not established.
- Names `ModelTransport` / `SerializerGradleProtocol` not located in examined KGP/Gradle source; require exact FQCN/installed source mapping rather than invent that call chain.
- 42,727 task proxies and live holder do not identify 385MB serialized content; task-model dedup alone is not a justified fix.
- Diagnostic ownership is plausible independently: Gradle 9.5.1 `VariantSelectionByAttributesException.java:27–34` retains structured failure; [NoCompatibleVariantsFailure:34–43](https://github.com/gradle/gradle/blob/v9.5.1/platforms/software/dependency-management/src/main/java/org/gradle/internal/component/resolution/failure/type/NoCompatibleVariantsFailure.java#L34-L43) retains AssessedCandidate list. Do not suppress real missing-variant failures.

## Coordinator-only private evidence requests — NOT executed

- PSM owner query: anonymous source-set/transformation owner groups; field-only paths to ChooseVisibleSourceSets → PSM and provider → PSM; report shared vs distinct identities/counts.
- Content equality: privately compare **all typed fields**, including ordered nested collections, aliases, format/root/host/layout/cinterop directories. Export only anonymous equality-group cardinalities and numeric size summaries, never strings/properties/paths/digests of private values.
- Map PSM equality groups to selected file snapshots in producer evidence; distinguish same-file reparsing from equal-but-different files, versions/projects/builds. A heap PSM lacks its source File, so heap-only inference is insufficient.
- Quantify within-project reuse first; report exclusive duplicate subgraph bytes excluding already-shared strings/collections. Do not sum overlapping retained sizes or claim all 817MB reclaimable.
- Buffer query: root frame method/class and reference-field path; chunks/currentWriteChunk/currentReadChunk array lengths and numeric cursors; incoming payload/stream/holder owner **classes**, no object contents.
- Separate action-result vs streamed-result vs other allocation; count simultaneously live original models, model ByteArrays, payload chunks and OOS handles, with non-overlapping sizes.
- Diagnostic query: configuration → resolver result/unresolved edge → exception.failure → candidates/attributes paths; anonymous configuration-role/failure-subtype counts, not exception messages or requested private coordinates.
- These are existing-heap queries for coordinator's locked MAT scope, not a request to launch new GUI or allocate another heap here.
- Runtime acceptance remains parent's same-model/binary/source/navigation/functional and full 4GiB genuine GUI gate. **STATIC READY is not runtime fixed, release ready, or production adoption.**
