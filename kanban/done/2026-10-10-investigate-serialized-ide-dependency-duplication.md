# Task Tree

- `Load repair scope, SPEC boundaries and private IDEA investigation`() // complete
- `Trace pinned KGP producer and installed IDE transport adapter`() // complete; static reads only
- `Record mutation counterexamples and coordinator measurement boundary`() // complete; not implemented or executed

# Details

- **Scoped investigation complete — no safe KGP-only serialized-array sharing fix established.** No implementation or memory repair is accepted.
- Parents: [full OOM repair](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L1), [private IDEA investigation](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-10-investigate-private-ide-receive-retention.md#L1).
- Loaded AGENTS/workflow skills, Gradle, better-code-abstract and [SPEC/Impl contract boundary](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/spec-impl-module-boundaries.md#L5); preserve ownership, behavior and wire shape.
- Fixed product `5b41be52`, KGP `2.4.0`, Gradle `9.5.1`; official source commit `add726ca8c82922b6ab4cb2a27ae738d6a780817`.
- Xiaoxin source root: `file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/`; only source/text summaries and installed/cached JAR static reads over SSH.
- No builds/tests, IDE/MAT, sampling, private heaps/serialized payloads/credentials, process control, Git, remote writes or local Python. Only this outer task file written; no temporary files created.
- Coordinator owns all runtime/heavy-lock work. Isolated official KGP source/build authorization is not IDE-adapter patch, fork/publication or production authorization.

## Exact experiment coordinates and contracts

- KGP producer FQCN: `org.jetbrains.kotlin.gradle.plugin.ide.IdeMultiplatformImportImpl`; methods `resolveDependenciesSerialized(String): List<ByteArray>` and `serialize(Iterable<IdeaKotlinDependency>): List<ByteArray>`.
- Public interface FQCN: `org.jetbrains.kotlin.gradle.plugin.ide.IdeMultiplatformImport`; [local declarations:47–55 and project ownership:347–355](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImport.kt#L47), [official contract](https://github.com/JetBrains/kotlin/blob/add726ca8c82922b6ab4cb2a27ae738d6a780817/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImport.kt).
- Ownership is the **original Project's** `kotlinIdeMultiplatformImport` via `projectStoredProperty`, not one cache across all 203 KMP owners or daemon/import lifetimes. Project ownership alone does not make returned arrays immutable.
- KDoc does not promise immutable arrays or explicitly describe freshness. Existing implementation's fresh-array/mutation isolation is nevertheless observable and explicitly required by this investigation.
- Producer [local:32–65,150–151,170–176,250–254](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImportImpl.kt#L32), [official implementation](https://github.com/JetBrains/kotlin/blob/add726ca8c82922b6ab4cb2a27ae738d6a780817/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImportImpl.kt#L55): resolve all phases/artifacts/transformers/effects, create a context per serialization call, then map each dependency to protobuf bytes.
- Context FQCN: `org.jetbrains.kotlin.gradle.idea.serialize.IdeaKotlinSerializationContext`; it supplies logger and extras serializers, not an immutable dependency snapshot.
- [Context composition:14–47](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeSerialization.kt#L14) selects serializers and reports conflicts; registration lists and serializer implementations can change or be stateful. Do not memoize by dependency identity/GAV while bypassing serialization or logger effects.
- Proto function `org.jetbrains.kotlin.gradle.idea.proto.tcs.IdeaKotlinDependency.toByteArray(context)` constructs the full dependency proto, then calls `toByteArray`; [local:16–43](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin-idea-proto/src/main/kotlin/org/jetbrains/kotlin/gradle/idea/proto/tcs/IdeaKotlinDependency.kt#L16), [official source](https://github.com/JetBrains/kotlin/blob/add726ca8c82922b6ab4cb2a27ae738d6a780817/libraries/tools/kotlin-gradle-plugin-idea-proto/src/main/kotlin/org/jetbrains/kotlin/gradle/idea/proto/tcs/IdeaKotlinDependency.kt#L41).
- Installed shaded `org.jetbrains.kotlin.gradle.idea.proto.com.google.protobuf.AbstractMessageLite.toByteArray()` bytecode uses `newarray byte` at offset 4; fresh arrays, including empty ones, not a content pool.
- Full bytes include subtype, coordinates, binary type/classpath and serialized extras. [Binary coordinates:15–23](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin-idea-proto/src/main/kotlin/org/jetbrains/kotlin/gradle/idea/proto/tcs/IdeaKotlinBinaryCoordinates.kt#L15) include sourceSetName, capabilities and attributes; no GAV/path-only key.
- [Extras:36–47](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin-idea-proto/src/main/kotlin/org/jetbrains/kotlin/gradle/idea/proto/extras.kt#L36), [official serialization](https://github.com/JetBrains/kotlin/blob/add726ca8c82922b6ab4cb2a27ae738d6a780817/libraries/tools/kotlin-gradle-plugin-idea-proto/src/main/kotlin/org/jetbrains/kotlin/gradle/idea/proto/extras.kt#L36): serialize each registered value and copy into protobuf `ByteString`. That immutable internal value does not replace public fresh `ByteArray` output.

## Existing sharing versus duplicate content

- **Existing sharing found:** `org.jetbrains.kotlin.gradle.plugin.ide.dependencyResolvers.IdeKonanDistributionLibsService` caches `Set<IdeaKotlinDependency>` per `CommonizerTarget`, shared across projects/source sets.
- [Local:27–37,43–79,83–87](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeKonanDistributionLibsService.kt#L27), [official service](https://github.com/JetBrains/kotlin/blob/add726ca8c82922b6ab4cb2a27ae738d6a780817/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeKonanDistributionLibsService.kt): cache stores dependency objects, **not serialized ByteArrays**; values carry mutable extras, so this is not the requested immutable serialization boundary.
- Reusing a dependency object still produces fresh protobuf arrays per occurrence. Exact repeated bytes are plausible, especially Native distribution dependencies, but unchanged object identity does not prove unchanged extras/context or total duplicate byte volume.
- **Not proved large:** earlier 203 complete Kotlin MPP model arrays total `367,942,312` bytes; Gradle transport buffer `385,903,112` bytes. Complete model bytes include more than dependency records; neither number measures duplicated dependency content.
- Two holder traversals were already identity-deduplicated in the parent evidence; do not call them two model copies or add overlapping retained groups again.

## Pinned installed IDEA adapter: static bytecode evidence

- Installed IC `2026.2.3`, build `262.10968.63`; verified `product-info.json` and `build.txt`, not inferred from a current upstream branch.
- Tooling JAR: [installed file](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/idea-oss/plugins/Kotlin/lib/kotlin-gradle-tooling.jar), SHA256 `f66b250459d965015186332822700267c1a67fab35a02a0480f89f6520d093a6`.
- Receiver JAR: [installed file](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/idea-oss/plugins/Kotlin/lib/kotlin-plugin.jar), SHA256 `f4c35876b9f35325cf75f3807eebef5e7effa39bd20607ce6155ae8b0612f48d`.
- Below, file/line references are **embedded SourceFile/LineNumberTable coordinates** from bounded read-only `javap`, not a claim that corresponding pinned IDEA source files were obtained. Guessed official tag URLs returned 404; installed class bytes are authoritative here.
- `org.jetbrains.kotlin.idea.gradleTooling.builders.IdeaKotlinDependencyContainerBuilderKt.buildIdeaKotlinDependenciesContainer`, `IdeaKotlinDependencyContainerBuilder.kt:13–17`: iterate original extension source sets; reflect `resolveDependenciesSerialized(sourceSet)`; `toList()`; associate name to list.
- `org.jetbrains.kotlin.idea.gradleTooling.reflect.KotlinMultiplatformImportReflectionImpl`, `KotlinMultiplatformImportReflection.kt:25–27`: invoke original KGP method by source-set name, `filterNotNull()`. These are reference-list operations, not array clones or content interning.
- `org.jetbrains.kotlin.idea.gradleTooling.IdeaKotlinSerializedDependenciesContainer`, `IdeaKotlinDependenciesContainer.kt:33–43`: private final map retains constructor argument; `get(String)` throws; `writeReplace()` creates `IdeaKotlinDependenciesContainerSurrogate` **with the same map**.
- Consequently, map/list/array references can already be shared between wrappers and serialization surrogate. This is not additional identical-content canonicalization, nor a proof of a second array allocation.
- Container constructor accepts an external map without deep copying; private/final storage and lack of a byte getter do **not** prove immutable ownership. KGP cannot intercept this container without crossing the IDE tooling boundary.
- `org.jetbrains.kotlin.idea.gradleTooling.KotlinMPPGradleModelBuilder`, `KotlinMPPGradleModelBuilder.kt:62`: embeds that container in the complete project model.
- `org.jetbrains.kotlin.idea.gradleTooling.KotlinMPPGradleModelBinaryBuilder`, `KotlinMPPGradleModelBinaryBuilder.kt:28–38`: build model, create new `ByteArrayOutputStream` and new `ObjectOutputStream`, `writeObject(model)`, then `toByteArray()` into `KotlinMPPGradleModelBinaryImpl.data`.
- **One new object stream per project model:** identity handles do not cross the 203 already-materialized model byte arrays. Sharing across projects cannot make those inner streams reference one another.
- Java object serialization deduplicates **identity**, not equal byte contents; verified JDK21 `ObjectOutputStream$HandleTable.hash` uses `System.identityHashCode`, consistent with the [official serialization specification](https://docs.oracle.com/en/java/javase/21/docs/specs/serialization/output.html#the-objectoutputstream-class).
- `org.jetbrains.kotlin.idea.gradleTooling.IdeaKotlinDependenciesContainerSurrogate.readResolve`, `IdeaKotlinDependenciesContainer.kt:49–57`: in IDE context, parse each byte occurrence then `toSet()` per source set into `IdeaKotlinDeserializedDependenciesContainer`. Even hypothetical wire sharing would not automatically share decoded mutable dependencies across source sets.

## Safe boundary verdict and unimplemented counterexamples

- **Reject public ByteArray interning**, including project-owned pools. Equal bytes returned for two slots/calls would alias; callers can mutate either. This breaks required behavior independently of any measured saving.
- **Reject copy-on-return as a retained/wire fix:** cloning protects callers but still gives each occurrence a different identity to `ObjectOutputStream`; a canonical pool adds retention rather than removing those arrays.
- **Reject clone-on-every-get List wrappers:** current repeated `list[0]` reads return the same slot object; fresh clones change identity and mutation persistence. Read-only Kotlin `List` does not freeze its ByteArrays.
- Counterexample pseudocode, **not implemented/run**, using existing `org.jetbrains.kotlin.gradle.idea.testFixtures.tcs.TestIdeaKotlinInstances.simpleResolvedBinaryDependency` and a real project's `IdeMultiplatformImport.instance(project)`:
```kotlin
val importer = IdeMultiplatformImport.instance(project) // deterministic fixture serializers
val sameDependency = TestIdeaKotlinInstances.simpleResolvedBinaryDependency
val a = importer.serialize(listOf(sameDependency, sameDependency))
val b = importer.serialize(listOf(sameDependency))
check(a[0].contentEquals(a[1]) && a[1].contentEquals(b[0]))
check(a[0] !== a[1] && a[0] !== b[0]) // original fresh arrays
val untouched = a[1].copyOf()
val ownSlot = a[0]
ownSlot[0] = (ownSlot[0].toInt() xor 1).toByte()
check(a[0] === ownSlot) // rules out clone-on-get
check(a[1].contentEquals(untouched) && b[0].contentEquals(untouched))
```
- Also require same public call repeated after input extras/classpath mutation; stateful or newly registered/conflicting serializers; changed sourceSetName/capabilities/attributes; Native/cinterop markers; independent Projects; concurrent callers; ordered occurrences and original errors/effects.
- An IDE-owned, exclusively held, finalized serialization surrogate could **potentially** canonicalize exact bytes after all public calls finish, retaining all occurrences/order/context and parsing separately on receive. Existing exclusive ownership/immutability is **not established**; this is neither a KGP-only patch nor authorized implementation.
- Do not substitute a new Provider/ModelManager, global cache, compression or new wire protocol. No safe KGP-only retention-sharing implementation identified at the inspected boundary.

## Coordinator-only numeric experiment, not a queued runtime action

- Authorized KGP experiment coordinates above are suitable for **diagnostic counters without changing public return semantics**, not for adopting interning. Coordinator decides whether/when to instrument and full-build under its lock.
- Count original serialized occurrences, summed lengths and distinct **array identities**; bucket full bytes by length/hash and verify `contentEquals` privately to avoid collision-based false equivalence.
- Separate (a) one serialize invocation, (b) all source sets in **one original Project/model build**, (c) all 203 owners, and (d) later imports. Export only numeric aggregates; no raw bytes, paths, credentials or identifying payload hashes.
- Report excess exact-content bytes in (b), not merely global duplicates in (c); the installed per-model object streams limit potential wire benefit to within-model identity sharing.
- Preserve source-set keys, subtype, coordinates, capabilities/attributes, classpath, extras, cinterop and source attachment occurrences. Hash decoded GAVs or equal source roots alone is insufficient.
- Include sampler/pool overhead and lifetime; an owner-wide cache across repeat imports may accumulate obsolete bytes. Hypothetical duplicate-byte totals are not measured peak savings or a complete full-4GiB repair.

## Root/module/order-entry augmentation

- Existing sanitized [text summary:2–16](file:///home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/private-ide-model-text-duplication.json#L2): `5,245` modules, `673,278` ORDER rows, `86,434` library-source roots, `328,720` class roots; max `1,317` orders/module, `1,158` modules above 200.
- `org.jetbrains.kotlin.idea.gradleJava.configuration.mpp.PopulateMppModuleDataNodeKt.createMppGradleSourceSetDataNodes`, `populateMppModuleDataNode.kt:268–473`: creates target/compilation/source-set data nodes. IDEA module count is not the 216 Gradle-project count; do not remove these semantic modules.
- `org.jetbrains.kotlin.idea.gradleJava.configuration.mpp.PopulateModuleDependenciesKt`, `populateModuleDependencies.kt:13–17,27–72`: container-present path iterates source sets, resolves project-artifact dependencies, converts the substituted result to a Set, adds dependency nodes and assigns original dependency order.
- Legacy compilation/platform-propagation/visibility augmentation is the **container-absent** branch (`populateModuleDependencies.kt:79–86`), not evidence that modern KGP dependencies are automatically populated twice.
- `org.jetbrains.kotlin.idea.gradleJava.configuration.mpp.AddIdeaKotlinBinaryDependencyKt.addDependency`, `addIdeaKotlinBinaryDependency.kt:31–75`: first find existing coordinate-derived library node; otherwise create it; preserve project/module level and Native identity; attach binary, source and documentation roots. `addToDependencyNode:83–85` calls `LibraryData.addPath`.
- `org.jetbrains.kotlin.idea.gradleJava.configuration.mpp.DependencyNodeUtilsKt`, `dependencyNodeUtils.kt:29–40`: lookup uses coordinate-derived Kotlin library name, not a source-root path. Therefore the same sources archive may be attached to distinct variant/capability libraries.
- No exact duplicate ORDER text within a module in the existing summary; `14,272` duplicate LIBRARY_SOURCE text rows across distinct library entries are **not safe entry-collapse evidence**. Root attachment multiplicity, decoded dependency multiplicity and byte duplication are different quantities.
- Trace establishes augmentation mechanisms, **not exact attribution of all 673,278 orders or their retained bytes**. Do not infer unnecessary dependencies from module/source-root counts.
- [B9 projection:123–135](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-08-investigate-dependency-fanout-cost.md#L123) removes only 70 API project-reachability pairs; other 198 modules' projections unchanged. No evidence it alone fixes this scale; API cuts are out of scope.
- Current brief's R34 active IDEA: used ≈2.68m KiB, committed ≈2.88m KiB, RSS ≈4.63m KiB; Other `201,589` KiB, metaspace `319,630` KiB. Growing Java heap is the main observed contributor, not a proved native leak or serialized-duplication diagnosis.
- Preserve V2/V3 19-test/model/source-group passes and two warm four-import passes, but fresh repeated-import reserve failure remains. **No runtime acceptance, measured duplication benefit or production repair claimed by this static investigation.**
