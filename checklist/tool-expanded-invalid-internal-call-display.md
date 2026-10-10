# Invalid Internal Tool Call Display

This design covers the invalid-call branch for built-in tools. External tools
retain the [existing two-level disclosure](tool-expanded-external-display.md).

## Confirmed design

- Show the actual qualified tool name as the first expanded line, without a
  `Tool:` prefix or a synthetic `invalid_tool_call` name.
- Display the complete original arguments or free-form input.
- Preserve invalid argument strings without parsing, repairing, or rewriting
  them. For already structured tool-search arguments, display their recorded
  JSON structure without interpreting them as valid search parameters.
- Display the recorded error message directly.
- Do not apply a normal tool's specialized field layout when its input could
  not be decoded.
- Remove the `Invocation` and `Error` disclosure groups.
- Hide internal call and item IDs.

## Examples

```text
exec_command
arguments: {"cmd":
error: <recorded error message>
```

```text
apply_patch
input:
<complete recorded patch input>
error: <recorded error message>
```
