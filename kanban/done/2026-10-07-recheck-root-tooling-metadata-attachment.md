# REVIEW READY — Session 556

- **Scoped deployment APPROVED for the exact frozen 11-file composition below.**
  Session 555's sole remaining static B1 is closed. This permits the coordinator's
  user-authorized, explicitly reviewed main deployment/manual CI dispatch;
  it does **not** certify runtime success, bypass pre-writer gates, or authorize
  a consumer-pin update.
- Independent reviewer, not implementer. Main may read this report asynchronously;
  no further reviewer, build or publication was awaited.

# Task Tree

- `Recheck the exact remaining tooling-attachment correction`()
- `Confirm truthful classifier ownership and unchanged deployment gates`()
- `Return the narrow deployment verdict`()

# Details

- Parent: [Fork CI](2026-10-07-implement-fork-package-ci.md).
- Previous [fixed review](2026-10-07-recheck-fork-package-publication-deltas.md)
  held deployment only for the genuine root tooling JSON mismatch.
- Current narrow snapshot: `file:///tmp/kodex-fork-ci-tooling-final-review-20261007/`;
  three files, `SHA256SUMS` SHA-256
  `15e95b0d4a2bdcf737e15396e5b07111970db2d11f537354cca6d17c1bd565db`.
- Other eight current files remain the previous 11-file snapshot, whose manifest
  hash is `c342b18fec3c305be01f5e721bd486aa231e6f80395aa2275525d54ae5f876e6`.
- Coordinator ran 67 offline cases successfully on Xiaoxin. Previous standalone
  classifier was rejected; new root-only classifier is structurally checked,
  kept in the checksum/upload inventory, and excluded only from component-file
  equality because GMM does not list it as a variant payload.
- Old real Mosaic layout is coordinator-reported PASS. No final normalized
  Lucene PASS/FAIL was supplied in the loaded handoff; see B2.
  Neither check is current package-production evidence.
- Read-only source review; own only this report. No runtime, publication or Git.

## Evidence boundary and exact composition

- Loaded AGENTS, build-change, kanban, programmatic-planning, checklist,
  ask-user, document, workspace and Gradle skills; Draft, this task, CI scope,
  Packages authorization, parent gates, and preserved 553/554/555 handoffs.
- Independently verified both manifest hashes and `sha256sum -c`, twice:
  [current manifest:1–3](file:///tmp/kodex-fork-ci-tooling-final-review-20261007/SHA256SUMS#L1)
  matched **3/3**; [previous manifest:1–11](file:///tmp/kodex-fork-ci-final-review-20261007/SHA256SUMS#L1)
  matched **11/11**. Current review inputs are three replacements plus eight
  verbatim previous-snapshot files, not mutable current CI scripts.
- Replacement SHA-256:
  - `contract.py`: `245097d0935977efe747e2658e73bccb2a046bf7daed7efc89caef9d9868179a`.
  - `test_publication.py`: `4c2d5cc3aa44ddf548268ab195d05d79cf05046637e944a369dd36955cb98112`.
  - `README.md`: `e91f6c09d3722217fddaec8e0f38837bd8979d6afb2444186b6378404b67b647`.
- Unchanged eight: all three workflows, `pipeline.py`, `publisher.py`,
  `publish.init.gradle`, `smoke.py`, `test_admission.py`. Their **8/8**
  bytes match the previous manifest. Deployment must use this exact composition;
  this reviewer did not compare or approve the dirty live workspace.
- [Session 555's report:24–29](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-recheck-fork-package-publication-deltas.md#L24)
  is preserved, digest
  `5c746c1319efbdcb6949cf949fb5966ec57c9f79c4c40308bd313ac88beb63be`.
  Its original Kodex `837494340036f433e5320fabedccd874b0f320e7`
  and [immutable fork commit/tree verification:50–64](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-recheck-fork-package-publication-deltas.md#L50)
  remain the source baseline: Mosaic `7b1a412918f7bc24db847343480f010242939788`,
  SDK `53f717661712d618cd4822d1760392e903989ead`,
  Lucene `549f8afc9570bd4289f1047653d4cd6dc9665bbd`.
  This reviewer did not repeat a whole-repository/blob audit or read original
  source from dirty `build/gradle-development-experience`.
- Independently hashed/read three old real tooling attachments and Lucene root
  GMM; their digests match old manifest entries
  [Mosaic:1624](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/manifest.json#L1624),
  [SDK:4904](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/manifest.json#L4904),
  [Lucene:6824](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/manifest.json#L6824),
  [GMM:6884](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/manifest.json#L6884).
  Manifest hash matches
  `46a2c88ea51b9fef84f2aaf1876d1c5ffb67301c1e31b77cb67a3e2e5955b14a`.
  Old bytes establish layout only, never current production success.

## R — narrow correction and truthful ownership

- Original producer: cached
  [KGP 2.4.0 source archive](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/local-native-20261003/gradle-home/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-gradle-plugin/2.4.0/8cc8a31f5f34bbace6166c7ad0187724c345932f/kotlin-gradle-plugin-2.4.0-sources.jar),
  entry `org/jetbrains/kotlin/gradle/plugin/mpp/publishing/Publishing.kt:93–110`,
  attaches classifier `kotlin-tooling-metadata` to `kotlinMultiplatform`,
  separately from the software component. Its independently checked SHA-256 is
  `2610c9c7f70b6adcb6ec5d5937ec7d376de92ca6808f4e743ff10e8db5a51f8e`.
- **Admission before → after:**
  [previous:252–259](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/contract.py#L252)
  rejected the real attachment; [current:252–272](file:///tmp/kodex-fork-ci-tooling-final-review-20261007/contract.py#L252)
  admits only exact `<artifact>-<version>-kotlin-tooling-metadata.json` on
  `target == "root"`, following coordinate/stem validation.
  Arbitrary JSON filenames and this classifier on JVM/JS/Native coordinates
  remain refused before any equality exemption.
- Content requires schema `1.1.0`, `Gradle`, exact
  `org.jetbrains.kotlin.gradle.plugin.KotlinMultiplatformPluginWrapper`,
  nonempty plugin version, `projectSettings` object and nonempty
  `projectTargets` list of objects with nonempty `target`/`platformType`.
  This matches the real producer shape; it is narrow structural validation,
  not comprehensive schema or source/toolchain provenance certification.
- **Component accounting before → after:**
  [previous:443–445](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/contract.py#L443)
  incorrectly required this standalone classifier in GMM's payload set;
  [current:457–462](file:///tmp/kodex-fork-ci-tooling-final-review-20261007/contract.py#L457)
  excludes exactly its already-validated filename from component/directory
  equality. Other unreferenced payloads still fail. No fake variant is added.
- Full [inventory:107–120](file:///tmp/kodex-fork-ci-tooling-final-review-20261007/contract.py#L107)
  and [return:482](file:///tmp/kodex-fork-ci-tooling-final-review-20261007/contract.py#L482)
  retain the JSON unchanged. Diagnostic targets do not define consumer closure.
  The real [SDK tooling:12–52](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/repo/io/modelcontextprotocol/kotlin-sdk-client/0.14.0-stream.53f717661712.r1/kotlin-sdk-client-0.14.0-stream.53f717661712.r1-kotlin-tooling-metadata.json#L12)
  includes iOS, and [Lucene tooling:12–45](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/repo/org/gnit/lucene-kmp/lucene-kmp-core/10.2.0-alpha14-stream.549f8afc9570.r1/lucene-kmp-core-10.2.0-alpha14-stream.549f8afc9570.r1-kotlin-tooling-metadata.json#L12)
  includes Android/iOS. Keeping truthful unpublished original targets is not
  advertising those packages in consumer `.module`.
- Existing [normalization:143–160](file:///tmp/kodex-fork-ci-tooling-final-review-20261007/contract.py#L143)
  removes only invocation buildId/unapproved root GMM redirects, not tooling
  JSON or root JAR compile data.
  [Project-structure checks:203–231](file:///tmp/kodex-fork-ci-tooling-final-review-20261007/contract.py#L203)
  preserve original source-set mappings and common payload; extra original
  mappings are not rewritten. [GMM closure:407–414](file:///tmp/kodex-fork-ci-tooling-final-review-20261007/contract.py#L407)
  still requires only the approved consumer target/role closure.
- [Two focused cases:189–213](file:///tmp/kodex-fork-ci-tooling-final-review-20261007/test_publication.py#L189)
  cover accepted standalone root tooling/inventory retention, wrong build
  system and forbidden platform placement without adding a GMM advertisement.
  **67 passing offline cases on Xiaoxin is coordinator evidence, not reviewer
  execution.** No broader re-audit of the earlier 65 tests was undertaken.
- README hook correction:
  [before:138](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/README.md#L138)
  → [after:138–140](file:///tmp/kodex-fork-ci-tooling-final-review-20261007/README.md#L138).
  Startup properties at [pipeline:245](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/pipeline.py#L245)
  and [JS assertion:94](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/smoke.py#L94)
  agree with pinned SDK capture:
  [archive](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/archives/KotlinMcpSdk-53f717661712d618cd4822d1760392e903989ead.tar.gz),
  entry `kotlin-sdk-core/build.gradle.kts:14–30`.
  No late hook, factory/provider, API change or pinned-source rewrite was added.

## B1 — confirmed previous blocker CLOSED

- Session 555 correctly held its own snapshot. This correction closes **both**
  rejection sites. No remaining static B1 is demonstrated by the exact delta.
- Actual production call chain remains:
  - [Init:59–65](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/publish.init.gradle#L59)
    keeps original components/tooling, removing only javadoc/signatures.
  - [Direct original tasks:115–119](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/pipeline.py#L115)
    produce; [stage:289–308](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/pipeline.py#L289)
    normalizes only `.module`, preserves real JSON and seals size/SHA-256.
  - [Merge:352–368](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/pipeline.py#L352)
    calls the corrected validator and manifests its entire returned inventory.
    [Bundle:415–423](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/pipeline.py#L415)
    repeats validation and full inventory/hash equality.
  - [Smoke:164](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/smoke.py#L164)
    verifies the bundle before real consumers;
    [writer:168–171](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/publisher.py#L168)
    verifies receipts/bundle before HTTP.
  - [Upload set:61–76](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/publisher.py#L61)
    includes **all** raw files plus SHA-1/SHA-256/SHA-512/MD5 sidecars, including
    tooling JSON. [Remote comparison:93–149](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/publisher.py#L93)
    requires every expected byte, not merely component files.
- This is static contract closure and controlled-CI admission.
  Unexecuted mandatory Native/registry gates are U, not new static B1 or
  grounds to keep a sound narrow fix pending forever.

## B2 — concrete follow-up, not reopening B1

- **Old Lucene shape:** [coordinator:96–101](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-implement-fork-package-ci.md#L96)
  reports Mosaic PASS/160 files but leaves normalized Lucene in progress.
  The old [Lucene root GMM:111–155](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/repo/org/gnit/lucene-kmp/lucene-kmp-core/10.2.0-alpha14-stream.549f8afc9570.r1/lucene-kmp-core-10.2.0-alpha14-stream.549f8afc9570.r1.module#L111)
  really contains Android redirects. Coordinator must apply the actual frozen
  production `normalize_module(..., root=True)` before `validate` and record
  **actual PASS or FAIL**, file count and any exact failing predicate in the
  main handoff. Do not use a handwritten filter, drop tooling/compile data,
  or call a raw pre-normalization rejection a current-producer failure.
  This reviewer did not execute that check or invent its final result.
- **Future protected push:** retain [555 B2:292–296](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-recheck-fork-package-publication-deltas.md#L292):
  record review-enforcing branch/environment policy or equivalent stronger
  reviewed marker before automatic publication. Protected flag alone is not
  human-review proof. Current unprotected main remains reviewed-manual-only;
  no global policy was changed.
- **Repeat/partial control:** retain the exact bundle and compare genuine
  remote/rebuild bytes. Partial/different state or checksum-format drift fails
  sealed: no overwrite, DELETE or blind resume. User-approved manual remediation
  is separate; old consumer pins remain.

## U — mandatory real gates, no runtime-green claim

- **Before writer:** exact reviewed live main HEAD and real maintenance-line
  ancestry must pass [guard:39–107](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/pipeline.py#L39).
  Explicit reviewed dispatch is not authorization for the dirty development
  branch. GitHub API SHA equality cannot replace ancestry.
- Standard Linux/macOS/Windows must actually produce from clean pinned
  archives using original wrappers, measured plugin/compiler identities and
  executed tasks. Validate current real POM/GMM, metadata, sources/generated
  source provenance and genuine JVM/JS/Native payloads. Old r1 output and
  67 offline fixtures do not satisfy these gates.
- Mosaic requires main cinterop/C bitcode, generated JNI headers/Zig,
  jextract/multi-release classes, **Java21 JNI and Java25 FFM**, Apple Native
  SDK plus task-local JNI SDK and Windows headers/link/console behavior.
  SDK requires real Mac JS generation and Linux Node API/lifecycle/default
  engine/`LIB_VERSION` checks. Lucene requires BreakIterator/GB2312 sources
  and SDK36/build-tools36. Three hosts require linked/running Native probes;
  approved Linux Arm64 cross-compile remains explicitly compile-only.
- Order stays build → merge/real-metadata validation → three-host smoke → writer:
  [Mosaic:170–257](file:///tmp/kodex-fork-ci-final-review-20261007/.github/workflows/fork-packages-mosaic.yml#L170),
  [SDK writer:213](file:///tmp/kodex-fork-ci-final-review-20261007/.github/workflows/fork-packages-mcp.yml#L213),
  [Lucene writer:209](file:///tmp/kodex-fork-ci-final-review-20261007/.github/workflows/fork-packages-lucene.yml#L209).
  [Smoke gates:70–83](file:///tmp/kodex-fork-ci-tooling-final-review-20261007/contract.py#L70)
  and [receipts:79–90](file:///tmp/kodex-fork-ci-final-review-20261007/scripts/fork-packages/publisher.py#L79)
  bind completed tasks/runtimes to the immutable bundle. These gates are
  mandated workflow obligations, not optional post-publication checks.
- Only the writer exposes the repository token/package-write permission.
  Actual registry permission/HTTP/checksum/redirect behavior and all-byte
  remote reread must succeed before publication acceptance. A failure may
  leave an explicitly fail-sealed partial remote version; Maven is not atomic.
- **Consumer pin UNCHANGED until actual package pass**, full remote byte
  verification and separate full default binary/JS consumer compatibility,
  accessor/regression and genuine IDEA/model/source-navigation acceptance.
  No whole-repository/model/performance acceptance is claimed here.

## Final scope and handoff

- **REVIEW READY / scoped controlled-CI deployment APPROVED.**
  Three replacements plus eight unchanged inputs close root-classifier B1.
  Coordinator may proceed within the existing user-authorized reviewed-main/
  manual route; all U gates remain at their stated pre-write/acceptance boundaries.
- Wrote only this report via `apply_patch`. No source/other-report/checklist/
  fixture writes, tests/recipe execution/builds, Gradle/Node, IDE, credentials,
  Git commands, network/SSH, process controls or resource operations.
  No temporary files or persistent resources were created.
- Prior reports and all three forks' source/API/protocol bytes are untouched.
  This is the requested thin producer/classifier review, not a reopened
  whole-repository or earlier 65-test audit.
