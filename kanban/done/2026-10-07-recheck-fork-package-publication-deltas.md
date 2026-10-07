# Task Tree

- `Independently recheck the fixed publication delta before deployment`()
- `Trace the approved SDK JS and real payload closure`()
- `Return scoped admission and host-validation limits`()

# Details

- Parent: [Fork CI](2026-10-07-implement-fork-package-ci.md).
- Previous [first review](2026-10-07-review-fork-package-ci-before-deployment.md)
  is immutable evidence of the original defects, not a review of fixed code.
- Frozen source: `file:///tmp/kodex-fork-ci-final-review-20261007/`;
  11 files, `SHA256SUMS` SHA-256
  `c342b18fec3c305be01f5e721bd486aa231e6f80395aa2275525d54ae5f876e6`.
- Own only this report; no builds, code edits, package writes or Git operations.
- Reviewed dispatch is required on unprotected main; automatic push additionally
  requires a protected ref. No branch policy was changed by the coordinator.
- User explicitly approved SDK-only JS and fail-sealed partial publication with
  old consumer pin unchanged; automatic deletion/overwrite is not authorized.
- Source review is not genuine host, registry, default-consumer or IDEA evidence.

## Review handoff

- **REVIEW READY — Session 555**, independent of implementers 550/552/554.
- **Scoped decision: HOLD this exact deployment snapshot for one confirmed
  producer/validator mismatch, B1 below.** This is not a hold merely because
  real platform tests have not run. Most original B1 corrections are present.
- Coordinator should read this report asynchronously **before main merge or
  manual publication**. No runtime builds or additional reviewers were awaited.
- User-confirmed fail-sealed/manual remediation supersedes the first review's
  unresolved automatic-cleanup contract. It is not a remaining defect.

## Evidence boundary

- Final [SHA256SUMS](file:///tmp/kodex-fork-ci-final-review-20261007/SHA256SUMS#L1)
  hash equals the supplied
  `c342b18fec3c305be01f5e721bd486aa231e6f80395aa2275525d54ae5f876e6`;
  `sha256sum -c` matched **11/11** files, including a final recheck.
- First [SOURCE_MANIFEST.json](file:///tmp/kodex-fork-ci-first-review-20261007/SOURCE_MANIFEST.json#L1)
  hash equals
  `ceeb1d21ee2bfbbf7fbf75e6b547909e1d59e2ba6c13af143f1fcd894b68e371`;
  in-memory SHA-256 comparison matched **10/10** entries.
- Current CI evidence is **only the final snapshot**; first-snapshot links are
  before/after mappings, not current implementation. No mutable Kodex CI
  scripts were used. All ten original recipe/workflow files changed; final
  additionally contains `test_admission.py`.
- Loaded AGENTS/workflow/document/Gradle/workspace skills, Draft, main Gradle
  parent, Packages SOP/task, CI child, first review 553, SDK handoff 552,
  validator handoff 554 and relevant JVM guidance.
- Original product callers were read from immutable Kodex
  `837494340036f433e5320fabedccd874b0f320e7`,
  tree `ae0f6b49aac9ebbc634a5f073bbec8be21f4f3f7`.
  Fork commit/tree pairs independently matched:
  - Mosaic `7b1a412918f7bc24db847343480f010242939788` /
    `11580f13f28d16076c2ff13e6c43b9cef5468079`.
  - SDK `53f717661712d618cd4822d1760392e903989ead` /
    `a1852fcfffb6be426e499864b78492d3048d4f1b`.
  - Lucene `549f8afc9570bd4289f1047653d4cd6dc9665bbd` /
    `ea33a2d3a3667dbe0a70b027c0c8e1247f0d1a10`.
- Read-only in-memory loose/pack/tree decoding verified accessed Git objects by
  their object hashes. Source locations below map those pinned blobs, not
  changes to the dirty `build/gradle-development-experience` workspace.
  This is not another full-worktree audit or proof of future CI cleanliness.
- Prior real r1 artifacts establish **layout only**, never current production
  success. Their [manifest](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/manifest.json#L1)
  hash matched `46a2c88ea51b9fef84f2aaf1876d1c5ffb67301c1e31b77cb67a3e2e5955b14a`;
  the 15 selected root POM/GMM/JAR/sources/tooling files matched its hashes.
- No recipe imports/execution, fixtures, tests, Gradle/compiler/Node builds,
  IDE, Git commands, credentials, network writes, SSH or resource controls.
  Only this report was written, via `apply_patch`; no temporary files,
  services, windows or persistent resources were created.

## B1 — genuine root tooling attachment is still rejected

- **Confirmed static blocker affecting all three forks.** Ordinary KMP root
  publications include `<artifact>-<version>-kotlin-tooling-metadata.json`.
  This is an additional publication attachment, **not** a GMM library/source
  variant and not the KMP metadata JAR.
- Primary cached KGP source confirms the producer:
  - [2.3.21 source archive](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/local-native-20261003/gradle-home/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-gradle-plugin/2.3.21/5121ff4e2e491f2814c387f1b1ff6fc02f56641b/kotlin-gradle-plugin-2.3.21-sources.jar):
    entry `org/jetbrains/kotlin/gradle/plugin/mpp/publishing/Publishing.kt:48–65`
    attaches the tooling classifier to `kotlinMultiplatform`;
    `PropertiesProvider.kt:245–246` enables it by default.
  - [2.4.0 source archive](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/local-native-20261003/gradle-home/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-gradle-plugin/2.4.0/8cc8a31f5f34bbace6166c7ad0187724c345932f/kotlin-gradle-plugin-2.4.0-sources.jar):
    the same entry at **93–110** unconditionally attaches it;
    `tooling/BuildKotlinToolingMetadataTask.kt:36–42,70–71` supplies the JSON.
    Archive SHA-256 is
    `2610c9c7f70b6adcb6ec5d5937ec7d376de92ca6808f4e743ff10e8db5a51f8e`.
- Real pinned-fork r1 evidence, independently hash-checked:
  [Mosaic root tooling:1–6](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/repo/com/jakewharton/mosaic/mosaic-runtime/0.19.0-stream.7b1a412918f7.r1/mosaic-runtime-0.19.0-stream.7b1a412918f7.r1-kotlin-tooling-metadata.json#L1),
  [SDK root tooling:1–6](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/repo/io/modelcontextprotocol/kotlin-sdk-client/0.14.0-stream.53f717661712.r1/kotlin-sdk-client-0.14.0-stream.53f717661712.r1-kotlin-tooling-metadata.json#L1),
  [Lucene root tooling:1–6](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/repo/org/gnit/lucene-kmp/lucene-kmp-core/10.2.0-alpha14-stream.549f8afc9570.r1/lucene-kmp-core-10.2.0-alpha14-stream.549f8afc9570.r1-kotlin-tooling-metadata.json#L1).
  Old Gradle/version strings differ from the new recipe; the attachment
  mechanism and pinned KGP versions are the relevant evidence.
- Exact caller chain:
  [final init:59–65](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/publish.init.gradle#L59)
  removes only javadoc/signatures, leaving this original attachment;
  [task generation:115–119](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/pipeline.py#L115)
  invokes the original root publication on Mac;
  [stage selection:289–303](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/pipeline.py#L289)
  retains JSON and records its checksum.
  [merge:352](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/pipeline.py#L352)
  calls `validate()`, whose
  [attachment allowlist:252–257](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/contract.py#L252)
  rejects it as `Unexpected publication attachment`.
- Merely extending that allowlist is insufficient:
  [payload equality:443–445](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/contract.py#L443)
  also assumes every non-POM/module attachment is referenced by a GMM variant.
  This real standalone tooling attachment is not. The same validator is
  reached again by bundle verification, smoke and writer preflight.
- Before/after:
  [first allowlist:128–132](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/contract.py#L128)
  already missed this attachment; final fixed optional Native `-metadata.jar`
  but left root tooling unhandled. The
  [final structural fixture:78–177](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/test_publication.py#L78)
  omits tooling JSON, so offline fixture success cannot cover this producer shape.
- Minimum correction: narrowly account for the original **root-only tooling
  JSON** as a separate attachment, check its structure/tooling identity, and
  keep it in manifest/checksum/upload inventory without inventing a GMM role.
  Alternatively explicitly exclude this diagnostic attachment at publication
  configuration with a reviewed/documented decision. Do not broaden arbitrary
  JSON acceptance, drop KMP metadata/sources, or fabricate a variant/JAR.
- This is a proved producer/validator contract conflict, not a claim that
  current Native generation has run or failed. It fails before the writer,
  preserving remote safety, but prevents approval of this exact deployable pipeline.

## R — fixed original B1 and integrated delta

### Admission, workflow syntax and scan boundary

- Original B1-1 is corrected:
  [first YAML:55](file:///tmp/kodex-fork-ci-first-review-20261007/.github/workflows/fork-packages-mosaic.yml#L55)
  → [final Mosaic:54–60](file:///tmp/kodex-fork-ci-final-review-20261007/.github/workflows/fork-packages-mosaic.yml#L54),
  [MCP:52–58](file:///tmp/kodex-fork-ci-final-review-20261007/.github/workflows/fork-packages-mcp.yml#L52),
  [Lucene:52–58](file:///tmp/kodex-fork-ci-final-review-20261007/.github/workflows/fork-packages-lucene.yml#L52).
  Admission is protected push **OR reviewed manual main dispatch**.
  [Python:91–107](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/pipeline.py#L91)
  enforces that same split; [publisher:159–171](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/publisher.py#L159)
  calls it again and checks live main before writes/completion.
- Main remains unprotected, so automatic publication is blocked. First dispatch
  reviews this exact code and pins `github.sha`; no branch/ruleset was changed.
  [identity:39–74](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/pipeline.py#L39)
  checks clean gitlinks, public HTTPS origin, actual maintenance-branch fetch
  and `merge-base --is-ancestor`, not merely matching an API HEAD.
- Original **12 job-level `runner.temp` uses** moved to step environment:
  [first Mosaic:96,183](file:///tmp/kodex-fork-ci-first-review-20261007/.github/workflows/fork-packages-mosaic.yml#L96)
  → [final:125–130,207–215](file:///tmp/kodex-fork-ci-final-review-20261007/.github/workflows/fork-packages-mosaic.yml#L125);
  MCP and Lucene likewise use step-local homes.
  Lucene retains constant
  [COPYFILE_DISABLE:97](file:///tmp/kodex-fork-ci-final-review-20261007/.github/workflows/fork-packages-lucene.yml#L97)
  without an empty job-env mapping. **Actual actionlint is reported final-clean
  by the coordinator/user**, not rerun by this reviewer; text fixtures alone
  are not actionlint evidence.
- Original B1-2 is corrected:
  [first producer:193](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/pipeline.py#L193)
  → [final startup command:237–247](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/pipeline.py#L237);
  [first consumer:143](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/smoke.py#L143)
  → [final:279–284](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/smoke.py#L279).
  Both supply `--no-scan` before settings evaluation, not a late init callback.

### Ownership, variant roles and real payload structure

- Original B1-3 is corrected:
  [first broad tty prefix:247](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/contract.py#L247)
  → [final exact base:247–256,459–464](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/contract.py#L247).
  Only `base == mosaic-tty` Native owns main cinterop and C bitcode.
  tty-terminal instead requires its terminal/tty edges; no nonexistent payload
  responsibility was copied into it. Pinned
  [tty build:266–293](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/build.gradle#L266)
  and [tty-terminal:26–32](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty-terminal/build.gradle#L26)
  confirm the ownership.
- Original B1-5's weak checks were substantially corrected, **subject to the
  remaining attachment B1 above**:
  [first:146–199](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/contract.py#L146)
  → [final roles:163–231](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/contract.py#L163),
  [POM/GMM edges:272–397](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/contract.py#L272).
  Correct platform/Native target/JS IR/environment/usage/category/source roles,
  redirect destination roles, fork versions and essential dependency scopes
  are required. JVM/JS API excludes implementation-only edges; runtime includes
  them; Native/common API includes them; source variants need no library edges.
- Prior real POM/GMM agrees with the revised distinction: roots declare common
  dependencies directly, target POMs select target artifacts, target GMM may
  depend on roots. A root-to-JVM POM forwarding edge is **not required**.
  Optional Apple `-metadata.jar` can be manifest-only; no false requirement for
  code in every empty hierarchy source set was added.
- Root JAR now requires real project-structure/source-set mappings, common
  manifest and nonempty `.knm`; source JAR needs packaged Kotlin source.
  These are structural checks, **not source-byte provenance or external
  dependency compatibility certification**.
- tty JVM requires all **seven** exact loader resources, ordinary Jni/
  NativeLibrary classes, real version-22 FFM class paths and `Multi-Release: true`:
  [final:446–458](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/contract.py#L446).
  Pinned [Zig:14–20](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/build.zig#L14)
  and [loader:24–49](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/src/jvmMain/java/com/jakewharton/mosaic/tty/NativeLibrary.java#L24)
  confirm those paths.
- Java21 JNI is no longer confused with Java25 FFM:
  [explicit JNI task:219–235](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/smoke.py#L219),
  [shared task/receipt closure:70–83](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/contract.py#L70),
  [writer receipt refusal:79–90](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/publisher.py#L79).
  Mosaic installs/provides Java21 on each smoke host and requires
  `verifyForkJvmJni` / `jvm-jni-java21` in addition to ordinary Java25 execution.
  The pinned pre-22 caller imports Jni; version-22 caller imports Libmosaic.

### SDK-only JS, generated version and toolchains

- Original B1-4 is corrected:
  [first target set:11–15](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/contract.py#L11)
  → [final SDK exception:24–30](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/contract.py#L24).
  Mosaic/Lucene stay JVM + four Native; SDK additionally publishes real JS
  for core/client/server. Mac alone owns root/JVM/Mac/SDK JS; Linux owns
  linuxX64/linuxArm64; Windows owns mingwX64. Task totals are **36 / 21 / 6**,
  through original direct publication tasks, not an umbrella wrapper.
- Immutable Kodex callers
  [stdio:2–9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/build.gradle.kts#L2),
  [HTTP:2–10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/streamable-http/build.gradle.kts#L2),
  [host convention:14–23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-host.gradle.kts#L14)
  prove genuine Node/SDK JS consumers. Blob IDs respectively are
  `ad045f2138dc011a6f32c624d543a8ff7340fec0`,
  `0394a4241363c8f7669640e09376287201247972`,
  `ec4fccfc78047fab29f638d8c80cec48b130777e`.
- SDK JS validation checks JS/IR variants, real IR/linkdata KLIB structure,
  core dependencies, sources and separate JS compiler identity:
  [final:414–440](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/contract.py#L414).
  Linux must run **`compileKotlinJs` and `jsNodeDevelopmentRun`**;
  provided Node >=22.14.0 cannot silently skip them.
  [real probe:73–146](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/smoke.py#L73)
  compiles SDK Client/Server/models/stdio/HTTP APIs and executes stdio
  frame/flush/cleanup, generated version assertion and ordinary Ktor defaults.
- `ktor-client-js:3.5.1` is an actual default-engine dependency supplied to the
  consumer, **not a mock/manual-engine protocol-smoke workaround**:
  [final:193–210](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/smoke.py#L193).
  SDK pin uses Ktor 3.5.1; original Kodex catalog uses 3.5.0 and the same
  `ktor-client-js` engine artifact. Full Kodex consumption must later resolve
  compatible versions; this isolated probe does not certify that.
- Startup `-Pversion` / `-PVERSION_NAME` precedes SDK's captured `project.version`:
  [final producer:245](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/pipeline.py#L245),
  pinned [generator:14–30](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/kotlin-sdk-core/build.gradle.kts#L14).
  JS asserts the compiled `LIB_VERSION == exact publication version` at
  [smoke:94](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/smoke.py#L94).
  The redundant task-input-only hook is removed; isolated root-name correction
  remains configuration-only, not a pinned-source rewrite.
- Actual loaded KMP JAR filename/hash is collected at
  [init:83–92](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/publish.init.gradle#L83).
  Native/JS compiler version resources and archive hashes are read at
  [pipeline:122–151,259–265](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/pipeline.py#L122),
  not assigned from KGP or stdlib manifests. The local real Native 2.4.0 and
  2.3.21 archives contain that version resource; current three-host layouts
  and actual selection still require CI evidence. Catalog `kgp` remains a
  declaration, and Zig/cklib/jextract are correctly labeled declared tools.

### Single writer and approved fail-sealed semantics

- Explicit manifest owner is now `Stream29/Kodex`, enforced by
  [merge/verify:360–390](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/pipeline.py#L360).
  Literal upstream versions remain
  `0.19.0-SNAPSHOT-kodex.7b1a412918f7`,
  `0.14.0-kodex.53f717661712`,
  `10.2.0-alpha14-kodex.549f8afc9570`, with full commit/tree identity.
- Only writer jobs grant `packages: write` and expose `GITHUB_TOKEN` to the
  final publisher step; no submodule checkout, Gradle or fork source runs there:
  [Mosaic:223–257](file:///tmp/kodex-fork-ci-final-review-20261007/.github/workflows/fork-packages-mosaic.yml#L223),
  [MCP:213–246](file:///tmp/kodex-fork-ci-final-review-20261007/.github/workflows/fork-packages-mcp.yml#L213),
  [Lucene:209–242](file:///tmp/kodex-fork-ci-final-review-20261007/.github/workflows/fork-packages-lucene.yml#L209).
  Producers use archived clean source and isolated file publication. Consumers
  use an exclusive fork-group repository, no global Maven Local, source
  fallback, provider or second publication manager.
- [classify/publish:93–149](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/publisher.py#L93)
  rejects existing partial/different state before writes. Fresh failure cannot
  declare completion; it reports possible partial state, preserves old pins
  and requires separately authorized remediation. No auto DELETE, overwrite
  or hidden resume path exists.
- Complete-exact skip compares **every expected byte**, including manifest and
  sidecars. Generated sidecars skip only when identical; final verification
  cannot intentionally accept missing/different checksums.
  Toolchain/JDK/recipe drift may honestly make same-version rebuilds fail;
  it does not authorize rewriting that version.
- Maven is not atomic; GET/conditional PUT is not a guaranteed server lock.
  Fail-sealed/manual remediation is the approved policy, not a promise of
  transactional cleanup or a manifest completion endpoint.

## B2 — concrete, scoped follow-up checks

- **B1 correction check before deployment:** coordinator should use a genuine
  root publication inventory including tooling JSON, not only the existing
  fixture. Account for both attachment allowlist and unreferenced-payload
  equality; refuse tooling JSON on target coordinates/arbitrary JSON names.
  Keep full hashes and writer byte verification. Reviewer did not execute it.
- **Documentation correction:** [final README:138](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/README.md#L138)
  still says `generateLibVersion` gets an input hook. Replace that stale claim
  with startup version capture plus real `LIB_VERSION` assertion. The code's
  ablation is correct; this sentence is not another runtime blocker.
- **Future automatic-push policy:** a protected ref alone does not prove human
  review of every code change. Before enabling that route, coordinator must
  record review-enforcing branch/environment policy or an equivalent stronger
  reviewed marker. Current unprotected main is manual-only, and first dispatch
  is explicitly reviewed; do not relabel it as unreviewed or change global rules.
- **Same-version repeat control:** retain the validated bundle; compare genuine
  rebuild/remote manifest bytes including toolchain and recipe. Any partial,
  different or sidecar-format mismatch must fail sealed, not trigger repair.
- Mutable/older Actions majors may produce warnings or need future provenance
  hardening. They are not a newly demonstrated publication-protocol blocker.

## U — mandatory real gates, not new static blockers

- **Before any writer:** original wrappers, loaded plugin/archive identity,
  actual executed publication graph, generated sources, real KMP/JS/Native
  artifacts and all three linked/running binary probes must succeed.
  The workflows enforce build → merge/validate → three-host smoke → writer;
  exact task/runtime receipts bind to the validated manifest.
- **Mosaic:** real main cinterop/C bitcode, generated JNI headers → Zig,
  jextract → multi-release JAR, Java21 JNI and Java25 FFM, selected Apple SDK
  plus task-local JNI SDK, and Windows main headers/link/console behavior.
  Presence/class-magic checks are not execution proof.
- **SDK/Lucene:** real SDK Mac JS generation and Linux Node lifecycle/default
  engine/version assertion; actual external/transitive variant resolution;
  Lucene BreakIterator/GB2312 generated sources and SDK36/build-tools36 on
  the required hosts. Offline structural fixtures do not certify these.
- **Source provenance before publication acceptance:** retain clean archive
  commit/tree and executed generator evidence; inspect current sources
  attachments against pinned fork APIs and generated sources/LibVersion.
  This review found structural checks, not a separate full source-fingerprint
  gate; do not claim such provenance was measured from the fixtures.
- **Registry:** actual token permission, direct Maven GET/PUT/redirect/status/
  checksum behavior, stale-main interruption and reread of every expected
  remote byte. Loopback tests are not GitHub HTTP evidence. Retain failure
  artifacts before seven-day expiry; remediation remains separately authorized.
- **Before consumer pin changes:** complete remote byte verification plus
  separate full Kodex default binary/JS consumer compatibility, root-accessor,
  regression and genuine IDEA/model/navigation acceptance. No blanket claim
  about all 203 projects or IDE speed/resource gains follows from this review.
- Linux Arm64 cross-compilation/compile-only receipt is the user-approved
  first-producer boundary, **not another B1**. Product full-runtime acceptance
  for the remaining architecture matrix stays explicitly open.
- Current Xiaoxin producer preflight failed Git HTTPS after 120 seconds
  **before Gradle**. Correct GitHub API main SHA is not ancestry proof; seeded
  caches are not cold-speed evidence. Neither result certifies publication.

## Final admission

- **REVIEW READY / EXACT SNAPSHOT HOLD for the root-tooling B1 only.**
  Correct that narrow real-shape mismatch and review the exact delta before
  main merge/manual publication. No new framework or broad source redesign
  is needed.
- Once that static blocker is closed, controlled deployment to the existing
  standard Linux/macOS/Windows pipeline can be approved in scope; named real
  production/smoke/provenance gates must still precede publication, remote
  verification must precede acceptance, and full consumer gates must precede
  any pin update. Unexecuted mandatory host gates alone are not deployment B1.
- No implementation fixes or operations were performed by this reviewer.
