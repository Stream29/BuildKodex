# History Scalability Benchmark

## Scope

- Tested the current Linux x64 release executable.
- Release SHA-256: `923a7a0f2d9d8bf00bd461384e3cd6497848c26f902f1b12a6c227d15fb47a2b`.
- Used an isolated copy of a real 13,388-stable-event session.
- Used a `120×40` tmux terminal on Linux x86-64.
- Filesystem and page cache were warm.
- Repeated startup five times and full-history traversal three times.

## Results

- Process startup: median `53 ms`, range `45–56 ms`.
- Session open to History ready: median `170 ms`, range `138–201 ms`.
- Full traversal reached the oldest item after `31.2–31.4 s` of active scrolling.
- End confirmation raised total wall time to `34.5–36.3 s`.
- The controlled driver requested about 20,500 terminal rows per traversal.
- `99.4%` of changing scroll chunks rendered within the first `60 ms` sample.
- The remaining five chunks rendered within the `260 ms` follow-up sample.
- Scroll CPU time was `33.0–34.4 s`, approximately one logical core under the aggressive input driver.
- Userspace file reads were consistently about `71.4 MiB`.
- Block-device read growth was zero because the page cache was warm.
- Initial PSS was `78.7–80.5 MiB`.
- PSS at the oldest item was `174.8–178.5 MiB`.
- Pre-return peak RSS was `228.2–230.1 MiB`.
- Scroll-to-latest completed in `20–41 ms`, with a `40 ms` median.
- No History read errors or exceptions were logged.

## Remaining Performance Risk

- Returning after a full traversal caused native worker growth in two of three runs.
- Thread counts changed from `22–24` to `26`, `84`, and `96`.
- The `84`- and `96`-thread cases retained the workers for at least five idle seconds.
- A shorter cache-miss probe retained six additional workers for at least 60 idle seconds.
- The extra workers were idle in `futex_do_wait`.
- The return read burst was only about `0.18 MiB` and `318–322` read syscalls.
- The evidence points to concurrent cache-miss reads rather than retained event payloads:
  - `Kodex/app/view/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt:278` starts one `produceState` load per composed item.
  - `Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt:130` dispatches each read.
  - `Kodex/utils/kotlinx-io-coroutines/src/nativeMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/PlatformCoroutineFileSystem.kt:9` configures file I/O with `limitedParallelism(Int.MAX_VALUE)`.

## Interpretation

- Initial open and distant return latency are already small.
- Incremental loading did not exhibit a multi-second UI stall.
- Loaded-history memory grew by about `94–99 MiB` PSS and fluctuated downward after transient peaks.
- The unbounded native file-I/O worker behavior remains a scalability risk before adding folding-related read concurrency.
- These measurements do not establish a speedup over the previous implementation because no previous release was benchmarked.
