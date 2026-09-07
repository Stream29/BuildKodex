# KodexDocs

- Public documentation repository: [Stream29/KodexDocs](https://github.com/Stream29/KodexDocs), root submodule `KodexDocs/`.
- Site: [GitHub Pages](https://stream29.github.io/KodexDocs/).
- Initial published commit: `d49bddc` (`feat: init`), branch publishing from `main:/docs`.
- Follow-up VitePress migration is local-only by explicit user instruction. Publishing requires separate authorization and switching Pages source to GitHub Actions before pushing; the prepared workflow deploys generated output rather than Markdown sources.
- Writing: `KodexDocs/docs/index.md`, `KodexDocs/docs/zh-CN/index.md`, `KodexDocs/docs/zh-TW/index.md`; headings and additional Markdown pages populate navigation automatically.
- Confirmed presentation: literal Markdown, language codes `zh-CN` / `zh-TW` / `en-US`, full viewport width, flush-left sidebar, hover preview and click pin/unpin using `[→]` / `[←]`.
- Do not fabricate application UI illustrations; any demonstration must use real application rendering.
- Terminal demonstrations: user chose playback of real operations, not a live interactive application. A real CLI → asciinema → [asciinema-player](https://docs.asciinema.org/manual/player/) isolated probe passed browser checks and the user approved its appearance; formal site integration remains pending. Reusable commands, evidence and limits: [recording findings](kodex-terminal-recording.md).
- Local commands, dependencies, theme structure, and publishing prerequisites: [KodexDocs README](../../KodexDocs/README.md).
