# Task Tree

- `Inventory every utils project and classify its contract boundary`()
- `Move pure utility contracts under utils/<domain>/spec`()
- **`Split platform and side-effect utilities into spec and impl`()**
- `Migrate shared and per-tool contracts and implementations`()
- `Migrate MCP contracts, composition, and transport pairs`()
- `Retarget all Gradle consumers and preserve KMP source sets`()
- `Validate each utility batch before starting the next`()
- `Remove stale root-level utils projects and guidance`()

# Details

## Goal and authorization

Move the whole `Kodex/utils` tree to the physical convention:

```text
utils/<domain>/spec
utils/<domain>/impl
```

The convention must describe real ownership. A pure, platform-independent
utility may be `spec`-only when its code is itself the reusable contract and
there is no meaningful replaceable implementation. Do not create an empty
`impl` project or duplicate DTOs merely to satisfy the directory shape.

Every implementation project must have a meaningful corresponding `spec`
project. Spec declarations and their attached KDoc remain the authoritative
source; implementation KDoc may describe mechanism but cannot redefine the
contract.

This migration batch now also includes the complete `Kodex/tool` and
`Kodex/mcp` trees. Their contracts are part of the same dependency closure:
tool and MCP specs may consume utility specs, while handlers, transports, and
host/process adapters consume utility implementations. This does not authorize
changing frozen RPC contracts or tool wire/persisted shapes.

## Progress and remaining work

- `utils/patch` is split into `spec` and `impl`; the same pattern has now
  been applied to the remaining utility projects with meaningful
  implementation boundaries.
- `coroutines`, `images`, `kotlinx-io-serialization`, `rpc-exception`, and
  `terminal-text` have moved to their `spec` projects with source and KDoc
  unchanged; `filesystem-lease/contract` has moved to `filesystem-lease/spec`.
- Those projects, filesystem-lease impl, and patch spec/impl passed targeted
  JVM main-source compilation. JVM tests passed for coroutines, images,
  rpc-exception, terminal-text, and filesystem-lease impl.
- `read-write-mutex`, `search-index`, and `external-url` have physical
  spec/impl pairs with matching Gradle consumers. The lock/index/opener
  interfaces are in spec; platform factories and implementations remain in
  impl.
- The remaining utility projects have been migrated. Pure or contract-only
  utilities are spec-only; `host-test-support` has a spec/impl pair whose
  spec owns the reusable mock-engine test dependency and whose impl preserves
  the compatibility coordinate. No production API is introduced. The other
  implementation-bearing utilities now have meaningful spec projects.
- The complete `tool` and `mcp` trees have also been relocated to the common
  `<domain>/<spec|impl>/<topic>` layout. Standalone callable tools have
  spec projects for their model/schema contracts and impl projects for
  registration and host effects; MCP core and both transports have explicit
  spec/impl boundaries.
- Targeted JVM main-source compilation passed for the new utility, tool, and
  MCP projects, and selected utility/tool/MCP tests plus downstream consumer
  compilation passed. Native, JS, live integration, and CLI smoke checks
  remain separately unvalidated.
- One explicitly retained mixed edge remains: `tool/spec/unified-exec`
  consumes `utils-shell-client-impl` because the existing public `Shell` type
  still includes host-resolution members in the implementation project.
  Unified Exec's own spec does not depend on a tool implementation; this
  utility edge remains a follow-up boundary task rather than being hidden as
  repository-wide spec closure.
- `settings.gradle.kts` derives Gradle project paths from directory locations;
  moving a project changes its Gradle path automatically, but every consumer
  declaration must be retargeted.
- Kotlin packages and serialized shapes should remain unchanged unless an API
  extraction makes a change necessary and explicitly reviewed.

## Initial classification and destination

| Existing project | Destination | Classification |
| --- | --- | --- |
| `utils/coroutines` | `utils/coroutines/spec` | Pure coroutine-scope and cancellation contracts; spec-only. |
| `utils/external-url` | `utils/external-url/spec` + `impl` | Result/API in spec; host URL launchers and process effects in impl. |
| `utils/filesystem-lease/contract` | `utils/filesystem-lease/spec` | Public lease contract and exception. |
| `utils/filesystem-lease/impl` | unchanged leaf | Lease acquisition, renewal, and filesystem coordination. |
| `utils/host-test-support` | `utils/host-test-support/spec` + `impl` | Test-only HTTP mock-engine dependency contract in spec; the impl project preserves the host-test-support compatibility coordinate. No production API is introduced. |
| `utils/images` | `utils/images/spec` | Pure image models, detection, data-url and prompt-image rules; spec-only. |
| `utils/images-codec` | `utils/images-codec/spec` + `impl` | Transformer/codec contract in spec; KorIM and target codecs in impl. |
| `utils/kodex-home` | `utils/kodex-home/spec` + `impl` | Stable home-path contract in spec; environment-derived host value in impl. |
| `utils/kotlinx-io-coroutines` | `utils/kotlinx-io-coroutines/spec` + `impl` | Raw I/O/filesystem interfaces in spec; default and platform implementations in impl. |
| `utils/kotlinx-io-serialization` | `utils/kotlinx-io-serialization/spec` | Pure serializer contract; spec-only. |
| `utils/ktor-client-ext` | `utils/ktor-client-ext/spec` + `impl` | Header/SSE observable contract in spec; Ktor pipeline/plugin adapters in impl. |
| `utils/logging` | `utils/logging/spec` + `impl` | Scoped logging contract in spec; file/application logging adapters in impl. |
| `utils/os-environment` | `utils/os-environment/spec` + `impl` | Environment query contract in spec; platform actuals in impl. |
| `utils/process-client` | `utils/process-client/spec` + `impl` | Process command/session contract in spec; platform process clients in impl. |
| `utils/read-write-mutex` | `utils/read-write-mutex/spec` + `impl` | Lock/state and `SafeRw` contract in spec; atomic mutex implementation in impl. |
| `utils/rpc-exception` | `utils/rpc-exception/spec` | Serializable remote-failure contract; spec-only. |
| `utils/search-index` | `utils/search-index/spec` + `impl` | Search document/index contract in spec; Lucene/platform index in impl. |
| `utils/shell-client` | `utils/shell-client/spec` + `impl` | Shell/session/buffer contract in spec; PTY/process/host implementations in impl. |
| `utils/terminal-text` | `utils/terminal-text/spec` | Pure terminal-cell model and algorithms; spec-only. |

The `host-test-support` row is intentionally test-only: its `spec` project
owns the shared mock-engine dependency contract and its `impl` project
preserves the existing compatibility coordinate. It must not gain a dummy
production API.

## Tool and MCP scope

| Existing area | Destination direction | Classification |
| --- | --- | --- |
| `tool/contract` | `tool/spec/contract` | Shared tool capability, invocation, route matching, and lifecycle contract. |
| `tool/image-generation/{contract,impl}` | `tool/{spec,impl}/image-generation` | Move model-facing tool models to spec; keep schemas, clients, handlers, and artifacts in impl. |
| `tool/multi-agent/{contract,impl}` | `tool/{spec,impl}/multi-agent` | Move task models to spec; keep request handling and host effects in impl. |
| `tool/request-user-input/{contract,impl}` | `tool/{spec,impl}/request-user-input` | Move request models to spec; keep UI/request handling in impl. |
| `tool/tool-search/{contract,impl}` | `tool/{spec,impl}/tool-search` | Move search models and observable result contract to spec; keep indexing, conversion, and dispatch in impl. |
| `tool/view-image/{contract,impl}` | `tool/{spec,impl}/view-image` | Move image-view models to spec; keep codec/image adapter and handler in impl. |
| `tool/unified-exec/{spec,impl}` | `tool/{spec,impl}/unified-exec` | Normalize the already split pair to the common layout; preserve the current session/process contract. |
| `tool/apply-patch`, `current-time`, `get-context-remaining`, `plan`, `web-run` | `tool/{spec,impl}/<topic>` | Extract each callable tool's public model/schema/behavior into spec; keep registration and host effects in impl. |
| `tool/tool-builder` | `tool/{spec,impl}/builder` | Spec owns the typed-handler/result adapter contract; impl owns JSON decoding and event construction. |
| `mcp/contract` | `mcp/spec/contract` | MCP protocol-facing configuration, client, manager, service, and tool contracts. |
| `mcp/impl` | `mcp/impl/composition` | MCP manager/service composition, client lifecycle, catalog, reconciliation, and implementation behavior. |
| `mcp/stdio` | `mcp/{spec,impl}/stdio` | Spec owns the process-to-transport ownership contract; impl owns process pipes and SDK transport adaptation. |
| `mcp/streamable-http` | `mcp/{spec,impl}/streamable-http` | Spec owns server-isolated HTTP/auth lifecycle semantics; impl owns Ktor client and SDK transport adaptation. |

Every moved topic must use the common physical shape
`<domain>/<spec|impl>/<topic>`. For example:

```text
tool/spec/image-generation
tool/impl/image-generation
mcp/spec/stdio
mcp/impl/stdio
```

Tool and MCP migration must preserve tool route names, JSON schemas,
serialization, MCP session lifecycle, transport framing, and existing
cross-module API exposure. The current naming of standalone tool entrypoints
is provisional until their source and consumer inventory is reviewed.

## Tool and MCP ownership rules

### Tool

- `tool/spec/contract` is the source of truth for the non-generic `Tool`
  lifecycle, `ToolName`, route matching, and the `ToolSpec` relationship.
- Each `tool/spec/<topic>` owns the topic's model-facing input/output
  contract, schema meaning, lifecycle and failure semantics, plus its KDoc.
- Each `tool/impl/<topic>` owns registration, JSON decoding, host resources,
  filesystem/process access, and concrete event construction.
- `tool/spec/builder` may describe the typed-handler result boundary, but
  decoding strategy selection, serialization failure handling, and clean-event
  construction remain in `tool/impl/builder`.
- `tool_search` remains an agent-loop primitive, but its callable models and
  observable search result shape still belong to `tool/spec/tool-search`;
  indexing and ranking remain in `tool/impl/tool-search`.
- A topic spec may depend on OpenAI or utility specs. A topic impl may depend
  on its topic spec, shared tool specs, utility impls, and other impls when
  composition requires it. No tool spec may depend on a tool impl.

### MCP

- `mcp/spec/contract` owns configuration, authentication state values, manager and
  service operations, client lifecycle states, catalog publication semantics,
  and MCP tool projection contracts.
- `mcp/spec/stdio` owns the observable contract that a configured stdio
  server becomes an MCP transport backed by one owned process session. Raw
  pipe adapters and SDK transport construction stay in impl.
- `mcp/spec/streamable-http` owns server isolation, authorization refresh,
  retry, cancellation, and transport ownership semantics. Ktor interceptors
  and SDK transport construction stay in impl.
- `mcp/impl/composition` may compose both transport impls and utility/process
  implementations, but it must consume transport specs at its public
  boundaries.
- MCP specs may depend on tool and utility specs. MCP impls may depend on
  MCP specs, tool specs/impls, utility specs/impls, and the SDK. No MCP spec
  may depend on a transport or composition impl.

### Compatibility gates

- Keep existing Kotlin packages and serialized field names unless a separate
  API decision approves a public rename.
- Keep `Tool` completion events and MCP `CallToolResult` envelopes as the
  existing source of truth; do not introduce parallel result DTOs.
- Preserve MCP `Connecting`/`Healthy`/`Failed`/`Closed` and authentication
  blocked semantics, catalog generation replacement, and server-isolated
  credentials.
- Preserve stdio stdout framing and the rule that protocol bytes do not pass
  through shell text buffering.
- Update tool/MCP checklist paths only after the corresponding batch has
  passed its source, dependency, and behavior checks.

## Dependency-order batches

1. **Pure contracts and existing pair rename** (completed)
   - `coroutines`, `images`, `kotlinx-io-serialization`,
     `rpc-exception`, `terminal-text`
   - `filesystem-lease/contract -> filesystem-lease/spec`
   - Update consumers and run JVM tests.
2. **Small API/implementation pairs** (completed)
   - `read-write-mutex`, `search-index`, `external-url`,
     `kodex-home`, `host-test-support`
   - Keep platform behavior and test-only target sets unchanged.
3. **I/O foundation** (completed)
   - `kotlinx-io-coroutines`, `os-environment`, `process-client`
   - Extract interfaces before moving platform source sets; verify expect/actual
     ownership and all transitive `api` edges.
4. **Transport and presentation utilities** (completed)
   - `ktor-client-ext`, `logging`, `images-codec`
   - Keep Ktor SSE semantics, logging scope payloads, and codec behavior.
5. **Shell boundary** (completed)
   - `shell-client`
   - Extract shell/process/session/buffer contracts first; keep PTY,
     cinterop, node-pty, and platform resolution in impl.
6. **Tools** (completed)
   - Move existing contract/impl pairs, separate the shared `Tool` contract
     from concrete builders, and split the six standalone tool projects.
   - Do not move durable sealed clean events into tool specs; existing tool
     DTOs and routes keep their serialization and names.
7. **MCP** (completed)
   - Separate core contracts, composition, and the two transport pairs.
   - Preserve raw stdio framing, server-isolated HTTP authorization, borrowed
     versus owned resources, and generation/lifecycle semantics.
8. **Repository closure** (in progress)
   - Complete utility/tool/MCP dependency closure, remove stale root-level
     project references, and update current checklists.
   - Historical `kanban/done` records remain unchanged.

## Guidance conflicts to resolve with implementation

- `checklist/tool-handler-decisions.md` previously forbade spec/impl
  hierarchies; the user's current authorization supersedes that physical
  layout rule. Its old raw `ResponseItem` handler signature also predates the
  current `PendingToolEvent -> StableCleanEvent.CompletedTool` boundary.
- `checklist/clean-model-rust-alignment.md` and
  `checklist/openai-model-alignment.md` still reference tool contract paths.
  Update them with the tool relocation rather than leaving live guidance at
  removed paths.
- MCP and tool-search checklists still refer to unsplit utility/transport
  projects. Retarget them in the corresponding implementation batch; their
  behavioral constraints remain in force.

## Per-batch checks

- `settings.gradle.kts` includes all new nested projects and no old root
  project is still referenced.
- Main-source dependency edges are classified separately from test edges.
  Specs may depend on specs; implementations may depend on specs and other
  implementations; no migrated spec may depend on a known implementation.
- KDoc moves with declarations. Public throwing operations retain explicit
  `@throws` conditions in spec.
- Kotlin packages, KMP target coverage, expect/actual pairing, serialized
  fields, and public FQCNs remain compatible.
- Run affected JVM compile/tests and downstream consumers. Native/JS/CLI
  checks are recorded separately when not available or when shared compute is
  occupied.
- Run `git diff --check` in both repositories and remove stale generated
  directories after each physical move.

## Stop conditions

- Stop before extracting a boundary that would require changing a frozen RPC
  contract, persisted JSON shape, or public serialization without explicit
  approval.
- Stop and revise the mapping if a spec would need a concrete host effect,
  platform cinterop, or a copied model merely to compile.
- Do not claim the whole `utils` graph is migrated until every row and every
  consumer has passed the closure checks.
