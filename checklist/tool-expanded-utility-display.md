# Expanded Utility Tool Display

This checklist records the confirmed expanded-content design for the
no-argument read-only utility tools `clock.curr_time` and
`get_context_remaining`.

## Confirmed design

- Show the raw tool name as the first expanded line:
  `clock.curr_time` or `get_context_remaining`.
- Show the tool's returned human-readable text immediately below the raw name.
- Do not add an `Arguments` section for these no-argument tools.
- Do not render an empty JSON argument object.
- Do not add a generic `Result` or `output:` heading.
- Do not transform the returned sentence into a new metric card or synthetic
  field structure.
- Preserve each tool's own result wording:
  - `clock.curr_time` returns the current UTC time text.
  - `get_context_remaining` returns the remaining-context token text.

## Examples

```text
clock.curr_time
It is 2026-10-04 12:34:56 UTC.
```

```text
get_context_remaining
You have 42000 tokens left in this context window.
```
