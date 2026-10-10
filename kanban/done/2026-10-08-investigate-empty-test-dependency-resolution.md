# Task Tree

- `Read fixed-model constraints and prior ablation`() // complete
- `Inventory registry-owned test roots and source producers`() // complete
- `Inspect cached published plugin and framework contracts`() // complete, version limitation below
- `Hand off bounded opt-in candidate and runtime gates`() // complete

# Details

- **STATIC READY / RUNTIME BENEFIT PENDING.** Investigation archived; no adopted OOM fix.
- Own only this outer report. No product changes, build/test/Gradle/IDE execution,
  process control, MAT, remote execution, lock acquisition, Git commands or network writes.
- No temporary files/resources retained; read-only web lookups provided no requested-version evidence.
- User-attested production `5b41be52`: 203 KMP modules / 5026 source sets;
  fixed full targets/source hierarchy and 4 GiB budget, no JVM-only profile.
- Coordinator retains heavy/GUI ownership. Coordinator reports selective-source `probeNative`
  passes, graph/model next, full GUI serialized; this report neither reruns nor certifies those gates.
- Parent: [full-model repair](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L1-L25).
- Prior `noTestBalloon` removed only the plugin, retained framework dependencies,
  and did **no dependency resolution** in its configuration census: 684.107 → 681.038 MiB.
  That ~3.1 MiB is not a test-dependency-resolution memory measurement.
  [Ablation](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-08-measure-fixed-model-build-cost-ablation.md#L49-L69).
- **Version limitation:** local catalog and published cache are **1.0.1-K2.4.0**,
  not requested **1.2.0-K2.4.0**. No readable local 1.2.0 artifact was located;
  remote cached sources were not accessed. Do not silently upgrade or transfer exact-version guarantees.
  [Catalog](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/gradle/libs.versions.toml#L16).

## Registry-filtered inventory, not recursive build-time detection

- Parsed literal `moduleTree`/`module` nesting: **203** registered build-script owners.
  Read v2 index's 1309 entries without Git commands; joined `owner/src/<*Test>/...` to owners.
  [Registration contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L51-L90).
- **140 test-owning modules; 63 without tracked test files**, not 64.
  **168 nonempty default test roots; 425 Kotlin files + 1 JSON resource**.
  On-disk roots agree; no untracked files in those roots, no additional empty default test directories.
- Test-root owner counts: commonTest 109, mosaicTest 30, jvmTest 14, jsTest 5,
  linuxX64Test 3, mingwX64Test 3, posixTest 3, fileLoggingTest 1; categories overlap.
- **31 owners lack commonTest files:** 29 have only JVM/Mosaic roots
  (28 Mosaic-only; `app/impl/view` has Mosaic + JVM), `utils/ktor-client-ext/impl`
  has JVM + Linux x64, `utils/logging/impl` has fileLoggingTest. **Zero JVM-only owners.**
  Empty commonTest does not mean empty downstream compilation.
- 139 owners reference TestBalloon; all 140 reference kotlin.test. Exception
  [kRPC patch](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/src/commonTest/kotlin/kotlinx/rpc/internal/utils/map/SynchronizedHashMapTest.kt#L7-L15)
  uses `kotlin.test.Test`, not TestBalloon imports; preserve its full stack in the first candidate.
- Base conventions: shared 12, host 90, CLI 44; view 29, viewmodel 27,
  CLI-executable 1 inherit CLI. Empty-owner split: shared 6 / host 35 / CLI 22.
- No matching Main-source reference to TestBalloon, kotlin.test or org.junit;
  no project Main dependency declaration exports these test libraries.
  Main helper/fixture projects remain real modules, not removable “test projects.”
- Registry excludes buildSrc, fork submodules, CI Python harness and native C fixtures.

### Exact 63-owner complement of the 140 direct opt-ins

- `app/component/{account-usage,agent,composer,context-source-settings,history-index,hook-settings,mcp-settings,new-session,path-picker,request-user-input,runtime-configuration,session-delete,session-rename,session-sidebar,settings,suggest-subagent-task,usage-reset,working-directory}/spec`
- `app/settings/spec/persistence`, `app/spec/{application,session}`, `app/test-support/rpc`
- `openai/spec/{account-usage,client,client-test,codex-cli-storage,json-codec,model-catalog}`
- `agent-state/{spec/contract,test}`, `agent-context/spec/{agents-md,available-skill,contract,prefix,skill}`
- `agent-runtime/impl/composition`, `agent-runtime/spec/{contract,decorator/compact,decorator/steer,decorator/tool}`
- `agent-session/{spec/contract,test}`, `agent-storage/spec/contract`
- `tool/spec/{apply-patch,builder,current-time,get-context-remaining,image-generation,multi-agent,plan,request-user-input,tool-search,unified-exec,view-image,web-run}`
- `utils/{external-url,filesystem-lease,images-codec,kotlinx-io-coroutines,kotlinx-io-serialization,process-client,read-write-mutex,search-index}/spec`

## Actual source producers and callers

- Indexed root-build scripts/buildSrc source-root/fixture/applied-script scan:
  **one extra declared root**, generated version on **commonMain**; **zero custom/generated test roots**.
  [Producer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/build.gradle.kts#L7-L24).
- **Zero on-disk `build/generated/testBalloon` directories** across the 203 owners.
  Published 1.0.1 Gradle source has no entry-point file generator; compiler-created
  symbols are not extra filesystem roots. Future generators/custom roots require explicit opt-in.
- Keep fixture Main/callers: `agent-state/test`, `agent-session/test`, `app/test-support/rpc`
  and OpenAI client-test modules; no TestBalloon Main imports, but Main/helper dependencies remain.
  [Fixture contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/test-support/rpc/build.gradle.kts#L5-L20).
- Host's JS Mocha timeout stays; view/executable Mosaic hierarchy and viewmodel
  Koin compiler/runtime stay. Ordinary platform test source classpaths stay complete.
  [Host](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-host.gradle.kts#L9-L23),
  [VM](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-viewmodel.gradle.kts#L3-L25).
- No indexed script/CI diagnostics/extension caller; `tasks.named("jvmTest")` configures Integration's probe inputs.
  [Caller](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/build.gradle.kts#L65-L71).
- [KGP 2.4.0][kgp-cache] owns JVM registration (`KotlinJvmTestRunFactory.create`, 24–43)
  and `allTests` (`KotlinTestsRegistry`, 26–45); detachment preserves registration, not proven execution/report equivalence.

## Published cache contracts, versus development source

- CLI/host always call `configureCommonTests`; shared duplicates the same injection:
  commonTest `implementation(kotlin("test"))` + `implementation(framework-core)`.
  This reaches inherited empty platform test sets without checking source existence.
  [Helper](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt#L50-L55),
  [Shared](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-shared.gradle.kts#L43-L54).
- Published Gradle source: `de.infix.testBalloon.gradlePlugin.TestBalloonGradlePlugin.isApplicable`
  matches default source-set name via `[tT]est(?!Fixtures)` or a registered JVM suite;
  it does **not** check files/suite count. `testBalloon.testSourceSetsRegex=^---NONE---$`
  can make ordinary KMP test compilations inapplicable; registered JVM suites override that filter.
  This does not disable task hooks/launcher injection. There is no inspected general `enabled=false` switch.
- Published plugin injects framework-shared into Native compiler-plugin classpath and
  JVM RuntimeOnly launcher 1.13.4; configures JUnit Platform/reporting; adds `testBalloonDiagnostics`.
  [Exact Gradle source archive][gradle-cache] (`TestBalloonGradlePlugin.kt` 30–71;
  shared `TestBalloonGradleProperties.kt` 38–80, 203–217; `configureWithTestBalloon.kt` 49–69, 183–202).
- Published compiler **binary**, read without class execution:
  `de.infix.testBalloon.compilerPlugin.CompilerPluginFrontendExtension.getTopLevelClassIds/getTopLevelCallableIds`
  advertise JVM class / JS-Wasm main / Native entry property without a suite-count gate.
  `ModuleTransformer.visitModuleFragment$lambda$0` dispatches to `irJvmEntryPointClass`,
  `irSuspendMainFunction`, `irTestFrameworkEntryPointProperty` even for zero discovered suites.
- JVM generated FQCN: `de.infix.testBalloon.framework.shared.internal.entryPoint.JvmEntryPoint`;
  `testFrameworkDiscoveryResult()` calls `de.infix.testBalloon.framework.core.internal.initializeTestFramework`
  and constructs `de.infix.testBalloon.framework.shared.internal.TestFrameworkDiscoveryResult(arrayOf(...))`.
  Configuration eagerly looks up shared framework symbols; active compiler generation needs framework code.
  [Exact compiler JAR][compiler-cache], SHA256 `c2ac99afe4636d204063397006b0a15dff1e256e8e96ff25058ae5310e75aa75`.
- **Zero-source execution unknown:** KGP may skip as NO-SOURCE; generation when compiler runs
  does not prove invocation on empty input. Do not remove core alone while keeping applicability.
- Cached JVM engine `de.infix.testBalloon.framework.core.internal.integration.TestBalloonJUnitPlatformTestEngine`
  returns an empty descriptor if entry class absent and returns on zero top-level suites.
  This is engine behavior, not proof Gradle tasks succeed with no engine/classpath.
  [Exact runtime source archive][runtime-cache] (`jvmMain/.../TestBalloonJUnitPlatformTestEngine.kt` 50–55, 153–158).
- Cached descriptor's JVM API frontier: stdlib 2.4.0, framework-shared 1.0.1-K2.4.0,
  coroutines-core/test 1.10.2; runtime adds JUnit platform-engine 1.13.4,
  datetime 0.7.1 and atomicfu 0.32.1. Common metadata includes the latter two too.
  Shared JVM exports stdlib only. These are published edges, **not selected full consumer resolution**.
  [Cached JVM descriptor][descriptor-cache]; source counterpart confirms API versus implementation
  [declarations](file:///home/stream/ACodeSpace/push/BuildKodex/shared-context/testBalloon/testBalloon-framework-core/build.gradle.kts#L55-L69).
- [Counterpart](file:///home/stream/ACodeSpace/push/BuildKodex/shared-context/testBalloon/gradle.properties#L37) is `1.0.2-dev-1`; renamed adapter/FIR classes are not exact 1.0.1/1.2.0 compiler-source verification.

## Minimal candidate and coordinator-only acceptance

- Proposal: small `kodex.kmp-tests` convention with existing plugin + commonTest dependencies,
  applied directly in 140 owner scripts; remove those injections/plugin applications from base conventions.
  Keep helper if useful; all targets/sourceSets/hierarchies, publishing, Mosaic, Koin and versions unchanged.
- No recursive filesystem decisions, registry/manager, `TestFrameworkAddons` plan objects,
  source-model replacement or blanket `api` deletion. Empty commonTest inside a test owner stays funded.
- Conditional safety: 63 qualify on inspected sources/declarations, not a resolved transitive graph.
  Prove no hidden consumers; detach plugin/dependencies together. New tests/custom producers must opt in.
  The annotation-only kRPC owner still needs kotlin.test plus appropriate JVM JUnit engine/launcher;
  do not infer that dropping TestBalloon is safe there from absence of its imports.
- Future minimal reproducer: full-target control/opt-in copies of empty `tool/spec/current-time`,
  helper-only fixture owner, Mosaic-only Path Picker view, its Koin VM and annotation-only kRPC patch.
- Coordinator must first compare named `jvmTest`/`allTests` seams, zero-suite/NO-SOURCE
  outcomes, generated symbols and actual resolved compile/runtime/import classpaths;
  preserve every active-test library/source attachment and every Main/helper edge.
  Empty-owner test-library removal is intentional, not whole-model byte identity.
- Measure actual empty-owner test/IDE resolution peak/live heap separately from configuration,
  then full 4 GiB genuine GUI import/navigation with all 5026 source sets.
  No claim that 63/64 empty blocks cost 62 MiB, no measured resolution saving, no broad static signoff.
  Exact 1.2.0 inspection, runtime/task equivalence and OOM benefit remain pending; no tests run here.

[gradle-cache]: file:///home/stream/.gradle/caches/modules-2/files-2.1/de.infix.testBalloon/testBalloon-gradle-plugin/1.0.1-K2.4.0/469bed73d31a455d56d9dbefeeb4d8bd38e747cf/testBalloon-gradle-plugin-1.0.1-K2.4.0-sources.jar
[compiler-cache]: file:///home/stream/.gradle/caches/modules-2/files-2.1/de.infix.testBalloon/testBalloon-compiler-plugin/1.0.1-K2.4.0/95986156dd4710b4e6dff0f707d688cca6c1e3b0/testBalloon-compiler-plugin-1.0.1-K2.4.0.jar
[runtime-cache]: file:///home/stream/.gradle/caches/modules-2/files-2.1/de.infix.testBalloon/testBalloon-framework-core-jvm/1.0.1-K2.4.0/8504c37134cf81dd583ce05e212e0ceb9a73c92f/testBalloon-framework-core-jvm-1.0.1-K2.4.0-sources.jar
[descriptor-cache]: file:///home/stream/.gradle/caches/modules-2/metadata-2.107/descriptors/de.infix.testBalloon/testBalloon-framework-core-jvm/1.0.1-K2.4.0/26c913274550a0b2221f47a0fe2d2358/descriptor.bin
[kgp-cache]: file:///home/stream/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-gradle-plugin/2.4.0/8cc8a31f5f34bbace6166c7ad0187724c345932f/kotlin-gradle-plugin-2.4.0-sources.jar
