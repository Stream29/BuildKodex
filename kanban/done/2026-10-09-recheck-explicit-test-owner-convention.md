# Task Tree

- `Independently review the fixed test-owner overlay`() // complete, read-only
- `Separate source equivalence from runtime and memory acceptance`() // complete

# Details

- **INDEPENDENT STATIC READY — bounded coordinator validation may proceed;
  adoption BLOCKED.** Not Session621's preparation; no OOM fix or 9% benefit proved.
- **R** = verified result; **B1** = confirmed blocker; **B2** = concrete pending
  acceptance gate; **U** = unknown. B1: none found in this fixed static overlay.
- Reviewed [handoff621][handoff], [archived583][prior], [parent repair][parent], actual bases and [fixed-model/spec requirements][spec].
- Only this report was written, with `apply_patch`. No code/other-document edits,
  builds/tests/plugin execution, IDE/MAT/device/process control, network writes,
  Git commands or credentials. ZIP reads/bytecode disassembly did not execute
  plugin classes. No temporary assets or owned live resources remain.

## R — fixed bytes and actual ownership

- Accepted ref read as `5b41be52d2f64a12ea3671bcecb9151fd2d1fa60`; all **143**
  [baseline inputs][baseline] match local bytes; new convention absent there.
  This is scoped input verification, not an independent whole-product clean audit.
- Rehashed **144/144** [candidate inputs][sums]; exactly those inputs plus three receipts. Manifest SHA-256:
  `f7c30b94dbec4ed5bd459246d1f0c767c27cb6a1ca0f50a36316a6aedbbfa5c5`.
- Baseline receipt SHA `c1ed3b51c03918edf542f5f43a2c02ba9a53d7d2414e7ff08cc93533fcf069cb`;
  [inventory][inventory] SHA `b2332d2857859e165511072f23a8f313939e3d2735740d448a0468fb7b51a9d0`.
- Independently parsed literal [registration][registration], then joined actual
  owner `src` directories: **203 owners, 140 positive, 63 empty, 168 roots**.
  All140 script differences are exactly one opt-in, including compact plugins blocks.
- **31 platform-only owners** retain opt-in: 28 Mosaic-only; app/impl/view
  JVM+Mosaic; ktor-client-ext/impl JVM+Linux; logging/impl fileLogging.
  All140 reference kotlin.test;139 reference TestBalloon; [annotation Patch][patch]
  remains opted in. Empty commonTest inside a positive owner still needs funding.
- Recomputed all **426** sorted `<SHA256>  <Kodex-relative path>\n` records (**425 Kotlin + 1 JSON**), fingerprint
  `5d4908fa7bede06f945fe585ed2bc63a47f5df4b7e614080d1edc4a3bf09c76f`.
  No Main Kotlin test-library references or explicit test-library exports found.
- Source-root declarations show only generated commonMain version input; no
  custom/generated Test declaration or on-disk owner `build/generated/testBalloon`.
  Main fixtures remain modules/dependencies. Future genuine Test generators,
  resources or source roots require explicit opt-in; never remove it merely
  because an owner's current directory becomes empty.

## R — minimal real responsibility, not a replacement model

- [Eight-line convention][convention] owns plugin+runtime together, reuses
  [actual helper][helper]; KMP application is idempotent and enables the typed
  accessor. It declares no targets/tasks/hierarchy, manager, reflection or detector.
  Removing it would scatter plugin/helper application across140 scripts; its KMP
  declaration supplies the precompiled accessor prerequisite, not useless complexity.
- Three bases lose only TestBalloon/commonTest injection and shared's unused
  import/block whitespace. All target/compiler/Werror/publishing declarations
  remain; helper SHA `526ef08473259796c5da825a2ee9a6fd12f7075b485c9cbc43950daeedc1139c`.
  No module merge, profile, API rewrite or fabricated entry point.

## R / B2 — later application is supported, not proven equivalent

- Exact cached **1.0.1-K2.4.0** [Gradle plugin][tb] SHA `2a65ccb7c08732eafa0ee2c8a7458f7b86b18859f103301932353cc7edc0e1e0`.
  TestBalloonGradlePlugin.kt L30–77 injects Native shared and defers applicability/options.
  configureWithTestBalloon.kt L49–119 uses configuration/target/compilation
  `configureEach`, covering existing and future containers; no source-count guard.
- Exact [KGP2.4 sources][kgp]: KotlinPluginWrapper.kt L279–283 creates Native
  compiler configuration before targets; native configuration factory L204–210
  extends it. JVM/common/native processors load subplugins after buildscript
  evaluation; JS uses whenEvaluated. Later opt-in is not inherently too late.
- **No JVMTest task replacement** in this exact plugin: it configures existing
  `Test` tasks with JUnit Platform (configureWithTestBalloon.kt L183–203);
  only explicit task registration is diagnostics L408–414. KGP owns jvmTest,
  Native test compilation and allTests registration.63 diagnostics disappear;
  standard KGP seams remain declared, not proven runtime/task-count equivalent.
- **B2/order:** TestBalloon moves after target creation and Koin/Compose on
  derived conventions. [KGP][kgp] SubpluginEnvironment.kt L20–60 iterates applied
  support plugins; relative plugin/options/classpath order can change.
  [Koin][koin] KoinGradlePlugin.kt L98–134 retains all-compilation applicability,
  strict-safety cache predicates and moduleId; [Compose][compose] L33–48 defers
  compilation options. Neither provides a static commutativity guarantee.
- **B2/task actions:** plugin taskGraph.whenReady L213–223 explicitly documents
  ordering sensitivity; JS L364–373 resets filters and may set
  failOnNoDiscoveredTests=false. No candidate override is added, but detachment
  removes those existing hooks on63; verify genuine empty outcomes, not fake success.
- Exact [compiler binary][compiler] SHA `c2ac99afe4636d204063397006b0a15dff1e256e8e96ff25058ae5310e75aa75`:
  frontend advertises entry symbols; ModuleTransformer's platform dispatch has
  no zero-suite guard and requires framework symbols. **Plugin and runtime must
  detach together on63.** [JVM engine][engine] L50–55/L153–158 permits absent entry/
  zero suites; that does not establish annotation-engine execution or Gradle success.

## B2 / U — coordinator-owned gates; stop static review here

- Fresh buildSrc compilation is **pending in this review**; check exact accessor/plugin/helper compilation first.
- Compare all203 final models: full targets/5026 control source sets, ancestry,
  navigation, Werror/options, binaries/cinterop/API and Main/active Test graphs.
  Only63 unused test-library/plugin ablations are allowed; enumerate total tasks.
- Compare actual compiler-plugin classpaths/arguments and loaded plugins, not
  guessed configurations. Fresh real JVM/Node/Native, Mosaic/Path Picker+Koin,
  fileLogging and enabled Integration tests must retain outcomes/counts/resources.
  Patch needs actual annotation-engine execution plus Native map identity/regression.
- Genuine compiler execution on opted-in **0-suite** owners must retain generated
  entry points/runtime symbols; detached63 must not generate TestBalloon entries.
  Cache hits/NO-SOURCE are not compiler execution evidence; detach launcher/reporting/Native shared hooks only on63.
- **U:** selected transitive/hidden consumers, runtime ordering, resolution peak/
  live heap and complete genuine **4GiB GUI import/navigation** remain unproved.
  Parent's live PayloadSerializer buffer is385,903,112 bytes (~368MiB); duplicate
  holder-map identity query remains pending. No task-manager/source-provider/reflection remedy follows.

[handoff]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-09-prepare-explicit-test-owner-convention-candidate.md#L1-L185
[prior]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-08-investigate-empty-test-dependency-resolution.md#L1-L143
[parent]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L199-L226
[spec]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-02-plan-gradle-model-resource-optimization.md#L49-L76
[baseline]: file:///tmp/kodex-explicit-test-owner-candidate-20261009/Baseline.sha256#L1-L143
[sums]: file:///tmp/kodex-explicit-test-owner-candidate-20261009/SHA256SUMS#L1-L144
[inventory]: file:///tmp/kodex-explicit-test-owner-candidate-20261009/Inventory.tsv#L1-L204
[registration]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L51-L90
[patch]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/src/commonTest/kotlin/kotlinx/rpc/internal/utils/map/SynchronizedHashMapTest.kt#L7-L42
[convention]: file:///tmp/kodex-explicit-test-owner-candidate-20261009/buildSrc/src/main/kotlin/kodex.kmp-tests.gradle.kts#L1-L8
[helper]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt#L13-L55
[tb]: file:///home/stream/.gradle/caches/modules-2/files-2.1/de.infix.testBalloon/testBalloon-gradle-plugin/1.0.1-K2.4.0/469bed73d31a455d56d9dbefeeb4d8bd38e747cf/testBalloon-gradle-plugin-1.0.1-K2.4.0-sources.jar
[kgp]: file:///home/stream/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-gradle-plugin/2.4.0/8cc8a31f5f34bbace6166c7ad0187724c345932f/kotlin-gradle-plugin-2.4.0-sources.jar
[koin]: file:///home/stream/.gradle/caches/modules-2/files-2.1/io.insert-koin/koin-compiler-gradle-plugin/1.0.1/91a688c7434df957b35529d324c1ec81f2bd59a8/koin-compiler-gradle-plugin-1.0.1-sources.jar
[compose]: file:///home/stream/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/compose-compiler-gradle-plugin/2.4.0/158a9df5c446e3f24566b370ee2cdd54e7e2b036/compose-compiler-gradle-plugin-2.4.0-sources.jar
[compiler]: file:///home/stream/.gradle/caches/modules-2/files-2.1/de.infix.testBalloon/testBalloon-compiler-plugin/1.0.1-K2.4.0/95986156dd4710b4e6dff0f707d688cca6c1e3b0/testBalloon-compiler-plugin-1.0.1-K2.4.0.jar
[engine]: file:///home/stream/.gradle/caches/modules-2/files-2.1/de.infix.testBalloon/testBalloon-framework-core-jvm/1.0.1-K2.4.0/8504c37134cf81dd583ce05e212e0ceb9a73c92f/testBalloon-framework-core-jvm-1.0.1-K2.4.0-sources.jar
