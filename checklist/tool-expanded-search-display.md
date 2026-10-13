# Expanded Tool Search Display

This checklist records the confirmed expanded-content design for local
`tool_search` and hosted `server_tool_search`.

## Confirmed design

- Treat both tools as one dedicated Tool Search display family.
- Keep the outer title a short action summary, such as `Search available tools`.
- Put input parameters inline in the first expanded-content line:
  `tool_search; query=calendar; limit=4`
- Start that line with the raw tool name without a `Tool:` prefix.
- Preserve complete inputs in the body, not the outer title; do not add generic
  `Arguments`/`Result` sections.
- Local input is typed `query`/`limit`; hosted input is recorded arbitrary JSON.
  Preserve all provided hosted fields rather than assume only `paths` exists.
- On the first expansion, show a compact list of returned tool titles.
- Give every returned tool its own title rather than flattening the result list
  into one text value.
- Keep the first-level result list compact even when each tool definition is
  large.
- Allow each returned tool title to expand independently a second time.
- On the second expansion, show that tool's recorded JSON definition without
  dropping description, namespace, parameters, strict/defer-loading metadata,
  nested tools, or other available definition fields.
- Results are stored as typed `LoadableToolSpec` values. Display their complete
  serializable definition, not a fabricated claim to reproduce provider wire
  bytes or unknown fields that were never retained.
- Keep namespace definitions intact; exposing member names must not discard
  their parent namespace description or the recorded nested definition.
- Preserve the information through progressive disclosure: the first level
  exposes every result tool, and the second level exposes each large raw
  definition.
- Render invalid arguments as a direct error result rather than inventing an
  empty tool list.
- Keep local and hosted result data specialized behind the shared Tool Search
  renderer; do not route either through the generic Tool detail template.

## Examples

```text
Search available tools
  tool_search; query=calendar; limit=4
  calendar.create_event
  calendar.delete_event
  calendar.list_events
```

```text
Search available tools
  tool_search; query=calendar; limit=4
  calendar.create_event
  {
    "type": "namespace",
    "name": "calendar",
    "description": "...",
    "tools": [
      {
        "type": "function",
        "name": "create_event",
        "description": "...",
        "defer_loading": true,
        "parameters": {
          "type": "object",
          "properties": {}
        }
      }
    ]
  }
```

```text
Search available tools
  server_tool_search; paths=["calendar","cloud"]
  calendar.create_event
  cloud.list_files
```
