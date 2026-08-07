# Kodex Curl Stream Failure Domain

- The application shares one `OpenAiClient`, `HttpClient`, Curl processor, and multi handle across sessions.
- libcurl multiplexed concurrent HTTP/2 streams over one TCP connection.
- A blackholed connection therefore blocked existing streams and new sessions using the same client.
- Request cancellation had two cleanup defects:
  - The cancellation handler was removed after response headers, before a streaming response ended.
  - Pre-header cancellation was launched onto the same continuously running single-thread Curl dispatcher.
- Removing a cancelled transfer alone still allowed its half-open connection to be reused.

## Controlled Reproduction

- An HTTP/1.1 streaming server observed that cancelling a started response did not close the native connection.
- A TLS HTTP/2 proxy froze only the client's first connection.
- Before isolation, another request on the shared client stalled while an independent client succeeded.
- With multiplexing disabled, the shared client opened a second connection and succeeded.
- After marking cancelled connections non-reusable, an immediate retry also opened a new connection and succeeded.

## Resolution

- `Kodex/utils/ktor-client-ext/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlProcessor.kt:81`
  processes HTTP cancellation through the Curl task queue.
- `Kodex/utils/ktor-client-ext/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt:27`
  retains cancellation handlers through the native transfer and protects against easy-handle pointer reuse.
- `Kodex/utils/ktor-client-ext/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt:62`
  disables HTTP multiplexing.
- `Kodex/utils/ktor-client-ext/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt:350`
  forbids connection reuse after cancellation.
