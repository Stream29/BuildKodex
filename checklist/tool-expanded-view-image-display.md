# Expanded `view_image` Display

This checklist records the confirmed expanded-content design for the local
`view_image` tool.

## Confirmed design

- Keep the image path in the collapsed title; the title already summarizes it
  as `View image: <path>`.
- The complete path must remain accessible when expanded. A character-limited
  preview or width-based ellipsis cannot be the only representation.
- Show the raw tool name as the first expanded line:
  `view_image`
- Do not repeat `path` in the expanded content.
- Put non-default input detail on the first line with `; `:
  `view_image; detail=original`
- Omit the default detail value.
- Show the actual returned detail when it differs from the requested detail:
  `output_detail: high`
- Do not display the returned `image_url`; it is an encoded media payload, not
  useful terminal text.
- Do not display `environment_id`. The user confirmed that it has no useful
  display meaning in the current implementation and may be removed in future.
  This is an explicit field-visibility exception, not a rule to hide all IDs.
- Do not load, decode, or render the image in the expanded terminal surface.
- Show a failure message directly when image loading fails.

## Examples

Default request:

```text
view_image
```

Non-default request whose effective output is downgraded:

```text
view_image; detail=original
output_detail: high
```

Failure:

```text
view_image
error: image file does not exist: /tmp/reference.png
```

## Implementation boundary

- `view_image` remains a specialized image-inspection renderer.
- The collapsed title owns the path summary; the expanded body must not repeat
  it.
- The renderer must not invent a media source or a visual placeholder.
- Future removal of `environment_id` is outside this display change; do not
  delete it from models or execution code in this round.
