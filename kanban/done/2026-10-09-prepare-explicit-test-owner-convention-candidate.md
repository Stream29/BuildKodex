# Task Tree

- `Verify the fixed inventory of real test owners`() // complete, static only
- `Prepare an explicit opt-in test convention in an isolated overlay`() // complete
- `Record source, task and runtime gates without claiming savings`() // complete; coordinator gates pending

# Details

- **PREPARED / VALIDATION PENDING.** Independent preparation is finished;
  no accepted OOM repair, resolved graph equivalence or measured heap benefit.
- Parent:
  [full import repair](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L1-L25).
- Rechecked the earlier
  [583 static report](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-08-investigate-empty-test-dependency-resolution.md#L1-L60).
  Fixed accepted source is `5b41be52d2f64a12ea3671bcecb9151fd2d1fa60`,
  KGP2.4.0 / Gradle9.5.1 / TestBalloon1.0.1-K2.4.0.
- All writes used `apply_patch`, confined to the candidate directory and this
  handoff. No production/fork/other-document edits, builds, test execution,
  Gradle/IDE/plugin execution, device/process control, network writes,
  credential access or Git commands. No Xiaoxin/MAT/lock operation.
- Only intended candidate assets are retained; no temporary scripts or live
  resources. Coordinator owns heavy work and adoption.

## Exact early handoff and scope

- Overlay root:
  [candidate directory](file:///tmp/kodex-explicit-test-owner-candidate-20261009/).
  Paths are relative to a **fresh isolated Kodex root**, not BuildKodex.
  This is a partial overlay, not a standalone build or source replacement.
- **144 build inputs:** 140 actual module `build.gradle.kts` files and 4
  `buildSrc/src/main/kotlin/*.gradle.kts` files. No business sources.
- Actual accepted source uses three precompiled base conventions, not the
  prompt's `Root`/`ModuleBasePlugin`/`StructuredSourceSets` classes. It has no
  base Poko application. Do not invent those types or transplant another build.
- Explicit user response authorized the minimum **four total buildSrc files**
  after the original 1–3-file ceiling was shown incompatible with this source.
  Existing `KodexHostKmp.kt` helper stays byte-identical.
- Added:
  [kodex.kmp-tests](file:///tmp/kodex-explicit-test-owner-candidate-20261009/buildSrc/src/main/kotlin/kodex.kmp-tests.gradle.kts#L1-L8).
  Modified only:
  [CLI base](file:///tmp/kodex-explicit-test-owner-candidate-20261009/buildSrc/src/main/kotlin/kodex.kmp-cli.gradle.kts#L1-L11),
  [host base](file:///tmp/kodex-explicit-test-owner-candidate-20261009/buildSrc/src/main/kotlin/kodex.kmp-host.gradle.kts#L1-L22),
  [shared base](file:///tmp/kodex-explicit-test-owner-candidate-20261009/buildSrc/src/main/kotlin/kodex.kmp-shared.gradle.kts#L1-L39).
- Three non-overlay receipts:
  [Baseline.sha256](file:///tmp/kodex-explicit-test-owner-candidate-20261009/Baseline.sha256#L1-L143),
  [SHA256SUMS](file:///tmp/kodex-explicit-test-owner-candidate-20261009/SHA256SUMS#L1-L144),
  [Inventory.tsv](file:///tmp/kodex-explicit-test-owner-candidate-20261009/Inventory.tsv#L1-L204).
  Do not copy receipt files into the isolated product build.
- Baseline manifest covers the 143 existing changed files; the new tests
  convention must be absent before application. Candidate manifest covers all
  144 destination files. Both manifests were checked against actual bytes.
- Receipt SHA-256:
  - Baseline: `c1ed3b51c03918edf542f5f43a2c02ba9a53d7d2414e7ff08cc93533fcf069cb`
  - Candidate sums: `f7c30b94dbec4ed5bd459246d1f0c767c27cb6a1ca0f50a36316a6aedbbfa5c5`
  - Inventory: `b2332d2857859e165511072f23a8f313939e3d2735740d448a0468fb7b51a9d0`
- This factor is independent of direct-kRPC. Four RPC scripts overlap the
  [separate direct overlay](file:///tmp/kodex-direct-krpc-candidate-20261008/SHA256SUMS#L1-L5).
  Coordinator must compose their exact diffs, not overwrite direct dependencies
  with these accepted-source copies. Choosing that combination is not authorized
  or validated by this preparation.

## Static source inventory

- Parsed actual literal `moduleTree`/`module` declarations:
  **203 distinct registered build-script owners**, not a guessed recursive
  module count. Joined only each owner's real `src` roots; no build-time detector.
  [Registration](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L51-L90).
- **140 test owners / 63 empty owners; 168 nonempty test roots;
  425 Kotlin files + 1 JSON resource.** Inventory records exact directory,
  flat project ID, root counts and file count for all 203 owners.
  Default `src/<name>Test` roots include additional `fileLoggingTest`.
- Root-owner counts, overlapping: common 109, Mosaic 30, JVM 14, JS 5,
  Linux x64 3, MinGW x64 3, POSIX 3, fileLogging 1.
- **31 owners have no commonTest files:** 28 Mosaic-only,
  `app/impl/view` JVM+Mosaic, `utils/ktor-client-ext/impl` JVM+Linux x64,
  `utils/logging/impl` fileLogging. All 31 explicitly opt in, funding inherited
  commonTest dependencies even where commonTest itself has no source files.
- Source reference scan: all 140 use kotlin.test, 139 use TestBalloon.
  Annotation-only
  [Native-map patch test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/src/commonTest/kotlin/kotlinx/rpc/internal/utils/map/SynchronizedHashMapTest.kt#L7-L42)
  stays a full plugin+framework owner, not an exception to the first gate.
- No inspected owner Main Kotlin source references TestBalloon/kotlin.test/JUnit.
  No Main test-library exports found. This is source/declaration evidence of
  no known source ABI consumer, **not proof about selected transitive graphs**.
  Baseline injects `implementation`, not literal `api`; no unrelated API is removed.
- Source-root/producer lookup found only generated **commonMain** version input,
  not custom/generated Test input declarations. No owner's on-disk
  `build/generated/testBalloon` directory was found.
  [Existing producer and exclusion](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/build.gradle.kts#L7-L48)
  remain identical, including its historical test exclusion.
- Compiler-created entry points are not filesystem Test roots. No new tests,
  generators, fixtures or helper projects are invented or merged.
  Main fixtures in agent-state/test, agent-session/test, app/test-support/rpc
  and OpenAI client-test projects remain ordinary dependencies.
- Source fingerprint over 426 sorted records:
  `SHA256(UTF8(concat(sorted("<file SHA256>  <Kodex-relative path>\n"))))`
  where `\n` represents a newline byte.
  = `5d4908fa7bede06f945fe585ed2bc63a47f5df4b7e614080d1edc4a3bf09c76f`.
  Records cover all files recursively under those actual nonempty Test roots,
  including excluded historical source and the JSON resource; not a test count.

## Minimal convention and intentional differences

- The new 8-line precompiled convention applies KMP for its actual typed Kotlin
  accessor, then the existing `de.infix.testBalloon` plugin and unchanged
  `configureCommonTests(project)`. It declares no targets/hierarchy/tasks.
  KMP plugin application is idempotent; no duplicate registration or reflection.
- Remove TestBalloon application from all three bases and their common-test
  injection. Shared's equivalent inline injection and now-unused import go too.
  Each of 140 modules only adds one tests-convention ID to its existing plugins block.
  No root callback/auto-application across 203 modules; 63 have no opt-in.
- Reuse the actual
  [helper](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt#L50-L55):
  commonTest `implementation(kotlin("test"))` and catalog framework-core.
  KGP hierarchy inheritance carries these libraries into the owner's actual
  JVM/JS/Native/Mosaic/fileLogging Test sets; do not change them to Main exports
  or assume the framework POM exports kotlin.test.
- Existing target/hierarchy declarations, coordinates, publishing, repositories,
  serialization/Compose/kRPC/Koin plugin applications and catalog/buildSrc
  dependency versions stay unchanged. No manager/registry/profile/facade or
  reflective classloader lookup. No macrobase integration is introduced.
- Koin ViewModel compileSafety/strictSafety/unsafeDslChecks stay true:
  [actual VM convention](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-viewmodel.gradle.kts#L3-L25).
  Actual Mosaic helper/hierarchy and per-module dependency configurations remain;
  JVM/Native test annotations and source files are untouched.
- All 140 scripts were byte-compared to originals plus exactly one plugin ID.
  This also preserves Integration's performance-probe properties, enabled branch
  and `tasks.named("jvmTest")` inputs/cache behavior:
  [candidate Integration](file:///tmp/kodex-explicit-test-owner-candidate-20261009/integration-test/build.gradle.kts#L7-L73).
- **Expected task difference:** 63 fewer `testBalloonDiagnostics` tasks.
  KGP's declared targets/test source sets/compilations/compile/test/report tasks,
  including empty `jvmTest`/Native test seams and `allTests`, are not removed.
  No task disabling, fake success, failOnNoDiscoveredTests override or engine shim.
- Compiler configurations/options/task inputs do intentionally change on 63:
  TestBalloon applicability, Native shared compiler-classpath injection, launcher
  dependencies and task hooks disappear together with commonTest injection.
  The new precompiled convention also changes buildSrc's generated plugin
  compilation/accessor work. **Actual total/compiler-related task counts pending**;
  unchanged declared KGP task seams are not a claim of identical task count.
- Plugin application moves later than its former base application on real owners.
  Exact support-plugin/engine/task behavior, including Koin interaction, requires
  real validation. Static script equivalence is not plugin execution equivalence.
- Exact cached 1.0.1 plugin source was read, not executed. Its applicability
  matches Test compilation names/JVM suites, not actual source/suite presence.
  Detaching plugin and runtime together avoids retaining an active zero-suite
  generator without its framework. Do not use a wrong compiler configuration
  classpath, presumed macrobase Context no-op or version-helper injection as
  evidence that runtime symbols are unnecessary.
- Future tests/resources/custom or generated Test roots require the developer
  to explicitly add `id("kodex.kmp-tests")` in that real module's plugins block.
  Do not automatically remove opt-in when a directory becomes empty; an owner
  may still compile zero suites or receive generated tests.

## Coordinator-only acceptance gates

- Check fresh baseline hashes, 144 overlay hashes and exact 140/63 ownership;
  enumerate generated/custom roots and hidden dependency consumers independently.
  Reuse no incorrect 203-owner inventory inferred from arbitrary build scripts.
- First compile the actual precompiled convention at unchanged versions.
  Reject inability to separate actual plugin applicability/helper symbols;
  investigate actual types rather than inventing a Root/ModuleBase abstraction.
- Compare all 203 module models: target names, complete source-set hierarchy
  (accepted 5026 control), language/compiler/Werror settings, dependencies,
  binaries/cinterop and navigation. Intentional 63-empty Test dependency removals
  are the only library-model ablation; active Test/Main graphs must stay funded.
- Compare actual compile/runtime/import and **compiler-plugin** classpaths and
  plugin arguments on positive, annotation-only, platform-only and empty owners.
  Verify the requested compiler plugin is really loaded for each positive
  compilation; a misselected configuration or cached/NO-SOURCE run is not proof.
- Controls: `rpc/spec/models`, `openai/spec/models` for real models/source ABI;
  `rpc/impl/krpc-utils-patch` for annotation-only test and Native map/module identity;
  `utils/ktor-client-ext/impl` for JVM+Linux-only ownership;
  `mcp/spec/stdio` and `utils/shell-client/impl` for actual JS Test behavior;
  `app/view/components`, Path Picker view and its VM for Mosaic/Koin;
  `utils/logging/impl` for fileLogging; Integration with probe enabled;
  `tool/spec/current-time` and Main-only fixture owners for empty seams.
- Execute existing tests with unchanged expected outcomes/counts, sources,
  annotations, resource loading and fixture dependencies. For Patch, explicitly
  verify actual annotation test execution/engine selection rather than treating
  zero discovered JVM suites as success; preserve its Native regression too.
- Require generated JVM/JS/Native entry points and framework symbols whenever
  the real compiler executes on an opted-in owner, **including 0 TestBalloon suites**.
  Distinguish actual execution from cache hits/NO-SOURCE. On 63 detached owners,
  verify no TestBalloon entry-point generation and genuine KGP empty-task outcomes.
- Enumerate standard and plugin/compiler-related task paths before/after; report
  intentional diagnostics and buildSrc differences openly. Preserve standard
  KGP registration even when no tests execute. Any extra loss is a blocker.
- Measure resolution-time peak/live heap separately, then genuine complete
  4GiB GUI import and full source navigation with all targets. Previous 3MiB
  configuration-only plugin delta is neither this experiment nor heap benefit.
- All runtime/model/source-navigation/heap/GUI gates remain **PENDING**.
  Preparation retained the isolated candidate only, not an adopted fix or commit.
- Final static audit passed: 147 candidate files, 144 scoped build inputs,
  both hash manifests, 203 inventory rows and exact module/base diffs.
  A first audit assertion modeled a removed shared-block blank line incorrectly;
  corrected the read-only checker and reran successfully without changing assets.
