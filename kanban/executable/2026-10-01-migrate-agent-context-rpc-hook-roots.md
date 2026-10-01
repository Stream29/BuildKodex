# Task Tree

- `Inventory every project under the agent-context, rpc, and hook roots`()
- **`Define the complete root-to-spec/impl mapping and unresolved helper boundaries`**()
- `Migrate the complete agent-context root`()
- `Migrate the complete rpc root`()
- `Migrate the complete hook root`()
- `Retarget all Gradle consumers and preserve KMP source sets`()
- `Review KDoc, serialization, lifecycle, and dependency direction`()
- `Run domain tests, downstream compilation, and stale-path checks`()
- `Update checklists and close the three-root migration`()

# Details

## Goal and authorization

The user selected the **complete** `agent-context`, `rpc`, and `hook` root
modules for the next migration wave. This is not limited to the previously
identified `prompt-dsl`, `rpc/models`, or `hook/contract` projects. Every
Gradle project below those three roots must be classified and relocated into
the domain convention:

```text
<domain>/spec/<topic>
<domain>/impl/<topic>
```

Pure, implementation-free projects may remain spec-only. Every
implementation-bearing project must either receive a meaningful corresponding
spec or stop at an explicit boundary decision; do not create marker
interfaces, copied DTOs, or empty projects only to satisfy the directory
shape.

Kotlin packages, public FQCNs, serialized names, RPC wire contracts, storage
formats, and observable Hook/Agent behavior remain unchanged. KDoc attached to
public declarations moves with the declarations; the spec-side KDoc is the
normative contract.

## Root inventory and target mapping

### Agent Context

The complete root currently contains these responsibilities:

| Existing project | Target | Responsibility |
| --- | --- | --- |
| `agent-context/contract` | `agent-context/spec/contract` | Shared Agent Context settings and model-facing context contract. |
| `agent-context/prompt-dsl` | `agent-context/spec/prompt-dsl` | Pure prompt XML/DSL model and parser rules; spec-only. |
| `agent-context/prefix/agents-md/contract` | `agent-context/spec/agents-md` | AGENTS.md discovery/model contract. |
| `agent-context/prefix/agents-md/filesystem` | `agent-context/impl/agents-md-filesystem` | Filesystem discovery and loading implementation. |
| `agent-context/prefix/skill/contract` | `agent-context/spec/available-skill` | Available-skill model contract used by prefix rendering. |
| `agent-context/prefix/contract` | `agent-context/spec/prefix` | Prefix composition and resolver-facing contract. |
| `agent-context/prefix/impl` | `agent-context/impl/prefix` | Prefix resolution and context assembly implementation. |
| `agent-context/prefix/render` | `agent-context/impl/prefix-render` | Prefix and planning/skill rendering implementation. |
| `agent-context/skill/contract` | `agent-context/spec/skill` | Skill model and resolver contract. |
| `agent-context/skill/filesystem` | `agent-context/impl/skill-filesystem` | Filesystem-backed skill discovery implementation. |
| `agent-context/skill/render` | `agent-context/impl/skill-render` | Skill document rendering implementation. |

The existing `agent-context/contract` and Unified Exec/Shell types still expose
the known mixed `utils-shell-client-impl` edge. This batch must preserve that
edge explicitly or extract only the minimum Shell-facing spec boundary; it
must not silently claim repository-wide `spec -> spec` closure.

### RPC

The complete root currently contains:

| Existing project | Target | Responsibility |
| --- | --- | --- |
| `rpc/models` | `rpc/spec/models` | Serializable RPC data models and compatibility tests. |
| `rpc/contract` | `rpc/spec/contract` | Frozen service and timeline RPC interfaces/value contracts. |
| `rpc/client` | `rpc/impl/client` | Client-side state restoration, caching, and observation adapters. |
| `rpc/server` | `rpc/impl/server` | Backend RPC composition and service implementation. |
| `rpc/in-memory` | `rpc/impl/in-memory` | In-memory transport and test composition. |
| `rpc/krpc-utils-patch` | `rpc/impl/krpc-utils-patch` | KRPC SDK/platform compatibility patch implementation. |

`rpc/spec/models` and `rpc/spec/contract` are spec-only sources of truth.
`rpc/impl/client`, `rpc/impl/server`, and `rpc/impl/in-memory` consume those
specs. The KRPC patch project was reviewed as an implementation-support
boundary with no project-owned protocol surface; it remains
`rpc/impl/krpc-utils-patch` without a dummy spec.

RPC compatibility gates:

- preserve service method signatures and generated KRPC exposure;
- preserve serialized field names and polymorphic values;
- run existing RPC model round-trip tests;
- run in-memory transport and server behavior tests;
- compile application and integration consumers.

### Hook

The complete root currently contains:

| Existing project | Target | Responsibility |
| --- | --- | --- |
| `hook/contract` | `hook/spec/hooks` | Hook settings, lifecycle, turn, tool, compaction, and error contracts. |
| `hook/impl` | `hook/impl/hooks` | Hook manager, execution, shell-backed commands, and event projections. |
| `hook/tool-utils` | merged into `hook/impl/hooks` | Pre/PostToolUse projection helpers over storage and OpenAI tool events; implementation-only helper code. |

`hook/spec/hooks` owns all public Hook behavior and KDoc. The implementation
must preserve Hook ordering, blocking/continuation semantics, resource
ownership, and shell command behavior. The former `hook/tool-utils` project
was reviewed and merged into `hook/impl/hooks`: its `runPreToolUse` and
`runPostToolUse` helpers are implementation algorithms over the Hook contract,
not an independently reusable protocol surface, so no empty or marker spec was
created.

## Dependency-order batches

1. **Agent Context contracts and pure DSL**
   - Move `agent-context/contract`,
     `prefix/{agents-md/contract,skill/contract,contract}`,
     `skill/contract`, and `prompt-dsl` into `agent-context/spec/*`.
   - Retarget all consumers before moving the filesystem and rendering
     implementations.
2. **Agent Context implementations**
   - Move the five implementation projects into
     `agent-context/impl/*`.
   - Preserve discovery precedence, prompt rendering, skill ordering, and
     filesystem behavior.
3. **RPC models and frozen contracts**
   - Move `rpc/models` and `rpc/contract` into `rpc/spec/*`.
   - Run serialization and contract tests before moving implementation
     projects.
4. **RPC implementations and patch boundary**
   - Move client, server, in-memory transport, and KRPC patch projects into
     `rpc/impl/*`.
   - Keep the KRPC patch implementation-only because it has no reusable
     project-owned protocol contract.
5. **Hook contract and execution**
   - Move `hook/contract` and `hook/impl` into the parallel `spec/hooks` and
     `impl/hooks` projects.
   - Merge the implementation-only `hook/tool-utils` helpers into
     `impl/hooks` and preserve Pre/PostToolUse behavior.
6. **Repository closure**
   - Remove old root project references and generated directories.
   - Update current checklists and record unresolved mixed utility edges.

## Per-batch validation

- Verify `settings.gradle.kts` discovers every new nested project.
- Check no migrated spec has a direct project dependency on a known impl,
  except explicitly recorded legacy edges.
- Compare public packages, signatures, KDoc, target sets, and `api` exposure.
- Run domain JVM tests and relevant downstream consumers after each batch.
- For RPC, perform serialization round trips and in-memory/server checks.
- For Hook, test ordering, blocking, cancellation, command execution, and
  Pre/PostToolUse projection behavior.
- Run `git diff --check` in both repositories and remove stale generated
  directories.
- Record Native/JS/CLI validation separately when unavailable; do not infer
  cross-target compatibility from JVM compilation alone.

## Execution checkpoint

- All eleven Agent Context projects, six RPC projects, and three Hook projects
  now follow the target nested layout; Kotlin packages and public names were
  preserved.
- Agent Context JVM tests passed for prompt DSL, AGENTS.md filesystem loading,
  prefix resolution/rendering, and skill filesystem/rendering.
- RPC JVM tests passed for models, contracts, client, in-memory transport,
  server, and the KRPC compatibility patch.
- Hook implementation JVM tests passed, including the former `tool-utils`
  helper behavior.
- JVM compilation passed for the migrated projects and selected downstream
  consumers: Agent State, Agent Session, Agent Runtime, shared settings,
  application/view-model RPC, RPC test support, and integration test.
- Native/JS/CLI targets were not rerun in this checkpoint. Existing cinterop
  warnings on the Linux host remain expected and are not treated as JVM
  compatibility evidence.

## Non-goals and stop conditions

- Do not change RPC wire contracts, generated method shapes, or persisted JSON.
- Do not change Hook behavior, execution order, or storage projection rules.
- Do not duplicate Agent Context, RPC, or Hook DTOs to fill a spec directory.
- Keep `rpc/impl/krpc-utils-patch` and the merged Hook tool helpers
  implementation-only when they expose no meaningful reusable protocol.
