# Task Tree

- `Prepare pinned upstream KGP source and verify its actual build route`() // complete; real dry-run succeeds
- `Build full unchanged upstream control with fixed official recipe`() // warm completion, failed controls retained
- `Implement minimal project-owned same-file metadata reuse with regressions`() // fixed V2 deployed after sealing control
- `Build reproducible isolated plugin artifacts without modifying installed JARs`() // complete upstream warm build and ten regressions pass
- `Compare metadata, full model and representative source attachments`() // equivalent; ten upstream regressions pass
- `Measure complete import and CInterop graph lifetime at the original budget`() // combination import passes; official controls OOM
- `Validate representative functions and trace the remaining source-navigation failure`() // 353 checks and CLI pass; Json navigation open
- **`Retain isolated evidence without adopting the rejected KGP maintenance route`()** // suspended; user decision

# Details

- Latest decision: the user rejects modifying or maintaining KGP. The completed
  source builds and controls remain research evidence, not an adoption route.
  No further KGP integration is scheduled; project-owned test configuration
  is handled in [a separate batch](2026-10-11-adopt-project-owned-gradle-test-configuration.md).
- Parent: [complete import repair](2026-10-08-repair-full-model-idea-import-oom.md).
- User explicitly authorizes **isolated KGP source patch/build experiments**.
  No remote fork creation, package publishing, production version change,
  commits or pushes are authorized by this confirmation.
- Upstream input is the actual accepted KGP2.4.0 implementation. Pin the
  upstream tag's full commit/tree and record the source/build recipe.
- Preserve all targets, source-set ancestry, cinterop, model/source navigation,
  public ABI and existing tests. Gradle import budget stays4GiB.
- Run heavy work only on Xiaoxin under the existing heavy lock; detect daemon
  JVMs before building. No concurrent GUI/MAT/build or local-machine execution.
- First verify upstream build requirements and bounded disk/resource cost.
  Report prerequisite failure rather than substituting a hand-edited installed
  JAR, undeclared classpath patch, reflection cache or copied model facade.
- Source-proven opportunity: the current factory is recreated per call and
  reads/parses the same selected metadata file per transformation. Reuse must
  be owned by the original consuming Project and preserve actual selected-file
  checks, file-content changes, errors and safe concurrent access.
- Do not share visibility decisions, selected component/provider graphs or
  consumer-specific state. Neither39k metadata objects nor the full817MB
  source-set group is an assumed recoverable saving.
- Required regressions include repeated equal selected-file reads, different
  content at the same path, independent projects/files, malformed input and
  concurrent readers. Compare all typed fields and ordered collection semantics.
- The finite preparation/build experiment may fail; parent OOM task stays
  open until actual full GUI/model/navigation acceptance.
- Warm R12 completes the Gradle action in2m48s and publishes625-module smart
  snapshots, but the original host-memory guard stops it before any external
  success callback. No Java OOM is captured; this is not repeatable import
  acceptance. Peak sampled RSS is about4.12GiB IDEA /4.74GiB Gradle, not live
  heap or unique physical-memory attribution.
- R13 repeats only to diagnose that distinct boundary. A passive `jstat -gc`
  sampler records numeric counters for verified owned JVMs; it does not request
  GC, change heap limits or weaken the safety guard. R12 duplicate caches are
  released after verified cleanup; R09 remains the shared warm control seed.
- R13 finishes the Gradle action in3m1s, but again stops at the host-memory
  guard without external success. Numeric sampling records a natural Full GC:
  subsequent old-generation usage remains about3.68GiB. That is stronger
  evidence of remaining occupancy than RSS, but not a retained-owner proof.
- R14 attempts a private `GC.heap_dump -all` after actual build completion.
  Cleanup interrupts the capture: only26MiB and no dump-completion marker,
  despite the attaching command returning0. Treat this as an incomplete
  diagnostic, not a usable heap or import success. R15 captures earlier while
  old-generation usage crosses2.2GiB, with the same guard and no requested GC.
- Completed R10/R11 private heaps were losslessly gzipped, read back and
  SHA-verified before removing their uncompressed duplicates. Mode0600,
  evidence and original-byte checksums remain; no private heap is uploaded.
- R15 obtains a complete4.86GB private mid-import heap with the JVM's dump
  completion marker, without requested GC. Its capture pauses the JVM and is
  therefore diagnostic-only, not an import-timing acceptance run. The original
  safety guard later stops the GUI. MAT parsing succeeds after process cleanup.
- At this earlier capture phase, actual factories are203 (one per Project),
  retaining about212MB; the source-set group retains about33MB. Remaining large
  groups are legacy configurations (~700MB) and their containers (~234MB).
  The older unmodified heap was captured at OOM, so these numbers are not a
  phase-aligned savings benchmark. They identify the remaining owner to inspect;
  a controlled query of configuration names is next.
- The name query identifies203 instances each of `nativeTestCInterop`,
  `linuxTestCInterop`, `nativeMainCInterop` and `linuxMainCInterop`, with summed
  per-object retained sizes about204/198/152/148MB respectively. These sums are
  not assumed disjoint. The original commonized-CInterop configuration resolves
  the whole dependency graph with a CInterop-only usage, even for published
  libraries lacking that variant; its view deliberately tolerates failure.
- Next diagnostic compares the original files and task dependencies with a
  variant-reselection view over the already-required metadata graph, filtered
  to actual Project components. Published CInterop metadata remains handled by
  the unchanged metadata-transformation path. First compare real consumers,
  including native resource modules and app/RPC; do not patch the producer until
  those boundaries and custom transitive/variant regressions are understood.

# Evidence

- [Exact parser/retention investigation](../done/2026-10-09-investigate-repeated-project-structure-metadata-retention.md).
- [Official KGP build](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/build.gradle.kts).
- [Official factory](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/KotlinProjectStructureMetadataExtractorFactory.kt).

## Source/build preparation

- Official tag commit `add726ca8c82922b6ab4cb2a27ae738d6a780817`;
  tree `e94052d76d53e57f3dc6592f89a245657d0906e4`.
- HTTPS Git clone fails before checkout because GitHub port443 is unreachable.
  Official commit API archive succeeds:47MiB,
  SHA256 `544efd79a12a591f1cfdf206a782e7c721de69349d49ab7577305661c3672f60`.
  Safe isolated extraction preserves130,349 entries under
  `~/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/source/JetBrains-kotlin-add726c`.
- Read upstream root/area instructions. No IDEA/MCP project exists for this
  new isolated source; no shared/user IDE is opened or operated. File reads
  and remote validation are the available experiment capabilities.
- Official wrapper9.4.1 has its recorded checksum; no replacement wrapper.
  Initial build-route probe requests Java25 explicitly,4GiB and one worker
  under the heavy lock, no other daemon initially available. It requests the real
  `:kotlin-gradle-plugin:embeddableGradle813Jar --dry-run`.
  This configures build logic; it does not compile or validate the patch.
- Disable only upstream's internal automatic personal-property setup and build
  scans; no compiler warning/test/verification policy override.
- Three-file source overlay is prepared separately, not yet copied into the
  active build-route probe. Original factory/extractor match the accepted
  published source. Six functional regressions cover the planned reuse contract.
- [Independent fixed-overlay review](../done/2026-10-09-review-kgp-metadata-reuse-source-overlay.md)
  runs without competing heavy execution. Actual build/functional/GUI results
  remain pending; upstream bootstrap/toolchain dependencies are not replaced
  with hand-edited binaries if they fail.
- Runtime observation catches an upstream daemon criterion: the official
  `gradle-daemon-jvm.properties` requires Java21 and takes precedence over the
  supplied Java25 home. The actual newly owned daemon is Java21/4GiB, not the
  requested25. Stop it by verified executable/start-time/private-home identity.
  R01 ends with an explicitly aborted route, not a patch/compiler failure.
  Its raw receipt records requested JVM25; this observation corrects that
  limited field rather than silently claiming actual25 execution.
- R02 explicitly supplies the available installed Java21, matches the unchanged
  upstream criterion, and checks the live daemon executable throughout.
  Reuses only the experiment's downloaded bootstrap cache; route remains
  dry-run/pending. No product Gradle JVM or resource default changes.
- R02 fails its actual official prerequisite after185.629s: task graph
  calculation requires Java8 for `:atomicfu:compileJava`. Foojay's automatic
  download cannot reach GitHub's release URL. This is not a patch compile
  failure; source overlay has not been installed. Owned JVMs are cleaned.
- Prepare only the required Java8 under the private experiment, using official
  Adoptium release asset8u504-b01 and its published digest; no SDKMAN/default
  JDK or product build setting change.
- First independent review confirms one test-oracle B1: the fixture omitted
  empty map entries that the unchanged parser normalizes. Revised isolated
  overlay V2 completes those entries, uses an actual serialized GAV for the
  same-length content change, and preserves the original unary JVM constructor
  descriptor with `@JvmOverloads`. Factory logic is byte-unchanged.
- V2 also compares serialization order and adds the original0.3.1 format
  fixture. Seven regressions are prepared, **not executed**. Current fixed
  overlay is `/tmp/kodex-kgp-metadata-source-overlay-v2-20261009`; the first
  reviewed overlay/report remains unchanged as historical evidence.
- [Independent V2 delta review](../done/2026-10-09-recheck-kgp-metadata-reuse-regression-delta.md)
  closes the fixture B1 statically; no new confirmed delta blocker.
  Runtime selector, configuration-cache, emitted ABI and full GUI gates remain.
- R03 passes the Java8 prerequisite but fails before compilation because
  official stdlib compilation requires Java11, and automatic GitHub download
  again cannot connect. Actual route duration334.635s; owned JVMs cleaned.
- Required private Java11 and upstream-used Java17 are official Adoptium
  artifacts, verified against release API digests after bounded range downloads:
  `5c3f68887c325d36d852ba534303e1f5f1f5cae7d6cc1e951d73e0d8e98a058d`
  and `3808d1d15e3ec6bd5b84057fb5d84c33d8a1536a258146bcea2e603fc726e08e`.
  A slow incomplete first transfer is not installed or counted verified.
- R04 explicitly supplies those private toolchains through the upstream's
  supported JDK environment inputs; original source/task/wrapper unchanged.
  Dry-run remains pending; source patch has still not entered a build.
- R04 official dry-run succeeds in84.976s, actual Java21 verified; all owned
  JVMs stopped afterward. Requested graph has1,155 tasks,813 compiler-named
  tasks across compiler/core/analysis/libraries as well as KGP. This is not
  patch compilation and does not measure a full build's time/disk requirements.
- The official route is viable but substantial. Before running that full
  source build, confirm whether to retain it or explicitly permit a smaller
  **partial source-rebuild experiment**: compile the changed original units
  against a checksum-pinned official plugin and generate a separate artifact
  with declared inputs/replacements/provenance. That alternative is not a
  full upstream rebuild and is not implicitly authorized by the current recipe.
  Never mutate installed/cache JARs, hide loader selection or publish the result.
- V2 static review is complete. Test-oracle B1 closed; seven prepared regressions
  and emitted binary ABI still require execution. Factory correction remains
  uncompiled and no runtime benefit is claimed.
- User explicitly chooses **direct full upstream build**, not the proposed
  partial rebuild. Full unchanged-source control now runs the real1155-task
  route, with source-built compiler dependencies and the official wrappers.
- This control uses upstream supported `deployVersion=2.4.0`,
  `build.number=2.4.0`, so generated KGP defaults stay the accepted base version.
  It is a separately rebuilt local control, never advertised as byte-identical
  to official published binaries, uploaded or installed into product caches.
- Actual Java21 daemon,4GiB, one worker; explicit private8/11/17 toolchains.
  Original Werror/verification retained. Bounded7200s, original4GiB minimum
  host-available-memory guard,10GiB remaining-disk guard and verified owned
  JVM cleanup. Source overlay is still separate until this control finishes.
- Recipes/receipts/source archives are under the private experiment root.
  Verified Java11/17 archives remain; redundant range fragments and unverified
  incomplete downloads have been removed. No owned Java/Xvfb remained before
  the full control launch.
- Full control R01 executes145 tasks (305 total including up-to-date) and
  reaches compiler CLI argument generation, then fails at298.947s on three
  exact IntelliJ platform251.27812.49 dependencies with connection resets.
  No patch source is loaded; this is an upstream dependency transfer failure,
  not a parser regression or GUI result. Resource cleanup succeeds.
- R02 repeats the exact full control recipe once, reusing its legitimate task
  outputs to get past that transient transfer. It does not skip the generator,
  change coordinates or disable dependency verification.
- R02 passes those transfers and fully compiles upstream compiler dependencies
  and KGP common sources, producing `commonJar`. Then the original host-memory
  safety guard stops at451.927s while preparing the Gradle813 variant.
  No Java heap OOM captured and no final embeddable artifact completed.
  Owned Gradle/compiler/Java worker processes are verified stopped.
- R03 continues the same full task graph from genuine already-produced outputs,
  with fresh owned JVMs. This avoids retaining the earlier long compiler
  session concurrently with final variant/packaging; no target/task/verification
  skip or resource increase. It is a warm completion control, not a cold
  full-build memory success or an IDEA repair claim.
- R03 full unchanged-source control succeeds in74.945s after fresh JVM startup;
  617 actionable tasks (6 executed,611 up-to-date). These legitimate outputs
  complete the full official route, not a partial binary rebuild. The failed
  cold/network/resource controls remain in the record.
- Resulting Gradle813 plugin is26,708,997 bytes; common plugin26,729,431 bytes.
  Their independent hashes/ZIP integrity and actual factory class were checked
  and copied to `control-artifacts/` before changing source.
- Deploy only the reviewed two original implementation units and new regression
  file, against exact original/new hashes. The production checkout and installed
  plugin caches are untouched; source-patch receipt records all three paths.
- Full patched R01 now executes official `functionalTest` filtered to the seven
  new regressions and three original serialization regressions, plus actual
  `embeddableGradle813Jar`. Full task dependencies, verification and resource
  guards remain; no test pass or patched artifact is yet claimed.
- Patched R01 compiles the changed common sources and several Gradle variants,
  but the host-memory safety guard stops the full graph after956.864s.
  No captured Java OOM; no regression tests have executed yet. Owned JVMs stop.
- The official functional-test prerequisites unexpectedly invoke `install` /
  `publishToMavenLocal`, creating user `~/.m2/repository`. This was not an
  intended experiment output location. All337 files (711,836,428 bytes), the
  repository and `.m2` directory have verified birth timestamps within this
  run; no preexisting files or settings are present.
- Hash-seal and relocate that entire newly created repository into the private
  experiment's `maven-local/`, then remove only the now-empty newly created
  `.m2` directory. Post-move hashes all match; user `.m2` is absent again.
  Provenance is in `unexpected-maven-local-provenance.json`.
- R02 continues the same full graph with explicit private `maven.repo.local`
  for both launcher and daemon. The upstream functional test already forwards
  this property into its test JVM. Original4GiB heap / host-memory / disk
  guards remain; reusing genuine upstream outputs is not skipping dependencies.
- Patched R02 succeeds in570.340s;1,713 actionable tasks,636 executed and1,077
  up-to-date. All seven reuse regressions and three original serialization
  regressions pass, XML failure/error/skipped counts zero. User `.m2` remains
  absent afterward and owned JVMs are cleaned.
- Full source-built patched Gradle813 artifact:26,717,446 bytes,
  SHA256 `bd42e4f796220b12832a0f969966e5ef03849d503b1ea6f6fc51e2bc12ccb176`;
  common artifact:26,749,923 bytes,
  SHA256 `1a5cd261c3808d69fed3baf3b0bfd697b6686f2f81dad8aac84914c5b2bc5968`.
  Private `patched-artifacts/` seals both artifacts, test XML and recipe.
- Versus the full unchanged-source control, nine changed and four added class
  entries all belong to the two edited source units; no class removed.
  Inspected original public descriptors remain, including the original unary
  file-extractor constructor. Additions are internal implementation methods /
  cache state, not a claimed public API redesign.
- Seal-script first ABI query used an incorrect nested extractor class name;
  correct the query to the actual top-level FQCN and rerun sealing. This was
  an evidence-script error, not a compile or test failure.
- Next consumption must use a declared private file repository and verify the
  loaded artifact hash / code source. Never mutate cached official JARs or
  advertise this locally rebuilt2.4.0 as official published bytes. Complete
  model, attachment/navigation, functional and genuine GUI results are pending.
- Release duplicate disposable dependency/Native caches from three finished
  GUI diagnostics (R01/R02/R03); preserve their source, logs, private heap /
  evidence, installed wrappers and later R04/R05/R07 controls. Exact path and
  stopped-run guards verify ownership.50,810,359,808 bytes freed; no user cache
  or producer build output removed.
- Real full-project candidate changes only `buildSrc/build.gradle.kts`, adding
  an exclusive declared private file repository for the exact KGP2.4.0 module.
  All product targets, source hierarchy, libraries and default source resolvers
  remain. R01 diagnostic fails because it resolves `runtimeClasspath` outside
  its owning task lock; fix the diagnostic task ownership, not the product.
- R02 verifies the resolved rebuilt artifact's exact hash and loaded factory
  method, and configures all216 projects. Native source-import preparation then
  fails honestly: official local upstream builds default to Native
  `2.4.0-dev-6762`, even with `deployVersion/build.number=2.4.0`. That development
  distribution is not the accepted Native2.4.0 and is unavailable from the
  production repositories.
- Upstream root build explicitly supports `versions.kotlin-native`; use
  `-Pversions.kotlin-native=2.4.0` in the **full upstream build recipe** to align
  the embedded default with the accepted compiler. No source constant edit,
  consumer Native override, target change or cache JAR mutation. Both initial
  source-built control and R02 artifact retain their original hashes/unaligned
  receipts; neither is advertised as a full Native-equivalent control.
- R03 reruns the original full patched task/test graph with that single
  supported release-default alignment. It will produce a newly sealed artifact;
  current raw source-import failure and diagnostic-added task/config entries
  remain recorded, not treated as a patch semantic pass.
- Aligned R03 succeeds in221.762s;313 executed /1,400 up-to-date tasks and all
  ten regressions pass again. Gradle813 artifact SHA256
  `2017706c121c1984b030a796823477c564b3aeb6a08ddf7d99c1a6233c1262b4`,
  common SHA256 `ae4633fd1be7b29580573ae687da212adccd5f1613406b68757c7c546f595bd1`.
  Both explicitly embed Native2.4.0; initial unaligned artifacts remain sealed.
- Real-project model R03 succeeds in46.854s. Resolved file hash and actual
  loaded patched factory verified. Entire model equals baseline after excluding
  the sole root diagnostic task;216 projects /1,344 targets /5,026 source sets.
- Real source importer R04 succeeds in18.605s. Initial comparison against an
  older different Native-cache snapshot finds three commonized binary changes.
  Do not claim byte equivalence until checking the cache confound.
- Unchanged official binary control then runs the same real importer using
  the same private Native cache in20.596s. Original plugin hash
  `ca5903a236a19a54a883a8695958d8516f9e39cc333bd1e3077f92ae38271cd6`
  and absence of the patched factory method verified. All four groups' binary
  records and source archive SHA/entries now equal exactly. This is a
  representative attachment gate, not all203 projects' navigation acceptance.
- Genuine full GUI R08 starts with fresh private IDEA/config/project, seeded
  declared dependency/Native caches and same Gradle binaries with their real
  official source tree. Original source resolvers, all test owners/targets,
  Gradle4GiB / IDEA3GiB / available-memory guard remain. No extra model probe
  or copied project-local cache. Private OOM capture is excluded from exports.
- R08 does **not reach Gradle import**: a first-start “Meet the Islands Theme”
  modal blocks the performance script. Private screenshot OCR and actual EDT
  stack identify `NewUiOnboardingService.showOnboardingDialog`. Escape /
  guessed close interaction does not dismiss it; stop the exact coordinator
  and verify cleanup rather than spend the remaining deadline.
- R09 uses a separate copy of the same GUI harness with only the installed
  IDE's `ide.experimental.ui.onboarding=false` registry key in private IDEA
  VM options, plus an explicit public evidence field. Disable only the optional
  UI tour, not Gradle import, indexing, source resolution or a product setting.
  R08 is a setup failure, not another patch OOM or successful import. Fresh
  R09 keeps the original deadline/resource/target gates.
- R09 reaches a **real full import success**: external start08:37:14.303Z,
  `EXTERNAL_SUCCESS`08:59:49.334Z and `EXTERNAL_END`08:59:49.353Z, then
  smart snapshots with625 nonempty modules. Gradle also reports22m18s success.
  This is stronger than R07's daemon-only completion; it is the first captured
  complete external import for this experiment at the original4GiB.
- The same run starts its next import at09:01:35.332Z and is stopped by the
  unchanged available-memory guard shortly afterward. Result remains
  `MEMORY_STOP_VALIDATION_PENDING`; no captured heap OOM or dump.
  Maximum sampled RSS: IDEA4,541,160KiB, Gradle4,868,288KiB, Native343,332KiB;
  maxima are not simultaneous or unique physical memory.
- Own JVM/GUI processes and staged credentials are cleaned. No repeated-import
  or navigation acceptance, no production switch. A same-condition official
  control is needed to distinguish producer benefit from the fully warmed
  source/dependency cache before investing in another correction.
- R10 official published KGP control uses the exact R09 dependency/Native cache
  seed, fresh private IDEA/project and unchanged resource/tour settings.
  It fails the first real import at252.683s with explicit `Java heap space`;
  `EXTERNAL_FAILURE` and a complete private5.6GiB heap captured. All resources
  and copied credentials cleaned. This argues against “warming alone fixes it”,
  but does not yet isolate source-rebuild differences from the parser patch.
- Restore both implementation files to checksum-verified official archive bytes,
  remove only the verified experimental test (reviewed copy/XML remain), and
  full-build an unchanged **aligned** upstream control. Same supported version,
  Native2.4.0, private `control-maven-local`, guards and official wrapper.
  Full publication completes in201.655s:681 tasks,48 executed /2 from cache /
 631 up-to-date. User `.m2` remains absent.
- Aligned original Gradle813 control SHA256
  `063f6643ea9ebd38161295ff41013ff3a6a630d605ed5e96acf8d048397456ad`;
  every class byte equals the earlier original control. Both aligned control
  and patch share the accepted Native default; their class delta remains the
  nine changed /four added entries from the two edited source units.
  Seal publication and exact source provenance before consumer use.
- Actual aligned-control importer in48.916s verifies the original factory /
  exact selected artifact. R11 now runs this full-source-built original
  control on the same R09 cache seed and original GUI budget. This is the
  causal control for the source patch, not another speculative project change.
- Release stopped R10 duplicate dependency/Native caches after capturing its
  failure; retain its private heap, logs, receipts, fixed project and wrappers.
  Stopped R08 disposable caches and initial cache seed are also released.
  Free-space changes measured during other copying are net changes, not an
  exact sum of deleted allocations.
- R11 **full-source-built original** control fails its first real import too:
  start09:35:11.157Z → explicit `Java heap space` / `EXTERNAL_FAILURE`
 09:42:15.936Z,424.779s. Original source/Native default/build recipe match the
  patched producer; complete private5.6GiB heap and exact control hashes retained.
  All owned resources and copied credentials cleaned. Thus merely rebuilding
  upstream does not account for the patched R09 initial success.
- Next rerun the patched variant against that same warmed R09 seed to check
  repeatability and the subsequent-import memory boundary. First successful
  import is supported; a completed stable development session is not yet proven.
- CInterop metadata-graph reselection passes24 real file/task comparisons, but
  controlled actual Gradle project fixtures reject its semantics: a mac-only
  dependency leaks into Linux, and metadata filtering still leaks a Native
  implementation dependency. Preserve these counterexamples; do not adopt.
- A deferred `copyRecursive` diagnostic retains the original graph's attributes,
  dependencies and task selection rather than reselecting metadata. Fixture R03
  matches original files/tasks in both counterexamples. Cold R04 removes only
  the three private selection payloads, discovers producer tasks from the file
  collection itself and succeeds in6.515s. Both original named configurations
  remain `UNRESOLVED`. Payloads are selection fixtures, not compiled KLIBs.
- Actual24-group comparison, copied resolution callbacks/constraints/locking,
  graph collectability and measured memory benefit remain pending. No source
  modification for this CInterop experiment, no new targets, no higher heap and
  no production change. Evidence stays in the existing private experiment root.
- Continue the remaining CInterop owner in the
  [isolated graph-lifetime experiment](2026-10-10-experiment-kgp-cinterop-graph-lifetime.md).
  Two headless controls support about259MiB less graph retention, not a GUI
  peak saving. Typed V3 full upstream build /19 tests, complete model and four
  source groups pass; actual initial/repeated GUI import is now the active gate.
