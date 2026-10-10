# Task Tree

- `Read the fixed-model constraints and accepted baseline`() // completed
- `Inventory tracked project declarations and actual public definitions`() // completed
- `Recompute candidate closures and verify existing callers`() // completed
- `Deliver the independent static report without executing measurements`() // completed

# Details

## Verdict and boundary

- **STATIC READY / runtime BENEFIT PENDING.**
- Found nine definition-proven unnecessary project exports in five Tool spec
  modules. They can be removed in a private measurement candidate without
  changing their Kotlin declarations or any existing caller's main project
  classpath closure. This is not a blanket `api` → `implementation` proposal.
- Verified benefit so far: fewer declared project edges and smaller compile/API
  closures for those five specs. **No measured Gradle/IDE memory, CPU, GC,
  configuration-count, variant-resolution or Sync benefit.**
- Production baseline supplied by the user:
  `5b41be52d2f64a12ea3671bcecb9151fd2d1fa60`; local HEAD agrees.
  Read the index directly, without Git commands. All 203 module scripts and
  1,279 tracked Kotlin/build/catalog/property files checked match their index
  blob hashes. This is not a separate Git clean-tree certification.
- Only this new outer report was written. No production edits, Git commands,
  network access, Gradle/build/test/IDE execution, process control, device
  access or Xiaoxin workspace/lock access. Static inspection used short local
  filesystem reads and offline Python; no temporary files were created.
- Parent remains responsible for remote experiments:
  [fixed-model ablation](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-08-measure-fixed-model-build-cost-ablation.md#L1)
  and [model/resource work](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-02-plan-gradle-model-resource-optimization.md#L1).
  Neither was edited or advanced by this investigation.

## Inventory and counting method

- Settings DSL and tracked build scripts agree: **216 registered projects =
  root + 12 source-less top-level containers + 203 KMP projects**. No tracked
  module build script outside the declared tree.
- Excluded Gitlinks `Mosaic`, `LuceneKmp`, `KotlinMcpSdk`, generated `build/`,
  ignored directories and experiments. Binary library aliases are not project
  edges; the root kRPC module substitution is recorded separately, not silently
  converted into an explicit project edge.
- Counted declarations in **all Main and Test source-set blocks**, not just
  text before `commonTest`. Included `mosaicMain` after `commonTest`, and
  `named("fileLoggingMain")` / `named("processClientMain")`.
- Main: **363 `api` + 274 `implementation` project declarations**.
  Test: **230 `implementation` project declarations**, no project `api`.
  These are declaration counts, not deduplicated graphs or resolved variants.
- `A(m)` is the union of project `api` edges across Main source sets.
  `E(m)` is its transitive reachable project set, excluding `m` itself.
  `D(m)` includes both Main `api` and Main `implementation` destinations.
  Compile projection:
  `C(m) = union(d in D(m)) ({ d } union E(d))`.
- `C` deliberately does not recurse through a dependency's `implementation`.
  Neither `E` nor `C` is a runtime graph, a Gradle configuration count or a
  target-specific resolved classpath. Main-source-set union can overstate
  availability on a particular platform; the candidates below only remove
  **commonMain** edges and each sole caller has the compensating edges in
  **commonMain**, avoiding a platform-only rescue.
- All 203 projects were checked for changed `E` and `C`, not just the edited
  modules. Counterfactuals used in-memory graph copies; production was untouched.

Relevant inclusion/source-set evidence:
[settings](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L70),
[History source-set ordering](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/build.gradle.kts#L1),
[fileLoggingMain](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/logging/impl/build.gradle.kts#L19),
[processClientMain](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/build.gradle.kts#L32),
[root substitution](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build.gradle.kts#L1).

## Exact candidate edges and public-definition proof

**A1:** remove only the apply-patch spec's Tool-contract edge.
**A5:** remove all five Tool-contract edges.
**B9:** A5 plus the four OpenAI-model edges below.
They are alternative diagnostic scopes, not cumulative measured wins.

| Spec project | Remove `api(project(...))` | Sole tracked Kotlin file and complete public boundary |
| --- | --- | --- |
| `:tool-spec-apply-patch` | `:tool-spec-contract`, [build line 8](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/apply-patch/build.gradle.kts#L8) | [ApplyPatchToolSpec.kt:4–29](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/apply-patch/src/commonMain/kotlin/io/github/stream29/kodex/tool/applypatch/ApplyPatchToolSpec.kt#L4): `io.github.stream29.kodex.tool.applypatch.ApplyPatchDescription: String` and `ApplyPatchGrammar: String`; only strings and `trimIndent()`. |
| `:tool-spec-builder` | `:tool-spec-contract`, [line 10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/builder/build.gradle.kts#L10); `:openai-spec-models`, line 9 | [JsonToolBuilderSpec.kt:10–28](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/builder/src/commonMain/kotlin/io/github/stream29/kodex/tool/builder/JsonToolBuilderSpec.kt#L10): `io.github.stream29.kodex.tool.builder.JsonToolHandlerResult<out Output>`, nested `Success`/`Failure`, `jsonToolSuccess`, `jsonToolFailure`; unconstrained generic value, Boolean and String only. |
| `:tool-spec-current-time` | `:tool-spec-contract`, [line 9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/current-time/build.gradle.kts#L9); `:openai-spec-models`, line 8 | [CurrentTimeSchemas.kt:3–18](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/current-time/src/commonMain/kotlin/io/github/stream29/kodex/tool/currenttime/CurrentTimeSchemas.kt#L3): `io.github.stream29.kodex.tool.currenttime.CurrentTimeParametersSchema` and `CurrentTimeOutputSchema`, both `kotlinx.schema.json.ObjectPropertyDefinition`. |
| `:tool-spec-get-context-remaining` | `:tool-spec-contract`, [line 9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/get-context-remaining/build.gradle.kts#L9); `:openai-spec-models`, line 8 | [GetContextRemainingSchemas.kt:3–25](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/get-context-remaining/src/commonMain/kotlin/io/github/stream29/kodex/tool/getcontextremaining/GetContextRemainingSchemas.kt#L3): `io.github.stream29.kodex.tool.getcontextremaining.GetContextRemainingParametersSchema` and `GetContextRemainingOutputSchema`, both `ObjectPropertyDefinition`. |
| `:tool-spec-plan` | `:tool-spec-contract`, [line 9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/plan/build.gradle.kts#L9); `:openai-spec-models`, line 8 | [PlanSchemas.kt:3–32](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/plan/src/commonMain/kotlin/io/github/stream29/kodex/tool/plan/PlanSchemas.kt#L3): `io.github.stream29.kodex.tool.plan.UpdatePlanParametersSchema: ObjectPropertyDefinition`. |

- This proof reads definitions and expression bodies, not existing comments or
  absence of imports alone. None references `Tool`, durable events, OpenAI
  models, project-owned serializers, a public inline body, `expect`/`actual`,
  RPC services or DI annotations.
- Each spec has exactly the one tracked commonMain Kotlin file shown above;
  no additional platform/test source files in the tracked inventory.
- **Delete these unused declarations**, rather than downgrade to
  `implementation`. Downgrading would reduce `E` but keep the same dependencies
  in these specs' own `C`, missing the verified compile-closure reduction.
- Retain the external schema dependency in the three schema specs, their
  exact schema definitions, builder's existing serialization plugin/core
  dependency, and all existing conventions. No plugin ablation is bundled here.

### Existing callers already carry the necessary dependencies

Across all tracked Main/Test project declarations, each candidate spec has
exactly one direct caller: its matching impl. There are no direct test callers.

| Sole caller | Preserved commonMain edges and real use |
| --- | --- |
| `:tool-impl-apply-patch` | [build:8–11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/apply-patch/build.gradle.kts#L8) explicitly exports its spec **and** Tool contract; [ApplyPatchTools.kt:18–35](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/apply-patch/src/commonMain/kotlin/io/github/stream29/kodex/tool/applypatch/ApplyPatchTools.kt#L18) assembles the grammar into the actual `ToolSpec` and `Tool`. |
| `:tool-impl-builder` | [build:9–11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/builder/build.gradle.kts#L9) explicitly exports its spec **and** Tool contract; [JsonToolBuilder.kt:22](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/builder/src/commonMain/kotlin/io/github/stream29/kodex/tool/builder/JsonToolBuilder.kt#L22) owns `jsonTool` using `ToolSpec` and `JsonToolHandlerResult`. |
| `:tool-impl-current-time` | [build:8–13](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/current-time/build.gradle.kts#L8) explicitly exports its spec **and** Tool contract; [CurrentTimeTools.kt:18–33](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/current-time/src/commonMain/kotlin/io/github/stream29/kodex/tool/currenttime/CurrentTimeTools.kt#L18) wraps the schema in `ToolSpec` and builds the executable `Tool`. |
| `:tool-impl-get-context-remaining` | [build:8–13](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/get-context-remaining/build.gradle.kts#L8) explicitly exports its spec, Tool contract **and** OpenAI models; [GetContextRemainingTools.kt:10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/get-context-remaining/src/commonMain/kotlin/io/github/stream29/kodex/tool/getcontextremaining/GetContextRemainingTools.kt#L10) creates the actual `ResponsesApiTool`. |
| `:tool-impl-plan` | [build:8–14](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/plan/build.gradle.kts#L8) explicitly exports its spec, Tool contract **and** OpenAI models; [PlanTools.kt:13](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/plan/src/commonMain/kotlin/io/github/stream29/kodex/tool/plan/PlanTools.kt#L13) creates the actual `ResponsesApiTool`. |

- Builder/current-time retain OpenAI access through
  `impl -> api(tool-spec-contract) -> api(openai-spec-models)`.
- No compensating dependency addition is needed in any existing caller.
  Public Tool factories stay in impl; meaningful spec declarations stay in spec.
- A hypothetical external consumer of these spec artifacts might have relied
  on accidental transitive exports. The in-repository caller proof does not
  certify arbitrary unpublished downstream sources. Publication dependency
  metadata intentionally changes; Kotlin declaration ABI does not.

## Recomputed closure counterfactual

| Scope | apply-patch `E / C` | Each of other four specs `E / C` | Changed modules | Total API reachability pairs |
| --- | --- | --- | --- | --- |
| Baseline | `14 / 14` | `14 / 14` | — | 1,925 |
| A1: one Tool-contract deletion | `0 / 0` | `14 / 14` | apply-patch spec only | 1,911 |
| A5: five Tool-contract deletions | `0 / 0` | `2 / 2` | five specs only | 1,863 |
| B9: all nine deletions | `0 / 0` | `0 / 0` | five specs only | 1,855 |

- A5 removes **62** `(requesting project, reachable project)` pairs.
  B9 removes **70**, not 70 projects or configurations.
- B9 Main `api` declarations: **363 → 354**.
  Main `implementation` declarations remain **274**; Test declarations **230**.
- **All other 198 modules' `E` and `C` remain exactly unchanged**, including
  the matching Tool impls, AgentState, CLI and integration-test main projection.
  Repeated alternate paths were recomputed, not summed as independent benefits.
- Each of the four direct OpenAI-edge deletions alone has **zero `E`/`C`
  effect** before A5: Tool contract still exports OpenAI models. After A5 each
  removes the two remaining reachable projects. Do not add per-edge deltas
  without recomputing the combined graph.
- Those last two projects are `:openai-spec-models` and
  `:utils-kotlinx-io-serialization-spec`.

The full fourteen-project baseline closure of every candidate spec:

```text
:tool-spec-contract
:agent-storage-spec-clean-models
:openai-spec-models
:tool-spec-image-generation
:tool-spec-request-user-input
:tool-spec-multi-agent
:tool-spec-tool-search
:tool-spec-unified-exec
:tool-spec-view-image
:utils-patch-spec
:utils-shell-client-spec
:utils-process-client-spec
:utils-kotlinx-io-coroutines-spec
:utils-kotlinx-io-serialization-spec
```

### Responsibility for that closure

- Candidate spec's unused edge reaches Tool contract.
- Tool contract [build:9–10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/contract/build.gradle.kts#L9)
  exports clean models and OpenAI models. Clean models then export eight
  direct projects [build:11–18](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/build.gradle.kts#L11).
- Unified Exec → Shell → Process → coroutine IO spec:
  [Unified Exec:9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/unified-exec/build.gradle.kts#L9),
  [Shell:9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/spec/build.gradle.kts#L9),
  [Process:10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/spec/build.gradle.kts#L10).
- Patch also exports coroutine IO spec
  [build:11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/spec/build.gradle.kts#L11).
  OpenAI models exports the Path serializer project
  [build:9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/build.gradle.kts#L9).
- These inner edges are generally **required**. Remove the unnecessary outer
  spec edges, not the durable Tool contract or its serialized child types.

## High fanout: what to query, not an invented memory result

Counts below are distinct requesting projects; `E` and `C` use the union
projection defined above.

| Dependency | Direct Main callers | Direct API callers | Requesters whose `E` contains it | Requesters whose `C` contains it | Its own `E` size |
| --- | ---: | ---: | ---: | ---: | ---: |
| `:utils-kotlinx-io-serialization-spec` | 2 | 2 | 110 | 125 | 0 |
| `:utils-kotlinx-io-coroutines-spec` | 5 | 5 | 104 | 125 | 0 |
| `:openai-spec-models` | 43 | 36 | 109 | 124 | 1 |
| `:agent-storage-spec-clean-models` | 15 | 15 | 79 | 98 | 12 |
| `:tool-spec-contract` | 17 | 16 | 51 | 54 | 13 |
| `:mcp-spec-contract` | 12 | 10 | 35 | 38 | 14 |
| `:utils-kotlinx-io-coroutines-impl` | 20 | 14 | 14 | 26 | 1 |
| `:app-view-components` | 27 | 4 | 6 | 27 | 14 |
| `:app-component-settings-spec` | 6 | 4 | 6 | 10 | 43 |

- Ranking all 203 destinations, **largest transitive compile fanout is tied:
  `:utils-kotlinx-io-serialization-spec` and
  `:utils-kotlinx-io-coroutines-spec`, each in 125 requesters' `C`**.
  Path serialization spec also has the largest exported reach, 110 requesters.
- **Largest direct Main fanout: `:openai-spec-models`, 43 callers**.
  Its Path serialization export explains why the two-direct-caller serializer
  spec actually reaches more requesters than OpenAI itself. These are query
  seeds, not removable public ABI or measured configuration costs.
- The coordinator's variant query should group requesting configurations by
  dependency target, requested platform/usage/category/attributes and selected
  producer variant; separate existing configurations from any additionally
  created/materialized ones. Include metadata and platform requests, not just
  JVM. Query the two IO specs plus OpenAI models, then compare Tool
  contract/clean models; do not rank solely by direct declarations.
- **Which dependency actually creates the largest additional configuration
  population is unknown here.** Static fanout does not reveal resolution
  multiplicity, selected variants, lazy configuration realization or retained
  object size. A resolved variant query must be separate from the parent's
  explicitly non-resolving configuration census.
- For the safe candidate, the single largest immediate edge is
  `:tool-spec-apply-patch -> :tool-spec-contract`: fourteen lost reachable
  projects versus twelve for each other A5 edge. A1 is an exact one-edge
  diagnostic if the coordinator wants the narrowest experiment.
- All projects remain included and all targets/source-set ancestry remain
  configured. **No fewer projects, tasks or configurations are guaranteed.**
  Five smaller spec classpath projections can coexist with unchanged IDE
  module counts and unchanged downstream application graphs.

## Required exports and rejected/deprioritized ideas

- **Reject removing Tool contract's clean-model edge.**
  `io.github.stream29.kodex.tool.contract.Tool.handle` accepts
  `PendingToolEvent` and returns `StableCleanEvent.CompletedTool`; `spec` is
  `io.github.stream29.kodex.openai.ToolSpec`.
  [Actual signatures:13–34](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/tool/contract/Tool.kt#L13).
  This is the definition proof, not its explanatory comment.
- **Reject cutting clean models' serialized child edges.**
  `PendingCommandExecutionAction.ExecCommand.arguments` is
  `io.github.stream29.kodex.tool.unifiedexec.ExecCommandArguments`, and
  `WriteStdin.arguments` is `WriteStdinArguments`, under `@Serializable`.
  [Declarations:9–23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/unstable/PendingCommandExecutionAction.kt#L9).
  Do not sever generated serializers from their DTO providers.
- **Reject cutting MCP's Tool edge.**
  `io.github.stream29.kodex.mcp.contract.McpTool : Tool` is a public supertype.
  [McpTool.kt:3–17](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpTool.kt#L3).
- **Reject mechanical Settings parent narrowing.**
  Settings spec's closure is 43, but its ABI exposes actual child handles:
  `GlobalSettingsViewModel` has ContextSourceSettings, SessionTitleSettings,
  ApplicationPreferences, AuthenticationSettings, AccountUsage, UsageReset,
  McpSettings and HookSettings ViewModel properties
  [definitions:42–58](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/GlobalSettingsViewModel.kt#L42).
  `SettingsViewModelArguments.target` is the actual
  `io.github.stream29.kodex.app.session.contract.SessionViewModel`
  [definition:95–100](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/SettingsViewModel.kt#L95).
  No replacement wrapper/Provider is justified.
- **Reject cutting RPC server's MCP impl export.**
  `BackendGlobalState.mcpManager` publicly exposes
  `io.github.stream29.kodex.mcp.impl.McpManagerImpl`
  [definition:98–109](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalState.kt#L98).
- **Reject changing StateFlow/Path exposure as an optimization.**
  Settings exports `StateFlow` and `Flow` in the definitions above;
  `PathAsStringSerializer : KSerializer<kotlinx.io.files.Path>`
  [definition:12–21](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-serialization/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxioserialization/PathAsStringSerializer.kt#L12)
  is referenced by the serialized OpenAI compaction model
  [CompactionModels.kt:68](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/CompactionModels.kt#L68).
- **Do not mistake an impl dependency for an impl-owned type.**
  Image codec IO extensions publicly take
  `io.github.stream29.kodex.utils.kotlinxiocoroutines.CoroutineFileSystem`
  [PromptImageCodecIo.kt:23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/codec/PromptImageCodecIo.kt#L23).
  The operation contract lives in IO spec. Any separate impl-edge narrowing
  requires spec/actual/factory tracing; no blanket downgrade is included.
- **Deprioritize small impl-export changes versus B9.**
  Downgrading [app/impl/rpc/build.gradle.kts:28](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/build.gradle.kts#L28)
  from API to implementation
  gives `E(app-impl-rpc) 51 → 50` and `E(app-test-support-rpc) 58 → 57`;
  only the History impl disappears. Direct use is inside the private
  `RpcAgentViewModel`
  [definition/use:68–112](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentViewModel.kt#L68).
  This report does not certify generated Koin ABI for that project, so it is
  **not** in the STATIC READY candidate.
- **Reject counting redundant paths as wins.**
  Removing only Tool contract's direct OpenAI edge still leaves OpenAI models
  reachable through clean models; its project-closure delta is zero.
  The same caveat applies to the four B9 OpenAI edges before A5.
- Targets/profiles, module merges, spec wrappers and framework/build-logic
  rewrites are outside this investigation. No recommendation of that kind is
  being substituted for measured benefit.

## TestBalloon and publishing: real attachment versus real use

- Convention inventory across the 203 tracked KMP scripts:
  host **90**, CLI **44**, view **29**, viewmodel **27**, shared **12**,
  CLI executable **1**.
- View, viewmodel and CLI executable inherit CLI, giving base applications:
  host **90**, CLI **101**, shared **12**.
- Each base applies `maven-publish` and TestBalloon:
  [host:1–12](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-host.gradle.kts#L1),
  [CLI:1–12](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-cli.gradle.kts#L1),
  [shared:5–9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-shared.gradle.kts#L5).
  Thus **203 publishing attachments and 203 TestBalloon attachments** in the
  static convention model. These are not 203 publications or a measured plugin
  event count; Gradle applies a plugin once per project.
- Every base also injects **both** `kotlin("test")` and
  `test-balloon-framework-core` into commonTest:
  [helper:50–55](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt#L50),
  [shared:43–52](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-shared.gradle.kts#L43).
  Dependencies form a union, not alternative frameworks.
- Actual commonTest inventory: **284 active tracked Kotlin files in 109
  modules**; TestBalloon framework references in **108** modules;
  `kotlin.test` imports in **109**; union **109**, intersection **108**.
- Across all Test source sets: **424 files in 140 modules**;
  TestBalloon references **139**; `kotlin.test` imports **140**; union **140**.
  Platform/mosaic tests also inherit commonTest dependencies. Therefore
  **94 modules without physical commonTest files are not 94 safe test-plugin
  detachments**; only **63** lack any active tracked Test Kotlin files.
- Important false negative: Session Tab Bar spec uses fully qualified
  `de.infix.testBalloon.framework.core.testSuite`, not an import
  [actual call:6](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/spec/src/commonTest/kotlin/io/github/stream29/kodex/app/sessiontabbar/contract/SessionTabBarContractTest.kt#L6).
  Import-only counting incorrectly yields 107 commonTest / 138 overall.
- kRPC utils patch uses `kotlin.test.Test`
  [declaration:7–15](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/src/commonTest/kotlin/kotlinx/rpc/internal/utils/map/SynchronizedHashMapTest.kt#L7).
  Do not delete commonTest's Kotlin-test dependency or claim only TestBalloon
  tests matter.
- Excluded exactly the known retired migration test using the production
  exclusion [migration build:38](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/build.gradle.kts#L38);
  its presence on disk was not counted as an active test owner.
- All five B9 specs have no tracked Test sources, but this candidate does
  **not** remove their plugin or commonTest dependencies. Publishing/test
  attachment ablations belong to the existing coordinator experiment and are
  not verified improvements from this report.

## Serialization, RPC, actuals and DI preservation

- B9 changes only nine dependency declarations. No Kotlin/KDoc/schema/grammar
  files, targets, hierarchy, compilation tasks, conventions or compiler plugins
  change. None of the five source files has a serializer/RPC/DI annotation or
  `expect`/`actual` declaration.
- The Tool impls still export Tool contract and its original generated
  serialization closure. RPC models/services and kRPC patch routing remain
  untouched; RPC compiler attachment remains
  [contract build:1–5](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/build.gradle.kts#L1).
- Koin compiler attachment/factory generation and strict safety remain
  [viewmodel convention:3–25](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-viewmodel.gradle.kts#L3).
  The candidate changes no Koin-bearing source/convention or existing impl's
  `E`/`C`; this is narrower than certifying every generated binary ABI.
- Generated symbol equality and actual downstream compilation still require
  the coordinator's authorized runtime validation. No such check was executed
  or claimed here.

## Handoff and completion checks

- [x] Definition proofs and exact edges delivered early.
- [x] Full source-set-aware graph recomputed, with alternative-path accounting.
- [x] Sole callers and public ABI checked from declarations.
- [x] Fully qualified TestBalloon call included in dependency-union counts.
- [x] Source and all target/profile constraints preserved.
- [x] Only the new authorized report written; no resources acquired.
- [x] Document reviewed for scope, definitions, arithmetic and claim strength.
- **Measurement boundary:** A1/A5/B9 are STATIC READY graph ablations only.
  The existing non-resolving census can observe declaration/configuration
  attachment changes, but cannot establish resolved dependency or IDE benefit.
  Any follow-up resolution/import/compile validation belongs to the parent,
  its private Xiaoxin copy and existing heavy lock.
- If controlled runtime results show no material benefit, record that outcome
  and do not present this dependency cleanup as the model-memory solution.
  This independent read-only investigation is complete; runtime acceptance
  remains **BENEFIT PENDING**, not a suspended operation in this session.
