# KMP Spec/Impl Source Sets

## Frozen design and scope

- Frozen on 2026-10-12: use `platformMain -> commonMain (impl) -> specMain`
  for the source-set migration, including retained contract projects.
  Arrows mean `dependsOn`. The 2026-10-16 clarification below makes this the
  uniform business KMP structure and requires retiring spec-named Gradle modules.
- This is the selected design, not evidence of a completed migration or validated
  build. The freeze covers the rules below, not candidate project counts, exact
  destination paths, convention implementation, or performance estimates.
  Finalize the per-project manifest and authorize implementation separately.
- Preserve the contracts, KDoc ownership, and compatibility rules in
  [Spec/Impl Module Boundaries](spec-impl-module-boundaries.md).
- Retaining an independent contract project does not exempt it from source-set
  migration. Preserve necessary contract/implementation project boundaries and
  place the retained project's contract code and dependencies in `specMain`.
  Its standard `commonMain` still depends on `specMain`; a contract-only project
  need not invent implementation code to fill `commonMain`.
- Select project boundaries by current business responsibility and dependencies.
  Historical contract boundaries are evidence to review, not an exemption from
  consolidation or the naming rules below.
- Build or IDE validation is an acceptance gate, not an outstanding architecture
  choice. If validation reveals a conflict with this design, record the evidence
  and obtain a revised design decision rather than silently changing the rules.

## Project naming and boundaries

- Confirmed on 2026-10-16: use `specMain` uniformly in business KMP modules;
  `spec` identifies a source-set responsibility, not a Gradle project category.
  Implementation/helper modules may leave `specMain` empty only when they have
  no module-owned exported API requiring a declaration there; do not invent a
  contract or remove the uniform layer.
- Gradle business project IDs and their module directory paths must not use
  `spec` or `impl` as naming segments or grouping directories. The 2026-10-16
  scope clarification includes removal of both categories, without merging
  unrelated View/ViewModel or composition responsibilities. This does not rename
  `specMain`, Kotlin API/package names, or convention plugin IDs merely because
  they contain spec or impl.
- Default to one business module containing contracts in `specMain` and shared
  implementation in `commonMain`. Do not preserve a separate spec project merely
  because it existed before or has several consumers.
- When a separate contract project is genuinely required to break a dependency
  cycle, name that boundary `contract`, never `spec`, and keep its contracts in
  its own `specMain`. Record the cycle that would otherwise result.
- Independent business libraries such as models, codecs, transports, and helpers
  use their actual business names; do not mechanically rename every old spec
  project to contract. Existing platform-coverage differences still require a
  coverage-preserving migration design, not an unsupported contract label.
- These are target rules, not a claim that all current paths already conform.
  Freeze per-project destinations and migrate registrations and consumers
  together; no alias or forwarding project should preserve an old spec ID.

## Source-set responsibilities

| Source set | Responsibility | Kotlin source directory |
| --- | --- | --- |
| `specMain` | Exported API contracts, contract models, and their normative KDoc | `src/specMain/kotlin` |
| `commonMain` | Shared actual implementations and non-exported declarations | `src/commonMain/kotlin` |
| Platform/intermediate source sets | Platform actual implementations and non-exported declarations with their required target subset | Platform-appropriate directories |

- Confirmed on 2026-10-20: this is an exported/non-exported API boundary inside
  every business module, including modules whose business responsibility is
  implementation, adaptation, composition, or helpers. Calling a module an
  implementation module does not exempt its exported API from specification.
- Only `specMain` declares module-owned public APIs. Public API implementations
  in every other production source set must be `actual` implementations of
  corresponding spec declarations; all other declarations must be effectively
  `internal` or `private`. Implicit public visibility is subject to the same rule.
  This also covers exports without current in-repository callers.
- Declare module-owned exported APIs in `specMain`, including
  creation entrypoints, functions, extensions, properties, types, and members.
  Keep their required signature types and normative behavior in the contract
  layer; keep concrete mechanisms in implementation source sets. An existing
  behavior interface does not by itself specify a separately exported constructor
  or factory entrypoint. Source relocation with the full mechanism is not
  declaration/implementation separation.
- Non-exported implementation declarations must use `internal` or `private`
  effective visibility. Source-set placement alone is not a Kotlin visibility
  barrier. An internal/private implementation type may override a spec interface;
  this does not export the implementation type or its members as a separate API.
  Merely implementing an interface does not exempt a public implementation type
  or a separate public factory from the `actual` rule.
- `commonMain` keeps its real KGP name; “implMain” describes its responsibility,
  not another source set or a rename. Do not introduce an extra `implMain` layer.
- Create a real `specMain` source set, not a second directory inside `commonMain`.
  Establish `commonMain.dependsOn(specMain)`; `specMain` must not depend back on
  `commonMain`.
- `platformMain` is schematic, not a new literal source set. Preserve existing
  platform and intermediate paths into `commonMain`, including Native, Mosaic,
  and posix groupings where present.
- Keep existing targets. Both shared layers participate in the targets reached
  through `commonMain`; unequal platform coverage of former spec/impl projects
  must be resolved when selecting migration units. Keeping those projects
  independent preserves their different target sets while still allowing their
  contract code to move to `specMain`.

### Confirmed platform boundaries — 2026-10-16

- IO, Process and Shell keep their Shared API in the parent business project and
  their real Host implementation in a `system` child project. These are platform
  adapters, not cycle-breaking contracts or forwarding aliases.
- A single JS target's browser/Node environments share its compilation API.
  Node-only IO/PTY factories cannot be isolated merely by adding `hostMain`.
  On the pinned KGP 2.4.20, splitting JS targets also fails ordinary project
  dependencies at NPM resolution. Do not repeat that merge without revalidating
  the toolchain; preserve the approved system boundary.
- Other approved partial-platform modules retain the following source coverage:

  | Business module | Wide API/helper layer | Restricted implementation layer |
  | --- | --- | --- |
  | `utils/external-url` | Shared | `cliHostMain` / `desktopNativeMain` |
  | `utils/logging` | Shared | `fileLoggingMain`: JVM + four desktop Native targets |
  | `utils/search-index` | Shared | `hostMain`: JVM/JS/four desktop Native; Lucene excludes JS |
  | `app/migration` | Host | `cliHostMain`, including generated application version |
  | `mcp` | Host contracts and transports | `cliHostMain` service composition |

- Keep corresponding tests on the same target subsets; do not add iOS/Wasm
  factory stubs. The `kodex.kmp-cli-host-layer` convention creates groups, not
  targets. Preserve the existing four Native network pins through
  `kodex.ktor-native-network-override` when replacing the old CLI convention.
- Architecture decisions and runtime/IDE acceptance are separate:
  [completed migration and evidence](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-16-plan-uniform-spec-main-module-layout.md#L31).

## Dependencies and consumption

- Declare contract dependencies on `specMain`, implementation dependencies on
  `commonMain`, and platform dependencies on the appropriate platform layer.
  `commonMain` inherits `specMain` dependencies and sees its declarations,
  including `internal`; spec must not rely on reverse propagation.
- The 2026-10-20 public/actual rule supersedes the earlier restriction on using
  `expect`/`actual` for API separation. Contracts fully declared in `specMain`,
  such as interfaces and value models, need not become `expect` mechanically;
  exported APIs requiring downstream implementation use corresponding spec and
  actual declarations. Prove the declaration shapes and source-set placement
  on the pinned toolchain before migrating them.
- Consumers may depend on the combined project; a separate spec-only artifact
  is not required. Across modules, both contract and implementation callers use
  the provider's exported spec API, not implementation-only declarations.
  Composition between implementation modules follows the same rule.
- Keep contract ownership distinct from the set of declarations present in a
  consumer's compile input. `api`/`implementation` dependency configurations do
  not themselves hide a project's public implementation declarations; technical
  accessibility is not permission to bypass the exported contract.
- Review existing conventions that inject dependencies or generated sources into
  `commonMain`; keep implementation-only inputs there and place inputs needed
  by contracts on `specMain`.

## Migration invariants

- Project consolidation and source-set migration are separate operations:
  consolidate suitable spec/impl projects, but migrate retained contracts too.
  A retained contract project keeps its own targets and normal consumer edges.
- For mixed sources, classify declarations and their dependencies by actual
  responsibility. Preserve valid contract default behavior; do not move its
  required helpers downstream or classify all code by the old directory name.
- Some legacy spec-named projects contain only concrete codecs, transports or
  helpers. Preserve their implementation in commonMain/platform layers and
  retain the uniform hierarchy. Apply the exported API rule even to these
  projects; an empty specMain is valid only when no module-owned export needs
  a declaration there. Do not invent a port or move implementation upstream
  merely to populate that layer.
- Audit actual cross-module references and module-owned public entrypoints,
  including implicit public declarations without callers, not only source-set
  wiring or existing interfaces. Check public implementations outside specMain
  for matching actualization, and non-exported declarations for effective
  visibility. Preserve restricted target coverage when separating partial-platform
  exports; resolve their declaration placement explicitly rather than widening
  APIs or adding unsupported stubs.
- Preserve public signatures, packages, normative KDoc, serialization shapes,
  resource ownership, cancellation semantics, and existing platform coverage.
  Apply the compatibility and hard-cutover rules in
  [Spec/Impl Module Boundaries](spec-impl-module-boundaries.md).
- Preserve main/test dependency placement, platform/intermediate test scope,
  required compiler plugins, generated inputs, and Native interop configuration.
  Check merged files, module-scoped declarations, and JVM file-facade collisions.
- Validate both the resulting project graph and actual compilation/metadata,
  consumers, and tests. A static acyclic graph does not establish build success;
  fewer projects do not establish faster Sync or incremental compilation.

## KGP integration

- The current structural lint entrypoints are `checkSpecApi` (strict) and
  `reportSpecApi` (diagnostic only); follow [Spec API lint](../Kodex/lint/README.md).
  It checks non-spec production Kotlin declarations, not full semantic contract
  coverage. Compiler, generated-API and migration acceptance gates remain separate.

- Official KGP `2.4.20`, used by the current implementation, diagnoses
  `commonMain.dependsOn(specMain)` as an error but has an internal suppression path.
  The selected design uses this narrow exception, not a KGP fork:

  ```properties
  kotlin.internal.suppressGradlePluginErrors=CommonMainOrTestWithDependsOnDiagnostic
  ```

- Do not disable all configuration checks. This diagnostic ID also covers
  `commonTest`; its suppression does not authorize additional source-set edges.
  Keep test wiring unchanged unless separately designed.
- Adding a manual `dependsOn` can disable automatic hierarchy-template application.
  Preserve the intended hierarchy through explicit template/convention setup;
  turning the default template off is neither required to suppress the diagnostic
  nor a substitute for preserving existing intermediate source sets.
- Business KMP projects use `kodex.kmp-spec-impl` before their existing
  target/renderer convention. It applies the default hierarchy explicitly and
  creates `specMain` with the `commonMain` edge. Existing conventions can extend
  that hierarchy afterward. Roll out missing wiring in scoped migration batches;
  do not apply the business rule blindly to build logic or external forks.
- Declare contract dependencies with `sourceSets.named("specMain")`; keep
  implementation and test dependencies in their appropriate existing layers.
  The convention does not move dependencies or generated inputs for a project.
- For the suppression control, clear the property and execute
  `checkKotlinGradlePluginConfigurationErrors` or an actual compilation task.
  `help` can report the diagnostic while exiting successfully; its exit code
  does not prove that the configuration passed.
- Internal-property use has its own warning and no stable compatibility guarantee.
  Recheck the exception on KGP upgrades; suppressed diagnostics do not establish
  correct metadata, compilation, publication, or IDE behavior.
- The original mechanism review used an official KGP fixture:
  [v2.4.0 build script](https://raw.githubusercontent.com/JetBrains/kotlin/v2.4.0/libraries/tools/kotlin-gradle-plugin-integration-tests/src/test/resources/testProject/suppressGradlePluginErrors/build.gradle.kts),
  [fixture properties](https://raw.githubusercontent.com/JetBrains/kotlin/v2.4.0/libraries/tools/kotlin-gradle-plugin-integration-tests/src/test/resources/testProject/suppressGradlePluginErrors/gradle.properties).
  Current [v2.4.20 checker](https://raw.githubusercontent.com/JetBrains/kotlin/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/diagnostics/checkers/CommonMainOrTestWithDependsOnChecker.kt)
  retains the diagnostic. Source evidence supports the mechanism, not acceptance
  of Kodex's complete model; record scoped build results in the migration task.
