# Task Tree

- `Compose frozen test-owner declarations with the full-source V3 plugin`() // complete
- `Verify complete model and actual plugin provenance`() // complete; four representative source groups equal
- `Run genuine complete import and trace the remaining Json navigation failure`() // full combination import passes; navigation open
- `Run fresh representative functions and complete native CLI`() // 353 checks and PTY pass
- **`Keep the KGP combination isolated after the maintenance route is rejected`()** // suspended

# Details

- Latest decision: do not adopt the source-built KGP combination. The user
  rejects KGP maintenance. Only [project-owned test configuration](2026-10-11-adopt-project-owned-gradle-test-configuration.md)
  proceeds to commits, without claiming this combination's GUI outcome.
- Parent: [full import repair](2026-10-08-repair-full-model-idea-import-oom.md).
  Both previous source changes remain isolated; production is unchanged.
- The [prepared test-owner overlay](../done/2026-10-09-prepare-explicit-test-owner-convention-candidate.md)
  and [independent review](../done/2026-10-09-recheck-explicit-test-owner-convention.md)
  preserve all targets/source sets and opt140 real test owners into the
  existing framework. Remove plugin and runtime together from63 empty owners.
  No runtime classpath/framework is removed from actual tests.
- The standalone candidate passed353 representative checks but did not pass
  complete GUI acceptance. V3 also remains unaccepted. Combining them is an
  experiment, not an inference that their measured benefits add linearly.
- Freeze exactly144 build-input files over the current V3 private consumer.
  No business source or private KGP route change. Compare the actual complete
  model with the prior test-owner model, verify the loaded348fc6... artifact.
- Keep4GiB Gradle /3GiB IDE /2GiB compiler and original4096MiB host reserve.
  Retain observed idle60 policy. R22 initially removes ineffective30s periodic
  tuning; later R24's separately measured5s policy is recorded below.
  Passive heap sampling may diagnose remaining pressure; never requested GC.
- Source navigation and meaningful tests remain mandatory. No source/profile
  truncation, module merge, publishing or production switch is authorized.
- Actual model gate succeeds in87.243s:216 projects /1,344 targets /5,026
  source sets /140 test owners. All recorded fields equal the prior standalone
  test-owner model after removing only the acceptance task; selected V3
  artifact and actual loaded source helper verified. Copied credentials and
  owned build processes are removed before the next serialized gate.
- The four actual binary/source groups also match in20.663s. First GUI launch
  is refused before creating a run: its new descriptive prefix does not match
  the harness's baseline/candidate **toolchain** discriminator. Preserve this
  setup failure; derive a new baseline-toolchain GUI identity with identical
  files/input ID and use an explicit test-owner run label. No validator removal.
- Actual R22 starts one resolve; the Gradle action completes with no captured
  heap OOM, but the original host reserve again stops IDEA before external
  success. Do not accept the combination. Final heap sampling is about
  3.33GiB used /3.89GiB committed; a subsequent normal young collection logs
  3730MiB to2397MiB with3988MiB still committed. Thus unused committed capacity
  is present in this phase, but it is not the whole retained-memory problem.
- Preserve R22; next isolate the JVM's documented free-ratio resizing policy
  at unchanged4GiB maximum, original collector and default periodic-GC setting.
  [JDK25 option reference](https://docs.oracle.com/en/java/javase/25/docs/specs/man/java.html)
  describes5% minimum /10% maximum free ratios as a footprint candidate with
  application-dependent performance. R23 measures it rather than assuming
  benefit; no forced collection, higher heap or reduced reserve.
- R23 reaches genuine `EXTERNAL_SUCCESS` in324.154s and smart snapshots
  with625 modules. Gradle RSS is around4.0GiB rather than about4.8GiB;
  the actual committed heap shrinks during collection. This is an initial
  success, not repeatability or complete GUI acceptance.
- Later receive/index work still trips the original host reserve; the four
  playback imports have not run. Peak IDEA RSS is4,905,448KiB. Preserve both
  the successful external event and the failed overall result.
- After action completion, passive samples remain constant at3,491,840KiB
  committed /3,442,239KiB used; no further allocation-driven collection occurs.
  These values do not establish post-delivery live retention without a later
  natural cycle. R24 isolates a5s standard concurrent periodic cycle with
  the same5/10 free ratios, all source inputs and budgets. No external GC.
- R24 again reaches initial external success (323.606s) and starts the next
  resolve after smart/index work. The second resolve hits the original
  reserve before success; whole repeatability is still blocked.
- Normal periodic concurrent cycles are observed. Post-delivery Gradle used
  heap falls from about3.3GiB to about1.8GiB. This is actual lifecycle evidence,
  not an externally forced collection or an accepted whole-GUI fix.
- Continue [private IDE retained-owner analysis](2026-10-10-investigate-private-ide-receive-retention.md).
  Do not repeat parameter trials without locating receive/index/old-model
  memory; copied credentials and all owned running resources are removed.
- The [receive-retention investigation](2026-10-10-investigate-private-ide-receive-retention.md)
  reaches two complete **warm** four-import passes, R27/R28. R28 needs no
  special IDEA GC settings and uses a separate streaming private observer.
  Final complete model is5,245 modules, not the625-module refresh intermediate.
  R29 starts a fresh private IDE config/system with frozen source and
  dependency/Native caches; fresh initial/repeated import, exact model roots,
  actual navigation and fresh combination regressions still gate adoption.
