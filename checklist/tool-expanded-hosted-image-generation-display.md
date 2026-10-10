# Hosted Image Generation Display

This design covers the provider-built-in image-generation event identified as
`hosted_image_generation` in the UI, not the local `image_gen.imagegen` tool.

## Confirmed design

- Use `Generate image` as the successful collapsed title.
- Show `hosted_image_generation` as the first expanded line without a `Tool:`
  prefix.
- Show the complete non-empty `revised_prompt` as a labeled field.
- Omit `result` when it contains unreadable image encoding; retain its original
  value when it is a readable reference.
- Do not load, decode, resolve, or render the image.
- Remove the synthesized `[generated image]` placeholder.
- The current event has no `saved_path`; do not invent a filename or path.
- Do not introduce artifact persistence merely to produce a filename title.
- Hide the internal event ID and keep status in the existing color
  presentation.
- Do not add a generic `Result` group or a second disclosure layer.

## Example

```text
Generate image
  hosted_image_generation
  revised_prompt: A watercolor illustration of a mountain village at sunset.
```
