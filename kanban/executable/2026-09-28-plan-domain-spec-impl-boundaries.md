# Task Tree

- `Confirm domain scope, naming, pairing, and KDoc ownership`()
- `Specify the Runtime and Unified Exec trial boundary`()
- `Commit outer repository records in coherent batches`()
- `Create a fresh Kodex experiment branch`()
- `Move pilot contracts and KDoc into spec projects`()
- `Adapt implementations and downstream Gradle dependencies`()
- `Validate the pilot and report results for review`()
- **`Review the trial diff and architecture tradeoffs with the user`()**

# Details

## Goal and authorization

- User goal: reorganize Kodex so each domain has one or several clearly identified `spec` modules as its source of truth; the remaining modules implement or adapt those specifications.
- Confirmed planning granularity: organize and migrate by domain. Confirmed project-level coverage: every implementation Gradle project, including private helpers, has a corresponding spec project; a domain may therefore contain several spec/impl pairs. This supersedes the earlier proposed exception for implementation-only helpers.
- Confirmed physical naming: place spec projects under each domain's `spec/` tree and implementation projects under its `impl/` tree; the directory placement must also appear in Gradle project paths (for example, `openai/spec/models` becomes `:openai-spec-models`). Apply this in reviewable batches, not by an unvalidated whole-repository rename.
- Confirmed first pilot: Agent Runtime and Unified Exec, rather than a low-risk rename-only slice. The pilot must resolve the existing runtime-spec dependency on a concrete tool implementation while preserving the session-scoped shell client behavior.
- Confirmed documentation requirement: a spec contains both its code declarations and their attached KDoc. Existing KDoc that states a public or internal module guarantee must move with that guarantee into the spec, not be stranded on an implementation-only declaration.
- The user authorized an experimental implementation on a new inner `Kodex/` branch after batching and committing the pending outer-repository records. The API sketch below is a trial hypothesis, not an accepted final architecture. Do not change the frozen RPC contract or implement the rest of the repository-wide migration in this trial.
- The user requested outer-repository commits, not an inner `Kodex/` commit or push. Leave the experimental code visible in the new branch for review unless separately asked to commit it.
- Primary work tree is `Kodex/`. `Kodex/settings.gradle.kts` currently includes module trees by directory; a physical rename changes Gradle project paths and every consumer declaration.

## Experiment state

- Outer `main` received three separate documentation commits before the inner branch was created: native crash/suspend findings `11b75f1`, history Work scroll investigation `fe9d396`, and this spec/impl task `6aa65f9`. No push was requested or performed.
- Inner `Kodex/` branch: `experiment/spec-impl-pilot-2026-09-28`, created from clean `main` at `66e6ba70`. Trial code remains uncommitted for user inspection.
- Trial changes so far: moved the two core contract projects to `spec` paths, added the Unified Exec client/session spec interfaces and KDoc, retained the concrete process manager in impl, and updated direct Gradle consumers.
- A first targeted Gradle compile was cancelled before completion: an existing Gradle Daemon was busy, and Gradle started a second daemon despite using the detected daemon JVM. The newly started daemon was stopped; the original daemon was left untouched. After `gradlew --status` reported the original daemon idle, the targeted JVM compilation was rerun using that daemon and passed (`:tool-unified-exec-spec`, `:tool-unified-exec-impl`, `:agent-runtime-spec`, and `:agent-runtime-impl`; `--max-workers=2`).
- Subsequent validation passed using the original Gradle Daemon with `--max-workers=1`: JVM tests for Unified Exec, Agent Session in-memory, RPC server, and all four Agent Runtime decorators; `:app-contract-agent:compileKotlinJvm` and `:agent-storage-clean-models:jvmTest`. Static inspection found 130 included project build files and no stale project references. The moved Unified Exec DTO file and `ResumableAgentLayer` are byte-identical to their originals.
- Native compilation and real CLI smoke were not run in this trial; the host has limited available memory while IDEA and other user processes are active. Report these as remaining validation, not as passed checks.

## Proposed meaning of `spec`

- A `spec` owns the corresponding implementation project's promised types and semantics, including internal module contracts: value models, interfaces, commands/events, invariants, error and lifecycle behavior, their attached KDoc, and compatibility tests where relevant. Pure deterministic rules may live there when consumers must agree on their result.
- An `impl` owns effects, resources, algorithms not promised as public behavior, adapters, platform or renderer code, and composition. A public class is not automatically a specification.
- Move existing KDoc with moved declarations. When introducing a spec interface for an existing concrete class, transfer or adapt its behavioral and lifecycle KDoc to the interface and its members; keep implementation-mechanics KDoc beside the concrete code. The spec is the authoritative description; avoid two conflicting copies of the same guarantee.
- A spec may depend on another domain's spec when its public API actually exposes that type. Spec-to-spec dependencies should remain acyclic and explicit. A spec must not depend on an impl, including through `api` re-export or public signatures.
- Each implementation project must have a corresponding spec project, even without an independent external consumer. A spec must still express a meaningful boundary; do not introduce duplicate DTOs, dummy interfaces, or behavior-free placeholders to satisfy the pairing mechanically. If a candidate implementation has no separable contract, determine whether its responsibility or project boundary should change before migration.
- Each domain's canonical semantics should have one owner. Wire DTOs, persisted models, runtime state, RPC contracts, and frontend presentation models are distinct authorities; do not create parallel mappers or state machines just to obtain a `spec` label.
- Target dependency policy: `spec -> spec`; `impl -> spec` and, where composition requires it, `impl -> impl`; hosts/entrypoints select implementations. Keep implementation dependencies out of specs' `api` and `implementation` configurations. Test-only dependencies are assessed separately. During staged migration, inventory and explicitly track dependencies on still-mixed legacy projects; do not report the target graph as achieved until those are classified and resolved.

## Initial domain map for review

| Domain | Candidate spec ownership | Candidate implementation/adapters | Review focus |
| --- | --- | --- | --- |
| Agent storage | `clean-models` for durable events, `contract` for storage operations | `filesystem`, `in-memory`, `filesystem-layout`, `contract-ext` as classified by actual API | Preserve one sealed event hierarchy and stored JSON shapes. |
| Agent state, runtime, session | Existing `contract` projects and any public state values now held elsewhere | `impl`, decorators, context-window projection, repositories, test fixtures | Remove the runtime contract's dependency on concrete Unified Exec without changing lifecycle semantics. |
| Agent context, hooks | Existing focused `contract` projects; prompt grammar only if it is a cross-consumer promise | Filesystem discovery, rendering, hook execution | Do not promote internal renderers into public protocol accidentally. |
| OpenAI | `models`, `client-contract`, model-catalog and account-usage contracts | HTTP client, auth integration, catalog implementation, Codex CLI storage | Keep wire types, transport API, and app-owned auth responsibilities distinct. |
| Tools and MCP | Generic tool contract, per-tool reusable DTO contracts, MCP contract | Tool handlers, command clients, stdio/HTTP transports, MCP manager | Keep tool DTOs reusable without moving sealed storage event subclasses into tool projects. |
| RPC | `models` and frozen `contract` | client, server, in-memory transport, version patch | Do not change cross-wire types or semantics without separate user approval. |
| Application | Frontend contracts, shared settings/auth contracts, migration contract | ViewModels, views, filesystem adapters, CLI, composition | Separate backend truth from frontend projection; avoid a monolithic application spec. |
| Utilities | A corresponding spec for every implementation project, including private helpers | Concrete filesystem, process, codec, terminal, and test helpers | `utils` is a collection, not one domain; find meaningful contracts rather than creating placeholders. |
| Integration tests | No production spec of their own | Cross-domain verification | Remain consumers of public specs and assembled implementations. |

Names in this table identify existing project candidates, not an exhaustive pair-by-pair inventory or final paths. The `spec/` and `impl/` placement and per-implementation pairing are decided; exact per-domain grouping, leaf names, and treatment of non-implementation entrypoints remain to be mapped. Test-support implementations are not exempt from the pairing rule.

## Known boundary and compatibility gates

- `agent-runtime/contract` has `api(project(":tool-unified-exec-impl"))`, because `AgentRuntime.unifiedExecToolClient` exposes the concrete `UnifiedExecToolClient`. Existing [AgentState/Runtime guidance](../../checklist/agent-state-and-runtime.md) explicitly requires this shared session-scoped instance. First map all consumers and ownership; propose the minimal spec-level capability or value view that preserves access, identity, observation, and closure rules. Do not remove or silently replace the property solely to satisfy a graph rule.
- The observed production consumer is `rpc/server/BackendAgentRuntimeRpc`: it reads the shell-session snapshot and Flow and requests close on a registered process. The runtime composition and tool handlers share the same client instance. The current `UnifiedExecProcessSession` observation/close interface lives in the implementation project, so extracting only the client type would leave another spec-to-impl leak. Keep the RPC wire contract unchanged.
- Unified Exec argument DTOs already expose `utils/shell-client`'s `Shell`, which currently shares a Gradle project with process execution, host resolution, and other implementations. Splitting Unified Exec alone cannot establish a fully spec-only transitive dependency graph. Record this as a legacy mixed-project dependency; later separate Shell's contract and its host effects without changing its persisted/model-facing encoding.
- `agent-storage/clean-models` uses `utils/patch`, which mixes public parsed patch models with parser/filesystem behavior. Investigate whether a smaller patch-model spec is necessary; do not move sealed clean events into per-tool modules or duplicate parsed patch DTOs.
- The storage clean-model sealed hierarchy and serialized fields must remain compatible; verify old JSON decode and re-encode when moving Kotlin types. See [Clean Model Rust Alignment](../../checklist/clean-model-rust-alignment.md).
- Existing [OpenAI Module Boundaries](../../checklist/openai-module-boundaries.md) and [Frontend Application Boundary](../../checklist/frontend-application-boundary.md) prescribe current physical locations and dependencies. Any agreed move must update those checklists in the same implementation batch, not leave contradictory rules.
- The [RPC contract freeze](../../checklist/rpc-architecture.md#rpc-契约冻结) remains in force. A module relocation may preserve public symbols and serialization, but any needed contract/behavior change is a separate approval gate, not a refactor detail.
- Gradle source-set target sets, Kotlin packages, `api` exposure, generated serializers/RPC artifacts, test fixtures, and executable assembly must be checked before each move. A directory-only migration is not assumed safe.

## Proposed implementation sequence, subject to review

1. **Inventory and baseline:** enumerate every Gradle project, its main-source dependencies, public API and attached KDoc, owned models/semantics, target set, and test consumers. Label each as candidate spec, implementation, entrypoint, test support, or mixed; map every implementation project to its corresponding spec, including private and test-support implementations. Produce a domain dependency graph; distinguish main from test edges.
2. **Resolve design blockers:** map each project into the chosen `spec/` or `impl/` layout and classify mixed modules and entrypoints. For projects without an identifiable contract, review project responsibility rather than generate an empty spec. For each spec-to-impl edge, record the public type causing it and the minimal semantics-preserving move. Stop at RPC approval gates if exposed types would change.
3. **Pilot Agent Runtime and Unified Exec:** establish their spec boundaries, migrate their production consumers and tests, and check the graph plus behavior. Use the result to tune the migration checklist before broad rollout. This trial is now authorized on the new Kodex branch; the broader migration remains outside its scope.
4. **Migrate in dependency order:** low-level models/utilities and OpenAI/tool contracts; storage clean models and storage contract; Agent state/runtime/session; MCP; RPC/application composition. Adjust order if the measured graph shows a different dependency DAG. Make each batch independently buildable, without leaving a half-moved production entrypoint.
5. **Enforce and document:** add an automated main-source dependency check for `spec -> impl` violations and spec cycles, with explicit classification of allowed third-party dependencies. Update only affected domain checklists after decisions are approved; remove superseded names and rules.

## Validation planned for each implementation batch

- Static: project inclusion and dependency graph; no direct edge from a migrated spec to a known impl or spec cycle; explicitly list dependencies on still-mixed legacy projects rather than claiming global closure. Check for accidental new public API dependencies, and compare public packages/signatures and KMP target coverage where moved.
- Documentation: compare KDoc on each moved declaration before and after; ensure behavior, cancellation, resource ownership, state meaning, and compatibility guarantees remain attached to the spec API. Check KDoc links after relocation and remove superseded or contradictory implementation copies without losing implementation-specific comments.
- Behavioral: relevant spec/impl unit tests; old fixture decode/re-encode for persisted data; RPC serialization and real round-trip tests when touched; frontend/backend ownership and lifetime tests when touched.
- Integration: compile affected downstream projects, JVM tests where available, supported Native compilation, and CLI smoke test for production assembly. Reuse the running Gradle Daemon JVM as required by the Gradle workflow.
- Scope: no deliberate product behavior, storage format, RPC wire shape, settings migration, credential handling, or UI interaction changes in this structural refactor. Any required exception gets an explicit review and its own checks.

## Proposed first pilot: Runtime and Unified Exec

### Physical project mapping

| Current project | Pilot destination | Treatment |
| --- | --- | --- |
| `:tool-unified-exec-contract` (`tool/unified-exec/contract`) | `:tool-unified-exec-spec` (`tool/unified-exec/spec`) | Move existing DTOs, exception, and attached KDoc without changing Kotlin package or serialized field names; add the client and observable session interfaces with their behavioral KDoc. |
| `:tool-unified-exec-impl` (`tool/unified-exec/impl`) | same path/name | Concrete process manager, factory, schemas, platform code, and tool handlers remain here and depend on the spec. |
| `:agent-runtime-contract` (`agent-runtime/contract`) | `:agent-runtime-spec` (`agent-runtime/spec`) | Move `AgentRuntime`, `ResumableAgentLayer`, and their attached KDoc; replace the impl dependency with `api(project(":tool-unified-exec-spec"))`. |
| `:agent-runtime-impl` (`agent-runtime/impl`) | same path/name | Composition stays here, re-exports `:agent-runtime-spec`, and retains the concrete Unified Exec owner. |

- Update all production and test Gradle project references to the two renamed spec projects in the same batch. `settings.gradle.kts` discovers the new paths automatically, but all `project(":...")` strings must be updated. Do not leave deprecated alias projects merely to keep old paths compiling.
- The four `agent-runtime/decorator/*` projects remain at their old paths during this pilot. They are implementation projects and need individual, meaningful spec pairs in a later batch under `agent-runtime/spec/decorator/*` and `agent-runtime/impl/decorator/*`. The pilot is an intentionally partial migration, not an exception to the confirmed pairing rule.
- Keep existing Kotlin package/FQCN for moved types wherever possible. This is a Gradle ownership change, not a wire-format or public naming cleanup.

### Proposed API extraction

- Introduce `UnifiedExecClient : AutoCloseable` in `tool/unified-exec/spec`, with the exact current operations: `activeSessions: StateFlow<Map<Int, UnifiedExecProcessSession>>`, `execCommand(ExecCommandArguments): UnifiedExecOutput`, and `writeStdin(WriteStdinArguments): UnifiedExecOutput`. Retain `UnifiedExecProcessSession`'s `sessionId`, original `arguments`, `completed: StateFlow<Boolean>`, and `close()` in that spec project under its current Kotlin package.
- Move `UnifiedExecProcessSession` with its KDoc intact, adjusting links only as needed. Adapt the existing concrete client's KDoc about the shared process owner, `activeSessions` registration, shell selection, and closing behavior onto `UnifiedExecClient` and its properties; leave concrete construction and synchronization explanations on `UnifiedExecToolClient`. Retain the `AgentRuntime` KDoc that explains turn ownership and the shared shell client, updating its type references rather than deleting it.
- Keep the concrete `UnifiedExecToolClient` class and its `CoroutineScope.UnifiedExecToolClient(...)` factory in `tool/unified-exec/impl`; have the class implement `UnifiedExecClient`. This avoids renaming the constructor/factory API or accidentally making the spec construct effects. `UnifiedExecTools` should consume the interface while the Runtime composition still creates exactly one concrete instance.
- Change only `AgentRuntime.unifiedExecToolClient`'s declared type from the concrete class to `UnifiedExecClient`; keep its property name and the exact shared instance. The implementation may retain the narrower concrete type internally. `rpc/server` needs only the spec-facing observable session members and should not gain a process-management dependency.
- Keep existing argument/output DTOs and `UnifiedExecToolException` in the moved spec. Preserve their existing `Shell` type and serialized shape, even though its current dependency is a mixed legacy project. The currently public yield/output limit constants are used by existing tests and describe observable tool behavior; move them to the spec with unchanged names/values, then verify no duplicate declaration remains in the implementation. Do not change normalization behavior in the pilot.
- Preserve current ownership: Runtime composition creates the client, its two tools share it, and tool closure/Runtime lifecycle still release it. `UnifiedExecProcessSession.close()` requests process-tree termination without removing the registry entry; `writeStdin` may still read the final output. The `activeSessions` registry remains the single source of live shell-session membership.

### Pilot-specific checks and stop conditions

- Check `:agent-runtime-spec` and `:tool-unified-exec-spec` have no **direct** main-source dependency on a known `-impl` project and do not form a spec cycle; in particular, the former must not depend on `:tool-unified-exec-impl`. Audit their transitive `api` exposure and record each mixed legacy project, notably `:utils-shell-client`. Do not present this pilot as proof of global spec-only closure.
- Compile all downstream consumers of the renamed projects, including Agent Session, Agent State tool projection, storage clean models, RPC server, integration tests, and the CLI. Run Unified Exec tests, Agent Runtime/Session tests, and RPC shell-session snapshot/Flow/close tests; include the registered-but-completed state and close-then-read-final-output case.
- Confirm the `rpc/contract` service signatures and serialized `ShellSessionState` remain byte/field compatible, and the backend keeps the [Shell process semantics](../../checklist/rpc-agent-runtime.md#shell-进程). Do not infer wire compatibility from compilation alone.
- The architecture rule becomes a whole-repository gate only after the remaining mixed legacy dependencies are inventoried and migrated. Until then, pilot acceptance requires an explicit unresolved-edge list, not a waiver silently treated as final compliance.
- If preserving the shared instance, lifecycle, or RPC behavior requires modifying the frozen RPC contract, stop and request separate approval. If a renamed public Kotlin type or serialization change appears necessary, document the exact impact before implementation.
- After the user reviews and accepts the experimental shape, update the concrete-type wording in [AgentState/Runtime guidance](../../checklist/agent-state-and-runtime.md), the tool DTO location in [Clean Model Rust Alignment](../../checklist/clean-model-rust-alignment.md), and affected dependency guidance. Do not rewrite confirmed checklists merely because the trial compiles.

The user added the KDoc requirement during pilot API review, then explicitly requested a branch-based code experiment to inspect before accepting the API. Success means presenting the actual diff, validation, and remaining architecture debt for review; it does not mean the user has approved the final spec design.
