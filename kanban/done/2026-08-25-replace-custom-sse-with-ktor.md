# Task Tree

- [done] Replace custom POST-SSE transport with Ktor SSE
  - [done] Confirm clean baseline and official API behavior
  - [done] Attempt direct official SSE migration
  - [done] Identify real response-header blocker
  - [done] Verify old transport against real API
  - [done] Add narrow missing-header compatibility
  - [done] Re-run tests, build, and end-to-end checks
  - [done] Compare final implementation and behavior

# Details

- Use Ktor 3.5.0 `SSE` and `HttpClient.sse` with `HttpMethod.Post`.
- Preserve the public `postSseEvents` Flow API and per-request packet-idle socket timeout.
- Keep the project-owned KodexCurl socket-timeout implementation.
- Replace the custom SSE parser/session adapter with Ktor's official implementation.
- Do not commit changes.
- Ktor extension tests and offline OpenAI tests pass.
- Linux release executable builds and resolves all dynamic libraries.
- Linux release CLI enters the main screen and exits cleanly in a fixed-size PTY.
- The direct official implementation fails against the authenticated ChatGPT backend because its successful SSE response omits `Content-Type`.
- The unchanged implementation succeeds against the same endpoint and `~/.kodex` credentials.
- Ktor 3.5.0 has no public switch for relaxing its SSE response `Content-Type` check.
- The final compatibility layer supplies `text/event-stream` only for marked `200` responses with no `Content-Type`; explicit wrong types remain errors.
- The final implementation and the unchanged baseline both pass the same real streaming probe with `~/.kodex`.
- Ktor extension tests pass on JVM, JS, and Linux X64.
- The 61 selected offline OpenAI JVM tests pass.
- The Linux X64 release executable builds and passes the fixed-size PTY smoke test.
- IDEA inspections report no errors in changed production files; IDEA composite build remains blocked in the unrelated `LuceneKmp` build by duplicate generated project accessors.
