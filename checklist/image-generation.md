# Image Generation

Use this checklist when changing OpenAI image generation support.

- Keep OpenAI image API DTOs in `CodexLite/openai/models`.
- Keep the OpenAI image HTTP client in `CodexLite/openai/client`.
- Keep the `image_gen.imagegen` tool layer in `tool:image-generation`.
- Keep generated image persistence outside `tool:image-generation`.
- Keep local prompt-image codec logic in `utils:images` and `utils:images-codec`.
- Keep local image inspection in `tool:view-image`.
- Return generated image bytes as API data, not filesystem side effects.
