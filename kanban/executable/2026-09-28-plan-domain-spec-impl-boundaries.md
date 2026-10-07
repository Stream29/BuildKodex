# Task Tree

- `Confirm domain scope, naming, pairing, and KDoc ownership`()
- `Specify the Runtime and Unified Exec first batch`()
- `Commit outer repository records in coherent batches`()
- `Create the Kodex refactor branch`()
- `Move the first-batch contracts and KDoc into spec projects`()
- `Adapt implementations and downstream Gradle dependencies`()
- `Review the first-batch API with the user`()
- `Split all four Runtime decorators into spec/impl pairs`()
- `Validate decorator specs, implementations, and consumers`()
- `Inventory remaining modules and plan migration batches`()
- `Plan and implement Session Configuration and Browsing component batch`()
- `Record Session Configuration and Browsing batch acceptance and archive`
- `Plan History Item, Composer, New Session, and Session Tab Bar component batch`()
- `Execute the authorized twelve-component batch`()
- `Track remaining History validation after the authorized hard cutover`()
- `Reaccept previous migrations and commit accepted scoped repairs`()
- `Review complete app/shared migration scope after repairs`()
- `Plan behavior ownership and reconsider notification Hooks`()
- `Receive start of the engineering-ready ownership cutover`()
- `Execute notification Hook, settings persistence, and backend capability cutover`()
- `Receive scoped ownership-cutover acceptance`()
- `Select the complete backend AgentState root as the next batch`()
- `Review the AgentState hard-cutover engineering plan`()
- `Execute the authorized complete AgentState hard cutover`()
- `Receive scoped AgentState acceptance with the known compaction defect recorded`()
- `Authorize coherent commits and push for the accepted AgentState migration`()
- `Select six complete frontend ownership units`()
- `Plan the Agent, Shell, Sidebar, Settings, Session, and Application hard cutover`()
- `Receive review and implementation authorization for the frontend root batch`()
- `Execute the authorized six-unit frontend hard cutover`()
- `Receive scoped frontend-root acceptance`()
- `Commit the accepted frontend-root batch in coherent slices`()
- `Select and approve UI foundations and Home bootstrap boundaries`()
- `Execute the authorized UI foundations and Home hard cutover`()
- `Independently reaccept and locally commit the UI/Home batch`()
- `Plan repository closure and remaining lifecycle or state-consistency work`()
- `Execute the authorized parallel repository reacceptance`()
- `Review qualified findings and authorize scoped closure work`()
- `Execute repairs and independently re-review actual ownership`()
- `Receive authorization to seal and commit scoped closure with residual risks`()
- `Prepare a pinned complete CLI for end-to-end acceptance`()
- **`Run full end-to-end acceptance against the sealed baseline`()**
- `Enforce actual contracts and validate affected end-to-end behavior`()
- `Update confirmed checklists and close the task`()

# Details

## Goal and authorization

- 用户选择并批准 [UI/Home 收口方案](../done/2026-10-07-cutover-ui-foundations-and-home-bootstrap.md)，
  随后明确直接执行。五个范围项目收敛为三个，History 保留原领域身份并解除滚动
  框架耦合；历史 migration 的内容和路径不动。用户随后明确要求独立验收，
  通过就提交并规划下一批；独立验收通过，四份任务归档 done。本轮仅本地提交，
  不推送。五个核心测试目标强制重跑 402 项，显式启用 Patch probe 重跑 1 项；
  新隔离 CLI 启动/Settings 开关/退出正常，冻结源与实际主图复核通过。
  既有 Home/State 故障保留，下一批以真实剩余职责为前提，不重复迁移已接管领域。
  本批内层提交 `fac1bdb6` / `ec9bf1c4` / `6b7129fa`，外层验收与指针已本地提交。
  下一批 [收尾与一致性任务](../done/2026-10-07-plan-spec-impl-closure-and-consistency.md)
  已建立：206 个实际项目的直接主图无 spec→impl/旧混合主边，不再机械创造目录迁移对象；
  初次 Planning 提出真实 lease 等待/取消交接与 compaction settings 保留规则；
  该初始未执行 checkpoint 随后被用户的修复授权推进，不是当前等待状态。
  用户随后要求赶紧并行核验“全仓是否已迁移”，
  [八线全仓终验](../done/2026-10-07-final-reaccept-spec-impl-repository.md#coordinator-final-verdict--2026-10-07)
  已完成并归档审查工作，固定 `6b7129fa`，206 个项目无遗漏或重复覆盖。
  结论为目录基本收口、真实接管主体成立，但严格兼容清理与行为终验未通过；
  kRPC substitution 不在直接项目图的证明范围内。新增修复/实验/政策门槛已关联
  收尾计划，迁移总任务不关闭。用户现明确授权“修，修完重新审查”，
  [修复主任务](../done/2026-10-07-plan-spec-impl-closure-and-consistency.md)及六线进入
  executable；先修确认缺陷、疑点先复现，政策变化不借修复自动决定。
  当前实际项目 203，固定快照的独立复审发现的缓存/共享 lease、启动 primary、
  Native terminal、MCP 回滚和 History completion 末项已补修并定向验证，
  没有新建项目。最终 35 个 JVM 目标 1152 项通过（含 up-to-date）；
  Node/Linux 行为门槛、最终 CLI 链接和隔离实操通过，准确证据见
  [限定范围交付](../done/2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)。
  原两次 View 压力失败仍保留；fixture identity/payload 门闩做消融 red 后恢复，
  continuous input 三次 fresh 通过，不证明历史失败唯一根因或所有 admission。
  因保留 B2、resolved 依赖债务与未验证平台，总任务不关闭。
  用户确认 remote compaction 保留 20、
  不增加历史图片/自动回答产品算法；修复五批现已本地提交，未推送。
  用户要求封存现场并构建完整二进制，
  [E2E 基线与产物任务](2026-10-07-seal-repair-baseline-and-build-e2e-cli.md)
  接管；修复记录归档，不等同完整 E2E 或残余 B2/U 验收。

- 用户因 Shell 契约未真实接入而要求重新验收此前全部重构，并要求多个独立 Session
  并行。重验期间 app/shared 与 Shell 新迁移暂停；共同标准、范围与证据报告见
  [Spec/impl reacceptance](2026-10-05-reaccept-spec-impl-refactor.md)。此前的完成记录
  作为历史输入，不作为本轮默认通过依据。
- 七条审查线已交接；协调者重验结论为不通过。已确认平台 utils/MCP 的未接管
  契约和冗余转发、Composer 生产接线回归、MCP effect 的 owner/composition 不匹配，
  另有 mock/契约文档问题。后端真实声明已直接迁移，原有缺陷单列，不做全面回滚。
- 用户随后授权四条修复线，现已接受当前修复并要求分批提交；集中 JVM 回归
  （869 项含 up-to-date 结果）及 JS/Linux x64 编译通过。结果、提交与未关闭
  策略/实验见重验任务的 `Central repair validation`。
- 用户随后要求重新讨论 DDD/MVVM 归属，并进入 Planning、复核多次演变的 Hook 设计。
  [按行为重新归属](../done/2026-10-06-reassign-shared-application-behavior.md)
  取代“按四个 app/shared 目录机械迁移”的前提；认证作为 Settings 功能、
  命名归后端、通知执行归 Hook。用户已确认完整退役旧控制 Hook，
  执行语义沿用 RPC 原 notification-only 决策，不把目录整理作为重新选择产品能力的理由。
  主计划及三条工程线已补齐：四个新项目、十个旧项目退役、原类型唯一接管、
  共享文件所有权和实际验收矩阵明确；当前无已知设计/外部前置阻塞。
  用户随后明确启动，四份任务移入 executable，三条实施线完成直接接管与旧路径删除；
  29 个 JVM 有测试目标共 846 项（含 up-to-date 结果）、CLI/Integration JVM 编译和
  十一个 Linux x64 主源码目标通过。用户已验收并要求提交，代码本地提交 `206266c8`；
  [统一证据与平台范围](../done/2026-10-06-reassign-shared-application-behavior.md#central-acceptance-evidence)，
  四份任务归档 done，本轮不推送，下一批仅做候选调研与选择。
  重验遗留风险继续独立跟踪。

- 下一批用户选定完整后端 `agent-state`，随后授权进入 Planning；
  [AgentState 硬迁移任务](../done/2026-10-06-hard-migrate-agent-state.md)
  覆盖 contract、context-window、tool、impl、test 与消费者。
  方案直接接管原 `KodexAgentState`，合并具体 tool 支撑到 State 实现，
  保留真实预算 spec 与测试支持；前端父级组合暂不纳入。
  工程映射与静态依赖投影已核对，用户随后授权“执行吧”，三份任务进入 executable。
  State/spec、唯一实现与全部消费方已硬迁移，tool 支撑项目并入实际实现；
  Xiaoxin Ubuntu 的 14 个 JVM 测试目标共 562 项、CLI/Integration 编译及
  四个 State Linux x64/JS 主源码目标通过；用户已范围验收并授权分批提交推送，
  主任务与两份子任务归档 done。
  内层分批提交为 `19bad603`（硬迁移）与 `034b778c`（补充验证）；
  外层归档记录和子模块指针分别提交，推送顺序为内层分支、外层 main。
  压缩期间成功更新 settings 被开始快照覆盖的基线缺陷已由 gated 测试复现，
  保留为未修复问题，不在目录迁移中偷偷改变并发协议。

- 用户随后选定六块完整前端范围，并明确“开始计划”：
  [Frontend ownership roots](../done/2026-10-06-hard-migrate-frontend-ownership-roots.md)
  及三份工程线覆盖 Agent 页面、Shell 会话面板、左右侧栏、Settings 根、
  Session 根和 Application 根。方案保留原领域身份，消融空装配项目，
  不机械建立六套新 ViewModel。Agent 双层合并和 index→exact-target 命令
  是明确提出的消融/行为调整，当时需随 Planning 评审、尚未授权实施。
  用户要求自查后，取消独立 Shell spec/View 两项目：原 Shell 接口留在 Agent spec，
  列表/菜单/hover 留在 Sidebar View。六块功能不减少，项目映射由 13→15 收敛
  为 13→13；不增加 Shell VM、错误中转服务或侧栏专用的通用 UI 层。
  用户随后明确授权执行，主计划与三份子计划进入 executable。现已直接接管原类型
  及唯一实现，十三个旧项目入口退出；最终 650 项定向 JVM 回归、CLI/Integration
  JVM 编译及十三个新项目 Linux x64 主源码编译通过。
  gated registry race 与已取消 Global 订阅迟到错误已证实并作最小修正，
  原 State compaction 风险仍保留。追加当前源码的真实 Linux x64 CLI 两次运行，
  导航、物化、错误反馈、重开、持久化与退出通过；窄屏双侧栏的基线限制单列。
  用户自行验收后授权本地分批提交，四份实施任务与 root 子线历史 handoff 归档 done；
  不包含推送或启动下一批，未覆盖链路与平台继续保留。
  内层提交为 `9272f26f`（真实所有权硬迁移）和 `8297d388`（独立新增回归）。

- User goal: reorganize Kodex so each domain has one or several clearly identified `spec` modules as its source of truth; the remaining modules implement or adapt those specifications.
- Confirmed planning granularity: organize and migrate by domain. Confirmed project-level coverage: every implementation Gradle project, including private helpers, has a corresponding spec project; a domain may therefore contain several spec/impl pairs. This supersedes the earlier proposed exception for implementation-only helpers.
- Confirmed physical naming: place spec projects under each domain's `spec/` tree and implementation projects under its `impl/` tree; the directory placement must also appear in Gradle project paths (for example, `openai/spec/models` becomes `:openai-spec-models`). Apply this in reviewable batches, not by an unvalidated whole-repository rename.
- Confirmed first batch: Agent Runtime and Unified Exec, rather than a low-risk rename-only slice. The batch must resolve the existing runtime-spec dependency on a concrete tool implementation while preserving the session-scoped shell client behavior. `AgentRuntime` inherits `ResumableAgentLayer`, retaining its own `resume()` KDoc.
- Confirmed documentation requirement: a spec contains both its code declarations and their attached KDoc and is the authoritative source for the module contract. Implementation declarations may also have KDoc, but it is descriptive of that implementation and cannot override or contradict the spec.
- The user has renamed the inner branch to `refactor/spec` and explicitly authorized formal work on the repository-wide split there. Migrate by coherent domain batches, not by generating placeholder specs for every existing directory at once. The first-batch API may still be refined as later domain boundaries are learned. Do not change the frozen RPC contract without separate approval.
- The user requested commits for the outer repository, but has not asked for an inner `Kodex/` commit or push. Keep inner changes inspectable and uncommitted unless separately requested.
- Primary work tree is `Kodex/`. `Kodex/settings.gradle.kts` currently includes module trees by directory; a physical rename changes Gradle project paths and every consumer declaration.

## Experiment state

- Outer `main` received three separate documentation commits before the inner branch was created: native crash/suspend findings `11b75f1`, history Work scroll investigation `fe9d396`, and this spec/impl task `6aa65f9`. No push was requested or performed.
- Inner `Kodex/` branch: `refactor/spec` (user-renamed from `experiment/spec-impl-pilot-2026-09-28`), created from clean `main` at `66e6ba70`. Changes remain uncommitted.
- First-batch changes so far: moved the two core contract projects to `spec` paths, added the Unified Exec client/session spec interfaces and KDoc, retained the concrete process manager in impl, updated direct Gradle consumers, and made `AgentRuntime` inherit `ResumableAgentLayer`.
- A first targeted Gradle compile was cancelled before completion: an existing Gradle Daemon was busy, and Gradle started a second daemon despite using the detected daemon JVM. The newly started daemon was stopped; the original daemon was left untouched. After `gradlew --status` reported the original daemon idle, the targeted JVM compilation was rerun using that daemon and passed (`:tool-unified-exec-spec`, `:tool-unified-exec-impl`, `:agent-runtime-spec`, and `:agent-runtime-impl`; `--max-workers=2`).
- Subsequent validation passed using the original Gradle Daemon with `--max-workers=1`: JVM tests for Unified Exec, Agent Session in-memory, RPC server, and all four Agent Runtime decorators; `:app-contract-agent:compileKotlinJvm` and `:agent-storage-clean-models:jvmTest`. Static inspection found 130 included project build files and no stale project references. The moved Unified Exec DTO file was byte-identical to its original at that point; `ResumableAgentLayer` subsequently gained contract KDoc.
- After the inheritance adjustment, targeted Agent Runtime, Agent Session in-memory, and RPC server JVM compilation passed. Native compilation and real CLI smoke remain outstanding; report them as remaining validation, not as passed checks.
- Next straightforward pairs were moved without initially changing source contents or Kotlin packages: `openai/account-usage/{contract,impl}` to `openai/{spec,impl}/account-usage` and `openai/model-catalog/{contract,impl}` to `openai/{spec,impl}/model-catalog`. Their direct Gradle consumers were retargeted; a static inventory finds no missing project references or old names. Both pairs' relocated JVM tests subsequently passed as part of the remaining OpenAI migration.
- KDoc review found additional detail on the model-catalog implementation (bundled initial snapshot, refresh publication, longest-prefix/fallback resolution, and close ownership). The user clarified that implementation KDoc may stay in place; it is not the normative source. Cross-implementation refresh, resolution, and ownership guarantees are now described in spec; the bundled initial catalog remains implementation detail.
- The user subsequently required **each** of the four Runtime decorators (`compact`, `steer`, `tool`, `turn-hook`) to have its own `spec/impl` pair. This supersedes the earlier deferral and any proposed merger. The migration is in progress on `refactor/spec`; each spec must contain an actual used contract and its KDoc.
- The user corrected the first decorator split: extracting routing or event projection into spec without a decorator interface does **not** define the decorator's contract. Each spec now declares its existing runtime name as a `ResumableAgentLayer`-derived interface with behavior KDoc (`KodexAgentCompactionRuntime`, `SteerRuntime`, `KodexToolRuntime`, `TurnHookRuntime`); each implementation class is named `*Impl` and implements its corresponding interface, and the four extension factories return interfaces. The compact retry-limit exception and atomic `SteerProvider` remain in their specs. `ToolRouting` and `TurnHookProjection` are implementation helpers again. Kotlin packages and runtime sequencing were retained. All eight decorator spec/impl JVM main-source compile tasks passed using the original Gradle Daemon (`--max-workers=1`); no second Daemon was started. The relocated decorator tests, except the compact test noted below, and broader downstream validation remain outstanding; do not describe the follow-up batch as fully validated.
- The user then corrected the core Runtime spec layout: `agent-runtime/spec/contract` must own `AgentRuntime` and `ResumableAgentLayer`, as a sibling of `agent-runtime/spec/decorator`, rather than placing source directly under `agent-runtime/spec/src`. The Gradle project becomes `:agent-runtime-spec-contract`; the Kotlin package is unchanged and the contract KDoc has since been expanded. All direct Gradle project references were retargeted, and the old spec project/source tree was removed. Static stale-reference and whitespace checks and targeted JVM compilation passed.
- The user clarified that `CompactionRetryPolicy` and its chosen retry count are implementation details. The policy now lives in `agent-runtime/impl/decorator/compact`; only the observable `AgentResponseRetryLimitExceededException` type remains in compact spec. Spec KDoc promises retry-limit failure without importing the implementation's numeric limit. After this adjustment, `:agent-runtime-spec-contract:compileKotlinJvm` and `:agent-runtime-impl-decorator-compact:jvmTest` passed using the existing Gradle Daemon with `--max-workers=1`; no second Daemon was started.
- The user additionally requires spec KDoc `@throws` on operations that expose an exception; `KodexAgentCompactionRuntime.resume()` now names `AgentResponseRetryLimitExceededException` and its trigger condition. The later Runtime spec compilation covered this KDoc-only adjustment.
- Review of this batch's other spec declarations found contract-level exceptions missing from `AgentRuntime.resume/compact` (occupied turn slot), `UnifiedExecClient.execCommand/writeStdin` (validation, session, or process failure), `KodexToolRuntime.resume` (dynamic route collision), `OpenAiModelCatalog.refresh` (structured provider error), and account-usage reset creation/consumption (invalid or unavailable account state and invalid attempt). Precise `@throws` KDoc now sits on those operations; unspecified delegate, Hook, transport, and cancellation exception types were not promoted into guarantees. The old Agent Runtime checklist still claimed four retries despite the implementation using 20; its number was removed because the retry policy belongs to impl. Targeted JVM compiles for the changed specs and `:agent-runtime-impl` passed with `--max-workers=1`; no behavioral tests were rerun for this KDoc/spec-signature audit.
- The user clarified the purpose of `SteerProvider`: hide the pending StateFlow's atomic claim-and-clear/CAS from decorator business logic. Its spec KDoc must require a linearizable ownership transfer against appends and competing claims, preserve batch order, return empty without waiting when nothing is pending, and distinguish claiming from later AgentState persistence. The current host uses `getAndUpdate { emptyList() }`; interruption claims from the same source.
- The user asked for a fuller `KodexToolRuntime` spec KDoc. It now documents borrowed resource ownership, catalog sampling and unique callable routes, pending-before-delegation order, invalid/search special cases, Pre/PostToolUse sequencing, state-bound completion avoidance, unavailable MCP versus unowned routes, observable return/resume boundaries, and propagated failures. Route indexing and execution code remain in impl. `:agent-runtime-spec-decorator-tool:compileKotlinJvm` passed; no behavioral tests were rerun for this documentation change.
- The user then requested the same behavior-level KDoc review for the rest of this batch. Compared the Runtime core and all four decorator specs, Unified Exec spec, and both OpenAI specs with their implementations and available tests. Expanded contract KDoc for turn admission/cancellation, compaction and Hook sequencing, steer delivery, process-session ownership and output semantics, catalog refresh/resolution, and account-isolated usage/reset states and DTOs. Kept concrete retry counts, initial bundled catalog contents, and process/synchronization algorithms in impl rather than promising them through spec. Corrected the Unified Exec byte-limit description and stale model-catalog checklist paths. Targeted JVM main-source compilation passed for all eight spec projects in this batch and the Runtime and both OpenAI implementations (`--max-workers=1`, existing Daemon); behavioral tests were not rerun because the edits are documentation-only except for explicit `close()` declarations already implemented by the OpenAI impls.
- The user corrected the core implementation layout: the standalone `agent-runtime/impl/src` project was moved alongside the decorator projects, without changing its Kotlin package or behavior. The first `impl/contract` name was provisional; the user selected `agent-runtime/impl/composition` and Gradle project `:agent-runtime-impl-composition` because this project assembles the concrete runtime against `agent-runtime/spec/contract`. Its three direct Gradle consumers use the new project path. Compilation and Agent Session JVM tests passed under the provisional path; JVM compilation of the final `composition` path and downstream application/integration-test compilation passed during the OpenAI migration. The removed `impl/contract` project has only an ignored generated `build/` directory remaining, which should be cleaned when it is safe to do so. The three `KodexDocs/drafts/execution-controls` links to the original `impl/src` path remain historical references in a separate submodule; do not silently edit that other repository on this Kodex branch.
- The user's follow-up request to complete the OpenAI split is tracked as [Complete OpenAI Spec/Impl Split](2026-09-29-complete-openai-spec-impl-split.md). All six remaining legacy projects have been physically migrated; migrated OpenAI JVM builds/tests and selected downstream JVM checks passed. Native compilation, live integration tests, and CLI smoke remain unrun.
- The next migration scope was the whole `utils` tree plus the complete
  `tool` and `mcp` trees. That relocation has now been executed on
  `refactor/spec`; the detailed mapping and validation record remain in
  [Migrate Utils, Tools, and MCP to Spec/Impl](2026-09-30-migrate-all-utils-to-spec-impl.md).
  The physical layout and direct Gradle consumers are updated, targeted JVM
  compilation/tests passed, and repository closure remains for stale-path
  cleanup, checklist synchronization, and recording unvalidated Native/JS/
  live/CLI checks. `host-test-support` is intentionally test-only; its spec
  owns the shared mock-engine dependency contract and its impl preserves the
  compatibility coordinate without introducing a production API.
- The complete `agent-context`, `rpc`, and `hook` root migration is recorded in
  [Migrate Agent Context, RPC, and Hook Roots](2026-10-01-migrate-agent-context-rpc-hook-roots.md).
  All three roots now follow the nested spec/impl layout, with JVM tests and
  selected downstream compilation passing. The KRPC patch and Hook tool helpers
  remain explicitly implementation-only; Native/JS/CLI validation is still
  outstanding.
- The complete `agent-storage` root is now migrated on `refactor/spec` with
  this physical mapping:
  `spec/{clean-models,contract,contract-ext}` and
  `impl/{filesystem,filesystem-layout,in-memory}`. The clean models,
  storage contracts, and contract extensions remain spec sources; filesystem
  and in-memory are implementations. `filesystem-layout` is deliberately
  classified as implementation support for now because its public helpers
  directly depend on the concrete `CoroutineFileSystem`; it is not presented
  as a pure protocol spec. The clean-models test source has a direct
  test-only dependency on `utils-kotlinx-io-coroutines-impl` for filesystem
  serialization fixtures; its production source remains spec-oriented.
  Agent-storage JVM tests passed (55 tests across the migrated projects), and
  selected downstream JVM compilation passed through Agent State, Agent
  Session, Agent Runtime, RPC, application, and integration-test consumers.
  Native/JS/CLI validation remains outstanding.
- The next authorized complex batch is the complete `agent-session` root,
  tracked in [Migrate Agent Session Root](2026-10-02-migrate-agent-session-root.md).
  Its target is `spec/contract`, `impl/filesystem`, and `impl/in-memory`, with
  the existing test-support project classified separately by actual reusable
  contract. Preserve Session lifecycle, repository ownership, persistence,
  fork/delete, and cancellation behavior. The known `spec/contract` edge to
  `utils-shell-client-impl` remains explicitly mixed legacy debt; it is not a
  reason to duplicate Shell types in this batch.
- The agent-session executable has now been implemented on `refactor/spec`.
  The three production projects use the target nested layout, while the
  shared `agent-session/test` factory remains test support at its existing
  path. Session JVM compilation/tests and selected downstream JVM compilation
  passed through Agent State, application consumers, RPC server, test support,
  and integration-test. Native/JS/CLI validation remains outstanding.
- The accepted first app-component batch is tracked in
  [Migrate Small App Contracts](../done/2026-10-02-migrate-small-app-contracts.md):
  `path-picker` and `session-catalog` are vertical components under
  `app/component/<name>/{spec,impl/<topic>}`. Their specs are the fact source
  for ViewModel dependencies, interactions, state/effect transitions, and
  renderer semantics; existing RPC/UI hosts are staged implementation debt.
- The first app-component batch is implemented and accepted. Path-picker has dedicated
  spec, ViewModel implementation, and View implementation topics. Session
  Catalog has a dedicated spec and framework-free `impl/viewmodel`; its RPC
  host is a dependency adapter while its renderer remains hosted by
  `app/view/application`. Component tests, RPC/Session/Application regression
  tests and direct downstream JVM compilation passed. Native/JS/CLI validation
  remains outstanding.

- The accepted Login component batch is
  [Extract OpenAI Login Component](../done/2026-10-02-extract-openai-login-component.md).
  Its spec, ViewModel and View are implemented under
  `app/component/openai-login/{spec,impl/viewmodel,impl/view}` without migrating
  all Settings or changing backend OAuth protocols. Component tests and
  Settings/RPC/Application JVM regressions passed; the user accepted continuation.
  Its dependency on the mixed legacy auth-contract project remains explicit.
- The next authorized family is
  [Extract Session Action Components](../done/2026-10-02-extract-session-action-components.md):
  migrate Session Rename and Session Delete together, including every
  Application, Settings and Catalog entrypoint. Group future batches by
  interaction/lifecycle similarity, preserving host-specific operation semantics.
  All six component projects are implemented; 25 component and 216 downstream
  JVM tests passed, and the user accepted the batch. Native/JS/CLI
  validation remains outstanding.
- The accepted Path Picker/Catalog, Login and Session Action component batches
  are submitted together in inner commit `fc9f1936`, preserving a buildable
  composition across their overlapping host files.
- The next similar interaction family is
  [Extract Working Directory Component](../done/2026-10-02-extract-working-directory-component.md),
  covering Application Draft/Agent/suggestion and Settings picker owners while
  preserving their target, callId, handle and revision admission rules.
  Spec, ViewModel and View are implemented; all 257 scoped JVM tests passed.
  This batch and the following four components are accepted and committed
  together as `7310a445`; Native/JS/CLI validation remains outstanding.

## Confirmed spec/impl rules

- 用户已选定 12 类应用交互并已授权实施：
  九个 History Item 组件、Composer、New Session Screen、Session Tab Bar。
  详见 [12-component executable](2026-10-03-extract-history-composer-and-navigation-components.md)
  及其三份工作线计划。History 已另获授权并进入
  [AgentHistory 硬迁移](2026-10-04-extract-history-aggregate-component.md)；
  不保留 parallel model/adapter，真实九类 item 随其内聚组件一起接管。Session Sidebar、Application
  Shell 和 `agent-state` 暂不纳入本批。硬迁移代码已提交为 `821b6627`，最终
  308 项定向 JVM 测试通过；压力用例偶发超时和未验证平台保持未闭合，不因提交而归档。

- 本批包含两个家族、五个完整组件：Session Settings、New Session Defaults、
  Runtime Configuration，以及 Session Catalog 的完整 renderer/interaction
  与 History Index 浏览组件。详见
  [Session Configuration and Browsing Components](../done/2026-10-03-extract-session-configuration-and-browsing-components.md)
  及五份子任务；15 个组件项目与宿主整合完成，975 项定向 JVM 测试通过，
  RPC server/client 及 Integration 测试源码编译通过。
  用户已验收通过；内层实现提交为 `721ac0fe`，外层设计记录与子模块指针随后
  分批提交并推送。未验证平台、混合依赖债务和首轮测试竞态记录见该主任务。

- 用户将下一批扩大为六个完整 Settings 组件：Context Source Settings、
  Session Title Settings、Application Preferences、Authentication Settings、
  Account Usage、Usage Reset。按配置编辑、账号/用量及独立复杂 reset 工作单元组织，
  共享宿主和构建集中整合；用户已审阅 planning 并授权实施。
  详见 [Settings Configuration and Account Components](../done/2026-10-02-extract-settings-configuration-and-account-components.md)
  及其六份子任务；三条实现 Session 已经用户确认。18 个组件项目与共享宿主
  整合完成，851 项定向 JVM 测试通过，Integration 测试源码编译通过；
  Native/JS/CLI、既有 spec 闭包债务及首轮取消用例的偶发风险见主计划。
  用户已验收通过，主任务与六份子任务已归档 done；内层实现提交为
  `05a472e9`。用户已授权提交和推送，外层设计记录与子模块指针分别提交。

- 前一批四个完整 app 交互组件，按待处理工具与设置资源两条线组织：
  Request User Input、Suggest Subagent Task、MCP Settings、Hook Settings。
  细化方案及四个子任务见
  [Pending Tool and Resource Components](../done/2026-10-02-extract-pending-tool-and-resource-components.md)。
  用户已确认两条实现 Session；12 个组件项目、完整宿主接线及四组件迁移完成，
  392 项定向 JVM 测试通过，RPC server/client 和 integration 测试源码编译通过。
  用户已验收，内层提交为 `7310a445`，六份任务已进入 done；
  Native/JS/CLI 与既有 spec 闭包债务见主任务。

- Use [Spec/Impl Module Boundaries](../../checklist/spec-impl-module-boundaries.md) as the canonical rule for directory placement, meaningful pairing, KDoc authority, and dependency direction. This task records migrations and temporary debt, not a second definition of those rules.
- For this migration, inspect `api` and `implementation` main-source edges separately from test dependencies. Track dependencies on still-mixed legacy projects until they are classified; do not describe a local pilot as repository-wide closure.
- Keep wire DTOs, persisted models, runtime state, RPC contracts, and frontend presentation models under their existing semantic owners. Do not duplicate mappers or state machines merely to fill a `spec` directory.

## Initial domain map for review

| Domain | Candidate spec ownership | Candidate implementation/adapters | Review focus |
| --- | --- | --- | --- |
| Agent storage | `clean-models` for durable events, `contract` for storage operations | `filesystem`, `in-memory`, `filesystem-layout`, `contract-ext` as classified by actual API | Preserve one sealed event hierarchy and stored JSON shapes. |
| Agent state, runtime, session | Existing `contract` projects and any public state values now held elsewhere | `impl`, decorators, context-window projection, repositories, test fixtures | Remove the runtime contract's dependency on concrete Unified Exec without changing lifecycle semantics. |
| Agent context, hooks | Existing focused `contract` projects; prompt grammar only if it is a cross-consumer promise | Filesystem discovery, rendering, hook execution | Do not promote internal renderers into public protocol accidentally. |
| OpenAI | `models`, `client-contract`, model-catalog and account-usage contracts | HTTP client, auth integration, catalog implementation, Codex CLI storage | Keep wire types, transport API, and app-owned auth responsibilities distinct. |
| Tools and MCP | Generic tool contract, per-tool reusable DTO contracts, MCP contract | Tool handlers, command clients, stdio/HTTP transports, MCP manager | Keep tool DTOs reusable without moving sealed storage event subclasses into tool projects. |
| RPC | `models` and frozen `contract` | client, server, in-memory transport, version patch | Do not change cross-wire types or semantics without separate user approval. |
| Application | Actual component contracts and configuration values, two persistence store contracts, migration contract | ViewModels, views, filesystem adapters, CLI, backend auth/naming and composition | Separate backend truth from frontend projection; app/shared is not a domain or a second fact source. |
| Utilities | Actual operation contracts and pure reusable models/functions | Concrete filesystem, process, codec, terminal, and test helpers | `utils` is a collection, not one domain; private support may stay impl-only rather than receiving a placeholder spec. |
| Integration tests | No production spec of their own | Cross-domain verification | Remain consumers of public specs and assembled implementations. |

Names in this table identify existing project candidates, not an exhaustive pair-by-pair inventory or final paths. The `spec/` and `impl/` placement and per-implementation pairing are decided; exact per-domain grouping, leaf names, and treatment of non-implementation entrypoints remain to be mapped. Test-support implementations are not exempt from the pairing rule.

## Known boundary and compatibility gates

- The four Runtime decorators have separate physical `spec/decorator/*` and `impl/decorator/*` projects. The spec for **each** decorator owns its runtime interface and behavioral KDoc; the corresponding implementation class implements that interface. Compact's retry-limit exception and steer's atomic input provider are additional spec declarations. The numeric retry policy, tool routing, and turn-Hook event projection are implementation details, not substitutes for runtime interfaces. Review the contract KDoc and tests before calling this batch complete.
- `agent-runtime/spec/contract` exposes the spec-facing `UnifiedExecClient`
  view; `agent-runtime/impl/composition` owns the concrete
  `UnifiedExecToolClient`. Existing [AgentState/Runtime guidance](../../checklist/agent-state-and-runtime.md)
  explicitly requires this shared session-scoped instance. Preserve access,
  identity, observation, and closure rules; do not remove or silently replace
  the property solely to satisfy a graph rule.
- The observed production consumer is `rpc/server/BackendAgentRuntimeRpc`: it reads the shell-session snapshot and Flow and requests close on a registered process. The runtime composition and tool handlers share the same client instance. The current `UnifiedExecProcessSession` observation/close interface lives in the implementation project, so extracting only the client type would leave another spec-to-impl leak. Keep the RPC wire contract unchanged.
- Unified Exec argument DTOs already expose `utils/shell-client`'s `Shell`, which currently shares a Gradle project with process execution, host resolution, and other implementations. Splitting Unified Exec alone cannot establish a fully spec-only transitive dependency graph. Record this as a legacy mixed-project dependency; later separate Shell's contract and its host effects without changing its persisted/model-facing encoding.
- `utils/patch` mixed public parsed patch models with parser/filesystem behavior; it is now split into `utils/patch/spec` for parsed models, parser, matcher, and the `PatchApplier` contract, plus `utils/patch/impl` for filesystem application. The remaining utility tree is tracked as the whole-utils migration; do not move sealed clean events into per-tool modules or duplicate parsed patch DTOs.
- `agent-storage` now follows the nested layout
  `agent-storage/spec/{clean-models,contract,contract-ext}` and
  `agent-storage/impl/{filesystem,filesystem-layout,in-memory}`. Keep the
  durable event hierarchy and storage operation contracts in spec. Treat
  `filesystem-layout` as implementation support until its dependency on the
  concrete `CoroutineFileSystem` is intentionally abstracted; do not
  manufacture a second layout protocol merely to make the directory name
  symmetrical.
- The storage clean-model sealed hierarchy and serialized fields must remain compatible; verify old JSON decode and re-encode when moving Kotlin types. See [Clean Model Rust Alignment](../../checklist/clean-model-rust-alignment.md).
- Existing [OpenAI Module Boundaries](../../checklist/openai-module-boundaries.md) and [Frontend Application Boundary](../../checklist/frontend-application-boundary.md) prescribe current physical locations and dependencies. Any agreed move must update those checklists in the same implementation batch, not leave contradictory rules.
- The [RPC contract freeze](../../checklist/rpc-architecture.md#rpc-契约冻结) remains in force. A module relocation may preserve public symbols and serialization, but any needed contract/behavior change is a separate approval gate, not a refactor detail.
- Gradle source-set target sets, Kotlin packages, `api` exposure, generated serializers/RPC artifacts, test fixtures, and executable assembly must be checked before each move. A directory-only migration is not assumed safe.

## Implementation sequence

1. **Inventory and baseline:** enumerate every Gradle project, its main-source dependencies, public API and attached KDoc, owned models/semantics, target set, and test consumers. Label each as candidate spec, implementation, entrypoint, test support, or mixed; map every implementation project to its corresponding spec, including private and test-support implementations. Produce a domain dependency graph; distinguish main from test edges.
2. **Resolve design blockers:** map each project into the chosen `spec/` or `impl/` layout and classify mixed modules and entrypoints. For projects without an identifiable contract, review project responsibility rather than generate an empty spec. For each spec-to-impl edge, record the public type causing it and the minimal semantics-preserving move. Stop at RPC approval gates if exposed types would change.
3. **First batch — Agent Runtime and Unified Exec:** establish their spec boundaries, migrate their production consumers and tests, and check the graph plus behavior. Use the result to tune the migration checklist before broad rollout. This batch is in progress on `refactor/spec`.
4. **Migrate in dependency order:** low-level models/utilities and OpenAI/tool contracts; storage clean models and storage contract; Agent state/runtime/session; MCP; RPC/application composition. Adjust order if the measured graph shows a different dependency DAG. Make each batch independently buildable, without leaving a half-moved production entrypoint.
5. **Enforce and document:** add an automated main-source dependency check for `spec -> impl` violations and spec cycles, with explicit classification of allowed third-party dependencies. Update only affected domain checklists after decisions are approved; remove superseded names and rules.

### Next domain batches and gates

| Batch | Work after the current clear pairs | Gate |
| --- | --- | --- |
| Foundational utilities | Separate `utils/shell-client`'s `Shell`/session API from process and host effects; `utils/patch`'s parsed model/parser/matcher and filesystem application are now split. | Preserve `Shell` JSON encoding and `Shell.default` behavior without making a spec depend on an impl; preserve parsed patch and clean-event serialization. Do not force a split by introducing a global mutable resolver. |
| OpenAI and tool capabilities | The remaining OpenAI split and the complete tool tree are migrated; continue closure checks for each topic's contract/handler pair and KDoc ownership. | Keep OpenAI wire shapes and client streaming semantics; leave the clean-model sealed hierarchy together. Do not add DTO copies merely to fill spec modules. |
| Agent and MCP | The complete MCP tree is migrated into core and transport spec/impl topics; continue storage/state/session integration checks. | Validate MCP transport lifecycle, each remaining spec/impl pair, ownership/lifecycle guarantees, and old storage fixture compatibility before calling the domain complete. |
| RPC and application | Move frozen RPC contract/models and transport/service projects, then application contracts, ViewModels, renderers, settings/auth adapters, host, and test support. | Do not change RPC wire/behavior without separate approval; preserve frontend/backend ownership and CLI entrypoint behavior. Decide host/entrypoint classification rather than giving them a dummy spec. |
| Repository gate | Classify remaining `utils` capabilities, remove legacy names and stale guidance, and enforce the project graph. | Only then claim repository-wide `spec -> spec` closure and one meaningful spec per implementation project. |

For each batch, update every consumer's Gradle project path in the same change, verify the moved KDoc and available platform targets, run targeted tests, and record unvalidated targets when shared compute is occupied. The two current OpenAI pairs are a path/KDoc-preserving migration pattern, not evidence that harder mixed modules have been solved.

## Validation planned for each implementation batch

- Static: project inclusion and dependency graph; no direct edge from a migrated spec to a known impl or spec cycle; explicitly list dependencies on still-mixed legacy projects rather than claiming global closure. Check for accidental new public API dependencies, and compare public packages/signatures and KMP target coverage where moved.
- Documentation: compare KDoc on each moved declaration before and after; ensure cross-implementation behavior, cancellation, resource ownership, state meaning, and compatibility guarantees are stated on the spec API. Check KDoc links after relocation; implementation KDoc may remain, but must not conflict with the spec.
- Behavioral: relevant spec/impl unit tests; old fixture decode/re-encode for persisted data; RPC serialization and real round-trip tests when touched; frontend/backend ownership and lifetime tests when touched.
- Integration: compile affected downstream projects, JVM tests where available, supported Native compilation, and CLI smoke test for production assembly. Reuse the running Gradle Daemon JVM as required by the Gradle workflow.
- Scope: no deliberate product behavior, storage format, RPC wire shape, settings migration, credential handling, or UI interaction changes in this structural refactor. Any required exception gets an explicit review and its own checks.

## First batch: Runtime and Unified Exec

### Physical project mapping

| Current project | Pilot destination | Treatment |
| --- | --- | --- |
| `:tool-unified-exec-contract` (`tool/unified-exec/contract`, historical) | `:tool-unified-exec-spec` (`tool/spec/unified-exec`) | The contract has been relocated; DTOs, exception, and attached KDoc remain the source of truth alongside the client and observable session interfaces. |
| `:tool-unified-exec-impl` (`tool/impl/unified-exec`) | same project | Concrete process manager, factory, schemas, platform code, and tool handlers remain here and depend on the spec. |
| `:agent-runtime-contract` (`agent-runtime/contract`, historical) | `:agent-runtime-spec-contract` (`agent-runtime/spec/contract`) | The contract has been relocated; `AgentRuntime`, `ResumableAgentLayer`, and their attached KDoc use the spec-facing Unified Exec API. |
| `:agent-runtime-impl` (`agent-runtime/impl`) | `:agent-runtime-impl-composition` (`agent-runtime/impl/composition`) | Composition re-exports `:agent-runtime-spec-contract` and retains the concrete Unified Exec owner. |

- Update all production and test Gradle project references to the two renamed spec projects in the same batch. `settings.gradle.kts` discovers the new paths automatically, but all `project(":...")` strings must be updated. Do not leave deprecated alias projects merely to keep old paths compiling.
- The four decorators are now migrated as a follow-up Runtime batch, each with a runtime interface in spec and a matching implementation. The initial Runtime/Unified Exec build results predate this follow-up and do not validate it.
- Keep existing Kotlin package/FQCN for moved types wherever possible. This is a Gradle ownership change, not a wire-format or public naming cleanup.

### Proposed API extraction

- Introduce `UnifiedExecClient : AutoCloseable` in `tool/spec/unified-exec`, with the exact current operations: `activeSessions: StateFlow<Map<Int, UnifiedExecProcessSession>>`, `execCommand(ExecCommandArguments): UnifiedExecOutput`, and `writeStdin(WriteStdinArguments): UnifiedExecOutput`. Retain `UnifiedExecProcessSession`'s `sessionId`, original `arguments`, `completed: StateFlow<Boolean>`, and `close()` in that spec project under its current Kotlin package.
- Move `UnifiedExecProcessSession` with its KDoc intact, adjusting links only as needed. Adapt the existing concrete client's KDoc about the shared process owner, `activeSessions` registration, shell selection, and closing behavior onto `UnifiedExecClient` and its properties; leave concrete construction and synchronization explanations on `UnifiedExecToolClient`. Retain the `AgentRuntime` KDoc that explains turn ownership and the shared shell client, updating its type references rather than deleting it.
- Keep the concrete `UnifiedExecToolClient` class and its `CoroutineScope.UnifiedExecToolClient(...)` factory in `tool/impl/unified-exec`; have the class implement `UnifiedExecClient`. This avoids renaming the constructor/factory API or accidentally making the spec construct effects. `UnifiedExecTools` should consume the interface while the Runtime composition still creates exactly one concrete instance.
- Change `AgentRuntime.unifiedExecToolClient`'s declared type from the concrete class to `UnifiedExecClient`; keep its property name and the exact shared instance. Make `AgentRuntime` extend `ResumableAgentLayer` instead of independently repeating `KodexAgentState` plus `resume()`, retaining the runtime-specific method KDoc. The implementation may retain the narrower concrete client type internally. `rpc/server` needs only the spec-facing observable session members and should not gain a process-management dependency.
- Keep existing argument/output DTOs and `UnifiedExecToolException` in the moved spec. Preserve their existing `Shell` type and serialized shape, even though its current dependency is a mixed legacy project. The currently public yield/output limit constants are used by existing tests and describe observable tool behavior; move them to the spec with unchanged names/values, then verify no duplicate declaration remains in the implementation. Do not change normalization behavior in the pilot.
- Preserve current ownership: Runtime composition creates the client, its two tools share it, and tool closure/Runtime lifecycle still release it. `UnifiedExecProcessSession.close()` requests process-tree termination without removing the registry entry; `writeStdin` may still read the final output. The `activeSessions` registry remains the single source of live shell-session membership.

### Pilot-specific checks and stop conditions

- Check `:agent-runtime-spec-contract` and `:tool-unified-exec-spec` have no **direct** main-source dependency on a known `-impl` project and do not form a spec cycle; in particular, the former must not depend on `:tool-unified-exec-impl`. Audit their transitive `api` exposure and record each mixed legacy project, notably `:utils-shell-client`. Do not present this pilot as proof of global spec-only closure.
- Compile all downstream consumers of the renamed projects, including Agent Session, Agent State tool projection, storage clean models, RPC server, integration tests, and the CLI. Run Unified Exec tests, Agent Runtime/Session tests, and RPC shell-session snapshot/Flow/close tests; include the registered-but-completed state and close-then-read-final-output case.
- Confirm the `rpc/contract` service signatures and serialized `ShellSessionState` remain byte/field compatible, and the backend keeps the [Shell process semantics](../../checklist/rpc-agent-runtime.md#shell-进程). Do not infer wire compatibility from compilation alone.
- The architecture rule becomes a whole-repository gate only after the remaining mixed legacy dependencies are inventoried and migrated. Until then, pilot acceptance requires an explicit unresolved-edge list, not a waiver silently treated as final compliance.
- If preserving the shared instance, lifecycle, or RPC behavior requires modifying the frozen RPC contract, stop and request separate approval. If a renamed public Kotlin type or serialization change appears necessary, document the exact impact before implementation.
- Update the concrete-type wording and inheritance in [AgentState/Runtime guidance](../../checklist/agent-state-and-runtime.md), the tool DTO location in [Clean Model Rust Alignment](../../checklist/clean-model-rust-alignment.md), and affected dependency guidance when the respective batch's shape is stable. Do not leave old physical locations as live guidance.

The user's branch rename and formal authorization supersede the earlier trial-only gate. Complete migration remains subject to domain-specific compatibility checks; no partial batch may be described as a repository-wide spec-only graph.
