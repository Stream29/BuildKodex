# Tool Handler Decisions

Use this checklist when changing tool specs, tool modules, or agent-loop tool dispatch.

- Keep tool specs and tool handlers separate by default.
- Do not introduce a generic `ToolExecutor` or `ToolRegistry` abstraction at this stage.
- Keep `ToolSpec` and related protocol DTOs as pure data contracts.
- Let concrete tool modules expose specs, DTOs, and reusable execution clients without requiring a shared handler interface.
- Put explicit tool dispatch in the agent loop while the tool set remains small.
- Prefer hand-written agent-loop dispatch because several tools have special behavior, including hosted tools, deferred `tool_search`, freeform custom tools, and multi-tool orchestration.
- Reconsider a shared handler abstraction only after repeated dispatch logic becomes larger than the special-case logic it would replace.
