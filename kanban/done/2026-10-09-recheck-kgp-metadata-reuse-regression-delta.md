# REVIEW READY — independent narrow V2 delta review

# Task Tree

- `Verify fixed snapshots and read official source instructions`() // complete
- `Recheck normalization, changed-content oracle and original constructor`() // complete
- `Deliver scoped build-readiness verdict and remaining gates`() // complete

# Details

- Parent: [isolated experiment][P]. Prior [first review][V1] and V1 snapshot remain preserved; this review does not revise their historical findings.
- Independent READONLY source review; only this report written with `apply_patch`. No source/other-document edits, builds/tests, IDE/MAT/process/device control, Git/network writes, credentials or artifact adoption.
- Loaded BuildKodex/workflow, documentation, Gradle, abstraction, workspace/device and IDE-collaboration skills; read actual upstream root/KGP `AGENTS.md`, `.ai/guidelines.md` and `.ai/testing.md`.
- Used explicitly authorized SSH-read fallback for isolated source; no IDE opened on new source while user gaming. No heavy wait or competing compute operation.
- Official v2.4.0 provenance: supplied commit `add726ca8c82922b6ab4cb2a27ae738d6a780817`, tree `e94052d76d53e57f3dc6592f89a245657d0906e4`; directly read original files under the pinned Xiaoxin source, not a replacement facade.
- Independently verified all three V2 SHA-256 values and `SHA256SUMS`:
  - [Factory][F]: `afb62a4addff1a0713c6447d67fd2e5ba0a72005da3c167ccd719aaa44ef450f`.
  - [Test][T]: `53abe944dd1ced27a988b31bafb6f50872f067524bcfcd9340c7741b1b1469c7`.
  - [Extractor][E]: `711c7dfe8e57ea41ddd79f126808d2b24955c50213f562440fdb8c689d963d94`.
- Factory additionally compared byte-for-byte equal to V1. Prior Plugin-state/consumer analysis is inherited, not claimed freshly re-audited.

## Verdict: B1 closed statically; controlled build verification ready

- **V1 normalization B1 is corrected in this fixed V2. No concrete new delta B1 found.** Ready to proceed within the authorized isolated source/build experiment, subject to official prerequisites; not a compilation/regression pass, 4GiB GUI success or adoption approval.
- [Sample21–33][T] explicitly includes `commonMain -> emptySet()` in dependsOn and `nativeMain -> emptySet()` in module dependencies, both binary layouts (KLIB and METADATA), and all remaining source-set fields.
- Original [serialize223–239][PS223] emits both source sets and empty arrays; [parse329–363][PS329] inserts dependsOn/dependency entries unconditionally. All nine fields at [99–125][PS99] now match the sample after unchanged parsing/freezing; test52/123/154 no longer has the missing-versus-empty contradiction.
- Original serializer writes `moduleDependency` as an array of `groupId + ":" + moduleId` strings, **not** module objects. V1's standalone quoted `"module"` replacement was a no-op here; [V2 test83–95][T83] replaces actual `group:module` with `group:other!`.
- `assertNotEquals(original, changed)` precedes writing; equal-length ASCII replacement, asserted byte length and successful mtime restoration prevent a false unchanged-file scenario. Subsequent assertions require new identity, changed first dependency, retained second dependency and unchanged old snapshot.
- [Test54–60][T54] checks source-set/variant/nested-name/dependency iteration and exact `sample.toJson() == first.toJson()`. Original [serialize201–269][PS201] drives output order; ordered wrappers preserve parsed iteration. Coverage is the fixture's observable order, not every collection shape.
- [Old-format test127–134][T127] reads the actual official 0.3.1 fixture and compares original-parser versus cached metadata with nine-field equality, exact `toJson()` and repeat identity. It compares normalized serialization, not original whitespace. [Official Test workingDir679][BUILD672] is `projectDir`, validating the relative fixture path.
- Original [extractor36–42][UE36] has unary constructor descriptor `(Ljava/io/File;)V`. [V2 extractor36–45][E] adds a trailing default lambda plus `@JvmOverloads`, preserving the unary JVM overload by Kotlin source rules; factory84–87 passes its actual reader. Emitted ABI remains unmeasured pending build.

## Unchanged responsibility and error boundary

- Internal Project extension still obtains the original implementation factory. Actual selected component file retains Usage/empty-marker/`singleOrNull` checks and is passed to the reader; no compatibility factory, facade or copied model registry.
- Factory42–67 reads current text before project-owned same-file comparison. Content, not path/size/mtime alone, establishes reuse; synchronized parse/freeze/publication retains the last successful snapshot but never serves it for a failing current read/parse.
- `file.readText()` remains uncaught: actual IOException propagates without wrapping. Malformed parsing is unchanged; test103's generic `assertFails` alone does not prove precise malformed exception parity.
- Seven collection fields remain deep-frozen through ordered Map/Set copies, with two scalar fields retained. Per-project/per-file latest-success cardinality and retained full text cost are unchanged; no measured savings claimed.
- Apart from the internal constructor/reader seam, public declarations and Jar JSON/XML/included-build/provider paths are unchanged in the reviewed extractor. No classpath, Native/cinterop, target, ancestry, navigation or public API change introduced by this delta; whole-plugin ABI/runtime compatibility remains unproven.

## Remaining gates and minimum ablation responsibility

- **Seven prepared regressions, zero executions claimed**: repeat/project identity; independent projects/files; equal-size/equal-mtime changed content; malformed/missing reads; immutable mutation attempts; original 0.3.1 equality/order; concurrent readers. Build owner must execute official tests and record receipts against these hashes.
- Inherited routine gaps, not new B1s: real factory `create`/selector integration; cold first factory access (test140 preinitializes); populated configuration-cache store/reload with changed/missing input; remaining frozen collection mutation cases/alternate root flag; whole ABI/model/source/navigation/JVM/Node/Native/cinterop gates.
- Minimum ablation: unchanged official parse-per-call versus reuse only, identical selected files/content/read failures, consuming-Project ownership and workload. Compare typed/order results, parse count, retained String/wrapper cost and model behavior; no visibility/provider/ModelManager sharing or presumed recovery of all 39k PSM objects/817MB.
- Coordinator/user-reported build infrastructure, **not patch results**: R01 discovered wrapper9.4.1 Java21 criterion overriding requested25 and stopped; R02 used actual21 but atomicfu Java8 auto-download could not reach GitHub; private official Adoptium8u504 digest verified; R03 **route dry-run running as supplied**, not awaited/reclassified here.
- Original [wrapper][WR]/[daemon criterion12][DJ] confirm 9.4.1/21. Prerequisites can still fail; no fake JAR, substituted wrapper, undeclared classpath or misleading local-version claim is authorized.
- User/coordinator reports clean production `5b41be52`, 353 representative passes and R07 Gradle action completion, but host-memory-guard stop/no `EXTERNAL_SUCCESS`; full GUI outcome remains unknown. None is V2 evidence. All targets and original 4GiB budget remain acceptance requirements; no fork/publish/production version switch.
- Source-fidelity, whole-document and ablation review completed: delta proofs/provenance/responsibility/gates retained; broad lifecycle analysis stays in V1. No temporary files/resources created.

## Evidence URIs (SSH references identify original Xiaoxin files)

[P]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-09-experiment-kgp-metadata-producer-reuse.md#L44
[V1]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-09-review-kgp-metadata-reuse-source-overlay.md#L1
[F]: file:///tmp/kodex-kgp-metadata-source-overlay-v2-20261009/KotlinProjectStructureMetadataExtractorFactory.kt#L42
[T]: file:///tmp/kodex-kgp-metadata-source-overlay-v2-20261009/KotlinProjectStructureMetadataReuseTest.kt#L21
[E]: file:///tmp/kodex-kgp-metadata-source-overlay-v2-20261009/MppDependencyProjectStructureMetadataExtractor.kt#L36
[T54]: file:///tmp/kodex-kgp-metadata-source-overlay-v2-20261009/KotlinProjectStructureMetadataReuseTest.kt#L54
[T83]: file:///tmp/kodex-kgp-metadata-source-overlay-v2-20261009/KotlinProjectStructureMetadataReuseTest.kt#L83
[T127]: file:///tmp/kodex-kgp-metadata-source-overlay-v2-20261009/KotlinProjectStructureMetadataReuseTest.kt#L127
[PS99]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/KotlinProjectStructureMetadata.kt#L99
[PS201]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/KotlinProjectStructureMetadata.kt#L201
[PS223]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/KotlinProjectStructureMetadata.kt#L223
[PS329]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/KotlinProjectStructureMetadata.kt#L329
[UE36]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/MppDependencyProjectStructureMetadataExtractor.kt#L36
[BUILD672]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/libraries/tools/kotlin-gradle-plugin/build.gradle.kts#L672
[WR]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/gradle/wrapper/gradle-wrapper.properties#L3
[DJ]: ssh://xiaoxin-ubuntu/home/stream/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c/gradle/gradle-daemon-jvm.properties#L12
