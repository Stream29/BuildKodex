# Task Tree

- `Read the approved publication contract and freeze all three gitlinks`()
- `Implement three independent fork CI workflows and the publication recipe`()
- `Complete coordinator validation of manifests, host closure and failure semantics`()
- `Hand off exact files, commands and remaining host gates to the coordinator`()

# Details

- Parent: [GitHub Packages production implementation](2026-10-04-plan-github-packages-fork-publication.md).
- Scoped completion: all three immutable forks are built, consumer-tested and
  remotely byte-verified. Mosaic's successful reviewed run at `49d78d99`
  verified 803 files, manifest
  `c3a84f667256ec29ce70f49d873fbf71511e9417f8209c7cf83bcbfdaa53f88c`.
  SDK/Lucene original three-host bundles were separately read-audited at
  453/453 and 133/133, with their historical failed writer statuses preserved.
  Normal default consumption is the
  [separate integration task](2026-10-07-consume-verified-fork-packages-by-default.md).
  Earlier pending handoffs below are historical, not the current CI verdict.
- Code baseline: Kodex v0.4.10 / `83749434`, clean worktree.
  Mosaic `7b1a412918f7bc24db847343480f010242939788`,
  LuceneKmp `549f8afc9570bd4289f1047653d4cd6dc9665bbd`,
  KotlinMcpSdk `53f717661712d618cd4822d1760392e903989ead`.
- Owner scope: Kodex `.github/workflows/` three fork publication workflows,
  `scripts/fork-packages/` recipe, validator and tests. Read pinned fork sources
  and old isolated publication evidence; do not change fork branches or gitlinks.
- Coordinator alone owns settings/catalog/consumer, credentials, all Git operations,
  actual remote CI triggers and heavy host validation. No worker launches Gradle,
  local CLI, IDE, process controls or network publication.
- Preserve upstream coordinates; immutable upstream-version plus `-kodex.<short-sha>`,
  manifest full commit/tree, recipe/toolchains/targets and file checksums.
  Build JVM/linuxX64/linuxArm64/macosArm64/mingwX64.
  **User-confirmed current-source exception: MCP SDK must additionally publish JS.**
  Its real stdio/HTTP Node consumers depend on SDK JS; no Wasm/iOS or fake variants.
  This updates the earlier five-target-only instruction for SDK, not other forks.
- Matrix outputs must merge deterministically and validate before a single remote
  writer; do not rewrite an existing complete version or silently accept partial
  coordinates. Publication failure leaves the consumer unchanged.
- Only main gitlink changes and reviewed manual dispatch may publish; no PR package
  write permission. Main is currently unprotected: gate exact main identity and
  record whether protection is configured rather than assuming it exists.
- CI uses standard runners, bounded concurrency/timeouts/artifact retention,
  repository `GITHUB_TOKEN`, recursive public submodules and explicit ancestry checks.
- Do not publish Develocity/Build Scan telemetry merely because `CI=true`;
  disable scan publication in the isolated fork recipe. Preserve token secrecy
  and capture locally auditable logs/artifacts instead.
- Reuse working native Maven publication from the previous research as evidence,
  not copied package bytes. Every current artifact must come from its pinned source.
- Current pristine composite baseline reproduces SDK `getKotlinSdk()` duplicate
  accessor. Audit the standalone publisher too; use the already approved
  `kotlin-mcp-sdk-fork` root-name correction in the explicit publication recipe
  if required, with its recipe/provenance recorded. Do not treat this known
  accessor failure as a missing package or silently modify upstream branches.

## Integration handoff

- Coordinator validation checkpoint: Xiaoxin fixed first snapshot passed 34
  offline tests. Actual actionlint v1.7.12 rejected 12 uses of `runner.temp`
  in job-level `env`; those variables must move to step-level environment.
  This is a real workflow-admission failure, not a Native build result.
- Coordinator owns the final workflow context/trusted-push correction and
  Mosaic scan-disable delta after the SDK handoff. The SDK worker keeps its
  JS closure changes only; immutable review snapshots remain unchanged.
- The first independent review is READY but HOLD. Coordinator has corrected
  runtime-only environment placement and rejects unprotected automatic push
  in both YAML and Python admission. Main remains unprotected; first publication
  therefore requires explicitly reviewed manual dispatch. No repository rules
  or user branch policy were silently changed.
- `--no-scan` is now supplied at Gradle startup, including binary smoke.
  Startup `-Pversion` remains the source of SDK generated version; the redundant
  task-input-only hook is removed. Actual compiler archives and KMP plugin JAR
  hash are collected; declared Zig/plugin versions are labeled as declarations.
  Package owner is explicit in the manifest.
- Confirmed tty ownership and truthful root/target/payload checks are assigned
  to [focused validator correction](2026-10-07-close-fork-publication-validator-review.md).
  Publication is still held pending its handoff, revised offline/actionlint
  checks, exact-byte independent delta review and real host CI.
- User has explicitly reconciled the old partial-publication cleanup promise:
  fail sealed, keep old consumer pin, no automatic deletion/overwrite/resume.
  A later remediation needs scoped audit and authorization. The canonical
  Packages task now matches the implemented writer.
- Revised actionlint is clean. The combined offline run exercised 65 cases:
  64 passed, one old task-list expectation needed the newly required Java21
  JNI gate. That expectation is updated without removing its matrix assertion;
  final rerun is pending. No Native or registry publication result is inferred.
- Xiaoxin producer preflight stopped at the exact-main Git HTTPS read timeout
  before Gradle or publication. Its seeded private cache is not a cold-speed
  measurement. Authenticated pinned SDK Maven POM returned 404, not a published
  package. GitHub API returned the correct baseline SHA; neither substitutes
  for the producer's ancestry gate.
- The first combined fixed snapshot passed 65 offline cases and actionlint;
  independent recheck found only the real root tooling JSON rejection.
  Coordinator reproduced the attachment in the old real producer inventory
  and fixed both the filename allowlist and non-component classification.
  The JSON is preserved and sealed, not misadvertised as a target variant.
  Final 67-case offline run passed; exact narrow delta review is next.
- Old real Mosaic output passes the current validator (160 files); this is a
  producer-shape check, not a current-version build. Lucene's old unnormalized
  root redirects are being checked through the exact current normalization.
  AppleDouble files are removed only from the coordinator's private diagnostic
  copy, never from original frozen research or source evidence.
- Controlled CI deployed at `2fe390800e421d72f9771bf4f9db58d763cd0d6b`;
  all three reviewed manual runs completed without entering the writer.
  [MCP 37603775353](https://github.com/Stream29/Kodex/actions/runs/37603775353)
  built all three host stages and merged them, but real Gradle consumption
  rejected normalized metadata: `formatVersion` must be the first JSON field.
  [Lucene 37603786792](https://github.com/Stream29/Kodex/actions/runs/37603786792)
  failed during Vanniktech plugin application because publication version was
  finalized before coordinates configuration.
  [Mosaic 37603763342](https://github.com/Stream29/Kodex/actions/runs/37603763342)
  failed in all host cklib Native tasks with `InvocationTargetException`.
  Coordinator is fixing the confirmed wire/configuration defects and collecting
  the Native cause in an isolated Xiaoxin build. No default dependency switched,
  no package was published, and no model/IDE speed acceptance is claimed.
- Second run at `13b1aab21ebccdef445f70887a80f99cd6053c4c` includes
  streaming-metadata ordering and delayed staging setup; 68 offline cases passed
  on Xiaoxin before deployment. Lucene still failed, so deferred repository
  configuration alone is **not** credited as resolving its error.
  Its real stack points to the fork's `coordinates()` callback; unlike the old
  successful Lucene recipe, the new command also injected `VERSION_NAME`.
  That property is being removed **only for Lucene**, leaving its real startup
  `version` and original coordinates callback authoritative.
- Mosaic's captured Linux stack identifies the missing `dependenciesUrl`
  property: cklib 0.3.5 constructs its distribution from hardcoded
  `user.home/.konan`, while KGP installed the actual compiler in the private
  `KONAN_DATA_DIR`. Bind cklib's existing `konanHome` to that distribution and
  require the original KGP download task before C compilation.
  These are recipe corrections, not fork source changes or reused user caches.
  Real three-host reruns remain mandatory before publishing.

- **IMPLEMENTATION READY / VALIDATION PENDING** — Session 550.
- Third deployed revision `dc3b258d52d935b13081d6859401b80fea72d1bf`:
  [Mosaic 37607930083](https://github.com/Stream29/Kodex/actions/runs/37607930083),
  [MCP 37607943182](https://github.com/Stream29/Kodex/actions/runs/37607943182),
  [Lucene 37607948319](https://github.com/Stream29/Kodex/actions/runs/37607948319).
  All nine real producer stages and all three merges succeeded. MCP/Lucene
  three-host binary smoke passed, including MCP's real Node execution.
  Mosaic's JNI Java21 probe failed on Java25 bytecode after JVM25/Native gates;
  compile toolchain21 plus explicit JVM25 FFM launcher is the narrow correction.
- MCP/Lucene writer jobs failed, so rollout was paused rather than resumed.
  Real registry GETs redirect to GitHub's official storage host; refusing every
  redirect prevented truthful final verification. The coordinator's read-only
  SDK audit against the original dc3 bundle found **453/453 files byte-exact**,
  with no missing/different/read-error files, including manifest and sidecars.
  Its failed CI status remains historical evidence; no repair upload was needed.
  Lucene's subsequent audit likewise found **133/133 files byte-exact**, with
  no missing/different/read-error files. Both audits used the old dc3 recipe
  to verify the original sealed bundle and only replaced its GET transport;
  they performed no PUT/DELETE. Evidence is in Xiaoxin's private
  `~/ACodeSpace/demo/kodex-gradle-rollout-20261007/registry-audit/`
  (`mcp-full-read.json`, `lucene-full-read.json`).
  Both versions are complete as audited; no cleanup/resume is required.
  Default consumer pins remain unchanged until Mosaic passes its corrected gate.
- [Independent GET/Java21 increment review](2026-10-07-recheck-registry-read-and-java21-probe.md).
  Revised reads strip Authorization after the original Maven request, allow only
  the exact HTTPS storage host, bound hops, reject PUT redirects and treat storage
  404 as read failure rather than absent publication. No signed URL is logged.
  Final revised bytes passed 74 offline cases on Xiaoxin (6.706s), including
  credential stripping, unsafe redirect/PUT/hop refusal, storage 404 scope,
  sanitized operation diagnostics and generated Java21/JVM25 probe settings.
- Only owned paths changed: three Kodex `.github/workflows/fork-packages-*.yml`
  files, `scripts/fork-packages/`, and this child handoff. No worker tests,
  Gradle/IDE, process controls, publication, commits/push, branches or gitlinks.
- [Exact CLI/task sets, runner/toolchain gates, immutable failure semantics and
  coordinator commands](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/README.md#L1).
- Independent workflows: each main push watches only its own gitlink; reviewed
  dispatch is main-only and exact live HEAD. PRs run only offline fixtures.
  Only the HTTP publisher job has `packages: write`; it runs no Gradle/fork code.
- Canonical root/JVM/macOS publications come from macOS; Linux contributes
  linuxX64/linuxArm64, Windows mingwX64. Real KMP/Native payloads, sources and
  transitive closure are validated before three-host binary smoke and publication.
- Publication version preserves the literal upstream property. In particular,
  Mosaic is `0.19.0-SNAPSHOT-kodex.7b1a412918f7`, not a stripped upstream version.
  MCP is `0.14.0-kodex.53f717661712`; Lucene
  `10.2.0-alpha14-kodex.549f8afc9570`.
- Offline tests cover metadata/ZIP/path/symlink corruption, merge identity and
  host collision gates, workflow permission boundaries, public ancestry fetch,
  literal versioning, and a loopback Basic-auth HTTP publisher with partial,
  duplicate, redirect, stale-HEAD, interruption and remote-corruption fixtures.
  **Written and manually reviewed, not executed by this worker.**
- Coordinator remote command:
  `PYTHONDONTWRITEBYTECODE=1 python -m unittest discover -s scripts/fork-packages -p 'test_*.py' -v`.
- Remaining concrete gates: real CI YAML/action validation; original fork wrapper
  and KGP across standard runners; SDK36 setup for Lucene; Mosaic JNI/FFM,
  Apple SDK and Windows main cinterop/console behavior; daemon/disk budgets;
  remote token permission/direct Maven HTTP behavior and exact-byte re-read;
  default Kodex consumer acceptance. Linux Arm64 runtime is not claimed.
- Current deployment gates supersede the earlier implementation-only pending
  list: YAML and real three-host producer/consumer gates have run. MCP/Lucene
  are complete as separately audited. Mosaic's remaining Windows Java21 JNI
  failure is handled by the user-approved
  [fork return-value repair](2026-10-07-fix-mosaic-windows-resize-package-gate.md),
  not by removing a gate. Historical runs and their failures remain evidence.
- Maintenance branch creation, review/protection policy, deployment/dispatch,
  credentials, Git operations and consumer version updates remain coordinator-owned.
  Existing partial/different versions fail without deletion or overwrite.
  No Maven atomic transaction or manifest-based completion endpoint is invented.
