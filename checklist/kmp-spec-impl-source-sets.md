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
  The confirmed MCP and ToolSearch boundaries below also use `contract` to
  isolate shared frontend/backend protocols from backend runtime dependencies.
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
| `specMain` | Exported API contracts, contract models, normative KDoc, and permitted short declarative inline extensions | `src/specMain/kotlin` |
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
  layer; keep concrete mechanisms in implementation source sets except for
  [short declarative inline extensions](#short-declarative-inline-extensions).
  Otherwise, moving full mechanisms into specMain is not
  declaration/implementation separation. An existing behavior interface does not
  by itself specify a separately exported constructor or factory entrypoint.
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
- Keep existing targets unless a separately confirmed boundary decision below
  revises them. Both shared layers participate in the targets reached
  through `commonMain`; unequal platform coverage of former spec/impl projects
  must be resolved when selecting migration units. Keeping those projects
  independent preserves their different target sets while still allowing their
  contract code to move to `specMain`.

### Short declarative inline extensions

- Confirmed on 2026-10-22: a very short inline extension whose code almost
  declaratively explains what it does may be implemented directly in `specMain`.
  Do not introduce expect/actual forwarding solely to split such a body.
- This permission does not require an explicit Kotlin contract. Preserve any
  existing contracts, inline/lambda semantics and public compatibility.
  Judge brevity and declarative clarity together, not by a fixed line count.
- This is not blanket permission for classes, member methods, state or
  substantial locking, I/O and platform mechanisms. Do not hide substantial
  mechanisms behind helpers just to qualify.
- The user explicitly approved converting SafeRw's readSession/writeSession
  members to spec inline extensions, retaining their EXACTLY_ONCE contracts.
  SafeRw's type, constructor and PublishedApi internal access declarations
  belong to spec; actual value/mutex storage and initialization remain common.
  Accept the scoped member-to-extension symbol/import and explicit type-argument
  changes, not a whole-class implementation exception or a waiver for other APIs.
- The public API placement rule is unchanged: public declarations still belong
  to `specMain`; ordinary implementations remain actual or effectively
  internal/private downstream. Previously validated migrations need not be
  moved back merely because a new option is permitted.

### Declaration-shape checks

- Preserve `const` API semantics and put initialized constants in `specMain`;
  uninitialized `const` members in an expect object do not compile on the pinned
  toolchain. For objects mixing constants and runtime operations, keep the public
  spec object and bind only runtime operations to an internal expect/actual
  implementation. Prefer direct expect/actual when this binding is unnecessary.
- For `SessionSidebarCollapsedButtonColumns` and `SessionSidebarCollapsedButtonRows`,
  the user approved direct relocation into the existing View spec with Kotlin
  names, Int values 3/1, const semantics and behavior preserved. Old JVM field
  ownership in SessionSidebarKt need not be retained; this is not a general
  waiver of binary compatibility for other APIs.
- Expect constructors declare parameters, with exported properties declared
  separately in the class body. Do not copy private implementation constructors
  or their state parameters into the contract. An expect class need not repeat
  internal implementation-only constructors or their resource defaults.
- Keep public default arguments on spec declarations, not actual implementations.
  Overrides inherit interface defaults; do not repeat those defaults on an
  expected override.
- Explicitly declare inherited abstract API in concrete expect classes and
  validate metadata, not only JVM compilation. Keep private sealed DSL builders
  downstream by actualizing their sealed contract rather than moving the builder
  mechanism into spec. Preserve annotation targets, including `@get:Composable`
  and `@get:ReadOnlyComposable` on expected getters.
- `runCatchingCancellable` uses a bodyless `expect inline` spec and an unchanged
  `actual inline` common implementation on the pinned toolchain. Preserve
  lambda modifiers, suspending calls and non-local returns; validate a real
  consumer as well as metadata. This result does not establish preservation
  of explicit contracts on APIs such as SafeRw.
- The three suspending RPC CAS extensions and restoreRemoteException follow
  the same bodyless spec/actual inline shape. Preserve paced retries,
  cancellation, return values and exception identity; retain the implementation's
  internal PublishedApi constant instead of adding public helpers. Cross-module
  tests verify all four non-local return paths; do not add false callsInPlace
  guarantees to make consumer definite-return analysis succeed.
- `CoroutineCloseable.use` also keeps its suspending inline declaration in spec
  and its cleanup body in the actual implementation. Preserve the suspending
  lambda, non-local returns, NonCancellable cleanup and exception priority;
  validate those paths from a consuming module without inventing a callsInPlace
  guarantee or moving the cleanup algorithm into spec.
- CLI executable entrypoints follow the same declaration boundary: keep the
  expected main in spec and the actual startup body in its renderer/host layer,
  without changing the Native entryPoint or adding a forwarding entrypoint.
  Validate a real linked binary and isolated command outputs/exit codes rather
  than granting main a blanket lint exemption.

### Current Native product targets — 2026-10-22

- The user confirmed that JVM is no longer supported as a product target.
  The current CLI/backend product target set is `linuxX64`, `linuxArm64`,
  `macosArm64` and `mingwX64`. Do not count a library's configured JS target
  as a supported Node backend; these products do not have a JS runtime target.
- This supersedes the JVM-inclusive client/CLI/Host coverage-preservation
  requirements below for the current Native product. Host/CLI/Client conventions
  now configure the four Native targets; Shared libraries keep their separate
  portable non-JVM coverage. Retired JVM/JS source directories are not active
  product variants. When changing targets, review the consumer variant chain,
  not only the provider.
- Gradle, detekt and compiler tooling may still run on a JVM. That does not
  justify a JVM product variant. Acceptance must cover metadata and real Native
  consumers, not rely on JVM tests as proof of the Native product.
  Retain prior JVM/Node results as historical evidence, not current support policy.
- The separately planned Web frontend remains a distinct scope. Do not
  mechanically remove its needed shared contracts or Wasm work when aligning
  the current Native backend and CLI.
- Native-only value classes retain their value/serialization behavior without
  JVM-only annotations rejected by shared Native metadata compilation. Do not
  restore a JVM target merely to keep `JvmInline`; validate the Native wire shape.

### Confirmed platform boundaries — 2026-10-16

- IO, Process and Shell keep their Shared API in the parent business project and
  their real Host implementation in a `system` child project. These are platform
  adapters, not cycle-breaking contracts or forwarding aliases.
- A single JS target's browser/Node environments share its compilation API.
  Node-only IO/PTY factories cannot be isolated merely by adding `hostMain`.
  On the pinned KGP 2.4.20, splitting JS targets also fails ordinary project
  dependencies at NPM resolution. Do not repeat that merge without revalidating
  the toolchain; preserve the approved system boundary.
- Current product boundaries after Native target convergence:

  | Business module | Wide API/helper layer | Restricted implementation layer |
  | --- | --- | --- |
  | `utils/external-url` | Client Native four | `cliHostMain` / `desktopNativeMain` |
  | `utils/logging` | Two independent Native-four backend libraries | Shared actual code and Native file output |
  | `utils/search-index` | Native four | Common actual factory; Lucene code/dependency on Native leaves |
  | `app/migration` | Native four | `cliHostMain`, including generated application version |
  | `mcp/contract`, `mcp/backend` | Both Native four; shared/runtime ownership split below | Backend spec/actual management APIs and transports |

- Keep corresponding tests on the same target subsets; do not add iOS/Wasm
  factory stubs. The `kodex.kmp-cli-host-layer` convention creates groups, not
  targets. Preserve the existing four Native network pins through
  `kodex.ktor-native-network-override` when replacing the old CLI convention.
- Architecture decisions and runtime/IDE acceptance are separate:
  [completed migration and evidence](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-16-plan-uniform-spec-main-module-layout.md#L31).

### Client-owned external URLs — 2026-10-21

- External URL opening is a client capability, not a backend operation. Its
  `specMain` API and result types follow the client target set; platform actuals
  open URLs in that client's environment. Do not route opening through backend RPC.
- View and external-url share `kodex.kmp-client` and follow the current Native
  product target policy above. This replaces external-url's former Shared-only
  result-type coverage; do not retain JS/Wasm/iOS variants without a client
  implementation or add unsupported actual stubs.
- Browser targets and real browser implementations belong to the approved Web
  frontend work. This migration does not implement them or expand all View/Host
  projects to Wasm. Client and backend conventions may reuse the desktop target
  configuration, but the client convention must not inherit backend target policy.

### Independent backend logging libraries — 2026-10-21

- Use the existing `utils/` root and group two sibling modules under
  `utils/logging/`: `backend-scope` attaches backend business context to
  KotlinLogging events; `backend-file` configures a backend process's file output.
  The logging directory is only a grouping directory, not a third project or
  forwarding alias.
- Neither production library depends on the other. A composition may use both;
  a combined test does not justify a production dependency. Each library owns
  its exported specMain declarations and downstream implementation.
- Both libraries now follow the four Native backend targets. Portability does not grant
  frontend ownership: frontend application/RPC/ViewModel code uses ordinary
  KotlinLogging, without dependencies on either backend logging library.
- TUI keeps internal file logging at its startup boundary, with the existing
  formatting, rotation and cleanup behavior. Its bootstrap does not invoke the
  backend logging initializer. A dual-mode CLI may still contain backend
  dependencies for its explicit backend subcommand.
- MCP uses the contract/backend split below rather than carrying backend
  logging through a mixed shared project. Verify resolved production compile
  and runtime graphs, not just direct dependency declarations; explicit CLI
  backend subcommands and test composition still legitimately use the backend.

### MCP contract/backend boundary — 2026-10-21

- Use two modules under `mcp/`: `contract` for the frontend/backend shared
  models, protocol types and necessary lightweight interfaces/pure helpers;
  `backend` for MCP execution and management. The parent is only a grouping
  directory, not a third project or compatibility alias.
- Frontend consumers and RPC models depend only on `mcp/contract`.
  `mcp/backend` depends on `mcp/contract`, never the reverse. SDK clients,
  transport/process ownership, management, credential IO and backend logging do
  not belong in the shared dependency closure.
  The existing `rpc/models` project remains an RPC aggregation consumer; move
  MCP-owned shared declarations, not unrelated RPC protocol ownership.
- Both modules retain the specMain/commonMain rule. Shared value models may be
  fully declared in specMain; shared pure helpers normally use specMain declarations
  and commonMain implementations, except for permitted short declarative inline
  extensions. Platform IO adapters remain backend-owned. Backend-owned exported
  APIs have their own backend specMain declarations rather than leaking into
  the shared module.
- Classify individual declarations, not entire existing contract-package or
  specMain files: those currently mix shared state with backend service APIs.
  Preserve existing type identities, wire/persisted shapes, defaults and KDoc;
  do not introduce parallel frontend DTOs.
- Both modules now follow Native four. The backend management classes, factories
  and validation API have backend-owned spec declarations and actuals; they do
  not belong in the frontend/shared contract module. Target alignment alone is
  not API separation: validate declarations and real Native consumers as well.
- Shared Agent signatures also require the lightweight `McpService`,
  `McpClient` and `McpTool` ports. Keep these pure interfaces in contract;
  manager, OAuth and persistence ports belong to backend specMain.
  Shared Tool signatures transitively expose process/shell ports, not their
  system implementations. This is dependency isolation, not removal of every
  backend-domain concept from the shared vocabulary.

### ToolSearch contract/backend boundary — 2026-10-21

- Split `tool/tool-search` into `contract` and `backend`; the parent is only a
  grouping directory, not a compatibility project. This separates shared event
  data from backend execution, rather than mechanically splitting the generic
  `utils/search-index` library into another pair of projects.
- Contract owns the shared search parameters, execution discriminator, result
  models and necessary protocol constants. Frontend history, clean event models
  and other shared consumers depend only on this module.
- Backend owns `ToolSearchEngine`, indexed-document construction and its input
  models, tool-description/schema construction and the search-index dependency.
  It depends on contract; contract must not depend on backend, search-index or
  Lucene. Agent execution consumers depend on backend where needed.
- Classify individual declarations: the current models file mixes shared event
  DTOs with backend indexing inputs. Preserve packages, serialized identities,
  defaults, KDoc and behavior without copied DTOs or forwarding projects.
  Both modules retain their own specMain/public and downstream actual rules.
- The split itself did not revise platform coverage. The later Native product
  decision above separately converges the search-index library and its factory
  to four Native targets; do not add Wasm/iOS search stubs or restore Node/JVM
  merely to preserve the earlier configured variants.

### Prompt-image codec implementation exports — 2026-10-21

- The user explicitly withdrew the public API commitment for the five concrete
  objects `JvmImageIoPromptImageTransformer`, `SharpPromptImageTransformer`,
  `SkikoPromptImageTransformer`, `GdiPlusPromptImageTransformer` and
  `WindowsNativePromptImageTransformer`. Treat them as internal mechanisms;
  lack of callers alone is not authorization to withdraw other public APIs.
- Preserve the public HostPromptImageTransformer spec/actual entrypoint,
  KorimPromptImageTransformer, transformer injection and other codec APIs.
  This is visibility-only: retain algorithms, fallback/resource/error behavior
  and targets without new modules or unsupported implementations.

## Dependencies and consumption

- Declare contract dependencies and inputs needed by permitted inline bodies on
  `specMain`, ordinary implementation dependencies on `commonMain`, and platform
  dependencies on the appropriate platform layer.
  `commonMain` inherits `specMain` dependencies and sees its declarations,
  including `internal`; spec must not rely on reverse propagation.
- Search-index keeps its shared Lucene implementation directory and dependency
  on each Native leaf, not in shared API metadata. This avoids exposing its
  concrete library metadata to common consumers without a new project,
  dependency rewrite or relaxed compiler checks.
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
  An explicit coverage revision, such as client-owned external URLs above, stays
  scoped to that approved boundary.
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
