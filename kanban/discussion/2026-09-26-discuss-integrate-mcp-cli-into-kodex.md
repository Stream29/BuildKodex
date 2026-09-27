# Task Tree

- **`Establish the Daemon prerequisite and frontend/backend dependency`()**
- `Clarify Kodex's in-process MCP CLI goal and boundaries`()
- `Review existing MCP service, client lifecycle, and CLI integration points`()
- `Compare embedded CLI semantics with external mcp2cli and mcpc`()
- `Decide ownership, configuration, lifecycle, output, authentication, and compatibility`()
- `Record the decision or promote the scoped design to planning`()

# Details

- Context: The user does not need generated standalone CLIs. The desired capability is to invoke configured MCP tools from Bash, similar to `mcp2cli`, while keeping the implementation inside Kodex.
- User constraints:
  - The solution must be open source.
  - It should be lightweight.
  - It should have strong community visibility to reduce maintenance and compatibility risk.
  - Some MCP servers, especially the IntelliJ IDEA MCP server, need repeated start, reconnect, and close behavior.
- Confirmed direction for discussion: evaluate extending Kodex's existing MCP implementation instead of adding an external MCP CLI dependency.
- Confirmed architectural prerequisite:
  - The MCP CLI capability depends on the future frontend/backend separation and should not be implemented as a per-invocation client that owns MCP connections directly.
  - A continuously running Kodex Daemon must own MCP server processes, transports, sessions, health state, reconnect, authentication, and lifecycle cleanup.
  - The future Bash CLI and interactive frontend should be thin clients of the Daemon over a stable local IPC or API boundary.
  - The Daemon must remain alive independently of the frontend so MCP sessions can survive UI restarts and support correct start, reconnect, and close semantics.
  - This discussion task should not be promoted to implementation planning until the Daemon process model and frontend/backend boundary are designed and accepted.
- Existing context:
  - `kanban/done/2026-07-21-build-mcp-foundation.md` records the application-level MCP service, dynamic tool projection, and shared connection ownership.
  - `kanban/done/2026-07-27-add-mcp-stdio.md` records stdio transport and native process ownership.
  - `kanban/done/2026-07-27-implement-mcp-service.md` records MCP client topology, refresh, and lifecycle behavior.
  - `kanban/done/2026-08-10-implement-observable-mcp-clients.md` records stable MCP client owners, reconnect, connection states, and failure-visible catalogs.
- Open questions:
  - Is the target a Kodex subcommand, a separate Kodex executable mode, or a reusable local command endpoint?
  - What Daemon API should the CLI use to discover servers/tools, call tools, inspect health, and control lifecycle?
  - How should the existing application-level `McpService` move behind or compose with the future Daemon?
  - How should the Daemon be started, located, versioned, single-instanced, and shut down?
  - Which server configuration scope should the CLI use: global Kodex settings, project settings, or an explicit config path?
  - How should `start`, `reconnect`, `close`, and automatic recovery map to existing MCP client lifecycle states?
  - Should the CLI support persistent named sessions, one-shot calls, or both?
  - What stable JSON envelope, exit codes, stdout/stderr rules, and stdin argument conventions are required for Bash pipelines?
  - How should OAuth credentials, environment variables, approvals, dangerous tools, and audit history be shared with the interactive Kodex client?
  - How should the CLI handle IDE-owned servers whose actual process lifecycle is controlled by IntelliJ IDEA rather than Kodex?
- Candidate references:
  - [`apify/mcpc`](https://github.com/apify/mcpc) for persistent sessions, reconnect, close, and JSON-oriented shell usage.
  - [`knowsuchagency/mcp2cli`](https://github.com/knowsuchagency/mcp2cli) for dynamic command and argument mapping.
  - [IntelliJ IDEA MCP Server documentation](https://www.jetbrains.com/zh-cn/help/idea/mcp-server.html) for HTTP Stream, stdio, and IDE-owned lifecycle constraints.
- Scope boundary: This task records a design discussion only. It does not authorize implementation, dependency adoption, configuration migration, or changes to existing MCP behavior.
