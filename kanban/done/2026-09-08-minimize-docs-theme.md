# Task Tree

- [done] Minimize handwritten documentation JS and CSS without changing confirmed behavior.
  - [done] Identify framework/native replacements and redundant styles.
  - [done] Simplify asset delivery, event handling, and responsive styles.
  - [done] Verify source fidelity, navigation, and terminal layout.

# Details

- Final integration uses Vite positive glob imports for page discovery and raw assets, replacing createContentLoader and redundant exclusions; no custom watcher or polling remains. Add/edit/delete in the already-running dev server passed. Source fidelity, three locales, five viewport widths, hover/pin/Escape/touch and no-JS behavior also passed; see [current acceptance](../../shared-context/findings/docs-interaction-acceptance-2026-09-08.md).
- Earlier byte counts below describe the theme-only reduction, before the separately requested terminal player and 21-page manual were added.

## Earlier implementation and paused verification
- User requested less handwritten JS/CSS; keep VitePress, literal Markdown, three language codes, flush-left/full-width layout, and hover/pin sidebar.
- Reuse the current KodexDocs environment and existing dependencies; do not switch themes or remove confirmed functionality merely to reduce line counts.
- Changes remain local-only; no new commits, pushes, or Pages configuration changes.
- Use Vite asset URL imports instead of a raw-file copy hook and custom development middleware.
- Reuse the already-installed VueUse event/mount utilities, declaring the existing version directly; retain explicit sidebar semantics and raw-source safety.
- Consolidate responsive CSS rather than adding a utility framework or hiding complexity in minified formatting.
- Validate builds and static source equivalence, raw asset URLs in dev/production, HMR/page discovery, hover/pin/Escape/touch, locale anchors, and full-width layout at desktop/mobile sizes.
- Baseline theme/config sources: 15,479 bytes; CSS: 57 rules, 162 declarations, 4,832 bytes.
- Current theme/config sources: 12,754 bytes; CSS: 40 rules, 107 declarations, 3,213 bytes. No additional installed package was needed; the existing VueUse version is now a direct dependency.
- Production build, 10 unit/static tests, audit, and dev/production browser checks pass: three locales at five widths, raw assets, hover/pin, keyboard/Escape, touch, language anchors, no-JS content, and no hydration errors.
- New-file discovery during an already-running dev session did not update the catalog/raw-asset map in the attempted HMR check. Do not claim that check passed; further investigation is paused while the user directs terminal-demo exploration. Temporary Markdown test page removed.
