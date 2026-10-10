# Task Tree

- `Freeze accepted source and reuse the authorised Xiaoxin resources`()
- `Measure isolated single-factor configuration-cost ablations`()
- `Reject candidates which change targets, public dependency semantics or required behaviour`()
- `Run the surviving candidate through the bounded genuine GUI gate`() // failed, preserved
- `Validate functional regressions and stop adoption at the unresolved GUI failure`()
- `Record measured benefits, rejected ideas and unresolved causes`() // diagnostic scope complete

# Details

- User requests evidence of actual benefit, not another list of speculative
  improvements. Parent: [model/resource work](../executable/2026-10-02-plan-gradle-model-resource-optimization.md).
- Accepted source: Kodex `5b41be52d2f64a12ea3671bcecb9151fd2d1fa60`.
  Production remains untouched. The failed compatibility candidate is not
  silently adopted as the new baseline.
- Fixed invariants:216 registered projects,203 KMP modules,1344 target
  instances including metadata,5026 source sets, actual source hierarchy,
  cinterop and public API. No target profiles, target deletion or module merge.
- Heavy work only on Xiaoxin under the existing `device-heavy.lock`. Detect
  available daemon JVM explicitly; stop rather than replacing a foreign
  daemon. No local build, test or IDE use.
- First measurements compare one-factor private copies: publishing attachment,
  DI compiler attachment, test-plugin attachment, and broad substitution
  registration. These are diagnostic ablations, not approved removals.
  Required tests, DI generation and patch routing must be restored or assigned
  to their real consumers before a candidate can be adopted.
- Capture actual registered tasks/configurations, platform/source-set model,
  plugin application counts and post-configuration heap. Resolve nothing in
  the configuration census; this cannot certify dependency graphs or Sync.
  Hold JVM/heap/workers constant and separate cold script compilation from
  repeated configured runs. A one-off help speed difference is not benefit.
- Only candidates with material measured reduction proceed to the existing
  private genuine-GUI harness at4GiB. No extra model probe during import.
  Compare importer outcome, wall/GC/RSS and model/navigation separately.
- Existing private heap and reports may contain credentials. Read only
  allowlisted class/count/size/retention information; do not upload raw values.
- Version and build-logic adoption remain paused at the unresolved GUI gate.
  A small refactor, fewer tasks, or help success does not close that gate.

## Evidence

- Serial Xiaoxin directory:
  `~/ACodeSpace/demo/kodex-build-cost-ablation-20261008/`.
  Scripts, inputs and private receipts are retained there. Initial shared-daemon
  results accumulated changing buildSrc classloaders and are not used for heap
  comparisons. Repeat runs start a fresh daemon for each factor; discard its
  first sample and use the median of three warm runs.
- Fixed actual toolchain: Java25/KGP2.4.0/Gradle9.5.1/Koin compiler1.0.1;
  4GiB Gradle heap, one worker, configuration cache off. No dependency resolution
  in the first census. Configuration/task counts are registered names, not
  claims that every configuration/task has been realized.

| Diagnostic factor | Registered configurations | Registered tasks | Warm post-GC heap MiB | Warm wall median seconds |
| --- | ---: | ---: | ---: | ---: |
| Unchanged baseline | 68,010 | 40,748 | 684.107 | 8.094 |
| No publishing attachment | 68,010 | 34,974 | 662.590 | 8.142 |
| Koin compiler only on annotation-bearing Path Picker | 68,010 | 40,748 | 683.809 | 8.546 |
| No TestBalloon plugin, framework dependencies retained | 68,010 | 40,545 | 681.038 | 8.442 |
| No global substitution, unsafe diagnostic only | 68,010 | 40,748 | 604.038 | 8.240 |

- All factors preserve the216/1344/5026 project/target/source-set projection.
  Heap is a coarse live-GC observation, not a retained-dominator calculation or
  peak RSS. No convincing warm wall-time improvement was measured.
- Publishing saves21.5MiB/5,774 registered tasks but does not reduce
  configurations. Koin-only and test-plugin detachments save about0.3/3.1MiB.
  Deprioritize those as OOM remedies; no production plugins/tests are removed.
  Entire substitution deletion is not acceptable: it drops Native patch routing.

### Surviving kRPC candidate

- Static reverse closure of all Main/Test project declarations from the three
  direct kRPC consumers gives17 actual consumers, including application/test
  hosts, CLI, Hook and Integration. They account for5,271 registered
  configurations rather than putting the rule on every subproject.
- An initial root-filter experiment saves62.4MiB versus a reverse baseline
  recheck. The actual project-local convention form subsequently saves
  **62.006MiB /9.021%**: warm post-GC median625.334MiB versus687.340MiB.
  This second paired experiment uses the same census plus owned routing tasks
  on both sides; do not compare its absolute heap to the first table.
- The actual form is one small `kodex.krpc-utils-patch` precompiled convention,
  explicitly applied by17 consumer scripts; no parallel graph registry or
  business abstraction. Production is unchanged. The first pilot had a
  missing newline when inserting a plugin into a one-line `plugins` block;
  preserve its failed fixture log, then use `project-local-candidate-v2`.
- Baseline and candidate compare68 resolved JVM/Linux compile/test component
  graphs byte-for-byte. The patch project is selected in51 of those graphs.
  The actual convention repeats that equality across four runs and preserves
  all registered configurations and target/source-set projection.
- Separate scoped-rule + unused-export regression gate passes253 JVM tests,
  one real Linux x64 map test, and five Tool spec Linux x64 compilations.
  Do not relabel that combined gate as the convention-only GUI test.
- [Independent candidate review](2026-10-08-recheck-project-local-krpc-cost-candidate.md).

### Definition-proven unused exports

- [Dependency audit](2026-10-08-investigate-dependency-fanout-cost.md) identifies
  nine unused exports in five Tool specs. Each has one real impl caller with
  the necessary dependencies already declared. Remove unused declarations,
  not broad `api`→`implementation` substitution or inner serialized contracts.
- Actual five-spec classpath/native metadata/cinterop-view experiment reduces
  optional view failure entries from50 to8 across repeated runs. Their observed
  outer class is `ModuleVersionResolveException`; this census does not classify
  every underlying cause or count failed builds. The static
  Main project closure falls14→0 for each spec; all other198 project compile
  projections remain unchanged.
- **No reliable heap reduction:** an A/B/A baseline recheck drops below the
  candidate, so cache/order effects prevent attributing the initial10MiB
  difference to the nine deletions. No measured Sync/speed improvement claimed.
- [Retention/source trace](2026-10-08-investigate-resolution-retention-cost.md)
  confirms live graph/diagnostic ownership and project-substitution's early
  graph-resolution branch. Its source-only suggestions remain separate
  candidates, not verified remedies.

### Genuine GUI gate

- The project-local candidate ran in
  `gui-runs/baseline-local-rule-r01` using the existing private IC2026.2.3
  harness, original3/4/2GiB IDE/Gradle/Kotlin budgets and no extra model probe.
  A control source and identical seed were staged; the paired control was
  not launched after the candidate itself failed the acceptance gate.
- Informational Islands Theme onboarding blocked the script before Gradle
  started; dismiss it only in the owned authenticated display. Native import
  timestamps, not launch-to-result wall time, must be used for comparison.
- Actual initial external import failed after1445.203 seconds. The IDE external
  system error chain contains `java.lang.OutOfMemoryError: Java heap space`.
  Startup `HeapDumpOnOutOfMemoryError` option text is not an OOM occurrence.
  No allocation stack/new heap dump identifies the precise origin in this run.
- Sampled peak RSS: IDEA3,433,116KiB; Gradle4,998,836KiB; native helper1,083,672KiB.
  Do not add role peaks as if simultaneous or equate RSS with live heap.
  This does not establish a lower GUI peak against a matched control.
- During import, an owned Gradle thread sample was blocked on HTTP HEAD through
  `DefaultArtifactResolutionQuery` and KGP's
  `IdeArtifactResolutionQuerySourcesResolver`. It confirms that acquisition
  was active at that moment, not that it uniquely caused the OOM or all delay.
- The source trace explains the default-enabled legacy query alongside
  variant-based source acquisition. Disabling it remains an untested
  counterfactual which may lose older Maven sources; require identical source
  attachments/navigation before considering adoption.
- Gate result: **configuration savings verified; complete Sync remedy not
  verified and production adoption stopped**. Do not compare this1445s failure
  with the earlier1790s probed/cache-different baseline as measured speedup.
- Harness reports `verified_owned_resources_stopped`; coordinator also verifies
  no Java process from the current experiment remains. Fixed source snapshots
  and private results stay for review, with no production changes.
- Experimental project restored byte-for-byte to its sealed control manifest;
  the additional convention source is absent from that restored copy.
  Its copied user-level credential file was removed. The19-file review snapshot
  remains because the unresolved adoption review references its exact bytes.
- Four diagnostic/audit records are archived; the parent model/resource task
  stays open.179 local links/line ranges and scoped diff checks pass.
  No source commit, push, package publication or toolchain adoption this round.
