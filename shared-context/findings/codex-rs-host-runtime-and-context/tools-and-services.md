# Tools 与宿主服务

## Tool Runtime

Rust 的工具不只是 handler function。

核心运行时 contract 是 `CoreToolRuntime`，位置是 `shared-context/codex/codex-rs/core/src/tools/registry.rs:43`。

它扩展了更底层的 tool executor，并增加 core-owned metadata：

- 匹配模型 payload kind。
- cancellation behavior。
- telemetry tags。
- pre-tool hook payload。
- post-tool hook payload。
- hook input rewriting。
- streamed argument diff consumers。

这解释了为什么把 `spec + handler` 绑定成一个万能对象会过于简单。某些工具需要 streamed argument diffs、approval handling 或稳定 hook payload。

输出路径也保持泛型。`AnyToolResult` 会把 tool result 转回 response item，见 `shared-context/codex/codex-rs/core/src/tools/registry.rs:159`。

Kotlin 侧的 tool module 可以保留窄的 `payload -> result` 接口，但 agent/runtime 层需要单独的 orchestration wrapper，负责 approvals、hooks、telemetry、streaming argument diffs 和 response-item conversion。

## Tool Orchestrator

tool orchestrator 是中心策略门禁。

模块注释说明它协调：

- approvals
- sandbox selection
- retry semantics

注释位置是 `shared-context/codex/codex-rs/core/src/tools/orchestrator.rs:1`。

`ToolOrchestrator::run` 处理：

- approval requirement
- strict auto review
- guardian review
- permission hooks
- filesystem sandbox policy
- network sandbox policy
- initial sandbox selection
- escalated sandbox retry
- network approval finalization
- telemetry decisions

主函数在 `shared-context/codex/codex-rs/core/src/tools/orchestrator.rs:132`。

这是宿主运行时逻辑，不应该埋进单个工具 handler。

这也支持我们之前对 Codex Lite 的判断：tool handler 可以保持简单，少量特殊工具在 agent/runtime 层手写分发，等抽象压力真实出现后再抽象。

## Apply Patch

`apply_patch` 是特殊工具的好例子。

Rust 内部有 `InternalApplyPatchInvocation`，分成两条路径：

- 直接返回 output。
- 委托 runtime filesystem 执行。

enum 位置是 `shared-context/codex/codex-rs/core/src/apply_patch.rs:13`。

决策路径会检查 patch safety：

- auto-approve
- ask user
- reject

之后要么返回模型可见错误，要么构造 `ApplyPatchRuntimeInvocation` 委托 runtime。函数位置是 `shared-context/codex/codex-rs/core/src/apply_patch.rs:33`。

这确认了 Kotlin 侧之前的方向：

- patch 解析和 apply 逻辑属于低层 stateless utility module。
- 真实 tool integration 属于 host/runtime code。
- approval/sandbox handling 不应该硬编码进 parser。

## Environment 与 Filesystem

Rust 会把项目和运行时文件操作尽量路由到 environment filesystems。

例子包括：

- 项目 `AGENTS.md` discovery 使用 `ExecutorFileSystem`。
- skill roots 可以绑定 repo filesystem 或 local filesystem。
- Session startup 会先 resolve environment selections，再 warmup plugins/skills。
- tool execution 会经过 sandbox 和 environment-aware runtime path。

这说明 host filesystem 不等于 project filesystem。

Kotlin 侧应保持分层 filesystem 策略：

- 低层 coroutine filesystem abstractions。
- environment-specific filesystem providers。
- host runtime 选择正确 provider。
- tools 通过 context 获取 runtime filesystem access，而不是直接用全局 file APIs。

## Auth 与 Model Provider

auth 和 model/provider state 也是宿主运行时职责。

`ThreadManager` 持有 `AuthManager` 和 shared model manager，位置是 `shared-context/codex/codex-rs/core/src/thread_manager.rs:205`。

`build_models_manager` 会基于 config 和 auth 构造 model provider，见 `shared-context/codex/codex-rs/core/src/thread_manager.rs:226`。

Session 初始化会在 turn loop 之前获取 auth，并计算 MCP auth statuses，见 `shared-context/codex/codex-rs/core/src/session/session.rs:603`。

这进一步支持我们已经讨论过的 OpenAI 模块拆分：

- model/client/auth storage 放在 OpenAI/host modules。
- agent runtime 消费 typed client 和 runtime snapshot。
- tool modules 不需要知道 auth refresh 怎么做。

## Telemetry 与 Analytics

telemetry 贯穿 runtime 边界：

- skill injection metrics
- skill invocation analytics
- plugin/app mention analytics
- tool decisions
- compaction attempts
- hook started/completed events

这不是 Codex Lite 最小 loop 的必要条件，但会影响源码架构。Rust 中很多结构会携带 session telemetry 或 analytics clients，因为这些操作都在 runtime boundary 上被审计。

Kotlin 侧可以把 telemetry 作为可选 runtime service。它不应该污染纯数据模型或低层 utility modules。

## Multi-Agent 与 Session Source

Rust 的 session 行为会随 `SessionSource` 改变。

例子：

- 非 root agents 继承 user instructions，而不是重新读取 host instructions。
- initial context 可以加入 multi-agent usage hints。
- hook routing 会区分 root stop hooks 和 subagent stop hooks。
- thread-spawn subagents 可以运行 session-start hooks，内部 synthetic subagents 不运行。

继承逻辑在 `shared-context/codex/codex-rs/core/src/thread_manager.rs:1136`。

initial context 的 multi-agent hint 路径在 `shared-context/codex/codex-rs/core/src/session/mod.rs:3063`。

stop-hook routing 在 `shared-context/codex/codex-rs/core/src/hook_runtime.rs:297`。

Kotlin 第一版可以不做 multi-agent。但 state model 不应该假设所有 session 都是 root user session。

## Extensions

Rust 有 extension registry，可以贡献上下文。

initial context 会调用 context contributors，并按 prompt slot 分类：

- developer policy
- developer capabilities
- contextual user
- separate developer

context contributor loop 在 `shared-context/codex/codex-rs/core/src/session/mod.rs:3011`。

per-turn extension injection 也在 `build_skills_and_plugins` 里发生，通过 `build_extension_turn_input_items`，位置是 `shared-context/codex/codex-rs/core/src/session/turn.rs:615`。

这再次说明：纯 loop 应该接收已经准备好的 context items，而不是自己构造所有 prompt sections。
