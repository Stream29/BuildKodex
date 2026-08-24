# Kodex Curl Engine

- Register the Linux-only engine implementation in Ktor's default engine registry from `Kodex/utils/ktor-client-ext`.
- Derive the engine from Ktor's Curl engine and retain its Apache-2.0 source notices.
- Preserve the upstream Curl engine's HTTP, SSE, and WebSocket capability set when it is the Linux default.
- Make request headers and Kotlin/Native stable references request-owned resources.
- Release request headers if a request cannot enter the Curl handler.
- Dispose request-owned resources after Curl cleans the easy handle on completion, cancellation, and engine close.
- Keep each HTTP request cancellation handler registered until its native transfer ends.
- Read `HttpTimeoutCapability.socketTimeoutMillis` per request and enforce it from actual response header/body packet activity.
- Close an idle response body with Ktor's `SocketTimeoutException`; do not replace packet-idle semantics with a low-speed average-rate threshold.
- Route HTTP cancellation through the Curl processor task queue.
- Match cancellation tasks with a request identity, not only an easy-handle pointer.
- Disable HTTP multiplexing so one stalled stream cannot block concurrent requests.
- Forbid reuse of a connection after its request is cancelled.
- Verify that a bare Linux `HttpClient()` selects the local engine before changing Ktor or Kotlin/Native initialization.
- Re-evaluate this local engine whenever the Ktor Curl dependency is upgraded.
