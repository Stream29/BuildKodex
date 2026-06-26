# Image Codec

Use this checklist when changing prompt-image codec implementations.

- Keep `utils:images` pure and free of platform codec dependencies.
- Keep platform codec implementations in `utils:images-codec`.
- Use Skiko for non-Windows native host targets that have Skiko K/N artifacts.
- Prefer Windows native codecs on `mingwX64`; fall back to KorIM when the native codec does not support the requested transform.
- Keep codec capability tests in `commonTest` through common APIs such as `HostPromptImageTransformer`.
- Treat WebP as unsupported in the prompt-image business chain; do not expose it as an `ImageMimeType`.
- Do not claim JPEG transform support on a target unless tests cover actual decode, resize, encode, and MIME preservation.
- Run native codec test binaries on their matching host OS; do not rely on cross-linked native test executables for codec validation.
