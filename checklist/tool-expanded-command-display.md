# Expanded Command Tool Display

This checklist records the confirmed expanded-content design for the
`exec_command` and `write_stdin` tools. It does not define their collapsed
history titles or outer actions.

## Confirmed design

- Show the raw tool name as the first expanded line without a `Tool:` prefix.
- Put compact execution context and short non-default parameters on that same
  line, separated by `; `:
  `exec_command; cwd=/project; shell=/bin/zsh; tty=true; yield_time_ms=100`
- Use `cwd` rather than `Working directory` in the compact first line.
- Omit null, empty, and default-valued parameters.
- This includes `max_output_tokens` when it equals the recorded tool default;
  show a non-default value on the compact first line.
- Treat omitted `shell` as default. Do not compare an explicit shell against
  today's mutable host configuration to decide whether to hide it.
- Do not repeat `command` in the expanded body when the collapsed title already
  displays its complete value. A shortened or whitespace-normalized title does
  not meet this condition.
- Keep the expanded content flat. Do not add `Execution`, `Result`, `Output`,
  `Arguments`, or other visual group headings.
- Render output metadata inline in the output label:
  `output(exit_code=0; wall_time=1.24s):`
- Include the recorded `original_token_count` in this same label. The example
  above illustrates field placement, not an exhaustive field list.
- Preserve meaningful zero values and the recorded wall-time value; do not
  confuse them with optional-input defaults.
- When the current output has no exit code, omit the exit-code entry regardless
  of why it is absent; do not synthesize a status label:
  `output(wall_time=1.24s):`
- Display the output body directly after its `output(...)` label.
- Treat `exec_command` and `write_stdin` as two specialized renderers in the
  command-execution family, not as instances of a generic Tool detail template.

## Process continuation

- `exec_command` may return output without an exit code when the requested yield
  time ends before an exit code is available.
- The underlying event data may require a later `write_stdin` call to send input
  or poll for more output, but the expanded renderer must not infer or invent
  that state from missing fields.
- `write_stdin` with empty `chars` is a poll and may omit the empty input value.
- `write_stdin` with non-empty `chars` should expose the meaningful input.
- Non-empty whitespace input is still input; do not trim it into an empty poll.
- Include `exit_code` in `output(...)` exactly when the current tool-event
  projection contains one; omit it otherwise. Do not add it through a later UI
  mutation or infer a replacement status.
- Do not show raw `session_id` values in the normal expanded display.
- Do not show `chunk_id`; it is an internal output-chunk identifier, not a
  user-facing Kodex concept.
- Do not source a completed row's title, parameters, color, or output from the
  live shell-session registry. A later poll or process completion must not
  change the data presented for this call.
- Preserve tool-returned output truncation markers; do not replace the recorded
  chunk with later or cumulative process output.

## Implementation boundary

- Shared low-level text/output primitives are allowed.
- The command renderers own their field order, first-line composition, and
  output metadata wording.
- The generic `Arguments`/`Result` disclosure surface must not determine this
  layout.
