# Task Tree

- `Preserve initial success and the failed second-import result`() // complete
- `Inspect passive heap/RSS and a private recovered-project IDE heap`() // complete
- `Identify concrete retained owners before another repair`() // complete; Java heap growth dominates R34
- `Test normal idle Gradle shutdown with unchanged complete imports`() // R36 combination passes
- **`Verify the minimum combination and actual navigation`()**

# Details

- Latest adoption boundary: the user rejects KGP modification/maintenance.
  Retain the normal-idle and full-source controls as research, not new production
  GC/idle defaults or a complete OOM fix. Project-owned test configuration is
  [adopted separately](2026-10-11-adopt-project-owned-gradle-test-configuration.md);
  full official-toolchain import and Json navigation remain blockers.
- Parent: [combined experiment](2026-10-10-experiment-v3-with-real-test-owners.md).
  R24 reaches initial external success at323.606s, completes indexing and
  starts the next resolve. The original reserve stops that second import.
  The combination is not accepted and production remains unchanged.
- Concurrent idle GC is actually observed. After the action, used Gradle
  heap falls to about1.8GiB and committed heap to about2GiB, without external
  collection. This supports temporary post-delivery garbage, not a claim
  that all3GiB is live or that the whole repair succeeds.
- Stop parameter-only retries. Recover only our imported R24 project in our
  isolated Xvfb/config/system, then read passive IDE heap/RSS and capture a
  private `GC.heap_dump -all` if625 modules are actually observed smart.
  No manual import is requested and automatic import is not disabled.
- This is a warm recovery diagnostic, not cold import or navigation
  acceptance. Keep the original max heaps, source inputs and reserve.
  Its private cache may still trigger an automatic resolve; record that
  instead of assuming no Gradle work.
- The user desktop, its IDE and its projects are not touched. Sensitive heap
  and logs remain private; export only class/count/retained-owner summaries.
- For disk safety, release only completed R16–R23 regenerable IDE systems
  and MAT indexes for the SHA-read-back-verified archived R15/R18 heaps.
  Preserve raw-equivalent archives, reports, source/model/GUI evidence and
  R24's imported state. Do not release user caches or upstream artifacts.
- Archived heap read-back SHA verification and regenerable-index/system
  cleanup release11,420,188,672 bytes. Recovery preflight nevertheless refuses
  its14GiB free-disk requirement before any launch/credential staging.
  Release only unused2.3.21/2.4.20 distributions in the dedicated private
  CLI cache; retain actual2.4.0, all targets and the original multi-version
  seed. Do not lower the disk guard.
- R25 recovery opens no project and supplies no625-module observation.
  Its EDT stack is inside `TrustedProjectStartupDialog`; the launcher had
  passed a new symlink rather than the already trusted canonical project.
  This is a diagnostic setup failure, not product OOM or valid small-heap
  evidence. Coordinator stop at891.274s runs verified owned cleanup, with
  no heap capture and copied credentials removed.
- R26 passes the original canonical project path, which exactly matches
  this private config's existing trusted path. Keep the failed R25 logs and
  thread evidence; do not disable trust checks globally or touch the user's
  IDEA. All budgets, source inputs and passive-capture criteria are unchanged.
- R26 canonical-path recovery captures a complete2,331,856,244-byte private
  IDEA `-all` heap at112.329s, without a requested GC. The recovered state
  starts an automatic resolve; observed smart snapshots include625 modules.
  This is a recovery/active-refresh phase, not completed repeated import.
  Passive maximum used heap is2,125,098.9KiB and committed heap2,400,256KiB,
  while IDEA RSS reaches4,341,416KiB. Thus RSS is not interchangeable with
  Java used heap. All owned processes and copied credentials are cleaned;
  analyze this private heap serially before proposing another change.
- MAT's reachable heap is about796.4MiB, including about191MiB in the
  external model tree. This dump is an active refresh: the restored snapshot
  has5,245 modules, then refresh replaces it through0/217/625 intermediate
  modules. The625-module snapshot has no library roots and is **not** a
  complete KMP model. Previous complete R24 snapshot contains5,245 modules,
  418 source roots and real library class/source roots after initial success.
- Preserve that distinction in future acceptance: four native imports and
  a final full model with source/library roots are required. R27 tests normal
  IDEA free-ratio/concurrent idle collection, justified by this observed
  reachable-versus-used gap, at the same3GiB IDE maximum. Gradle/Kotlin maxima,
  targets, source inputs and original reserve remain unchanged. This warm
  repetition gate is not a cold-import, source-navigation or production pass.
- R27 completes all four native imports at895.235s and satisfies actual
  external successes, smart/playback markers, final60s and full5,245-module
  state. Final counts exactly match prior complete state:418 source roots,
  86,434 library-source roots,328,720 class roots. This is a warm repetition
  pass, not fresh IDE, interactive navigation or whole repair acceptance.
- Before adopting IDE tuning, inspect observer cost. Its actual full snapshot
  is202,842,210 bytes. The original observer constructs one complete
  `StringBuilder` and then writes the complete string, generating additional
  whole-snapshot encoding/copy allocations. This instrumentation may amplify
  IDEA memory; it does not explain the independent Gradle OOM controls.
- Build a separate private observer from the same source/API, changing only
  snapshot output to `BufferedWriter` with unchanged row order/escaping.
  Original and changed compiled private helpers produce equal output for
  six encoding cases. Plugin/source hashes and the missing-JBR-jar packaging
  correction are recorded privately; installed IDE and original observer
  remain untouched. R28 repeats with this observer and **original IDEA GC
  defaults**, holding Gradle policy, source inputs and all budgets fixed.
- [Independent observer review](../done/2026-10-10-review-private-streaming-observer.md)
  completes static responsibility/encoding checks. It does not certify the
  old positive-snapshot collector or an OOM repair.
- Preserve old snapshots in per-file SHA-read-back-verified gzip archives
  to release diagnostic disk cost. Keep the last complete raw snapshot for
  exact-content comparison; no product cache or user evidence is deleted.
- R28's first preflight refuses the unchanged14GiB disk guard before launch.
  Release only finished R10/R11 regenerable IDE systems, preserving their
  verified heap archives/reports. Then R28 completes all four real imports at
  847.393s with streaming observer and **original IDEA GC defaults**.
  This removes any requirement inferred from R27 for special IDEA GC tuning.
- R27/R28 final row identities, source/content/library roots and all other
  non-SDK text are identical. Exactly4,620 Kotlin SDK descriptors differ from
  `Kotlin SDK home= version=null` to `Kotlin SDK home=/ version=null` after
  recovery; report this rather than claiming byte-identical snapshots.
  The625 Java SDK descriptors are unchanged. All full counts match. Recovered
  state/GC differs across runs, so timing/RSS is not an observer-only causal
  benchmark: R28 IDEA peak is5,271,048KiB, not an observed peak-RSS reduction.
- Preserve both warm passes; retire only completed R24 copied Gradle caches
  and Native cache before a new private-config/system R29 initial/repetition
  run. Original seed, current full Native cache, all source/artifacts/reports
  and model evidence remain. R29 keeps original IDEA GC defaults and bounded
  observer output. Navigation and fresh product regressions are still gates.
- R29 preparation initially fails before display/IDE launch because the
  selected private `uv` Python3.12 lacks `os.pidfd_open`, required by the
  verified ownership helper. No JVM or model import starts. A follow-up
  private trace records this setup error; copied credentials are removed
  after checking no owned Xauth/JVM is live. Preserve both attempt receipts.
- Resume only that unlaunched prepared root, checking source/plugin/resource
  identity and absence of IDE events/display. Use system Python3.14 with
  verified pidfd APIs, as successful R25–R28 runners do; do not weaken process
  provenance or re-copy all caches. Source and fresh config/system are still
  unimported, with prior setup evidence saved under distinct filenames.
- R29 genuinely opens the fresh private IDE. Initial external import and its
  following resolve succeed, but the original reserve stops the next resolve;
  two manual commands are observed, not all four. No captured Gradle heap
  OOM. Peak RSS is IDEA5,216,864KiB, Gradle4,201,320KiB,
  compiler1,201,508KiB. Verified cleanup removes all own resources/credentials.
  Streaming observation alone is insufficient for a fresh initial/repeat pass.
- R30 keeps this fresh-state path and tests both previously justified normal
  idle-reclamation policies, including IDEA's5/10 free ratios and5s concurrent
  periodic cycle, at unchanged3GiB IDE/4GiB Gradle/2GiB compiler maxima. This
  is not a lower safety threshold or forced collection. Complete model,
  navigation and fresh meaningful regressions remain mandatory.
- R30 also reaches initial and subsequent external successes, but stops at
  the original reserve on the following resolve. Compiler lifetime evidence
  shows normal disappearance well before failure, so its early peak cannot
  be added to later peaks as simultaneous failure pressure. Do not accept
  either fresh-state repetition run or continue blind GC parameter trials.
- R32 native-memory recovery opens the complete private model, but playback
  rejects one600000ms delay: installed command permits at most60 seconds.
  IDEA exits before the60-second classification trigger. The coordinator
  stops and verifies cleanup; no NMT snapshot or OOM result exists.
  Preserve that setup error. R33 uses twelve supported60000ms delays under
  the original finite900s diagnostic deadline; no manual import is requested.
- R33 obtains an actual full-model recovery classification at62.465s:
  used Java heap1,175,002.6KiB, committed1,282,048KiB, RSS2,892,188KiB.
  smaps reports anonymous2,225,776KiB and file PSS620,052KiB; NMT total
  committed2,220,066KiB, including Java heap1,282,048KiB, Other273,453KiB
  and metaspace274,104KiB. This warm idle phase is not the5GiB failing phase.
- R34 requests one actual reimport and passively classifies at receiver
  RSS≥4.3GiB or available memory<4.25GiB, before the original4GiB reserve.
  Capture initial and active NMT summaries plus numeric smaps; keep all
  maximum heaps and source inputs fixed. Do not infer a native leak merely
  from subtracting Java used heap from RSS.
- R34 active receiver classification completes at198.391s before the reserve:
  Java used2,681,422.4KiB / committed2,879,488KiB, RSS4,627,916KiB;
  smaps anonymous3,900,076KiB / file PSS685,260KiB.
  NMT total committed3,858,481KiB includes Other201,589KiB and
  metaspace319,630KiB. Relative to initial NMT, Java committed grows about
  1GiB; Other grows only about11MiB. This does **not** support a large
  untracked-native leak as the primary explanation in this sampled phase.
- Stop native/GC parameter speculation. Measure actual serialized dependency
  multiplicity through KGP's public importer on six representative owners.
  Emit only counts/lengths, never payloads, private fields or fingerprints.
  Exact content repetition is not proof that mutable ByteArrays may safely
  share identity, or that all byte savings become retained/transport savings.
  [Independent boundary investigation](../done/2026-10-10-investigate-serialized-ide-dependency-duplication.md)
  runs read-only; coordinator alone runs the locked numeric census.
- Completed R30 cache copies are released after all associated processes and
  copied credentials stop; retain source, system/config, NMT/model evidence,
  upstream artifacts and original cache seed. The source/navigation gates are
  still open and production remains clean.
- Actual six-owner census succeeds in40.752s with selected V3 artifact checked:
  152 source sets /20,995 serialized records /13,684,964 payload bytes.
  Global unique content is1,346,524 bytes; project-local repeated payload
  bytes total9,926,270. Largest record is2,171 bytes. No payload/fingerprint
  is exported. This is real repetition, not a claim that those exact bytes
  can safely share mutable identity or recover the whole model's memory.
  All own Gradle resources and staged credentials are removed.
- Reject model-parallelism tuning before another GUI run: installed IC262
  bytecode matches its default-disabled `gradle.parallel.model.fetch` feature.
  Actual R30 logs include `org.gradle.tooling.parallel.ignore-legacy-default=true`,
  with no explicit tooling-parallel=true. Gradle9.5.1 gives that guard precedence
  over the ordinary parallel-task default. The `idea.parallelModelFetch.enabled`
  flag alone does not prove concurrent Gradle model production.
- Independent [serialized-array investigation](../done/2026-10-10-investigate-serialized-ide-dependency-duplication.md)
  rejects aliasing public mutable arrays. [Parent dependency investigation](../done/2026-10-10-investigate-ide-parent-dependency-propagation.md)
  rejects dropping identical child libraries: current parent exports and
  ordered classpaths do not preserve that behavior. Neither is a repair.
- R35 tests one supported lifecycle change only: private
  `org.gradle.daemon.idletimeout=10000`. A completed import's idle Gradle may
  shut down normally while IDEA indexes, releasing its retained heap/RSS.
  Active commands must remain alive; no outside process kill or forced GC,
  no extra playback delay, and no changed model, target or maximum heap.
  This may trade daemon reuse/latency for lower overlap; measure actual
  starts/exits, import times and memory rather than promising a free speedup.
  Require all four native imports and complete final roots under the original
  deadline/reserve. A pass still needs navigation and fresh functional gates.
- R35's first preflight uses a nonexistent observer filename and exits before
  root creation, credentials or launch. Correct to the existing sealed
  `streaming-observer/observer-streaming.jar`; its original SHA guard remains.
- R35 records external successes0–3 and several normal idle Gradle exits,
  but diagnostic/cache writes exhaust disk before the remaining playback and
  final capture. Do not count this as a complete import pass or a heap failure.
  The failed writer also interrupts ordinary cleanup. Reacquire the same lock;
  verify saved PID/start/executable/cwd plus marker/live parent, then stop only
  those three surviving resources with pidfds and remove copied credentials.
  Release completed R35 regenerable Gradle/Native cache copies, retaining
  upstream seed, source/system, logs and the failed evidence.
- Archive only earlier large TSVs in four explicitly finished private runs:
  per-file gzip read-back SHA equality, receipt before raw-copy deletion, keep
  each latest raw observation. Do not delete unverified evidence or user data.
  Require a larger disk preflight and a runtime disk guard before retrying;
  heap budgets/reserve and deadlines remain unchanged.
- R36 completes the fresh-private-IDE automatic import plus all four native
  manual commands, their smart/after markers and normal exit. Final model is
  exactly5,245 modules /673,278 orders /418 source roots /86,434 library-source
  roots /328,720 class roots. All original memory thresholds remain.
  This proves the combination, not the necessity of each source patch.
- Continue in [minimum-combination verification](2026-10-10-verify-minimum-oom-repair-combination.md).
  R39's focused-editor and native source assertions pass, but Json target
  navigation does not. A stock-KGP control must distinguish inherited
  navigation behavior and unnecessary source-patch maintenance.
- Completed snapshot archiving is corrected to preserve the highest numbered
  **module snapshot**, rather than the later `events.tsv`. Earlier compressed
  final snapshots are restored after full SHA checks; earlier evidence copies
  are also verified/compressed. No bytes are lost;68 archive proofs are kept.
- R37 navigation's first native assertion fires3s after GotoDeclaration but
  the actual `CoroutineStdioIo.kt` sources archive opens at7.2s. Stop this
  failed finite probe and clean resources; do not call it missing sources.
  Replace fixed sleep with a private bounded public playback helper that
  waits for the real selected editor filename, then still execute the native
  assertion. The helper cannot open files, repeat actions, mutate models or
  bypass indexing; preserve30s per-target /900s overall and fail on timeout.
- Twenty-eight earlier snapshot archives pass full decompressed SHA checks.
  Also release only the completed upstream build's regenerable Gradle caches
  after checking no live source-build process and the final sealed813 artifact
  SHA. Preserve source/build outputs,19-test reports, both controls, private
  repositories, wrapper/JDKs. Future upstream runs may need cache regeneration.
  R36 requires26GiB free initially and stops before the remaining2GiB disk
  reserve is consumed; disk errors remain separate from memory results.
- Later finished navigation snapshots have14 additional read-back-verified
  archives, retaining each latest numbered raw module snapshot in both areas.
  This releases enough space for an unmodified-source control without deleting
  any source/artifact/failure evidence or touching user caches. R40 and R44
  fail actual heap OOM despite the normal idle policy; idle shutdown alone
  does not cure the producer. The minimum-combination child records the
  artifact controls and the separate stock Json navigation counterexample.
