# External Tool Display Boundary

## Confirmed design

- This round specializes built-in Kodex tools only.
- External tools, including dynamically provided MCP tools, keep the existing
  two-level disclosure format: expand the tool, then independently expand its
  arguments and result.
- Preserve the existing external-tool presentation rather than replacing it
  with inline arguments or a flat output layout.
- Do not add per-name or server-specific external-tool renderers in this round.
- The flat layouts recorded for built-in tools do not override this external
  tool fallback.
