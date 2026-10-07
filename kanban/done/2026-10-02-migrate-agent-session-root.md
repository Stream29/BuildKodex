# Task Tree

- `Inventory agent-session public contracts and implementation edges`()
- `Classify the shared test-support project`()
- `Move the session contract into spec/contract`()
- `Move filesystem and in-memory repositories into impl topics`()
- `Retarget all production and test consumers`()
- `Validate lifecycle, persistence, and repository behavior`()
- `Update current checklists and close the executable task`()
- `Archive delivered Session migration with later lifecycle acceptance`()

# Details

## Reconciled status — 2026-10-07

- **DONE — Session 原契约与仓库迁移已交付**。早期 mixed Shell 主边已在
  [真实平台契约修复](2026-10-05-repair-platform-contracts.md)中解除；
  `agent-session/test` 是明确测试支撑，不为形式新增空 spec。
- [Storage 清理验收](2026-10-05-repair-storage-cleanup.md)、
  [后端终验](2026-10-07-final-audit-backend-roots.md)及
  [限定收尾](2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)
  覆盖原 Session/缓存/lease 真实调用、故障回归与独立增量审查。
  [核心 native E2E](2026-10-07-accept-sealed-cli-core-e2e.md)实际验证持久化重开、
  fork/revert/delete/archive 与进程租约；[发布](2026-10-07-release-kodex-0-4-10.md#publication-verified--v0410)
  交付 main `83749434`。
- 下文是原迁移时的验证范围，不改写当时 Native/JS/CLI 未跑事实。OS publication
  后取消、故障矩阵与其他 B2/U 仍按后续收尾跟踪；
  [迁移总任务](../executable/2026-09-28-plan-domain-spec-impl-boundaries.md)保持打开。

## Authorization and scope

- The user approved this task for executable implementation after reviewing the
  agent-session migration proposal.
- Migrate the complete `agent-session` root as one complex batch.
- Do not migrate `agent-state`, `app`, or unrelated roots in this task.
- Preserve Kotlin packages, public names, Session lifecycle behavior, persisted
  storage semantics, and existing test coverage unless a separate approval is
  requested.
- Do not commit or push unless the user explicitly asks after review.

## Target mapping

| Current project | Target project | Responsibility |
| --- | --- | --- |
| `agent-session/contract` | `agent-session/spec/contract` | Session, repository, entry, and dependency contracts with authoritative KDoc. |
| `agent-session/filesystem` | `agent-session/impl/filesystem` | Filesystem Session repository, cached storage, leases, and persistence composition. |
| `agent-session/in-memory` | `agent-session/impl/in-memory` | In-memory Session repository and runtime-backed test behavior. |
| `agent-session/test` | `agent-session/test` or a nested spec/impl topic after inspection | Shared test dependency factory; classify by actual reusable contract rather than mechanically renaming it. |

The test-support project must not introduce a production dependency on an
implementation merely to create an empty spec. If its public factory is a
reusable test contract, give it an explicit spec owner; otherwise keep it as a
test-support implementation and document the exception.

## Boundary and dependency gates

- `spec/contract` may depend on stable specs and third-party libraries, but
  must not gain a direct dependency on `agent-session/impl/*`.
- The existing `utils-shell-client-impl` dependency of the Session contract is
  a known mixed legacy edge. Preserve behavior first and record it explicitly;
  do not invent a duplicate Shell DTO or silently expand this task into a
  shell-client migration.
- `impl/filesystem` may depend on Agent Runtime composition, Agent State,
  Storage filesystem, and filesystem utility implementations.
- `impl/in-memory` may depend on Agent Runtime composition, Agent State, and
  Storage in-memory.
- Test-only edges may use implementations, but must remain in test source
  sets and must not leak through `api` from the Session spec.
- Keep Session repository ownership, close/cancellation behavior, fork/delete
  semantics, cached index behavior, and filesystem lease behavior unchanged.

## Required validation

- Compare public signatures, attached KDoc, Kotlin packages, source-set
  targets, and Gradle `api`/`implementation` exposure before and after.
- Run all agent-session JVM tests, including filesystem and in-memory tests.
- Compile direct downstream consumers: Agent State, app session/title/history
  consumers, RPC test support, and integration-test where applicable.
- Run relevant Session lifecycle and persistence tests, including repository
  close/cancellation, cached storage, fork/delete, and filesystem layout
  compatibility.
- Run `git diff --check` in both repositories.
- Scan current checklist, planning, executable, and shared-context documents
  for stale active `agent-session/{contract,filesystem,in-memory,test}` paths.
- Record Native/JS/CLI validation separately if not run on the current host.

## Stop conditions

- Stop and request approval if moving the contract requires changing public
  Kotlin names, persisted JSON, RPC wire types, Session lifecycle semantics,
  or the frozen Shell dependency boundary.
- Do not claim repository-wide spec-only closure; unrelated mixed legacy edges
  remain outside this task.

## Execution checkpoint

- `agent-session/spec/contract`, `agent-session/impl/filesystem`, and
  `agent-session/impl/in-memory` are now the active project locations.
- The shared `agent-session/test` project remains at its existing path; it is
  test support rather than a production Session implementation and is not
  mechanically given an empty spec.
- Migrated Session JVM compilation and tests passed:
  `:agent-session-spec-contract:compileKotlinJvm`,
  `:agent-session-impl-filesystem:compileKotlinJvm`,
  `:agent-session-impl-in-memory:compileKotlinJvm`,
  `:agent-session-test:compileKotlinJvm`,
  `:agent-session-impl-filesystem:jvmTest`, and
  `:agent-session-impl-in-memory:jvmTest`.
- Selected downstream JVM compilation passed through Agent State, application
  Session/title/history consumers, RPC server, test support, and integration
  test.
- No Kotlin package, public symbol, persistence, or Session lifecycle changes
  were introduced. Native/JS/CLI validation remains outstanding.
