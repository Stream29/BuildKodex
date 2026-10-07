# Task Tree

- `Independently inspect the fixed CI recipe before any package writes`()
- `Trace real target metadata, provenance, credential and failure boundaries`()
- `Report deployment blockers and the smallest corrections`()

# Details

- Parent: [Fork package CI implementation](2026-10-07-implement-fork-package-ci.md).
- Read-only review of the first handoff; SDK JS closure is a separately approved
  increment and must be reviewed after its exact bytes are available.
- Snapshot and manifest provenance will be supplied in the worker prompt.
- Own only this report; no code changes, tests, Gradle, network writes, IDE/resource
  operations or Git changes.
- Distinguish genuine Maven immutability/atomicity limits from fabricated completion
  guarantees. Inspect real consumers, KMP root/target metadata, standard runner
  gates, exact main/ancestry, partial publication, redirects and token secrecy.
- Main is unprotected: exact HEAD equality is not branch protection. Automatic
  publication must satisfy the approved trusted-admission boundary.

## Review handoff

- **REVIEW READY — Session 553, independent of implementer Session 550.**
- **First snapshot is NOT DEPLOYMENT READY.** Confirmed static blockers below
  must be corrected and reviewed; real platform and registry gates remain open.
- This handoff is for asynchronous coordinator reading **before Git commit or
  deployment**. It is not approval to publish or update a consumer pin.

## Evidence boundary and hash verification

- Reviewed only
  [first snapshot](file:///tmp/kodex-fork-ci-first-review-20261007/SOURCE_MANIFEST.json#L1).
  Manifest SHA-256:
  `ceeb1d21ee2bfbbf7fbf75e6b547909e1d59e2ba6c13af143f1fcd894b68e371`.
- `sha256sum` verified the manifest; `jq` emitted its entries to
  `sha256sum -c -`: **10/10 file hashes matched**. The snapshot contains ten
  recipe/workflow files plus the manifest, not eleven recipe files.
- Read project guidance, associated workflow/document/Gradle skills, Draft,
  executable main Gradle plan, Packages plan, CI child, and approved SDK JS child.
  Current decisions override historical planning-only restrictions, but do not
  remove the publication/consumer acceptance gates.
- No live dirty CI implementation was read. Original Kodex file links below are
  **location mappings for the reviewed first snapshot**, not evidence that those
  mutable paths still contain these bytes. Snapshot links identify actual evidence.
- The three permitted fork worktrees were checked against immutable Git objects:
  read-only, in-memory loose/pack/tree decoding and Git blob hashing; no Git
  commands, index updates, fetches, checkouts, or resource operations.

| Fork | Pinned commit | Verified tree | Tracked blobs checked |
| --- | --- | --- | --- |
| Mosaic | `7b1a412918f7bc24db847343480f010242939788` | `11580f13f28d16076c2ff13e6c43b9cef5468079` | 430 |
| LuceneKmp | `549f8afc9570bd4289f1047653d4cd6dc9665bbd` | `ea33a2d3a3667dbe0a70b027c0c8e1247f0d1a10` | 4,151 |
| KotlinMcpSdk | `53f717661712d618cd4822d1760392e903989ead` | `a1852fcfffb6be426e499864b78492d3048d4f1b` | 429 |

- All checked blob bytes matched; no nested gitlinks appeared in those trees.
  Ignored caches/build outputs were not used as evidence. This does not certify
  future CI worktree cleanliness; the recipe must still enforce it.
- Real consumer callers were read directly from **immutable Kodex `83749434`
  objects**, never from its current mutable worktree. Relevant blob IDs:
  stdio `ad045f2138dc011a6f32c624d543a8ff7340fec0`,
  HTTP `0394a4241363c8f7669640e09376287201247972`,
  host convention `ec4fccfc78047fab29f638d8c80cec48b130777e`,
  root settings `8fc0e0d2308a2ad9aca17c9c7d7b0e0381d42233`.
- Reviewer executed **no recipe, tests, Gradle, CLI smoke, IDE operations,
  fixture creation, extraction, temporary HTTP server, or network writes**.
  Public primary documentation was read. Only this report was written, using
  `apply_patch`; no temporary files or persistent resources were created.

## B1 — confirmed static defects and smallest corrections

### B1-1: Unprotected-main push bypasses reviewed admission

- All three workflow gates admit `push` independently of `inputs.reviewed`.
  Python guard/publisher admit either push or dispatch. Exact live-main equality,
  clean gitlinks, and maintenance-branch ancestry prove identity, **not review**.
- Evidence: original
  [Mosaic workflow:55](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/.github/workflows/fork-packages-mosaic.yml#L55),
  reviewed [snapshot:55–58](file:///tmp/kodex-fork-ci-first-review-20261007/.github/workflows/fork-packages-mosaic.yml#L55);
  original
  [pipeline:88](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/pipeline.py#L88),
  [snapshot:88–96](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/pipeline.py#L88);
  original
  [publisher:159](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/publisher.py#L159),
  [snapshot:159–163](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/publisher.py#L159).
- Main is currently unprotected, as explicitly supplied and recorded in
  [review scope:18](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-review-fork-package-ci-before-deployment.md#L18).
  Publisher executes `publisher.py`, `pipeline.py`, and `contract.py` from that
  same revision with a write token. An unreviewed main edit can change those
  programs; successful offline fixtures do not establish trust.
- Minimum first-deployment correction: make publication admission
  **main-only `workflow_dispatch` with `reviewed=true`**, retaining live-SHA and
  ancestry checks. Do not open automatic push publishing until the coordinator
  establishes and records the approved branch/ruleset or protected-environment
  review boundary. No protected-tag route is implemented in this snapshot.
- Script-only deployment with unchanged gitlinks does not match `push.paths`.
  First publication therefore needs three separately reviewed manual dispatches
  on exact live main, not dispatch from `build/gradle-development-experience`.
  Coordinator already created maintenance branches at the pins; do not recreate
  them or mistake branch existence for branch protection.

### B1-2: Mosaic CI can publish unauthorized Build Scans

- Pinned
  [Mosaic settings:1–13](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/settings.gradle#L1)
  applies Develocity 4.5.0. `CI=true` accepts terms and does **not** install the
  non-CI `publishing.onlyIf { false }` restriction.
- Recipe command has no `--no-scan`, and the init script starts at
  `beforeProject`, not before settings. File-only Maven tasks do not prevent
  independent settings-plugin telemetry.
- Evidence: original
  [pipeline:193](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/pipeline.py#L193),
  [snapshot:193–204](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/pipeline.py#L193);
  original
  [init:13](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/publish.init.gradle#L13),
  [snapshot:13](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/publish.init.gradle#L13).
- Minimum correction: supply **`--no-scan` as a startup argument before settings
  executes**, including any other fork-wrapper invocation that evaluates these
  settings. A scoped settings-plugin callback is an alternative, but a late
  project callback is not. `-Dscan.uploadInBackground=false` merely changes
  upload scheduling; it does not prohibit publication.
- Develocity 4.5 documents default publishing and the supported per-invocation
  opt-out. Actual upload was not observed by this reviewer.
  [Develocity 4.5 publication controls](https://docs.develocity.ai/gradle/4.5/gradle-plugin/#controlling_when_a_build_scan_is_published).

### B1-3: Validator assigns tty-owned Native payloads to tty-terminal

- `artifact.startswith("mosaic-tty-")` matches both
  `mosaic-tty-linuxx64` **and** `mosaic-tty-terminal-linuxx64`, and likewise all
  other Native targets. The latter is required to contain its own
  `-cinterop-mosaic.klib` and embedded `native/mosaic.bc`.
- Evidence: original
  [contract:247](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/contract.py#L247),
  reviewed [snapshot:247–252](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/contract.py#L247);
  the overly broad attachment allowance also occurs at
  [snapshot:128–131](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/contract.py#L128).
- Actual owner:
  [tty build:266–293](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/build.gradle#L266)
  registers main cinterop and cklib. In contrast,
  [tty-terminal build:26–32](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty-terminal/build.gradle#L26)
  has a project dependency on tty, but registers neither payload.
- Consequence: a correctly built Mosaic closure is rejected at merge before
  smoke/publication. This is a confirmed ownership error, not an untested
  Windows/compiler hypothesis.
- Minimum correction: derive the base module from the known target suffix and
  require these attachments **only when base == `mosaic-tty`**. Restrict the
  attachment allowlist identically; validate tty-terminal's dependency on tty
  instead. Do not synthesize or copy C payloads into tty-terminal.

### B1-4: First-version SDK JS closure is knowingly missing

- Fixed global `TARGETS` excludes JS, host tasks have no `Js` publication,
  root normalization removes JS redirects, and task fixtures explicitly reject
  `Js`. This is **not unknown** and not a pass under the superseded five-target
  instruction.
- Evidence: original
  [contract:11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/contract.py#L11),
  [snapshot:11–15,92–109](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/contract.py#L11);
  original
  [tests:459](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/test_publication.py#L459),
  [snapshot:459–467](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/test_publication.py#L459).
- Actual `83749434` callers:
  [stdio:2,9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/build.gradle.kts#L2)
  and [HTTP:2,10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/streamable-http/build.gradle.kts#L2)
  apply `kodex.kmp-host` and depend on SDK client in commonMain;
  [host convention:14–23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-host.gradle.kts#L14)
  declares Node JS libraries/tests. These links map the immutable blobs listed
  above, not current worktree reads.
- SDK itself has real JS targets:
  [convention:50–58](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/buildSrc/src/main/kotlin/mcp.multiplatform.gradle.kts#L50).
- Minimum correction is the already approved **SDK-only** JS increment:
  core/client/server real JS payloads and sources, root redirects, dependencies,
  manifest/task/receipt closure and a real API consumer compilation. Preserve
  current Node compilation/tests; do not expand Mosaic/Lucene or add Wasm/iOS.
- **Exact-delta review requested:** coordinator must supply the second immutable
  snapshot, manifest SHA-256 and changed-file list, explicitly including
  contract/task/merge/validator/smoke/receipts/tests/README and any MCP workflow
  changes. This first-byte review does not approve the parallel worker's
  unfinished or subsequently mutable increment.

### B1-5: “Metadata closure” certification is weaker than true variants

- Root validation checks the **set of destination artifact names**, but does
  not require correct JVM/Native attributes on redirect variants or the real
  usage/category/API/runtime/source variant structure. POM checks inspect
  existing same-group dependencies without requiring expected ones to exist.
- External dependency links/versions/variants are not resolved by the offline
  validator. Payload ZIP checks establish integrity/path safety, not that root
  JARs contain KMP metadata or source JARs contain the pinned sources.
- Mosaic JVM check accepts any Native-library suffix anywhere plus any entry
  under `META-INF/versions/22/`; it does not check exact resource roots,
  required architectures, actual FFM classes, or `Multi-Release: true`.
- Evidence: original
  [contract:146](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/contract.py#L146),
  [snapshot:146–150,171–199,215–246](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/contract.py#L146).
  Structural fixture deliberately uses `fixture.txt`, an attribute-free root
  redirect list, and an empty dependency POM:
  [snapshot tests:32–73](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/test_publication.py#L32).
- Minimum correction: validate the actual pinned publication structure, correct
  redirect attributes and essential POM/project edges; check genuine root/source
  payload structure and exact tty library entries. Keep external compiler/variant
  compatibility as an explicit **real resolution/compile gate**, not an inference
  from matching the fork's own klib `compiler_version`.
- Do not claim an actual generated module is corrupt without its bytes. The
  confirmed defect is the validator's overbroad certification; B2/U below
  distinguish concrete counterexamples from still-unrun real outputs.

## Actual project, version and target trace

| Fork | Actual publication projects / immutable version | Declared wrapper / KGP |
| --- | --- | --- |
| Mosaic | `:mosaic-{runtime,animation,testing,terminal,tty-terminal,tty}`; `0.19.0-SNAPSHOT-kodex.7b1a412918f7` | 9.6.1 / 2.3.21 |
| SDK | `:kotlin-sdk-{core,client,server}`; `0.14.0-kodex.53f717661712` | 9.6.1 / 2.4.0 |
| Lucene | `:core`, publication artifact `lucene-kmp-core`; `10.2.0-alpha14-kodex.549f8afc9570` | 9.5.1 / 2.4.0 |

- Pinned module registration, plugins and target declarations support the recipe's
  standard task-name pattern
  `:<project>:publish<Publication>PublicationToForkStagingRepository`.
  First snapshot requests 36 Mosaic, 18 SDK and 6 Lucene tasks; SDK needs three
  additional real JS publications in the approved increment. Task-name generation
  alone does not prove registration/execution on a host.
- Actual coordinate/target sources:
  [Mosaic group/version:1–2](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/gradle.properties#L1),
  [all targets:1–12](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/addAllTargets.gradle#L1);
  [SDK properties:10–11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/gradle.properties#L10),
  [SDK publishing plugin:3–10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/buildSrc/src/main/kotlin/mcp.publishing.gradle.kts#L3);
  [Lucene coordinates:75–82](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/LuceneKmp/build.gradle.kts#L75),
  [Lucene targets:142–161](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/LuceneKmp/build.gradle.kts#L142).
- Mosaic closure is real: animation/testing depend on runtime; runtime depends
  on terminal and tty-terminal; tty-terminal depends on terminal and tty.
  SDK client/server depend on core. Lucene core main dependencies are external;
  codecs/test-framework are test edges, not required production publications.
  Validator should enforce these **per-variant** links, not merely allowed names.
- Source navigation must use genuine source attachments, not just a nonempty
  ZIP or an upstream URL. SDK's inherited
  [Dokka source link:16–19](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/buildSrc/src/main/kotlin/mcp.dokka.gradle.kts#L16)
  points to upstream moving `main`, and its
  [POM SCM:40–44](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/buildSrc/src/main/kotlin/mcp.publishing.gradle.kts#L40)
  remains upstream. These are not pinned-fork provenance. Javadoc is excluded
  by this recipe; do not rewrite unshipped documentation as an incidental fix.
  Verify actual source contents/generated sources and use the manifest's full
  fork commit/tree for authoritative identity.
- Mac owns all root/JVM/macOS publications; Linux owns linuxX64/linuxArm64;
  Windows owns mingwX64. Root, target component URLs, local payload references
  and same-group version links are merged and checked. Mac's root must still
  advertise the full approved target closure with true variants, including SDK
  JS after the delta, not a host-filtered subset.
- This uses original component publications to an isolated **file repository**,
  not fabricated root/native JARs or a new publication owner/cache/provider/
  bundle manager. Kotlin documents separate root/target publications and Apple
  cinterop/final-binary host requirements.
  [Kotlin publication and host requirements](https://kotlinlang.org/docs/multiplatform/multiplatform-publish-lib-setup.html#host-requirements).

### Version timing and generated SDK constant

- Identity reads the pinned upstream property and appends the commit suffix.
  Mosaic's embedded `SNAPSHOT` is preserved; the resulting version does not end
  in `-SNAPSHOT`. No catalog or upstream file is rewritten.
- Build passes **both `-Pversion=...` and `-PVERSION_NAME=...` at startup**:
  [snapshot pipeline:201](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/pipeline.py#L201).
  Mosaic assigns project versions from `VERSION_NAME`; Lucene assigns from the
  version property. SDK uses Gradle project properties, with no later version
  assignment found in its pinned publication/core scripts.
- SDK
  [generateLibVersion:14–30](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/kotlin-sdk-core/build.gradle.kts#L14)
  captures `project.version` when its task is configured. Gradle loads project
  properties before project script evaluation:
  [Gradle 9.6.1 project-property loader](https://raw.githubusercontent.com/gradle/gradle/v9.6.1/subprojects/core/src/main/java/org/gradle/initialization/ProjectPropertySettingBuildLoader.java).
  The current startup `-Pversion` path therefore has a static basis for the
  suffixed constant; it is not merely a late publication-version change.
- Init's later `publication.version = version` and
  `inputs.property("forkPublicationVersion", version)` **cannot themselves
  change a captured constant**. Dropping the startup property or replacing it
  with only a catalog/late-publication update would regress identity.
- Generated source and compiled `LIB_VERSION` must still be inspected and
  asserted equal to the exact publication version in a real consumer. Current
  smoke checks async stdio/server behavior, not the generated constant.
- Original parent source `help` is a **known FAIL**, duplicate
  `RootProjectAccessor.getKotlinSdk()`:
  [main-plan evidence:101](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-02-rescue-gradle-development-experience.md#L101).
  Corrected-init/root-name control remains pending. SDK standalone settings do
  not enable type-safe accessors in the inspected pin; do not automatically
  transfer the composite failure to standalone CI. If it reproduces there, use
  only the approved `kotlin-mcp-sdk-fork` recipe correction, with provenance and
  an isolated control; never silently rewrite the upstream fork/settings.

## B2 — specific reproducer/check recipes, not executed

- **Cinterop ownership:** coordinator feeds a real Mosaic closure with ordinary
  tty-terminal Native klibs and tty-only cinterop/C bitcode into `validate()`.
  First snapshot rejects tty-terminal at contract:248. After the narrow fix,
  ordinary tty-terminal must pass; deleting tty's cinterop/bitcode must fail.
- **False root variants:** take the existing Lucene structural fixture, preserve
  all five root redirect destinations, but label all redirect attributes as
  mingwX64 or remove their platform/usage attributes. First validator does not
  inspect them; corrected validation must reject the wrong/missing attributes.
- **Missing graph/POM edge:** use actual SDK client/root metadata and delete the
  core dependency or the generated root-to-JVM POM forwarding edge while
  retaining valid file hashes. Checking only dependencies that remain misses
  this. Corrected validation must reject the missing expected edge; JVM Maven
  and KMP Gradle consumers must resolve the intended dependency graph.
- **False JNI ZIP:** move a required tty DLL/SO/DYLIB out of its loader path,
  leaving one arbitrary `.so` entry and one arbitrary version-22 entry; refresh
  module hashes. The first ZIP predicate is insufficient. Require exact roots
  from the pinned Zig build and test real loading.
- **Version timing:** coordinator's isolated control compares startup
  `-Pversion` with late-publication-only configuration, records project version,
  generated `LibVersion.kt`, sources attachment and runtime `LIB_VERSION`.
  Expected distinction follows the pinned capture site; reviewer ran neither.
- These are concrete scoped checks for the coordinator, **not reviewer-produced
  execution results**. The snapshot's 562-line test file and 2,540 total
  recipe/workflow/document lines do not prove real Native branches ran.
  Workflow fixtures are text assertions, not GitHub YAML/action validation;
  HTTP fixtures model chosen responses, not universal registry return codes.

## U — exact platform, registry and provenance gates still open

### U-1: Real three-host artifact production and dependency compatibility

- Labels are standard runners: `ubuntu-24.04` x64, `macos-14` Arm64,
  `windows-2022` x64. Mac build explicitly checks Arm64. Public standard runners
  do not imply unlimited artifact/package storage; Mac's documented RAM/disk
  budget is especially relevant.
  [GitHub runner specifications](https://docs.github.com/en/actions/reference/runners/github-hosted-runners#standard-github-hosted-runners-for-public-repositories).
- No host tasks ran here. Require executed task graph and original wrapper,
  actual KGP codeSource/version, Kotlin/Native compiler version and klib
  manifests, JDK 25 Gradle/JDK 21 SDK toolchain evidence for **each host**.
  Init currently records KGP by catalog regex and pipeline assigns
  `nativeCompiler = kgp`; these are declared values, not measured compiler
  provenance:
  [snapshot init:61–75](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/publish.init.gradle#L61),
  [pipeline:210–223](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/pipeline.py#L210).
- Real resolution must establish compatibility of **all external dependency
  links** and the actual Kodex 2.4.0 consumer with Mosaic 2.3.21 and SDK/Lucene
  2.4.0 publications. Own-klib compiler equality is not transitive compatibility.
  The later 2.4.20/9.6.1 migration is a separate gated batch.
- Ubuntu must actually build linuxArm64 and compile its binary consumer.
  Current smoke records it as `compileOnly`, correctly not Arm64 runtime.
  Linux x64, Mac Arm64 and Windows x64 require their own linked/running probes.
- Lucene must retain real generated BreakIterator/GB2312 sources and AGP
  configuration with SDK36/build-tools36 on each host:
  [core build:21–22,76–88](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/LuceneKmp/core/build.gradle.kts#L21),
  [snapshot SDK setup:139–158](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/pipeline.py#L139).
  Standard image availability/licenses/time/disk were not validated.

### U-2: Mosaic JNI, FFM, Apple SDK and Windows main cinterop

- Actual task chain includes Java-generated JNI headers → `jvmJniZigBuild`
  (Zig 0.15.1) → JVM resources; jextract bindings/JDK22 compilation → multi-release
  JAR; main cinterop plus cklib C bitcode → Native publications.
  [tty build:74–131,266–293](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/build.gradle#L74).
- Exact JNI JAR root is `com/jakewharton/mosaic/tty/jni/`, not the JAR root.
  Pinned Zig builds SO for `aarch64`, `riscv64`, `amd64`; DYLIB for `aarch64`,
  `x86_64`; DLL for `aarch64`, `amd64`.
  [Zig destinations:14–20](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/build.zig#L14).
  Loader uses `Jni.class.getResource("jni/<os.arch>/<library>")`:
  [NativeLibrary:24–49](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/src/jvmMain/java/com/jakewharton/mosaic/tty/NativeLibrary.java#L24).
- Smoke runs Java 25 only. Its version-22 `TestTerminal` implementation calls
  `Libmosaic` FFM, while the ordinary JVM implementation calls `Jni`.
  [FFM caller:3](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/src/jvmJdk22/kotlin/com/jakewharton/mosaic/tty/TestTerminal.kt#L3),
  [JNI caller:3](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/src/jvmMain/kotlin/com/jakewharton/mosaic/tty/TestTerminal.kt#L3).
  A Java25 pass does **not** exercise the pre-JDK22 JNI branch. Minimum real
  acceptance: retain Java25 FFM probe and add a Java21 JNI execution, with
  truthful launcher/receipt evidence, on the required hosts.
- Mac must prove selected Kotlin/Native Apple SDK and the isolated JNI
  `FORK_DEVELOPER_DIR`/CommandLineTools SDK work; no `xcode-select` mutation.
  Windows must actually run `cinteropMosaicMingwX64` with the main
  `mosaic.h`/included headers, compile C bitcode and link the main consumer.
  Do not substitute test cinterop or a metadata-only success.
- Windows probe has a specific console dependency: native C opens `CONIN$`
  and `CONOUT$`; current code can fail if the runner process has no console.
  [Windows test terminal:47–77](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-tty/src/commonMain/c/mosaic-test-windows.c#L47).
  This is a concrete host check, not a confirmed runner failure or permission
  to skip the probe/change product source.

### U-3: Real GitHub Packages HTTP behavior and repeatability

- Remote reader/preflight, payload PUTs, generated sidecars, classifier manifest,
  and exact-byte reread have not run against GitHub here. The documented
  authentication split is correct: classic read-only PAT for developers;
  repository `GITHUB_TOKEN` for publishing associated packages.
  [GitHub Gradle registry authentication](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-gradle-registry#authenticating-to-github-packages).
- `NoRedirect` fails closed for redirects without leaking Basic credentials.
  A real GET/PUT can encounter registry-specific redirects; this code will fail,
  not secretly follow them. Coordinator must record endpoint/status behavior
  without secrets; any later safe redirect handling needs its own review.
  Neither official generic registry docs nor the loopback fixture establish
  universal 404/409/201 behavior or `If-None-Match` enforcement.
- Sidecars are exact lowercase digest bytes, **MD5 is an actual MD5 digest**,
  not another algorithm with a misleading extension. Registry-generated
  sidecars are skipped only when byte-identical; whitespace/case/format
  differences currently fail. Inspect actual behavior before trusting a skip.
  [snapshot publisher:57–71,128–143](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/publisher.py#L57).
- `classify()` enumerates expected paths, not the entire remote namespace.
  GET-before-PUT is not a remote atomic lock; an external writer can race it if
  the registry ignores conditional PUT. Central concurrency reduces known CI
  races, not all remote writers.
- Complete-exact skip requires byte-identical rebuild manifest/payload/sidecars.
  Floating setup-java patch versions, changing recipe bytes and compiler
  reproducibility can make a later same-commit rebuild differ. Require a
  repeat-run control and retained validated bundle; never “repair” by overwrite.
- Manifest records fork identity, full tree, coordinates, scripts/own-workflow
  hashes, host tasks and files. Publisher endpoint identifies `Stream29/Kodex`
  ownership, but the build manifest does not explicitly declare the publication
  repository/owner. Add/enforce that small provenance field if required by the
  [Packages contract:52–53](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-04-plan-github-packages-fork-publication.md#L52).
  Actual compiler/toolchain provenance remains U-1, not “verified” by a catalog.

## Security, source isolation and failure semantics

- Static permission separation is sound **subject to B1-1 admission**:
  PR has only offline tests, no `pull_request_target`/`workflow_run` writer;
  build/merge/smoke jobs have no package-write grant; only publisher has
  `packages: write`, no submodule checkout, Gradle or fork/native execution.
  [snapshot Mosaic publish:215–249](file:///tmp/kodex-fork-ci-first-review-20261007/.github/workflows/fork-packages-mosaic.yml#L215).
- `GITHUB_TOKEN` is supplied only to publisher's final step, not matrix code,
  generated consumer files, Gradle homes, heredocs, source tar or built files.
  Basic-auth errors omit response bodies/credential-bearing exception text.
  No reviewer read user credentials. The four configured developer machines
  and read-only PAT remain coordinator-owned; this review adds none.
- Actions use mutable major tags (`@v4`/`@v5`), not immutable action SHAs.
  Recipe/workflow hashes bind the tag text, not resolved action source. Include
  action provenance in coordinator deployment review; immutable SHA pinning
  is the documented hardening option, particularly for the write-permission job.
  [GitHub action supply-chain guidance](https://docs.github.com/en/actions/reference/security/secure-use#using-third-party-actions).
- Public recursive checkout uses no SSH key and `persist-credentials:false`.
  Identity reintroduces a **command-local** correct
  `url.https://github.com/.insteadOf=git@github.com:` bridge after checkout
  removes temporary auth/rewrite configuration, then fetches ancestry over an
  explicit HTTPS URL. No global Git config mutation is needed.
  [snapshot pipeline:37–61](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/pipeline.py#L37),
  [checkout v4 auth implementation](https://raw.githubusercontent.com/actions/checkout/v4/src/git-auth-helper.ts).
- Workflow `GIT_CONFIG_COUNT=1` correctly supplies index 0 `core.autocrlf=false`;
  the command-local `-c` adds the URL bridge without discarding that setting.
  If another setting is added via environment, append at the current count and
  increment it; never overwrite index 0/count and lose CRLF control.
  [Git runtime configuration semantics](https://git-scm.com/docs/git-config/2.48.1#Documentation/git-config.txt-GITCONFIGCOUNT).
- Build requires clean pinned fork/nested sources before assembly, uses
  `git archive <commit>` and an isolated extraction/output directory, and fails
  on unsupported nested-source archives. Safe in-tree source document links
  are allowed through the data extraction filter; staged files/payload ZIPs
  reject symlinks, traversal and nonregular files. This is not a sandbox for
  untrusted build code; reviewed source and publisher admission remain essential.
- Builds modify only fresh archived copies; init changes publication/task
  configuration, not upstream source or dirty pins. Private Gradle/Konan homes
  and Zig cache are isolated. Successful staging deletes source/raw stage;
  failures can retain runner-temporary files, with daemon stop attempted in
  `finally`. No user device/resources should be reclaimed for those checks.
- Binary smoke uses an exclusive file repository for the fork group, no
  `includeBuild`, substitution, `mavenLocal` or hidden missing-package fallback:
  [snapshot smoke:76–105](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/smoke.py#L76).
- Three workflows have separate concurrency/artifact/receipt namespaces;
  coordinates/groups are disjoint. Local stages drop rolling metadata/checksums;
  publisher regenerates sidecars under each coordinate/version path. There is
  no shared fork checksum filename or three-fork graph/bundle writer.
- Existing complete-exact state skips only after byte verification. Existing
  partial/different state **fails before writes**. Fresh writes recheck each
  path; present differing bytes cause an error, never intentional overwrite.
  Payloads precede POM/module visibility, then manifest; final acceptance reads
  every expected file. `remoteVerified=false` in the immutable build manifest
  is honest, not a completion marker.
- A cancellation/stale main/transport/verification failure can leave a partial
  version. The writer honestly reports that possibility and has no delete,
  rollback, resume or administrative cleanup capability:
  [snapshot publisher:93–149](file:///tmp/kodex-fork-ci-first-review-20261007/scripts/fork-packages/publisher.py#L93).
  Ordering metadata last is not a Maven transaction.
- This is safer than fabricating the parent plan's
  [automatic cleanup/retry requirement:77](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-04-plan-github-packages-fork-publication.md#L77),
  but does **not fulfill it**. Coordinator must explicitly reconcile that
  acceptance contract: keep the consumer pin unchanged, retain evidence, then
  obtain separate approval/permissions for guarded removal or a new reviewed
  fork commit/version. Do not grant admin deletion, resume, or overwrite
  authority as an incidental CI fix.

## Minimum ablation and deployment decision

- Keep the actual responsibilities: standard Gradle file publication, explicit
  build/merge validator, tiny binary consumers, isolated HTTP writer and focused
  offline checks. None requires another owner/cache/provider/bundle abstraction.
  A blanket deletion of a module is not justified merely by line count.
- Concrete ablation: narrow the two tty prefix predicates (B1-3), removing the
  nonexistent tty-terminal C-payload responsibility. Once startup-version
  assertions are added, remove init's redundant `generateLibVersion` input-only
  hook: it cannot update the captured constant and fresh builds disable cache.
- Remove/rename unmeasured `nativeCompiler = kgp` and hardcoded Zig/cklib/jextract
  “evidence” rather than treating declared strings as execution results; retain
  pinned input hashes and collect actual versions only where needed.
- Replace weak generic success claims with a small number of actual KMP
  metadata/payload negative cases and host receipts. Do not grow the fixture
  suite or dynamically rewrite project/target models to manufacture green CI.
  These are proposals only; reviewer changed no implementation.
- **Deployment decision: HOLD.** Minimum correction set is trusted admission,
  pre-settings no-scan, exact tty ownership, approved SDK JS delta and truthful
  variant/payload certification; then exact-byte delta review and the named
  real host/provenance/registry checks.
- Only after complete remote byte verification and separate default-consumer
  acceptance may coordinator change package pins. Isolated smoke does not
  validate the full 203-project model, genuine IDEA Sync/navigation gains,
  root accessor fix, full regressions, or Linux Arm64 runtime.
- **REVIEW READY / FIRST SNAPSHOT NOT DEPLOYMENT READY.**
  Coordinator should read this report asynchronously before committing or
  deploying, and return the precise revised snapshot/manifest for scoped review.
