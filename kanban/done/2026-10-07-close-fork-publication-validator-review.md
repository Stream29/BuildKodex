# Task Tree

- `Correct the reviewed tty ownership and publication structure checks`()
- `Add small counterexamples against the real pinned publication shape`()
- `Hand off the exact delta for centralized host validation`()
- `Complete centralized structural validation and exact delta review`()

# Details

- Parent: [Fork CI](2026-10-07-implement-fork-package-ci.md).
- Scoped completion: corrected tty ownership, real metadata/source/dependency
  checks and root tooling attachment passed exact-byte independent rechecks,
  remote offline tests and actual three-host staging/merge for all three forks.
  Later writer and whole-Kodex consumer gates remain in the parent, not this
  structural-validator task. Earlier handoff pending statements are historical.
- Independent [first review](2026-10-07-review-fork-package-ci-before-deployment.md)
  confirms tty-terminal was incorrectly required to own tty C payloads, and
  root/target metadata validation overstated its guarantees.
- SDK JS worker is ready. Own only `contract.py` and focused validator fixtures
  in `test_publication.py`; coordinator owns workflow contexts, trusted
  admission, no-scan, toolchain evidence, smoke, publishing and Git.
- Preserve the approved real SDK JS closure and all publication algorithms.
  Derive required intra-fork edges from pinned sources; do not invent payloads
  or certify external dependencies without the actual compile gates.
- No builds/tests/IDE/credentials/network writes or Git operations by the worker.
  Use the existing helpers, not another validator/provider abstraction.

## Integration handoff

- **IMPLEMENTATION READY / CENTRAL TESTS PENDING — Session 554.**
- Changed only
  [contract.py](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/contract.py#L1),
  [existing fixture and eight focused cases](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/test_publication.py#L34),
  and this child handoff. SDK JS closure/compiler/archive checks and its test
  methods were preserved; no workflow/pipeline/publisher/smoke/README or review
  report edits by this worker.
- Read B1-3/B1-5 and the original first-review manifest. Its SHA-256 remains
  `ceeb1d21ee2bfbbf7fbf75e6b547909e1d59e2ba6c13af143f1fcd894b68e371`.
  Current mutable files include the separate SDK/coordinator deltas; this is
  not a new immutable review snapshot or deployment approval.
- Handoff-byte SHA-256 before coordinator integration:
  `contract.py = 6373dfab2cd8a0eadc8cb20812201313d88852038e8f929ab76d673d08eac412`;
  `test_publication.py = 37eb784aead38cbcc07c738b462263e4fe575ebc7d88326a1303c1bf5de51aec`.
  Coordinator must freeze the combined exact delta for independent review.

### Expected producer shape

- Every root/JVM publication: `.pom`, `.module`, `.jar`, `-sources.jar`.
  Every Native/SDK JS publication: `.pom`, `.module`, `.klib`, `-sources.jar`.
  All names use the publication artifact/version stem.
- Only exact Native base `mosaic-tty` owns `-cinterop-mosaic.klib` and embedded
  `/native/mosaic.bc`. Ordinary tty-terminal Native publications without either
  pass the model; a tty-terminal cinterop attachment is rejected.
- Native `-metadata.jar` is allowed only as a referenced `kotlin-metadata`
  library-role payload. Real prior Apple publications advertise this attachment;
  inspected examples are manifest-only. Do not fabricate metadata or require
  that every Native target emits it.
- Root/target GMM requires true platform/Native target/JS IR attributes,
  JVM environment, usage/category and source docstype/bundling roles.
  Root redirects stay within the same base and match destination roles.
  Required roles: root metadata API + sources; JVM/JS API + runtime + sources;
  Native API + sources, with optional Native metadata.
- Root JAR requires `META-INF/kotlin-project-structure-metadata.json` containing
  `projectStructure`, root publication flag, sourceSets with `dependsOn`,
  `moduleDependency`, `binaryLayout=klib`, and actual platform `sourceSet`
  mappings. It also requires commonMain manifest and nonempty `.knm` entries.
  Genuine empty hierarchy source sets need not each have compiled payloads.
- Source attachments require nonempty Kotlin files with package declarations,
  not `fixture.txt` or a comment-only stand-in. This is structural checking,
  not proof that every source byte equals the pinned source tree.
- Essential edges are
  [pinned main-project API/implementation dependencies](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/contract.py#L40):
  animation/testing → runtime; runtime → terminal + tty-terminal;
  tty-terminal → terminal + tty; SDK client/server → core.
  Root GMM/Native API includes implementation edges; JVM/JS API excludes them,
  runtime includes them. Source variants do not require library dependencies.
- Genuine target GMM can depend on root modules; target POMs select matching
  platform artifacts. Root POMs inspected from prior real publications declare
  common edges directly, **not** a root-to-JVM forwarding dependency. Scope
  checks follow those real POMs: root runtime; JVM/JS API compile and
  implementation runtime; Native compile.
- tty JVM requires all seven exact loader resources from
  [pinned Zig destinations](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/build.zig#L14),
  ordinary Jni/NativeLibrary classes, version-22 Libmosaic/Tty/TestTerminal/
  StandardStreams class entries with class magic, and `Multi-Release: true`.
  Path/class presence is not JNI/FFM execution evidence.

### Evidence and central gates

- Read pinned fork build sources and prior real publication metadata/JAR lists
  under `file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/repo`.
  Prior artifact bytes establish layout only, not current-version production.
  Read-only Xiaoxin directory discovery also finished; no remote job/build.
- Eight focused cases cover tty-terminal valid absence/misplaced attachment,
  tty missing C payloads, wrong root Native attributes/missing runtime role,
  missing POM/GMM edges, junk root metadata/invalid sourceSet mapping, junk
  sources and misplaced JNI/FFM/multi-release payloads.
- Fixture bytes remain an **OFFLINE, UNPUBLISHED MODEL**, not compiler output.
  Production still uses original Gradle publications; no fake artifact recipe.
- Worker performed static reading/diff/hash review only. No tests, Gradle,
  local application runtime, IDE, credentials, publication or Git operations.
  No temporary files or retained sessions/resources were created.
- Coordinator owns centralized offline tests, actual current three-host
  publication outputs and any evidenced producer-shape differences.
  External library resolution, transitive/consumer variant compatibility,
  pinned source-byte provenance and runtime loading remain real CI gates;
  this validator does not certify them.

## Coordinator integration requirement

- Mosaic smoke now defines a real `verifyForkJvmJni` task using supplied Java21,
  in addition to Java25 FFM. Its `smoke_gates` task/receipt closure must include
  that task and label the extra runtime `jvm-jni-java21`. Other forks unchanged.
  Coordinator will apply that small integration delta after the validator handoff
  unless this work line includes it explicitly.
