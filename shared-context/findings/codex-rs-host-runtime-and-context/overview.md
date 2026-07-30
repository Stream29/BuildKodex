# 总览

## 范围

- 本文分析 Rust Codex 在纯模型循环之外做了哪些宿主运行时工作。
- 重点覆盖 `AGENTS.md`、skills、上下文构造、hooks、工具运行时、存储、上下文压缩，以及其他不属于纯 agent loop 的输入来源。
- 分析依据来自 `shared-context/codex/codex-rs` 下的 Rust 源码。
- 本文不直接规定 Kotlin API 形状，只记录 Rust 侧真实设计，以及它对 Kodex 架构的约束。

## 核心结论

- Rust Codex 不是“一个纯 agent loop 加几个工具”。
- 模型采样循环被包在更大的宿主运行时里面。
- 宿主运行时负责配置、鉴权、环境、文件系统、项目文档、skills、plugins、MCP/app connectors、hooks、权限、工具编排、线程存储、上下文压缩、遥测和多 agent 状态。
- 稳定的数据流是：收集宿主态信息，把它们规整成模型可见的 `ResponseItem` 或 tool spec，运行模型采样，通过运行时边界执行工具，然后持久化模型可见历史和宿主侧元数据。
- `AGENTS.md` 和 `SKILL.md` 不是同一类东西：
  - `AGENTS.md` 会变成长生命周期的 contextual user instructions。
  - 可用 skill 的元数据会变成 developer capability instructions。
  - 显式选择的 skill 正文会变成当前 turn 的 contextual user message。
- Kotlin 侧不应该把这些都塞进一个巨大的 `agent loop`。loop 应该接收已经准备好的上下文、历史、工具定义和运行时服务。

## 总体心智模型

- `ThreadManager` 是最外层的宿主协调器。
- `Session` 是单个 thread 的运行时持有者。
- `TurnContext` 是单个 turn 的运行时快照。
- `ContextManager` 和 rollout/thread storage 保存模型可见历史和可回放数据。
- 工具不是直接从模型输出调用 handler，而是经过 orchestrator。
- hooks 和 extensions 可以注入额外模型上下文，也可以中止 turn。
- compaction 是替换历史的运行时任务，不是普通 assistant response。

可以把整体流程理解成：

- 启动或恢复 thread。
- 解析 config、auth、model provider、environment、filesystem、plugins、MCP/app 状态、skills 和 user instructions。
- 构造 `Session`。
- 每个 turn 构造一个 `TurnContext`。
- 采样前记录 initial context 或 context diff。
- 如果用户显式要求，注入当前 turn 的 skills/plugins/extensions。
- 运行 input hooks，并记录用户输入。
- 调用模型采样。
- 把流式模型输出投影成 response items 和 tool invocations。
- 通过 approval、sandbox、hook、telemetry 等运行时门禁执行工具。
- 追加 tool result，需要时继续采样。
- 运行 stop hooks。
- 持久化 rollout items，并重新计算 token usage。
- 需要时执行上下文压缩。

这里的关键点是：很多宿主操作发生在模型请求之前。它们不是普通 tool call。

## ThreadManager 边界

`ThreadManager` 持有长生命周期的宿主服务。它的状态包括 thread storage、auth、model management、environment management、skills、plugins、MCP、extensions、user instructions、analytics、attestation 和 state DB。源码位置是 `shared-context/codex/codex-rs/core/src/thread_manager.rs:205`。

这说明这些依赖不是 agent loop 的职责，而是宿主运行时的职责。

构造路径也很明确：

- `ThreadManager::new` 基于 config 和 session source 构造 `PluginsManager`、`McpManager` 和 `SkillsManager`，见 `shared-context/codex/codex-rs/core/src/thread_manager.rs:258`。
- spawn thread 时会先解析 environment selections、加载或继承 user instructions、计算 parent/fork 元数据，然后把宿主服务传进 `Codex::spawn`，见 `shared-context/codex/codex-rs/core/src/thread_manager.rs:1386`。
- root agent 会重新从 provider 加载 instructions。非 root agent 会从 parent 或 fork source 继承 instructions，而不是自己重新读宿主文件，见 `shared-context/codex/codex-rs/core/src/thread_manager.rs:1136`。

对 Kotlin 的启示是：需要一个 host/session runtime 边界。它负责构造 session、准备服务和快照。纯 loop 不应该自己发现 `AGENTS.md`、枚举 skill roots 或打开 plugin/MCP 连接。

## Session 初始化

`Session::new` 把 thread persistence、state DB、auth/MCP setup、plugin/skill warmup 接起来。

plugin 和 skill 的 warmup 是 session 初始化的一部分，不是 turn 执行的一部分：

- `warm_plugins_and_skills_for_session_init` 从 environments 解析 primary filesystem。
- 它读取 plugin config。
- 它拿到 plugin 提供的 skill roots。
- 它调用 `skills_manager.skills_for_config`。
- 入口在 `shared-context/codex/codex-rs/core/src/session/session.rs:436`。

session startup 还会并行执行几类互相独立的初始化任务：

- thread persistence
- state DB lookup
- auth and MCP config
- plugin and skill warmup

这些任务通过 `tokio::join!` 合并，位置在 `shared-context/codex/codex-rs/core/src/session/session.rs:640`。

这说明宿主运行时状态倾向于被预热和缓存，而不是每一步模型采样时重新发现。
