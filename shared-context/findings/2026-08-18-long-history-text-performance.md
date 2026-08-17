# Long History Text Performance

- The former `wrapToTerminalWidth` repeatedly segmented and copied the remaining suffix. Long single-line text therefore had near-quadratic time and heavy allocation.
- The minimum fix segments each hard line once, uses direct chunking for printable ASCII, and caches the latest `(text, width)` wrapping inside `WrappedHistoryText`.
- JVM benchmark at width 120:
  - 50,000 ASCII characters: 514.100 ms → 1.772 ms.
  - 100,000 characters: 1,654.394 ms → 3.676 ms.
  - 200,000 characters: 16,401.984 ms → 2.082 ms.
  - 400,298 characters: old implementation exceeded its approximately 101-second sample window; the new implementation took 2.883 ms warm and 6.82 ms median across ten fresh JVMs.
- A release Linux CLI opened a real 448,816-character tool result at width 120:
  - First visible render completed in about 0.49 seconds.
  - Composition and layout settled after about 1.04 seconds of dense CPU work.
  - Process RSS increased from about 95 MiB to 133 MiB.
  - Subsequent same-width scrolling remained responsive and reused the wrapped lines.
- The remaining initial cost is composing and measuring roughly 3,741 `Text` children. Further improvement would require visual-line virtualization or a viewport-aware single text node rather than another wrapping cache.
