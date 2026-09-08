# KodexDocs

- Public documentation repository: [Stream29/KodexDocs](https://github.com/Stream29/KodexDocs), root submodule `KodexDocs/`.
- Site: [GitHub Pages](https://stream29.github.io/KodexDocs/).
- Initial published commit: `d49bddc` (`feat: init`), branch publishing from `main:/docs`.
- Follow-up VitePress migration is locally committed under explicit batch-commit authorization, not pushed. Publishing requires separate authorization and switching Pages source to GitHub Actions before pushing; the prepared workflow deploys generated output rather than Markdown sources.
- Writing: one `index.md` per locale under `KodexDocs/docs/`, `docs/zh-CN/`, `docs/zh-TW/`. Only UI showcase remains; eighteen old pages, including installation and connections, were deleted. Installation is one Agent prompt in `Kodex/README.md`.
- Confirmed presentation: literal Markdown, language codes `zh-CN` / `zh-TW` / `en-US`, full viewport width, flush-left sidebar, hover preview and click pin/unpin using `[→]` / `[←]`.
- Do not fabricate application UI illustrations; any demonstration must use real application rendering.
- Terminal demonstrations are integrated locally: 8 isolated native CLI recordings and 4 real-component recordings, distinguished in the provenance manifest. All twelve autoplay and loop indefinitely, without controls; an inert mount prevents mouse/keyboard/focus interaction. The execution recording uses a real model under explicit read-only credential authorization, in a disposable project; the website has no live backend or credentials. Sources and hashes: [manifest](../../KodexDocs/docs/public/recordings/manifest.json).
- User-facing content is limited to demo headings/players and the explicitly requested Codex reuse note. Generated provenance captions, small-screen hints, and extra explanatory paragraphs were removed.
- Detailed Hooks/MCP recordings, persisted-operation checks, and the unsupported-import issue: [integration recording acceptance](docs-integrations-recording.md).
- Markdown is still literal text; only registry-owned player markers mount a player. Prototype names such as `constructor` do not mount components. Navigation and raw assets use Vite imports, without a custom file server or watcher; add/edit/delete discovery was verified in the running dev server.
- Do not equate a visible button, fixture callback, or unit test with a filmed end-to-end operation. Current coverage, navigation animation, and remaining gaps: [refinement acceptance](docs-showcase-refinement-2026-09-08.md). Reusable commands: [recording findings](kodex-terminal-recording.md).
- Local commands, dependencies, theme structure, and publishing prerequisites: [KodexDocs README](../../KodexDocs/README.md).
