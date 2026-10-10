# Task Tree

- `Accept the independently reviewed moduleTree and immutable SDK consumer batches`()
- `Prepare the isolated Kotlin2.4.20 and Gradle9.6.1 candidate`()
- `Observe actual plugin classpaths and compare the unchanged full model`()
- `Validate daemon-first and separate in-process compilation controls`()
- `Validate real JVM JS Native consumers and explicit source-composite compatibility`()
- `if (allVersionGatesPass()) {`
  - `Review the exact candidate independently before production adoption`()
- `} else {`
  - `Stop the version and build-logic migration and return the concrete failure to the user`()
- `}`
- `Apply the separately user-approved Koin1.2.1 compatibility fix`()
- `Resolve the JS lock preparation failure with the user before repeating remaining gates`()
- `Use the approved native lock-update task and rerun Node before Native`()
- `Validate the complete Linux CLI link and real Native regression targets`()
- `Regenerate the wrapper and start the genuine GUI gates`() // wrapper complete
- `Return the genuine full-import failure and attribution controls for a decision`() // decision received
- `Run the user-approved same-budget no-model-probe diagnostic`() // OOM reproduced
- `Choose the next bounded memory diagnosis without changing production defaults`() // private heap approved
- `Capture private OOM retained-memory evidence at the unchanged4GB budget`() // captured and analyzed
- `Withdraw target-profile candidates after the user decision`()
- **`Investigate configuration and resolution retention with all existing targets preserved`()**

# Details

- Parent: [Gradle development experience](2026-10-02-rescue-gradle-development-experience.md).
  Prerequisite: [moduleTree integration](../done/2026-10-08-integrate-explicit-module-tree.md)
  and [SDK root-name/package gate](../done/2026-10-08-maintain-mcp-sdk-composite-root-name.md).
- User-approved compatibility target is Kotlin/KGP2.4.20 with Gradle9.6.1.
  Official fully-supported range includes9.6.1; this establishes eligibility,
  not actual Kodex/plugin compatibility.
  [Kotlin compatibility table](https://kotlinlang.org/docs/gradle-configure-project.html).
- Isolated first candidate changes the canonical Kotlin catalog version,
  replaces the independent buildSrc KGP literal with a catalog library alias,
  and updates the wrapper URL with its official distribution checksum.
  Build-logic migration, target/profile, kRPC and resource defaults remain unchanged.
- Existing TestBalloon, kRPC, Koin and Poko versions are not silently upgraded
  to make the compatibility test pass. Observe their actual compiler/plugin
  behavior; any concrete compatibility failure stops this line for a user decision.
- Default binary mode does not load fork Gradle plugins. Explicit source mode
  remains a separate required gate. Any fork adaptation belongs only to its
  `kodex-submodule` maintenance branch and a newly reviewed immutable version,
  never an upstream cooperation branch or overwritten package.
- Heavy execution only on Xiaoxin under the existing device lock. A wrapper
  transition occurs only after the completed, verified owned daemon is released;
  then explicitly select the actual JVM. No incompatible available daemon may
  be ignored while a new server is launched.
- Model equivalence compares IDs/directories, targets, source roots/parents and
  dependency edges; expected plugin codeSource and compiler version changes
  are separately recorded. CLI `help` and model extraction do not certify IDEA Sync.
- No production version commit, build-logic implementation or performance claim
  before the specified gate succeeds. Preserve failed outputs and exact hashes.

## Exact first experiment

- Accepted product predecessor: settings-only9458be6f and SDK consumer2e5c0df4.
  Isolated source archive9458be6f has the subsequently accepted51ca6e catalog
  suffix plus exactly the three compatibility candidate files.
- Kotlin2.4.20 catalog + real `libs.kotlin.gradle.plugin` buildSrc dependency;
  official Gradle9.6.1 archive SHA256
  `9c0f7faeeb306cb14e4279a3e084ca6b596894089a0638e68a07c945a32c9e14`.
  [Official distribution checksum](https://services.gradle.org/distributions/gradle-9.6.1-bin.zip.sha256).
- Our completed owned9.5.1 daemon PID1100746 was explicitly released only after
  all SDK/model/source/PTY gates finished. No available Gradle server remained
  before the9.6.1 invocation; selected JVM stays Temurin25.0.4.
- First isolated `help` passed in118s. Actual full model/classpath observation
  reused9.6.1 daemon PID1120008. `help` alone does not pass the compatibility
  line; no production version change is declared.

## Compatibility gate failed — migration suspended

- Actual full model equals the accepted predecessor exactly except intended
  KGP codeSource, now `kotlin-gradle-plugin-2.4.20-gradle96.jar`. Registered216,
  targets/source roots/parents, included-build closure and39 fork observations
  remain equal. Evidence: Xiaoxin `results/toolchain-2420-961-model-comparison.json`.
- Representative JVM compilation failed in99s at
  `:app-component-history-impl-viewmodel:compileKotlinJvm`. Koin compiler
  plugin1.0.1 warns that2.4.20 exceeds its newest tested2.4.0 adapter; the
  original `allWarningsAsErrors` policy correctly rejects that warning.
  This run proves a compatibility gate failure, **not** a reproduced IR ABI
  crash. Do not disable Werror, mute the warning or report the partial tasks
  as a successful suite.
- Exact failed output: Xiaoxin
  `results/binary-consumer-toolchain-2420-961-jvm.log`, lines919–946.
  Main stays the accepted Kotlin2.4.0 / Gradle9.5.1 build. Version migration,
  source-fork adaptation and build-logic are suspended at the agreed stop boundary.
- Primary-source follow-up found compiler-plugin1.2.1 is explicitly verified
  for Kotlin2.4.20;1.2.0 is not an acceptable substitute. Existing Koin core4.2.2
  satisfies its documented4.2.0+ requirement.
  [Official compatibility table](https://insert-koin.io/docs/setup/compiler-plugin/#kotlin-version-compatibility),
  [1.2.1 release](https://github.com/InsertKoinIO/koin-compiler-plugin/releases/tag/1.2.1).
  Upgrading the compiler plugin and unifying its catalog dependency is a proposed
  next isolated fix, pending the user's decision; no such edit/run has occurred.
- Completed owned9.6.1 daemon PID1120008 was verified and released after failure.
  No further compilation or build-logic operation is queued. Evidence and isolated
  candidate remain available for the decision; unrelated processes are untouched.

## User-approved continuation

- User explicitly selected upgrading Koin compiler plugin1.2.1 and revalidating
  the original compatibility line. Keep Koin core4.2.2, actual DI definitions,
  original Werror and all target/profile policies unchanged.
- Only the isolated candidate's plugin version and independent buildSrc Koin
  literal change: a canonical catalog library shares `koin-plugin` with the real
  plugin declaration. Kotlin/KGP already use the same candidate catalog alias.
- Preserve the first99s failed run. New help/classpath/compile/tests have distinct
  output names; no production version or build-logic changes before they pass.

## Second gate — stale JS toolchain lock

- Koin1.2.1 closes the original failure without warning suppression. New help78s,
  JVM compile/tests118s covers110 tests (including up-to-date results) across stdio8, HTTP4, in-memory RPC19 and
  History View79, zero failures/errors/skips; CLI JVM compilation also passed.
  Loaded Koin plugin is the genuine1.2.1 JAR; KGP/Compose2.4.20-gradle96,
  kRPC0.10.3 and TestBalloon1.0.1-K2.4.0 are recorded, not inferred from aliases.
- Separate in-process stdio JVM compilation control passed39s with56 actual
  tasks rerun; default daemon-first receipt remains daemon with fallback=false.
  This is not an in-process full-target/GUI pass.
- Node validation stopped in52s at `:kotlinStoreYarnLock`, before test execution.
  The original committed lock records the older KGP-managed JS tooling.
  Generated diff includes webpack5.101.3→5.108.1, webpack-cli6.0.1→7.2.1
  and their dependency closure. No sharp/node-pty entries changed in the diff.
  This is a dependency-input synchronization failure, not a reproduced Node
  compiler/runtime defect; it still does **not** count as a passing JS gate.
- Preserve `results/binary-consumer-toolchain-koin121-js.log`.
  Standard `kotlinUpgradeYarnLock` is the proposed isolated update, followed by
  diff review and the same Node gates. Do not turn off lock mismatch checking
  or apply a blind Yarn upgrade. Await user decision at the agreed stop boundary.
- Native task was not started after the failure; build-logic and production
  toolchain remain unchanged.
- Completed owned daemon1126604 was verified and released; no Native or GUI
  operation is queued while awaiting the lock-update decision.
- User explicitly approved the standard lock update and revalidation. Run
  `kotlinUpgradeYarnLock` only in the isolated candidate, retain and independently
  review its full diff; normal lock mismatch checking and Werror remain enabled.
  Node must pass before the dependent Native command is admitted.

## Standard lock-update continuation

- Standard `kotlinUpgradeYarnLock` passed28s. Updated isolated lock SHA256 is
  `60c4d2a402b4da36cc1555cefee9bbfda15aa6b21f5b0fc8ac452f6cb73470ac`.
  Production lock and toolchain remain unchanged; no mismatch bypass was added.
- Repeating the original stdio/HTTP Node targets passed52s with the updated lock.
  Preserve the prior failed lock-store run separately; it did not execute Node tests.
- Complete Linux CLI release linking and Process/Shell/Curl Native tests passed
  507s under the same device lock and explicit available9.6.1 daemon JVM.
  XML confirms64 Native and12 Node tests, zero failures/errors/skips.
  This does not establish source-mode or IDEA Sync compatibility.
- Three-file Koin/toolchain static review and the generated lock delta are distinct
  inputs. Do not attribute lock approval to the earlier three-file snapshot.
- Fresh candidate binary SHA256
  `984761cdc8344fee6edc2770c6162912bfdbf549cd95b6fec9b9e3c17809ee11`.
  Actual private PTY startup, Settings pointer-open/Escape-dismiss and Ctrl+C
  exit0 passed; Home and lease cleanup checked. Original accepted binary output
  and smoke records were not overwritten.
- Standard wrapper regeneration is now running in the candidate only. Genuine
  GUI baseline/compatibility and uniform fork/source maintenance remain required
  before production adoption and build-logic migration.

## Current wrapper and real GUI gate

- The [three-file independent review](../done/2026-10-08-recheck-koin-toolchain-compatibility-candidate.md)
  and [managed lock review](../done/2026-10-08-recheck-kgp-managed-js-lock-update.md)
  are complete with no confirmed B1 in their scopes. Production remains5b41be52.
- Additional actual Node targets bring the candidate Node total to75; Native64
  remains passed, zero failures/errors/skips. Separate in-process compilation
  reran57 tasks in42s, recording `IN_PROCESS` and fallback=false on the actual
  stdio compiler task; this does not stand in for in-process GUI Sync.
- Online standard Wrapper regeneration failed at the distribution redirect with
  `NoRouteToHostException`. Preserve both failed logs. The official9.6.1 archive
  obtained through GitHub Release API matches the official SHA256; standard
  `wrapper --offline` subsequently succeeded5s. No URL/checksum validation policy
  was disabled. Cold distribution-URL connectivity remains unverified.
- Four generated Wrapper assets are frozen separately under
  `file:///tmp/kodex-toolchain-wrapper-final-review-20261008/`.
  JAR remains the official bootstrap bytes; final properties SHA256 is
  `89f62533208a72b7a8cc2892b6b3540c445fa6175508297d932dae57d653591a`.
- Genuine private GUI A/B started on Xiaoxin after releasing only the verified
  finished CLI daemon. First baseline `baseline-5b41be52-r01` was blocked by the
  first-launch agreement and stopped through its owned cleanup; not a Sync pass.
- Existing acceptance record and the same installed `euaCommunity.html` both
  identify version1.3. Only that public already-accepted entry is reconstructed
  for a fresh run; no new agreement is accepted or user window used.
  Run02 refused the encoded node punctuation before GUI launch; run03 uses
  validated printable node components with traversal/backslash rejection.
  Initial/warm Sync, indexing/model, resource and navigation gates remain pending.
- [Private GUI preparation](../done/2026-10-08-prepare-current-private-idea-sync-gate.md)
  and [uniform fork input preparation](../done/2026-10-08-prepare-uniform-fork-toolchain-maintenance.md)
  are handed off and archived as preparation only. Their empirical execution,
  source-composite and publication obligations remain in this active parent.

## Genuine baseline import failure

- Baseline run03 passed the known-source trust prompt and dismissed first-launch
  theme onboarding only inside its authenticated private display. It then ran
  the real native `RESOLVE_PROJECT:0`, not a CLI or Tooling API substitute.
- That initial automatic import failed after1790.409s with
  `OutOfMemoryError: Java heap space`, reported by IDEA as a Gradle memory issue.
  Controls were original Gradle4g, explicit IDEA3g, Kotlin2g, workers1,
  daemon/fallback=false. These are measured experiment controls, not a new
  production memory default. No heap enlargement or model reduction was applied.
- Preserve `idea-current-runs/baseline-5b41be52-r03/evidence/result.json`
  and private diagnostics on Xiaoxin. Configured-model capture and intermediate
  625-module snapshots do not make the import or four scripted imports pass.
  No warm-import timing or navigation acceptance is available from this run.
- The failure is on accepted5b41be52/KGP2.4.0/Gradle9.5.1, not evidence of a
  new candidate compatibility regression. An identically budgeted candidate
  `candidate-koin121-961-r01` is now the next controlled observation.
- Verified owned baseline teardown completed, credential copy removed and
  shared lock released before starting candidate. Other user resources remain
  untouched. Production adoption and build-logic still wait for actual gates.

## Candidate GUI failed — version/build-logic migration suspended

- Candidate `candidate-koin121-961-r01` also failed its initial real import under
  the identical3g/4g/2g, workers1 controls. IDEA received
  `UTFDataFormatException` in the daemon log-event Throwable deserializer.
  Keep that actual error, rather than relabeling the GUI as a passing CLI suite.
- The daemon log independently records `OutOfMemoryError: Java heap space`
  while KGP's `KotlinProjectSharedDataProvider.parse` reads secondary-variant
  sharing data during `GranularMetadataTransformation`, followed by failures
  in original/additional IDE dependency resolvers. The UTF/log-transport error
  appears in the same failed run; its independent root cause is not proved.
- An owned `GC.heap_info` sample before failure observed Gradle's committed4GiB
  heap using3,924,044KiB, while IDEA used1,263,032KiB of its3GiB reservation.
  This is direct daemon pressure evidence, not a claim that every failure in
  this sequence has one proven cause.
- Baseline sampled peak RSS: IDEA3408.0MiB, Gradle4912.4MiB. RSS is not heap.
  Neither run completed initial import or yielded the required four scripted
  imports; no current203-project warm timing/navigation benefit is certified.
- Failed roots, public evidence and private diagnostics remain sealed in their
  separate labels. Both completed owned teardowns removed credential copies;
  no new build or GUI server remains launched by these runs.
- Migration stops at the agreed gate: main remains5b41be52 with accepted binary
  consumption/moduleTree and old production tools. No uniform fork commit,
  new package rollout, build-logic or resource-default change is made.
- Before attributing all pressure to production, the prepared coordinator-only
  `--without-gradle-model-probe` control can remove eager all-project model
  inspection while retaining genuine GUI/observer/import commands and the same
  resource budget. At that checkpoint it was **not run**; the subsequent
  approved control below cannot satisfy the normal model/phase
  comparison by itself. Any heap/resource policy experiment is a separate
  decision, not an automatic default enlargement.

## User-approved attribution control

- User selected the same4GiB diagnostic with eager Gradle model capture removed.
  No production change, target removal, Werror bypass or default heap increase.
- `candidate-no-model-probe-r01` retains the installed GUI, native importer,
  compiled UI observer, four scripted imports/smart waits and original budgets.
  Only the experimental `init.d/current-gui-phase.gradle` is absent.
  Normal full model/phase comparison cannot pass from this diagnostic alone.
- Dependency/distribution caches are seeded from the completed failed candidate's
  private Gradle home, not its project `.gradle`, IDE configuration or indexes.
  That seed remains immutable during the new run. This is a changed cache state,
  so one successful diagnostic alone would not prove the probe caused OOM;
  a matching with-probe control would be needed for that causal conclusion.
- Source attestation and all six approved candidate differences remain identical.
  Migration stays suspended while this diagnostic runs under the shared device
  lock. No fork maintenance commit or package version is advanced.

## No-model-probe result — importer pressure remains

- `candidate-no-model-probe-r01` failed the genuine initial `RESOLVE_PROJECT`
  after1068.261s. Both observer and IDEA report `OutOfMemoryError: Java heap space`;
  no scripted warm imports completed. Its experimental Gradle init directory
  was empty, confirming the eager model probe was not present.
- Therefore the extra probe is **not necessary** for this failure. The changed
  dependency-cache seed prevents timing or quantitative memory comparisons
  against the earlier run; it does not invalidate this narrower observation.
- The first recorded daemon OOM occurs in Kotlin reflection metadata decoding
  (`ReadersKt.toKmType`, `KClassImpl`), followed by IDE dependency/compiler-model
  failures. An allocation stack does not establish the dominant retained object
  or prove a particular package/project caused the exhaustion.
- Sampled peak RSS: IDEA3332240KiB, Gradle4949760KiB. These are resident-memory
  observations, not retained-heap measurements. The sharing-data-variant check
  found no matching variant among cached fork `.module` files; it provides no
  evidence for the suspected large fork sharing-data payload.
- Evidence remains under the exact private run label: `evidence/result.json`,
  `events.json`, `owned-processes.tsv`, and private IDEA/daemon diagnostics.
  Verified owned teardown completed, temporary credential copy was removed,
  and no production source or budget changed.
- The next step needs retained-memory evidence or an explicitly approved
  isolated resource experiment. Do not repeat unchanged imports, attribute
  the failure solely to the probe, or proceed with version/build-logic adoption.

## Approved private heap diagnosis

- User approved an OOM heap capture at the unchanged4GiB Gradle budget.
  `candidate-private-heap-r01` uses the same failed-candidate cache seed and
  no eager model probe; only the OOM dump options are added to the private JVM.
- Heap contents may contain package credentials or other live values. Keep
  `private/gradle-oom.hprof` under the mode0700 run directory, mode0600 file,
  analyze only on Xiaoxin, and never upload, commit or quote raw object values.
  Publish only allowlisted class/count/size and retention-path summaries.
- No new production default, fork commit, build-logic change or migration
  adoption is authorized by a diagnostic run. A stack/class histogram alone
  does not prove the retained-memory root cause.

## Retained-memory evidence

- The private run again failed actual import, with a complete6078771239-byte
  mode0600 HPROF. Its size is the dump encoding, **not** the4GiB heap limit.
  Eclipse MAT1.17.0 scanned78870957 objects and completed its dominator/suspect
  analysis after the verified GUI/daemon cleanup. Analysis held the same device
  lock, using a separate6GiB analyzer limit; this does not change the importer.
  Tool archive matched the official SHA512.
  [MAT batch analysis](https://help.eclipse.org/latest/topic/org.eclipse.mat.ui.help/tasks/batch.html).
- Top-level dominator-class groups in this candidate snapshot:
  `DefaultLegacyConfiguration_Decorated`1523953384 bytes/36.40%;
  `DefaultConfigurationContainer_Decorated`672297544 bytes/16.06%;
  `DefaultKotlinSourceSet_Decorated`501625528 bytes/11.98%.
  These groups locate the pressure in retained configuration/resolution and
  source-set graphs; they do not by themselves prove a library defect or leak.
- The histogram includes17656 `VariantSelectionByAttributesException` objects.
  A private, limited first50-message query identifies ordinary Kotlin/test
  dependencies and native commonizer requests; failure objects are
  `NoCompatibleVariantsFailure`. This is a sample, not all failures classified.
- KGP's actual commonized-cinterop view deliberately uses lenient resolution
  when a dependency provides no corresponding elements. Thus these retained
  failures cannot be equated to broken production dependencies or failed CLI
  compilation. Investigate their retention/repetition, not blind package
  republishing or global removal of cinterop.
  [Exact KGP configuration source](https://raw.githubusercontent.com/JetBrains/kotlin/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/targets/native/internal/CInteropCommonizerConfigurations.kt).
- Reports, indexes and raw message queries remain private alongside the heap.
  Only class/count/size and known coordinate summaries were read out. No heap,
  system-property report, raw exception message or credential was uploaded.
- The proposed `kodex.targets` property came from the resource sub-plan,
  not an implemented or individually confirmed user decision. After querying
  its origin, the user explicitly rejected the switch and further target
  changes: the current target matrix has already been reduced.
- Withdraw JVM-only/host-only profile candidates. Continue only fixed-model
  configuration/resolution diagnosis; do not change target declarations,
  hierarchy, cinterop or publishing semantics to pass the memory gate.
  Version/fork/build-logic migration stays paused at its unresolved GUI gate.

## Own-project scale — read-only count at5b41be52

- Git-tracked Main Kotlin:637 files/77946 physical lines/3164113 bytes;
  Test Kotlin:425 files/80216 lines/3565532 bytes. Counts include comments and
  blank lines, exclude external submodules and generated sources. Main/Test
  classification follows source-set directories, so Main test-support code is
  included in Main, not claimed to be solely application business logic.
- Actual own build scripts:204 including root,203 non-root KMP projects.
  Registered model:216 including12 namespace projects and root.
  Median Main Kotlin lines per non-root module166, average384;
  145/203 modules have at most300 Main lines, including one with zero.
- Saved accepted/candidate models agree:1141 platform target instances plus
  203 metadata targets,5026 source-set instances. Platform instances are
  JVM203; linuxX64/linuxArm64/macosArm64/mingwX64 each203; JS102; iOS12; Wasm12.
  This counts module×target pairs, not1141 distinct platforms or5026 populated
  source directories. Both have zero fork included builds.
- Source quantity is not a retained-heap estimate. The measured candidate
  configuration/container/source-set dominator groups account for64.44% of its
  heap. Module granularity and KMP graph expansion explain substantial fixed
  model overhead, but do not prove4GiB is intrinsically insufficient or identify
  one unique defect. Continue tracing redundant construction/retention without
  changing the fixed targets or meaningful spec/impl boundaries.
