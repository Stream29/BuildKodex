# Kodex Curl Engine

- Register the Linux-only engine implementation in Ktor's default engine registry from `Kodex/utils/ktor-client-ext`.
- Derive the engine from Ktor's Curl engine and retain its Apache-2.0 source notices.
- Preserve the upstream Curl engine's HTTP, SSE, and WebSocket capability set when it is the Linux default.
- Make request headers and Kotlin/Native stable references request-owned resources.
- Release request headers if a request cannot enter the Curl handler.
- Dispose request-owned resources after Curl cleans the easy handle on completion, cancellation, and engine close.
- Verify that a bare Linux `HttpClient()` selects the local engine before changing Ktor or Kotlin/Native initialization.
- Re-evaluate this local engine whenever the Ktor Curl dependency is upgraded.
