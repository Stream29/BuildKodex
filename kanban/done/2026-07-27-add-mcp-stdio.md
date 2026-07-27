# Task Tree

- [done] Add MCP stdio transport support
  - [done] Model Streamable HTTP and stdio configurations as an explicit union
  - [done] Preserve native Codex stdio settings through global settings
  - [done] Let `mcp:impl` open stdio transports through `mcp:stdio`
  - [done] Extract direct pipe-process ownership from `ShellClient` into `utils:process-client`
  - [done] Reuse `ProcessClient` from ordinary shell sessions and MCP stdio
  - [done] Wire stdio support into the CLI application
  - [done] Verify configuration decoding and a real stdio MCP lifecycle

# Details

- The MCP SDK owns JSON-RPC framing through `StdioClientTransport`, but it does
  not start or terminate the server process.
- `mcp:impl` directly uses the CLI-target `mcp:stdio` module. Node.js stdio
  remains a separate planned capability.
- `ProcessClient` owns direct `executable + arguments` execution and raw pipes.
  `ShellClient` remains the shell/text/PTY layer.
- Stdio stdout is protocol data and bypasses the bounded text buffers used by
  `ShellClient`.
