# Codex.rs 宿主运行时与上下文机制分析

本目录拆分自原单篇报告 `codex-rs-host-runtime-and-context.md`。

阅读顺序：

- [overview.md](overview.md)：范围、核心结论、总体模型、ThreadManager 和 Session 初始化。
- [instructions-and-skills.md](instructions-and-skills.md)：`AGENTS.md`、skills、plugins、MCP、apps。
- [context-and-hooks.md](context-and-hooks.md)：TurnContext、initial context、context diffs、hooks。
- [tools-and-services.md](tools-and-services.md)：工具运行时、apply patch、filesystem、auth、telemetry、multi-agent、extensions。
- [storage-and-compaction.md](storage-and-compaction.md)：compaction、thread storage、rollout items。
- [termination-and-interruption.md](termination-and-interruption.md)：Responses 结束语义、turn 完成条件、暂停、steer、interrupt、hook block。
- [agent-loop-and-architecture.md](agent-loop-and-architecture.md)：纯 agent loop、Codex Lite 架构影响。
- [source-anchors.md](source-anchors.md)：源码锚点。
