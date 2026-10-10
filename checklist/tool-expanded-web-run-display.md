# Expanded `web.run` Display

This checklist records the confirmed expanded-content design for the local
`web.run` tool. Hosted web search is a separate tool and is not covered here.

## Confirmed design

- Show `web.run` as the raw tool name at the start of the expanded content.
- Treat one `web.run` call as a batch that may contain multiple operations.
- Generate one human-readable operation title for every actual operation item;
  do not choose only the first non-empty operation type.
- Put the operation's meaningful input fields into its title. For example:
  `Search Kotlin MCP SDK in kotlinlang.org within 30 days`.
- Use semantic compact wording when it preserves the actual values, including
  domains, recency, line numbers, link ids, tickers, dates, locations, and
  other operation parameters.
- Put batch-level parameters such as `response_length` on the `web.run` line.
- Omit null, empty, and default-valued fields.
- Let long operation titles wrap naturally; do not truncate or hide their
  meaningful input values.
- Do not render a generic `Arguments` section or repeat the original JSON
  payload after the operation titles.
- Render the tool's returned `output` after the operation titles.
- Render a failure message when the current result is a failure.
- Do not add another visual grouping layer around the operation titles or
  output.
- Treat the operation titles as the complete input presentation; the expanded
  result area is for returned output and errors.

## Field boundary

- Operation input fields belong in their operation title.
- Batch-level input fields belong on the `web.run` line.
- `SearchResponse.output` belongs in the returned output body.
- `SearchResponse.encryptedOutput` is backend side-channel state and is not
  normal user-facing content.
- Internal call/storage identifiers do not belong in the normal display.

## Examples

```text
web.run; response_length=short
Search Kotlin MCP SDK in kotlinlang.org within 30 days
Open result_1 at line 120
Click link 3 in result_1
output:
...
```

```text
web.run
Search Kotlin
Search MCP SDK in modelcontextprotocol.io
Open result_1
output:
...
```
