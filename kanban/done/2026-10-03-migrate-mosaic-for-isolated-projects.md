# Task: make Mosaic compatible with Gradle Isolated Projects

**Stage:** executable  
**Status:** completed_research; production IP deferred  
**Owner:** Session 445  
**Parent:** [Rescue Gradle development experience](../executable/2026-10-02-rescue-gradle-development-experience.md)  
**Research device:** Xiaoxin Ubuntu, isolated exact-main validation copy only

- Scoped prototype, rollback and blocker report are complete. The original
  results below do not certify production IP or full Native/IDE acceptance.
  User subsequently made IP optional; remaining research is tracked in
  [deferred native mechanisms](../planning/2026-10-02-verify-native-gradle-sync-mechanisms.md).

## Task tree

```text
Mosaic IP remediation
├── [x] Inventory all cross-project configuration sources
├── [x] Preserve a normal-help/project-graph baseline
├── [x] Prototype project-local Mosaic conventions
├── [x] Run normal and Isolated Projects diagnostics
├── [x] Compare model and resource evidence
└── [x] Stop, rollback, and record the next blocker
```

## Scope and safety

- Use the exact main validation snapshot and gitlinks recorded in
  [the main validation checkpoint](2026-10-03-validate-gradle-plan-on-main-xiaoxin.md).
- Do not modify the active local `Kodex` worktree, public fork branches,
  gitlinks, or remote repositories.
- Keep the prototype under the Xiaoxin research directory; it is not a
  production patch.
- Acquire
  `~/ACodeSpace/demo/kodex-gradle-research-445/device-heavy.lock` for every
  Gradle configuration, compilation, or diagnostic run and hold it until all
  owned processes stop.
- Do not use `org.gradle.unsafe.isolated-projects.dangerously-ignore=true`.
- Keep the weak-device controls at two workers and approximately 2 GiB Gradle
  heap unless a measurement requires a separate, explicitly recorded control.

## Acceptance gates

1. The inventory identifies every Mosaic `allprojects`, `subprojects`,
   `afterEvaluate`, and other cross-project access relevant to the failure.
2. Normal `help` remains successful and the project graph is unchanged.
3. The prototype removes root-to-subproject configuration and applies the
   equivalent convention project-locally.
4. Isolated Projects diagnostics no longer report Mosaic violations, or the
   next failure is isolated to a named non-Mosaic/plugin boundary with a
   reproducible log.
5. No model or target is silently removed; any reduced target profile is
   explicit and separately recorded.
6. All owned daemons and samplers are stopped, the shared lock is free, and
   the report/checkpoint contains exact commands, logs, timings, and rollback
   paths.

## Stop conditions

- Stop and report if preserving Mosaic's normal project graph requires
  `dangerously-ignore`, cross-project callbacks, or an unreviewed plugin
  behavior change.
- Stop if the prototype changes published coordinates, source-set semantics,
  target coverage, or task behavior without a separate equivalence check.
- Stop before touching a public fork or creating/pushing `kodex-submodule`;
  those require a later production authorization.

## Checkpoint

### Completed on Xiaoxin Ubuntu

- The exact main validation snapshot and Mosaic gitlink were preserved:
  root `3008a6de`, Mosaic `7b1a4129`, Gradle `9.6.1`.
- The original Mosaic blocker was root `allprojects {}` in
  `Mosaic/build.gradle`, configuring `group`, `version`, repositories, task
  options, test logging, and Spotless on every project.
- A project-local convention prototype removed that root cross-project block
  and applied equivalent configuration to all 16 existing project build
  scripts. It preserved the 18-project Mosaic project graph.
- The prototype's normal `help` succeeded in about 11 seconds, and
  `-Dorg.gradle.unsafe.isolated-projects=true help` succeeded in about
  9 seconds with configuration cache stored. No Mosaic IP violation remained
  once the external jextract plugin was isolated.
- The external `de.infolektuell.jextract` 1.0.0 plugin itself accessed
  `project.rootProject.layout` from `mosaic-tty`, producing the next IP
  violation. Upgrading to 1.4.0 did not remove that violation and introduced
  an additional KMP/Java plugin incompatibility.
- An isolated Mosaic-local copy of the jextract plugin source, changing only
  its shared-service cache directory from `project.rootProject.layout` to
  `project.layout`, compiled and passed IP `help`. The jextract task remained
  registered and its `--dry-run` path succeeded.
- Under the existing Mosaic KGP `2.3.21` line, `:mosaic-tty:compileKotlinJvm`
  succeeded after the convention prototype set the JVM compiler target
  correctly. This is semantic evidence, not a production patch.

### Decision gate

- A separate KGP `2.4.20` experiment passed Mosaic IP `help`, but
  `:mosaic-tty:compileKotlinJvm` failed in the unrelated
  `app.cash.burst` compiler plugin with an
  `IrGenerationExtension`/`ProjectExtensionDescriptor` `ClassCastException`.
- Per the confirmed version-line rule, this failure stops the KGP/Gradle
  migration and returns the version decision to the user. Do not continue
  toward a production `kodex-submodule` patch until that decision is made.

### Evidence

- Remote prototype root:
  `~/ACodeSpace/demo/kodex-gradle-research-445/main-validation-445/mosaic-ip-local-jextract-445/`
- KGP 2.4.20 result:
  `~/ACodeSpace/demo/kodex-gradle-research-445/main-validation-445/mosaic-ip-local-jextract-445/results/mosaic-tty-compile-kgp2420.log`
- IP success:
  `~/ACodeSpace/demo/kodex-gradle-research-445/main-validation-445/mosaic-ip-local-jextract-445/results/mosaic-ip-local-jextract-v3.log`
- Compile success:
  `~/ACodeSpace/demo/kodex-gradle-research-445/main-validation-445/mosaic-ip-local-jextract-445/results/mosaic-tty-compile-jextract-v5.log`
- The public fork, gitlinks, local active worktree, and remote branches were
  not modified. Owned daemons were stopped and the shared device lock is
  free.
