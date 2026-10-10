# Task Tree

- `Compare upstream build structure and identify actual failing acquisition paths`() // complete
- `Measure source-resolution corrections at the fixed budget`() // complete; no accepted memory repair
- `Validate direct actual kRPC dependencies without global substitution`() // functional pass; GUI memory fail
- `Locate retained objects in the current production-toolchain private heap`() // complete: bounded retention/transport evidence
- `Validate explicit real test ownership without changing target declarations`() // model and representative runtime gates pass, isolated fixture delta explicit
- `Build isolated actual KGP metadata producer experiment`() // complete full upstream build and ten regressions
- `Require equivalent binary model and source attachments before adoption`() // full model and four representative source groups equal; whole navigation pending
- **`Run full genuine GUI import, navigation and functional regressions`()** // R36 fresh complete repetition passes; navigation/minimal-combination gates open
- `Adopt only a verified repair and preserve failed controls`()

# Details

- Current adoption boundary: the user explicitly rejects modifying or
  maintaining KGP. Preserve source-built patches as isolated evidence only.
  [Project-owned configuration adoption](2026-10-11-adopt-project-owned-gradle-test-configuration.md)
  proceeds separately; official KGP controls still OOM, so this parent remains
  blocked rather than declaring the experimental combination adopted.
- User explicitly requests continuing the repair rather than accepting OOM
  because the build has many modules. Parent:
  [model/resource gate](2026-10-02-plan-gradle-model-resource-optimization.md).
- Product is still clean `5b41be52`; previous project-local kRPC candidate has
  measured configuration savings but fails complete GUI import. Do not repeat
  that as an accepted OOM repair.
- Fixed targets, source hierarchy, cinterop, public ABI, existing source
  navigation and4GiB Gradle budget. Heavy work only on Xiaoxin under the
  existing device lock, with explicit available daemon JVM and owned cleanup.
- Compare `kotlinx.serialization` official source, not presumed memory use.
  Its current settings list14 explicit projects; build-settings-logic/buildSrc,
  actual targets and host/runtime budgets require separate accounting.
  Source LOC is not the IDE model or an independently measured OOM guarantee.
- Actual previous import sampled the legacy Sources Jar query waiting on
  HEAD. KGP keeps that fallback for pre-1.8.20 publication compatibility;
  turning it off is initially a diagnostic, not a safe repair.
- Probe real binary dependencies/source attachment acquisition on representative
  owners at fixed toolchain, using KGP's real importer API and required
  preparation tasks. Compare source archive identities and actual entries.
  Any missing sources reject a blind global disable. No replacement model
  cache, fake SourceProvider or duplicated business contracts.
- A GUI gate failure remains a blocker. No silent toolchain upgrade, heap
  increase, target profile or suppression of missing binaries/errors.
- [Minimum-combination gate](2026-10-10-verify-minimum-oom-repair-combination.md):
  preserve R36's successful full-model repetition, validate actual navigation,
  and test whether stock KGP with the same build ownership/lifecycle policy
  suffices before proposing a maintained Kotlin fork.

## Evidence

- Prior [single-factor results](../done/2026-10-08-measure-fixed-model-build-cost-ablation.md).
- [Official serialization settings](https://github.com/Kotlin/kotlinx.serialization/blob/master/settings.gradle.kts),
  [resource properties](https://github.com/Kotlin/kotlinx.serialization/blob/master/gradle.properties).
- [Actual KGP2.4.0 legacy source acquisition](https://github.com/JetBrains/kotlin/blob/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeArtifactResolutionQuerySourcesResolver.kt).

## Current isolated gates

- [Independent source-fallback investigation](../done/2026-10-08-investigate-legacy-source-fallback-fix.md)
  identifies transformed metadata as a real source-navigation risk; recent
  publication versions alone do not justify disabling the legacy resolver.
- Actual JVM/JS importer calls on four owners/five source sets succeeded:
  legacy disabled 57.972s, enabled 111.265s, fresh daemon for each.
  These are ordered cached acquisition probes, not a repeatable GUI speed claim.
  Binary records agree. Source archive SHA/entries agree; Lucene enabled attaches
  a second filename with byte-identical contents, not additional source content.
- A second actual probe includes common/native metadata and the image-codec
  JVM Java libraries. Disabling legacy **loses** stdlib/serialization source
  archives on commonMain/nativeMain. Blind global disable is rejected.
- A representative full-preparation probe previously timed out at 900s during
  `app-cli:transformNativeMainCInteropDependenciesMetadataForIde`, before
  source comparison; not evidence the legacy source query caused that timeout.
  Initial probe also had a wrong project ID; corrected. Native directory
  classpaths required real recursive hashing, not treating every KLIB as a file.
- Current source-only candidate uses the existing public KGP post-additional
  artifact extension: retain modern sources, supplement only missing imported
  exact GAVs with the public Gradle artifact query, never a newly traversed
  configuration graph or guessed capability module. Experimental Groovy
  reflection is confined to the acceptance adapter, not product code.
- Separate [direct actual kRPC dependency overlay](../done/2026-10-08-prepare-direct-krpc-patch-dependencies.md)
  is prepared, not applied. Removing substitution must retain the actual Native
  map patch, all compiler/library graphs and public API; no new routing registry.
- Remote private receipt root:
  `~/ACodeSpace/demo/kodex-import-sources-repair-20261008/` on Xiaoxin.
  Credentials are staged only for locked runs and removed on exit. Individual
  earlier probe scripts were revised in place; their logs/JSON are retained,
  not retrospectively claimed to have immutable first-revision script manifests.
  Current metadata and selective probe source copies are separately preserved.

## First genuine GUI diagnosis and guarded follow-up

- First source-only GUI experiment, original toolchain/full targets/4GiB:
  `baseline-selective-sources-r01` ends
  `DEADLINE_VALIDATION_PENDING`, not accepted import and not an observed OOM.
  No external success was captured within the original 2400s deadline.
  All owned resources stopped; copied credential properties removed.
- Actual sampled Gradle binary-resolver thread and heap at the later stage:
  `IdeBinaryDependencyResolver -> resolveDependenciesSerialized`,
  reserved4194304K / committed3858432K / used2778853K.
  This one sample is not peak/live-retained memory or proof of an OOM repair.
- [Independent diagnostic review](../done/2026-10-08-recheck-selective-source-fallback-diagnostic.md)
  permits bounded diagnosis, **blocks adoption** pending whole source/navigation
  coverage. Its opening “legacy=true GUI” description is a label error:
  the attested GUI source sets legacy=false and applies the supplement.
- Reviewer identified real Native-distribution coordinate misuse. First GUI
  event trace confirms pointless requests such as
  `org/jetbrains/kotlin/native/CoreNFC/2.4.0/CoreNFC-2.4.0.pom`.
  These are local platform libraries, not repository modules. They are excluded
  using KGP's existing `isNativeDistribution` extra in the v2 adapter.
- V2 also preserves KGP's exact paired source-download opt-out. Original v1
  helper/source/run evidence stays unchanged; no edits during the GUI run.
- V2 real importer flag=false versus original legacy-enabled path: four groups
  agree, now including explicit GAV/sourceSetName/capability fields and Native
  distribution markers. Common/native source archives remain attached.
  Flag=false23.201s / enabled20.186s are probes, not an established speed gain.
- Second fresh-private-IDE complete GUI gate:
  `baseline-guarded-sources-r02` also ends `DEADLINE_VALIDATION_PENDING`.
  Same resource/deadline/toolchain, seeded with first run's downloaded
  dependency/Native caches; **not an empty-cache speed comparison**. No external
  success or recorded OOM. All owned resources stopped and copied credentials
  removed. Production stays unchanged.
- [Empty-test dependency investigation](../done/2026-10-08-investigate-empty-test-dependency-resolution.md)
  finds63 test-empty modules, not64, but plugin-generated entry points prevent
  safely dropping just the framework dependency. No candidate is adopted;
  this is a separate possible resolution-cost experiment, not measured savings.

## Gradle script-source acquisition: separate observed blocker

- Second GUI sampled the actual Gradle daemon in
  `SourceDistributionResolver -> SourcePathProvider -> KotlinScriptTargetModelBuilder`.
  The observer repeatedly reported resolving `gradle-9.5.1-src.zip`; this is
  not proof of repeated physical downloads. No successful import was recorded.
- The downloaded source ZIP is 75,062,830 bytes. Its SHA-256 agrees with the
  official checksum:
  `7196ad0b60c1cd37d16a99e0282c643773876920783503236d1259fe8901d79c`.
  Do not describe the archive as corrupt or assert a unique OOM root cause.
- [Gradle's source-path implementation](https://github.com/gradle/gradle/blob/v9.5.1/platforms/core-configuration/kotlin-dsl/src/main/kotlin/org/gradle/kotlin/dsl/resolver/SourcePathProvider.kt)
  uses real installation `src` directories before the downloaded-source
  resolver. A same-version official `all` distribution is the next bounded
  diagnostic: retain script navigation, toolchain binaries and all KMP targets.
- Official `services.gradle.org` download of `9.5.1-all` failed connecting to
  its GitHub redirect before a usable archive was obtained. This is a network
  failure, not a failed build. The direct official CDN also redirects to GitHub;
  its 240,633,647-byte archive transferred only a few MiB before the verified
  owned low-throughput download was stopped. No partial ZIP is used.
- Third diagnostic instead stages a **private bin installation with verified
  official source roots**, not an official `all` ZIP. The source mapping follows
  Gradle's `FindGradleSources` transform; all 683 pre-existing installation files
  remain byte-identical. It adds 10,942 official main-source files in 210 roots.
  Original/shared installations and previous run evidence are untouched.
- [Independent guarded-source/installation review](../done/2026-10-08-recheck-guarded-source-and-installed-gradle-source-diagnostic.md)
  reviews the fixed two experimental inputs. The first installation-stage
  launch, `baseline-installed-gradle-sources-r03`, omitted the harness's curated
  already-accepted policy preference and stopped at User Agreement before
  import. No agreement was auto-accepted; the exact owned gate was stopped and
  cleaned. This is an orchestration failure, not a Gradle result.
- Fresh `baseline-installed-gradle-sources-r04` supplies that existing accepted
  preference input and uses original resource/deadline/full model, source-only
  v2 plus the explicit private source-installation asset. Actual daemon
  classpath/owned home confirms the prepared installation was selected.
  Cache reuse is recorded; no empty-cache or single-factor speed claim.
- R04 **fails with an actual external Java heap OOM**, approximately 300.736s
  after resolution started. This rejects the combination as an accepted memory
  repair. Earlier no-OOM timeouts were incomplete observations, not evidence
  the underlying OOM had been fixed. Source availability bypasses a separate
  acquisition path; it does not establish memory sufficiency.
- All R04 owned resources stopped; copied credential properties removed.
  One pre-failure heap sample reports committed4194304K / used3679666K;
  this is neither post-GC retained size nor peak-memory improvement.

## Next bounded gate: direct actual kRPC dependencies

- Apply the prepared five-file overlay only to a fresh isolated accepted-source
  project. Keep patch implementation/source bytes, `org.jetbrains.kotlinx:utils`
  Native identity, all targets and API symbols. No B9 export removal or test
  plugin change is mixed into this gate.
- First compare actual selected components and variant attributes across all
  retained JVM/four Native Main/Test paths, common/native metadata, and actual
  RPC-root compiler-plugin configurations. An unpatched original utils binary,
  duplicate utility identity or unresolved graph rejects the candidate.
- Root project substitution is removed, not the Native patch: real library
  exclusions and existing-project API dependencies replace the hidden global
  dependency rule. Component equivalence is not a measured OOM benefit;
  functional and genuine GUI gates remain mandatory.
- Gate R03: 17 owners / 300 configuration graphs, selected component IDs,
  variant attributes and capabilities **equal**, no original utils component.
  Includes all four retained Native targets and actual compiler configurations.
  Earlier harness attempts had a wrong input-file path and cross-project
  resolution without owner task locking; corrected, retained as harness failures.
- Actual functional build succeeded in72.402s: 240 JVM tests and one Linux
  Native patch test, all four test tasks executed. Three actual RPC producer
  Linux Native compilations executed; patch and much of CLI JVM compilation
  reused cache. These are functional gates, not GUI memory measurements.
- Genuine GUI `baseline-direct-patch-r05` keeps original KGP source resolvers
  enabled, no selective supplement, unchanged toolchain/targets/4GiB.
  Same-version installation sources are available. It **fails with actual heap
  OOM**, about678.570s after external resolution started. No completed GUI
  import; direct dependencies alone are not an accepted OOM repair.
- Effective Gradle executable, owned user home, `-Xmx4g` and private heap-dump
  flag were checked against the actual daemon. A 5.6GiB-class mode0600 heap
  was produced; all owned GUI/Gradle resources stopped and credential copies
  removed. Heap bytes can contain credentials and never leave Xiaoxin.
- One earlier heap sample committed2994176K / used2616240K. Different stages
  and GC histories prevent treating this as a peak-memory improvement.
- Existing MAT tool now analyzes this heap under the same device lock, with
  private home/workspace. Its8GiB analysis budget is not a Gradle resource
  change; no GUI/build runs concurrently. Return only class/count/size and
  structural owner paths, not object strings or private properties.
- [Independent direct-overlay review](../done/2026-10-08-recheck-direct-krpc-dependency-overlay.md)
  is separate from implementation and actual GUI acceptance.

## Current heap: concrete retention and transport boundary

- MAT parsed the actual R05 Gradle heap successfully. Sanitized class/count/size
  summary is retained only on Xiaoxin as
  `current-heap-class-summary.txt`; raw heap and reports remain private.
- Dominator groups include source sets817,618,472 bytes,
  configuration diagnostics712,343,928 bytes, configuration containers237,226,056
  bytes, and live threads849,625,056 bytes. These are partition groups, not
  additive class-loader histograms or proof all objects in a group are waste.
- Source-set retention includes39,360 parsed project-structure metadata objects
  and39,360 visibility decisions. [Exact parser investigation](../done/2026-10-09-investigate-repeated-project-structure-metadata-retention.md)
  proves same-file parsing can repeat per transformation, but no supported
  producer replacement API was found. A patched KGP needs separate scope and
  publication prerequisites; do not ship a reflection cache.
- Actual failing frame is Gradle `PayloadSerializer.serialize` called by the
  client action adapter. Payload type is installed IDEA's
  `GradleModelHolderState`; its output `StreamByteBuffer` retains385,903,112 bytes.
  This is an observed model-transport allocation boundary, not merely a presumed
  source-query leak.
- Private structural queries identify two holder traversals with2,363 entries
  each. A second query deduplicates actual byte-array identities:203 Kotlin
  MPP model arrays occupy367,942,312 bytes;217 task models10,404,376 bytes.
  The former dominates model bytes. Double traversal is not two copies.
  The model arrays and live output buffer coexist; do not add overlapping
  dominator groups again or call this a universal per-model double-copy rule.
- Actual fetch-action fields show streaming=true, phased=true,
  projectLoadedAction=false. Turning on already-enabled streaming is not a
  repair. The serializer is within the build-model action stack; that does not
  prove direct action-return serialization rather than its nested streamed send.
- [Explicit test-owner candidate](../done/2026-10-09-prepare-explicit-test-owner-convention-candidate.md)
  is a separate pending experiment: keep the test plugin and runtime together
  on140 actual owners, avoid loading them on63 empty owners, retain all KMP
  targets/source sets and real tests. Earlier plugin-only3MiB measurement did
  not remove test dependency graphs and does not predict this experiment.
- Candidate buildSrc compiles. Actual model comparison retains216 projects,
  1,344 targets and5,026 source sets with identical ancestry. Exactly63
  `testBalloonDiagnostics` tasks disappear; no other declared tasks change.
  TestBalloon applies to140 actual owners. One configuration-after-GC sample
  is775,147,856 bytes; this is not an observed saving or a resolution peak.
- [Independent fixed-overlay review](../done/2026-10-09-recheck-explicit-test-owner-convention.md)
  finds no static blocker, but later compiler-plugin order and real empty-task
  behavior require runtime validation. Its finite review/preparation reports
  are archived; this repair task remains active.
- First functional launch used a nonexistent `mcp-impl-stdio` project and failed
  before tests. Preserved as a harness failure. Corrected requests are checked
  against the actual model, using existing MCP stdio/HTTP source-owning projects;
  no production fix follows from that failed command.

## Test-owner runtime gates and bounded GUI control

- First actual functional gate executes103 tests and has one Native socket
  EPIPE in `stale unpause token cannot resume a forced reused easy address`.
  Accepted-source single-case control passes; neither result identifies a
  unique historical root or proves plugin-order equivalence.
- [Independent fixture review](../done/2026-10-09-recheck-native-fixture-send-before-cancel.md)
  confirms a permitted old schedule: the token assertions precede real peer
  response writes, then call cancellation can close the connection before
  those writes finish. The isolated copy adds the existing bounded `sent.await`
  after both assertions, before cancel; no assertion, engine or timeout removed.
  Original production test bytes remain unchanged.
- With that explicit one-test readiness delta, all11 non-Integration owners
  pass352 tests across JVM/Node/Linux, including22 Curl Native,165 Mosaic
  components, annotation-only Patch and Koin/Path Picker. Some task outputs
  restore from cache; source-sensitive compilations and many test tasks execute.
- Accidentally requesting the entire credential-dependent Integration project
  fails30 live tests because no credentials were loaded. This is not30
  configuration regressions or an authorized live-model workload. The failed
  receipt/log remains. A correctly filtered, enabled offline Patch probe then
  executes one test successfully. Earlier task XML directories are reused by
  subsequent runs; do not claim immutable XML snapshots for those failures.
- Empty current-time spec and Main-only State fixture `jvmTest` remain declared
  and return genuine NO-SOURCE; empty Native Test compilation is also NO-SOURCE.
  CLI JVM compilation executes successfully. No fail-on-empty override added.
- Genuine GUI `baseline-test-owner-r06` retains the same old toolchain,
  all targets/ancestry,4GiB, original source resolvers, official installation
  sources, cache reuse recorded. Fixed146-path delta consists of144 build inputs,
  that one fixture readiness copy and a same-budget owner marker.
  No extra Gradle model probe. It ends `MEMORY_STOP_VALIDATION_PENDING` after
  394.875s of process observations, not external success or a captured Java OOM.
  The original4,096MiB minimum host-available-memory guard stopped it.
- R06 summed resident peak9,098,648KiB (shared pages may double-count);
  per-role maxima: IDEA3,334,908KiB, Gradle4,836,044KiB,
  Kotlin1,163,864KiB, Native child392,404KiB. RSS is not Java retained heap,
  and maxima from different timestamps must not be added as one peak.
  Verified owned resources stopped and copied credentials removed.
- R07 repeats the same semantic input/resource limits with R06's now-compiled
  private cache/Native seed; only the owned-run marker changes. Fresh private
  IDEA is recorded, **not cold Gradle caches**. This separates additional
  compiler/cache work from repeated model acquisition, not a retry-to-green
  acceptance claim. It remains a bounded import diagnosis, not full navigation
  or production acceptance. Product remains clean5b41be52.
- R07 also ends `MEMORY_STOP_VALIDATION_PENDING`, after755.578s of process
  observations. No external import success/failure or actual Java heap OOM
  was captured. The same host-available-memory guard stopped the run.
  This warm control has no Kotlin daemon; cold compiler overlap alone does
  not explain the failed import.
- R07 summed sampled resident peak9,203,064KiB (not unique physical memory);
  role maxima are IDEA4,220,352KiB, Gradle4,940,796KiB and Native338,552KiB.
  Maxima are not simultaneous. An `OutOfMemoryError` string in IDEA startup
  arguments is not an OOM event. A daemon `BUILD SUCCESSFUL` is not an
  `EXTERNAL_SUCCESS` import event.
- There is a limited positive observation: R07's actual daemon reports
  `BUILD SUCCESSFUL in 10m 30s` and `has finished executing the build` at
  2026-10-09T02:30:36.282Z, immediately before cleanup. This differs from
  R05's actual Gradle heap OOM, but does not establish which model phase
  finished or a completed IDEA import. No source/navigation acceptance follows.
- Harness verifies owned resources stopped; no Java/Xvfb remains and the
  copied package credential is absent. The test-owner candidate remains
  unadopted: representative tests/model equality do not repair this GUI gate.
  Further repeated GUI runs of this same candidate are not justified.
- After the warm control stopped, the locked cleanup removed only R06's
  duplicated private dependency/Native caches. Logs, receipts, fixed inputs,
  R05 private heap and R07 warm control cache remain; available disk recovers
  to31GiB. No user cache, installed IDE, product or unrelated process changed.
- Next producer experiment requires explicit scope confirmation: KGP's
  internal metadata parser/factory has no public replacement hook. A real,
  reproducible isolated KGP source patch is preferable to reflection injection
  or hand-edited installed JARs, but introduces a maintenance prerequisite.
  Begin with project-scoped repeated selected-file parsing, not whole-model
  replacement. Same model/source/ABI/function and real GUI gates still apply;
  no publishing, fork creation or production version switch is implied.
- User now authorizes that [isolated KGP source/build experiment](2026-10-09-experiment-kgp-metadata-producer-reuse.md).
  It is a new experimental prerequisite, not authorization to publish a fourth
  fork or adopt a plugin version. Failed project-side controls remain recorded.
- Full upstream source build and ten regressions pass; the private variant's
  embedded Native default is aligned to accepted2.4.0 using the upstream build
  property, not a consumer override. Resolved Gradle813 artifact SHA and actual
  loaded factory method are verified.
- Product candidate differs only by one declared private KGP repository in
  `buildSrc/build.gradle.kts`. Complete project/config/task model equals the
  accepted baseline after excluding the one explicit diagnostic task:
  216 projects,1,344 targets including metadata and5,026 source sets.
- Four real importer groups (common/native/linuxX64/JVM) have identical binary
  records and source archive SHA/entries against the actual official-plugin
  control using the same private Native cache. Three commonized binary hashes
  differ from the older user-cache snapshot; same-cache official control also
  yields those newer bytes, so that first comparison was cache-confounded.
  It is not dismissed as “harmless nondeterminism” without the control.
- R08 now runs the genuine full IDEA import, unchanged targets/source resolvers,
  Gradle4GiB, IDEA3GiB and original host-memory safety guard. No extra Gradle
  model probe; source-built producer/model passes are not full GUI acceptance.
- R08 was blocked before import by the optional new-theme tour; stopped without
  claiming a patch result. Private R09 skips only that tour using the installed
  registry key. R09 captures actual `EXTERNAL_SUCCESS` and subsequent smart /
 625-module snapshots at4GiB, not merely Gradle daemon completion.
- Its following reimport still triggers the original host-memory guard, with
  no captured Java heap OOM. Repeated-import/navigation acceptance is not
  closed. Next compare the actual official plugin under the same fully warmed
  dependency/Native/source-cache conditions; no causal claim or adoption yet.
- Same-warm-cache official KGP R10 fails its first genuine import with actual
  Gradle heap OOM at252.683s. A separately full-built original KGP control,
  aligned to the same Native default and differing only in the two source
  units, is now being tested to isolate patch benefit from source-rebuild
  differences. Repeated import and full navigation remain open.
- Aligned full-source original control R11 also fails its first genuine import
  with actual Gradle heap OOM at424.779s. Its selected artifact is independently
  verified; compared with the aligned patch it has only the expected nine
  changed and four added class entries from the two source units.
- Patched warm R12 uses the same R09 cache seed, complete model,4GiB budget and
  original safety guard. This tests repeatability rather than attributing the
  R09 success to warmed caches or an upstream rebuild. R11's duplicated caches
  were released only after verified process cleanup; private heaps and evidence
  remain. No production change or stable repeated-import acceptance yet.
- Passive R13 sampling shows a natural Full GC followed by about3.68GiB old
  generation occupancy. R14's interrupted26MiB dump is unusable; R15 obtains
  a complete private4.86GB mid-import dump and successful MAT analysis.
  These diagnostics pause or stop the import and are not performance passes.
- The patched factory is actually one per Project (203). Remaining large
  configuration retention is concentrated in commonized CInterop dependency
  graphs. The four203-instance Native/Linux Main/Test families have individual
  retained-size sums near700MB; sums can overlap and are not promised savings.
- Reject metadata-graph variant reselection: although24 real file/task groups
  match, genuine project fixtures expose a mac-only API dependency, then a
  Native implementation dependency even after metadata-visibility filtering.
  This is a semantic failure, not a candidate to ship.
- Next investigate temporary copies of the **original** CInterop declaration
  for IDE resolution only. Cold fixture execution discovers the actual producer
  tasks without explicit dependencies, returns the original selected files and
  leaves the named configuration unresolved. Copy callbacks, constraints,
  dependency locking, actual retained-memory benefit and full GUI remain gates;
  no CInterop producer patch or production adoption yet.
- The subsequent [CInterop source experiment](2026-10-10-experiment-kgp-cinterop-graph-lifetime.md)
  completes the official V3 build and19 regressions, full model and four
  actual source/binary groups. A Tasks-only projection is rejected by a cold
  artifact-transform fixture; direct work-node delegation preserves it.
- R17/R18 complete the actual Gradle build action but still stop at the
  unchanged host reserve before external import success, without captured
  heap OOM. The R18 private heap shows substantially smaller retained
  configuration groups, not a proved whole-process peak reduction.
- Next [measure normal compiler idle shutdown](2026-10-10-experiment-idle-compiler-lifecycle.md)
  privately; no external compiler kill, heap increase or target change.
  Complete import, repeatability, source navigation and fresh product
  regressions are still required. Production remains unchanged.
- Subsequent [combined test-owner experiment](2026-10-10-experiment-v3-with-real-test-owners.md)
  and [private IDEA analysis](2026-10-10-investigate-private-ide-receive-retention.md)
  preserve all targets and source attachments. Two warm four-import runs
  now complete; the streaming-observer run uses original IDEA GC defaults.
  Full imported model is5,245 IDEA modules with real source/library roots.
  Earlier625-module smart snapshots were intermediate, not complete model
  acceptance. Original aligned controls retain their actual Gradle OOM.
- Fresh-private-IDE R29, exact full model/roots, genuine source navigation
  and fresh combined product regressions remain open. No maintenance fork,
  publishing, production switch or completed OOM-repair claim follows yet.
