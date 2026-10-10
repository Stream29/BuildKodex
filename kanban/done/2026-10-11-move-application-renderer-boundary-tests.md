# Task Tree

- `Locate the renderer-only dependency and original test boundaries`() // complete
- `Move real root renderer tests to the existing integration project`() // complete
- `Remove renderer compilation support from the Application implementation`() // complete
- `Verify original cases and supported Native compilation on Xiaoxin`() // complete
- `Record scoped results and update the board`() // complete

# Details

- User first authorizes moving tests, then explicitly authorizes committing
  and pushing this batch. No new release is authorized.
- Baseline is published product `f22629c3847ca58c16d85690d27157f3ff50bb65`.
- `ApplicationRootBoundaryTest` needs Compose/Mosaic only for JVM renderer tests;
  its `commonMain` compileOnly runtime declaration leaks into four Native targets.
- Reuse `integration-test`; preserve all eight generated cases, RPC fault gates,
  exact-instance assertions, cancellation behavior and structured cleanup.
- Keep internal implementations internal. Expose typed creation functions used
  by both the actual composition root and integration fixture, returning existing
  Application/Sidebar contracts; do not add test-only production hooks.
- Leave non-renderer implementation tests in their current project.
- Keep official KGP, versions, target declarations, source hierarchy and heap
  budgets unchanged. Do not suppress the warning or export Mosaic as Application API.
- Local machine is reserved for the user. Use Xiaoxin's existing heavy-resource
  lock and explicit Java25 for Gradle; do not take over an existing IDE/daemon.
- Gates: original Application tests, all eight relocated root-renderer cases,
  offline integration cases, affected JVM/Native compilation, and absence of
  `IncorrectCompileOnlyDependencyWarning` in the actual build output.
- General IDE OOM/navigation work remains open; this task is not its repair.

## Implementation

- Move the original file to
  `Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/ApplicationRootBoundaryTest.kt`.
  The textual comparison changes only one contract import, the fixture's
  Application type and two construction calls. No test body, fault gate,
  assertion, timeout or teardown order changes.
- `createApplicationViewModel` and `createSidebarSettingsViewModel` construct
  the existing internal implementations and return the existing contracts.
  Both actual `withKodexApplication` and the integration fixture use these
  factories; no new owner, facade, RPC hook or renderer dependency is introduced.
- Remove Application's Compose plugin, JVM View/Mosaic test dependencies and
  commonMain Mosaic compileOnly. Put the fixture's actual dependencies in the
  integration project's JVM test configuration; reuse its existing Compose
  plugin and Mosaic runtime/testing dependencies.
- No changes to catalog, wrapper, build conventions, versions, targets,
  persisted data or wire shapes. The non-renderer Application tests remain.

## Xiaoxin validation

- Isolated tracked-source snapshot: 1,312 files; SHA256 manifest digest
  `2b7f4bf4f5f0267b1a3d802ed8377c8114587142962c49ca8da2387fdae06fa7`.
- Explicit Java25, Gradle9.5.1, official KGP2.4.0. Raw loaded plugin SHA256:
  `ca5903a236a19a54a883a8695958d8516f9e39cc333bd1e3077f92ae38271cd6`.
  Provenance confirms the experimental parser/CInterop implementations are absent.
- Existing private dependency/Native cache reused; Gradle build cache and
  configuration cache disabled. Tests forced to execute. Keep the original
  4GiB Gradle/2GiB Kotlin maximum heaps, 4GiB host reserve and heavy-resource lock.
- Stage one: Application **44** and CLI **4** JVM tests pass. Application
  `compileKotlinLinuxX64`, `compileKotlinLinuxArm64` and
  `compileKotlinMingwX64` actually execute successfully.
- `compileKotlinMacosArm64` is **skipped** because the Linux host cannot process
  required macOS cinterops. It is not counted as a successful macOS compilation;
  no macOS runtime or four-platform release is claimed.
- Stage two: relocated root-renderer **8** tests and the enabled offline patch
  renderer probe **1** pass. Stage three: root-renderer **8** pass again.
  **65 executions, 57 distinct cases; zero failures, errors or skips.**
  All three Gradle requests succeed; controller elapsed time is 324.690 seconds.
- All three actual build logs contain zero occurrences of the unsupported
  compileOnly diagnostic. No suppression property or warning-policy change.
  This verifies Gradle configuration/compilation, not a new successful IDEA Sync.
- No model-provider calls, release or CLI installation. Copied credentials
  removed; post-cleanup inventory finds zero Gradle/Kotlin daemons.
- Private result, XML and logs remain under Xiaoxin
  `~/ACodeSpace/demo/kodex-renderer-split-20261011/gate-r01/`;
  the regenerable temporary project and local staging scripts are removed.

## Submission

- Signed product commit `1b6ad5d3`:
  `fix: isolate Application renderer tests in integration project`,
  pushed to `Stream29/Kodex` main. Git recognizes the test move as a 98% rename.
- The corresponding BuildKodex batch contains only this validation record,
  the frontend test-boundary guidance and the new Kodex gitlink.
- Preserve unrelated user changes. Do not bump a version, modify release tags
  or replace the installed CLI as part of these commits.
