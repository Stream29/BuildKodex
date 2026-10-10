# Task Tree

- `Freeze the fork-independent Kodex Gradle baseline`()
- `coroutineScope {`
  - `val modelInventory = async {`
    - `Inventory settings, project count, KMP targets, source sets, variants, repositories, and configuration-time inputs`()
  - `val buildLogicInventory = async {`
    - `Inventory buildSrc, convention plugins, cross-project callbacks, eager configuration, and task realization`()
  - `val ideBaseline = async {`
    - `Measure real IDEA Sync, model import, smart completion, CPU, RSS, and repeatability on the weak Linux host`()
  - `val cliBaseline = async {`
    - `Measure representative JVM and Native task paths with daemon, configuration cache, build cache, and file-system watching states`()
  - `modelInventory.await()`
  - `buildLogicInventory.await()`
  - `ideBaseline.await()`
  - `cliBaseline.await()`
- `}`
- `if (baselineIsReproducibleAndForkIndependent()) {`
  - `Rank project-only bottlenecks by measured wall time, invalidation scope, CPU, RSS, and developer frequency`()
- `} else {`
  - `Stop optimization experiments and return the missing baseline or environmental confounder`()
- `}`
- `coroutineScope {`
  - `val settingsTrack = async {`
    - `Prototype explicit moduleTree registration without changing project identity or source-set semantics`()
    - `Measure settings I/O, module graph construction, and model equivalence`()
  - `}`
  - `val buildLogicTrack = async {`
    - `Prototype buildSrc to build-logic migration and configuration-avoidance repairs`()
    - `Measure build-script compilation, invalidation scope, and real Sync impact`()
  - `}`
  - `val targetTrack = async {`
    - `Freeze every existing target and verify unchanged model source and release semantics`()
    - `Verify source roots, expect/actual, hierarchy templates, and navigation before claiming a performance gain`()
  - `}`
  - `val resolutionTrack = async {`
    - `Audit repositories, dependency resolution, variant selection, cinterop registration, and eager metadata work`()
  - `}`
  - `settingsTrack.await()`
  - `buildLogicTrack.await()`
  - `targetTrack.await()`
  - `resolutionTrack.await()`
- `}`
- `for (candidate in measuredCandidates()) {`
  - `ApplyExactlyOneCandidateInAnIsolatedWorktree(candidate)`
  - `if (modelNavigationCompileAndTargetSemanticsRemainEquivalent()) {`
    - `Run repeated real IDEA Sync and representative CLI measurements`()
    - `KeepCandidateOnlyIfTheRelevantDeveloperScenarioImproves()`
  - `} else {`
    - `RollbackCandidateAndRecordTheConcreteRegression()`
  - `}`
- `}`
- `Validate native Gradle mechanisms without making them assumed solutions`()
  - `Validate configuration cache for CLI tasks and record that it is not an IDEA Sync cache`()
  - `Validate org.gradle.tooling.parallel independently from task parallelism`()
  - `Run Isolated Projects diagnostics only after project-local violations are inventoried`()
  - `Do not use dangerously-ignore or undocumented internal model caches`()
- `Validate the final project-only optimization set on Linux, macOS, and Windows`()
- `Review measured rollout, rollback, fixed-target acceptance and developer documentation with the user`()

# Details

## Current implementation status

- 用户已授权落地并部署本期 Gradle 改进。三个固定 fork 包与默认
  binary consumer 已完成；显式moduleTree和SDK维护版消费也已单独落地。
- 初始产品基线v0.4.10 /83749434；当前main已包含9458be6f与2e5c0df4，
  203个非root实际build-script项目、216个注册项目，原targets/source模型等价。
  每个候选单独验证并回滚，旧142项目计数仅为历史研究记录。
- [主实施树](2026-10-02-rescue-gradle-development-experience.md)
  统一构建/资源锁与提交；本机不运行构建或接管 IDE，Xiaoxin 承担隔离测量。
- 2026-10-08 用户否决目标开关：targets 已削减过，不再调整。
  撤回 JVM-only/host-only profile；优化只减少无效配置和解析工作，
  不减少现有 target、source set、生成代码或平台导航。
- 本文后续 planning 禁止语句属于历史授权边界；本节替代其实施授权，
  不替代模型等价、资源停止条件、版本兼容性和有证据才保留的验收规则。

## Scope

- This plan extends the fork publication work but deliberately excludes the three external forks from the primary optimization target.
- Forks are consumed as already-published binary dependencies during this study. Source composite behavior remains a separate explicit maintainer mode.
- The subject is the current Kodex root build: explicit settings, its203-project
  build-script model, `buildSrc`/convention logic, KMP targets/source sets,
  repository/variant resolution, compiler execution and genuine IDE model import.
- No API, spec/impl boundary, serialization contract, RPC contract, or source-root meaning may be changed merely to reduce Gradle cost.
- This is a planning task. It does not authorize changes to the active local Kodex worktree, fork branches, gitlinks, CI, package versions, or user IDE state.

## Evidence baseline

- Existing real GUI evidence on Xiaoxin shows complete binary-fork Sync at approximately 71.7 seconds native import, while source-fork Sync is approximately 108.3 seconds. The remaining cost is therefore primarily in Kodex’s own model/import path, not only the external forks.
- Existing evidence separates approximately 4.1 seconds of root/subproject configuration from approximately 57.2 seconds of Gradle model work and approximately 14.0 seconds of post-callback IDE import for the binary scenario.
- The full binary scenario still has about 3,437 `Kodex.` IDE modules and 295 repository source roots. These are IDE model counts, not a claim that Kodex has 3,437 Gradle subprojects.
- The weak-host evidence shows approximately 5 GiB IDEA RSS and approximately 5 GiB Gradle RSS in the full binary Sync control group; a JVM-only profile reduced Gradle RSS but intentionally hid Native model semantics and is not a full replacement.
- Maven Local binary consumer validation succeeded for Mosaic, LuceneKmp, and KotlinMcpSdk, including `:mcp-stdio:compileKotlinJvm`; it did not prove final GitHub Packages publication, full Native closure, or IDEA Sync.
- Evidence reports:
  - [IDEA Sync and resource report](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/REPORT.md)
  - [Maven Local consumer report](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/package-consumer-mavenlocal-20261004/REPORT.md)
  - [GitHub Packages planning](../done/2026-10-04-plan-github-packages-fork-publication.md)

## Xiaoxin baseline and first candidate result

- 2026-10-04 在 Xiaoxin 的 exact main binary-consumer 隔离副本上完成了项目级 CLI profile；Gradle 9.5.1、Temurin 25、2 workers、`-Xmx2g`，所有重型操作持有 `device-heavy.lock`。
- 当前设置注册 142 个 Kodex root modules；`gradle.properties` 已启用 configuration cache 和 parallel execution，但未显式设置 `org.gradle.tooling.parallel`。静态扫描未发现 root/convention 中广泛使用的 `allprojects`、`afterEvaluate`、`configurations.all` 或 `tasks.all` 模式；root 只有一个 `subprojects { configurations.configureEach { ... } }` kRPC substitution。
- 将 13 次 filesystem-walking `includeModuleTree` 替换为保持相同 142 个 project path 的显式 `includeModuleDir` 候选后，project list 完全一致；warm `help --no-configuration-cache --parallel` 为 baseline `13.92–15.58s`、候选 `14.85–19.06s`，没有可重复收益，候选不进入生产。
- Gradle profile 的 warm baseline 中 `Settings and buildSrc` 约 `0.03s`、`Loading Projects` 约 `0.25s`、`Configuring Projects` 约 `6–7s`；第一次运行的 `11m39s` 包含 `6m33s` startup/distribution/cache 初始化和 `54s` project configuration，不能归因给 `buildSrc` 单项。
- 结论：显式 `moduleTree` 仍有输入确定性、命名和失效范围的维护价值，但当前证据不支持把它当作主要 Sync 提速手段。后续优先研究 KMP 模型/variant/cinterop、依赖解析和真实 Tooling model。
- 远端报告：`~/ACodeSpace/demo/kodex-gradle-research-445/project-gradle-optimization-20261004/REPORT.md`；原始 profile、日志和 model list 在其 `results/`、`baseline/build/reports/profile/`、`explicit-modules/build/reports/profile/`。

## Configuration Cache and Declarative Gradle result — finalized 2026-10-04

- `gradle.properties` 已有 `org.gradle.configuration-cache=true`；此前 project-only profile 使用了 `--no-configuration-cache`，所以这轮专门在同一 binary consumer 上验证正常 CLI 路径。
- `help` 第一次/第二次分别为 `15.13s / 3.63s`；`:mcp-stdio:compileKotlinJvm` 第一次/第二次为 `18.50s / 5.54s`。日志分别出现 `Configuration cache entry stored` 和 `Configuration cache entry reused`。
- 对 `mcp/stdio/build.gradle.kts` 加一行受控注释后，Gradle 正确报告 configuration cache 因该文件变化而失效，并在 `22.49s` 内重建；文件随后从未修改的隔离副本恢复到原始 SHA。
- 结论：Kodex 的 CLI Configuration Cache 不是“没开好”；第二次 CLI task 已经明显变快。它不能解释第二次 IDEA Sync，因为 Gradle 官方当前仍说明 IDE import/sync 不使用 Configuration Cache。
- Declarative Gradle 官方仓库当前为 EAP3 实验状态，明确没有兼容性保证；其 Getting Started 文档明确“不适合 plugin authors、build engineers 或 software engineers 采用”。官方 unified KMP prototype 仅覆盖 `nodeJs`、`jvm`、`macOsArm64`，无法覆盖 Kodex 的 Windows/Linux/macOS 产品矩阵。
- Xiaoxin 尝试运行官方 KMP sample 时，Gradle 9.6.0-milestone-1 因需要 JDK 17 且没有可用 toolchain download URL 在模型执行前失败；没有把该失败误判成 Declarative Gradle 本身的模型性能结果。实验 clone 已恢复 clean。
- 结论：Declarative Gradle 可保留为观察/小型 prototype，不进入 Kodex 当前生产迁移或 Sync 优化主线。
- 远端报告：`~/ACodeSpace/demo/kodex-gradle-research-445/project-gradle-optimization-20261004/CC-DECLARATIVE-REPORT.md`。
- 本节定稿：Configuration Cache/Declarative Gradle 不再作为当前 Kodex IDEA Sync 优化实施项；后续只在 Tooling model、KMP/Native variant/cinterop 或 Gradle/IDE 版本变化时重新打开。

## Optimization hypotheses to test

### 1. Settings and module registration

- Replace filesystem walking and file-probe discovery with the approved explicit `moduleTree` settings-local DSL.
- Preserve current project paths where possible; any rename is a separate decision and must have a migration map.
- Measure settings evaluation and model differences separately. The report already indicates settings hooks are tens of milliseconds, so this track must not claim a large Sync win without evidence from downstream model construction.
- Registering a physical intermediate directory only as a namespace is acceptable if it is not exposed as a Gradle project and does not alter IDE source roots.

### 2. Build logic and configuration avoidance

- Inventory `buildSrc`, precompiled scripts, root `subprojects`/`allprojects`, `afterEvaluate`, `configureEach` versus eager APIs, cross-project property reads, and task realization.
- Compare keeping `buildSrc` with moving to `build-logic`; treat build-logic as a classpath/invalidation-scope and maintainability experiment, not an automatic Sync optimization.
- Prefer lazy APIs such as `tasks.register`, `tasks.named`, `configurations.configureEach`, and Provider-based values. Do not mechanically rewrite code when the measured realization graph does not improve.
- Make convention plugins project-local and avoid reading mutable state from sibling projects or the root project during configuration.

### 3. Fixed target model

- Preserve every existing target, architecture, hierarchy and source set; the
  user rejected further target reduction and developer-profile switches.
- Compare the unchanged full model, generated symbols, expect/actual, cinterop
  and platform navigation before and after each configuration/resolution fix.
- Do not split developer and release target models or hide targets by disabling
  tasks. Historical JVM-only measurements are not the current implementation route.

### 4. Dependency and variant resolution

- Audit repeated repositories, dynamic versions, metadata probes, dependency constraints, KMP variant matching, cinterop declarations, and eager resolution during configuration.
- Preserve dependency graph semantics while reducing unnecessary metadata/variant work.
- Verify that `PREFER_PROJECT` and project-level repositories do not hide or override the intended package/binary route.
- Measure dependency resolution separately from project configuration and IDE import.

### 5. Native Gradle mechanisms

- Keep configuration cache enabled for supported CLI scenarios, but do not use its CLI hit as evidence of IDEA Sync speed; Gradle documents that IDE imports/syncs do not yet use configuration cache.
- Explicitly measure `org.gradle.tooling.parallel` independently from task parallelism on the actual IDE scenario.
- Treat Isolated Projects as an optional diagnostics/migration track. Gradle documents that it is incubating, configures all projects, has memory/parent-hierarchy limitations, and still requires plugin/build-logic compatibility.
- Do not enable `dangerously-ignore`, internal intermediate model caches, or fork-SHA detection in settings.

## Measurement protocol

- Every candidate uses a clean isolated copy and the same fork binary fixture.
- One heavy operation at a time under Xiaoxin’s `device-heavy.lock`.
- Run at least:
  - first import after a controlled cache state;
  - three repeated warm real IDEA Syncs;
  - a no-op source edit;
  - a settings/module/build-logic change;
  - representative JVM compile;
  - representative Native configuration or compile where the host supports it.
- Record raw/model completion, native import completion, explicit smart completion, IDE/Gradle/Kotlin daemon/Native RSS and CPU, network/cache state, the fixed target inventory, JDK, Gradle, KGP, IDE and plugin versions.
- Use the same model/module/source-root/navigation comparison as the existing Sync study. A faster import that loses source roots, targets, or navigation is a failure.
- Use Build Scan or local profile reports only as diagnostics; do not upload proprietary source or dependency data without explicit authorization.
- Stop and return to the user if a candidate causes model loss, API/spec/impl semantic drift, OOM, unsupported host behavior, or fails the previously approved KGP/Gradle compatibility gate.

## Candidate acceptance

- A candidate must improve the named scenario, not merely `help` or a configuration-cache hit.
- It must preserve project graph, source roots, KMP hierarchy, `expect/actual`, cinterop declarations, navigation, representative compile/test tasks, and package consumer behavior.
- It must document developer impact, invalidation scope, memory impact and rollback under the unchanged complete target model.
- If the measured improvement is below noise or only affects an unimportant scenario, revert the candidate and keep the evidence rather than accumulating speculative Gradle “best practices”.

## Official references

- [Gradle performance guide](https://docs.gradle.org/current/userguide/performance.html)
- [Gradle Build Scan and performance inspection](https://docs.gradle.org/current/userguide/inspect.html)
- [Configuration cache](https://docs.gradle.org/current/userguide/configuration_cache.html)
- [Lazy versus eager evaluation](https://docs.gradle.org/current/userguide/lazy_eager_evaluation.html)
- [Creating configurations without eager realization](https://docs.gradle.org/current/userguide/declaring_configurations.html)
- [Isolated Projects](https://docs.gradle.org/current/userguide/isolated_projects.html)
