# Expanded `image_gen.imagegen` Display

This checklist records the confirmed expanded-content design for the local
`image_gen.imagegen` tool. Hosted image generation is a separate tool and is
not covered here.

## Confirmed design

- Show the raw tool name as the first expanded line:
  `image_gen.imagegen`
- For a successful result with `saved_path`, use the basename in the collapsed
  title:
  `Generate generated.png`
- For a successful result without a usable `saved_path`, fall back to
  `Generate image`, including for historical events. Do not derive a filename
  from the call ID or turn this recorded success into a synthetic failure.
- Keep the complete `saved_path` in the expanded content.
- Put non-default, meaningful optional inputs on the raw-name line with `; `:
  `image_gen.imagegen; referenced_image_paths=/tmp/input.png`
- Omit null, empty, and default-valued optional inputs.
- Do not use generic `Arguments` or `Result` visual groups.
- Always show the complete `prompt` in the expanded content.
- Include all referenced image paths and `num_last_images_to_include` when
  present; do not repeat fields already on the raw-name line.
- Do not display the `result` base64 payload; it has no useful terminal
  readability.
- Show `saved_path` when the tool result provides one so the user can inspect
  the generated image separately.
- If no `saved_path` exists, do not invent a path or a media placeholder.
- Do not try to load, decode, resolve, or render an image from a path or data
  URL.
- Show non-empty `output_hint` directly as a labeled field.
- Show a failure message directly when generation fails.

## Implementation boundary

- `image_gen.imagegen` remains a specialized image-generation renderer, not a
  generic JSON or function-tool detail renderer.
- Do not mix its layout with `hosted_image_generation`; that tool has a
  different provider-owned result model.
- The unreadable binary representation is intentionally omitted from the
  terminal surface; the UI must not invent a loading source or fallback media
  representation.
- The filename title requires a recorded `saved_path`. The no-path fallback
  handles the existing persistence-failure edge case without changing backend
  result semantics.
