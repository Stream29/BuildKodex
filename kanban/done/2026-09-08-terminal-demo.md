# Task Tree

- [done] Explore browser-based demonstrations of real Kodex terminal output.
  - [done] Compare mature terminal-emulation and recording-player components.
  - [done] Confirm the required interaction depth.
  - [done] Recommend a minimal documentation integration and first demonstration.

# Details

- User selected watching real operation demonstrations, not operating the TUI directly in the browser.
- Selected approach: `asciinema-player` with recordings of actual Kodex output; reuse bundled player controls/styles.
- No invented UI, browser-side reimplementation of Kodex, live process backend, or personal-account connection is needed for this approach.
- This task is exploration only. No demo has been recorded, embedded, committed, or published.
- Recommendation: one reusable VitePress player wrapper and static `.cast` assets; start with a short sidebar hover/pin recording before adding session switching and tool-detail demonstrations.
- Before publication, verify Chinese character widths, symbols, colors, recorded terminal dimensions, responsive scaling, and absence of credentials/private conversations. Playback controls are not live interaction with Kodex.
- Sources: [player overview](https://docs.asciinema.org/manual/player/), [embedding guide](https://docs.asciinema.org/manual/player/quick-start/), [xterm.js capabilities](https://github.com/xtermjs/xterm.js#features).
- Selected direction preserved in [KodexDocs context](../../shared-context/findings/kodex-docs.md); actual recording/integration is outside this exploration task.
