# Hosted Web Search Display

This design covers the provider-built-in web search event identified as
`hosted_web_search` in the UI, not the local `web.run` tool or external MCP tools.

## Confirmed design

- Use operation titles to expose the available action fields:
  - `Search <query>`; give each entry in `queries` its own title.
  - `Open <url>`.
  - `Find "<pattern>" in <url>`.
- Preserve all provided query values, URLs, and patterns in the titles.
- Let long titles wrap naturally rather than truncate meaningful values.
- Omit missing fields without inventing replacement values.
- Show `hosted_web_search` as the first expanded line without a `Tool:` prefix.
- Do not repeat fields already represented in the operation titles.
- Hide the internal event ID.
- Keep status in the existing color presentation rather than repeat it as text.
- Do not add generic argument/result groups or a second disclosure layer.
- The current event model has no separate output field; do not synthesize
  search results or an output block.
