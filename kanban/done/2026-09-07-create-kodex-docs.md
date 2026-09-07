# Task Tree

- [done] Create public KodexDocs and publish initial documentation.
  - [done] Confirm repository, publishing authorization, and product behavior.
  - [done] Create public repository.
  - [done] Implement minimal navigation and unrendered Markdown in three languages.
  - [done] Publish initial commit and register submodule.
  - [done] Migrate to VitePress with Markdown-only content maintenance.
    - [done] Replace the bespoke generator with VitePress and a minimal raw-text theme.
    - [done] Preserve three languages and application-style hover/pin navigation.
    - [done] Use the full viewport width with the sidebar flush left; no centered reading container.
    - [done] Prepare local Pages workflow without changing the live deployment.
  - [done] Verify build, local browser interactions, and initial public Pages deployment.

# Details

- User requested a public `KodexDocs` repository and root BuildKodex submodule.
- User authorized the new repository's initial commit with message `feat: init`; do not commit BuildKodex.
- Inspect CLI source and public release assets for documentation accuracy.
- Confirmed languages: `zh-CN`, `zh-TW`, `en-US`.
- User approved migrating to stable VitePress with a minimal custom theme; store Markdown in `docs/`, ignore generated output, and prepare a GitHub Actions deployment workflow.
- User confirmed direct display of unrendered Markdown; keep only navigation, language switching, raw-file links, and core documentation.
- Do not fabricate product UI. Initial site has no UI illustration; any future demonstration must capture real application rendering.
- Validate source/output consistency, links, desktop/mobile wrapping, language switching, and unrendered Markdown fidelity.
- Public repository created at `Stream29/KodexDocs` and registered as root submodule.
- Initial `feat: init` commit: `d49bddc`; registered root submodule and enabled Pages.
- Latest request: use application's `[→]` / `[←]` left-sidebar controls, hover preview and click-to-pin; locale labels are exactly `zh-CN`, `zh-TW`, `en-US`.
- User explicitly chose local-only changes for this follow-up; do not commit or push the sidebar/locale update.
- Migration is also local-only. Keep remote Pages on the initial deployment until a later authorized publication; document the required switch from branch publishing to GitHub Actions.
- Latest layout requirement: remove centered/max-width containers and generous whitespace; keep the sidebar at the viewport's left edge.
- Validation: VitePress production build and 10 unit/static-output tests pass; `npm audit` reports zero vulnerabilities with pinned Vite 6.4.3 override.
- Local Chrome verification: Markdown fidelity, literal HTML/Vue safety, automatic page discovery, Markdown/heading HMR, hover/pin/unpin, keyboard/Escape, touch, locale anchors/history, raw endpoints, no-JavaScript content, and 404.
- Final layout checked in all three languages at widths 320, 390, 768, 1440, and 2560: sidebar x=0, workspace fills viewport, no horizontal overflow.
- Initial remote Pages status is `built` and public HTML is reachable; remote `main` remains `d49bddc`. No follow-up commits, pushes, or Pages source changes.
- Durable context: [KodexDocs](../../shared-context/findings/kodex-docs.md).
