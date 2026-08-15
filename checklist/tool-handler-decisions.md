# Tool Handler Decisions

Use this checklist when changing tool specs, tool modules, or agent-loop tool dispatch.

- Keep each tool's spec, typed models, and implementation together under `tool:<tool_name>`.
- Use one non-generic `Tool` interface with `spec` and `suspend handle(ResponseItem.ToolCall): ResponseItem.ToolCallOutput`.
- Do not wrap OpenAI tool-call input or output DTOs with contract-local payload/result types.
- Return the output variant matching the input call variant; do not add a contract-level failure branch.
- Express model-facing tool failures with `FunctionCallOutputPayload.success = false`.
- Make `Tool` extend `AutoCloseable`, but do not put a no-op close implementation in the contract.
- Put no-op close behavior in builders or concrete implementations that truly own no resources.
- Use `tool:tool-builder` to adapt typed JSON DTO handlers into the raw `Tool` interface.
- Keep OpenAI protocol DTOs, including tool specs, in `openai:models`.
- Treat normal tools as JSON function tools by default.
- Treat `apply_patch` as the freeform custom tool path.
- Treat `tool_search` as an agent-loop primitive, not a normal `Tool` handler.
- Keep `tool_search` under `tool:tool-search` because that module owns indexing, search, and result projection behavior.
- Model client `tool_search_call` / output as `ResponseItem.ToolCall` / `ToolCallOutput`, then dispatch that special call from the same `KodexToolRuntime` loop as local tools. Never hand it to a normal `Tool` implementation.
- Construct every fixed local tool handler in AgentSession composition. `Direct`/`Deferred` controls request-time visibility, never whether a fixed handler is executable.
- A `ClientToolSearchOutput` extends request history for the model; it never mutates the durable tool settings.
- Do not split tool modules into `impl` and `spec` directory hierarchies.
- Keep spec-only tools in their normal `tool:<tool_name>` module until a handler exists.
- Put state-bound handlers such as `update_plan` and Multi-agent tools in their corresponding tool module as `KodexAgentState` factories.
- Do not introduce a generic `ToolExecutor` or `ToolRegistry` abstraction at this stage.
- Keep `ToolSpec` and related protocol DTOs as pure data contracts.
- Put explicit tool dispatch in the agent loop while the tool set remains small.
- AgentState derives the complete model-visible `List<ToolSpec>` for every request from fixed tools, persisted mode, and its required dynamic Tool Search callback; never pass that complete list through runtime or persist it in `KodexAgentSettings`.
- Treat hosted `web_search` as a direct request `ToolSpec`. It has no local `Tool` or `KodexToolRuntime` handler.
- Keep `request_user_input` schema/spec separate from runtime handlers. Expose its spec only when the request snapshot uses `RequestUserInputMode.AskUser`; `NoQuestion` hides it from both ordinary and remote-compaction requests without cancelling an already-issued or pending call. The host UI recognizes a sole matching `ToolPending` call, renders the form, and completes it directly through AgentState; do not route it through generic local-tool dispatch or create a dedicated Runtime.
- Use KodexToolRuntime for normal local tools. It borrows a fixed `List<Tool>`, dynamic `StateFlow<List<Tool>>`, and `StateFlow<ToolSearchEngine>` supplied by composition.
- Keep tool construction, resource ownership, and catalog rebuilding outside KodexToolRuntime.
- Reject duplicate fixed or dynamic routes, and leave unmatched pending calls for another runtime.
- Do not add a per-tool runtime when the tool only needs normal call routing and completion.
- Prefer hand-written agent-loop dispatch because several tools have special behavior, including hosted tools, deferred `tool_search`, freeform custom tools, and multi-tool orchestration.
- Reconsider registry abstraction only after repeated dispatch logic becomes larger than the special-case logic it would replace.
