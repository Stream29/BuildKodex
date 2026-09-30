# Task Tree

- [done] `Confirm the GPT-6.1-Sol version-gating cause`
- [done] `Align the Codex API client compatibility version`
- [done] `Run OpenAI client JVM tests`
- [done] `Create signed Kodex and BuildKodex release commits`
- [done] `Run release migration gates and MacBook builds`
- [done] `Publish and verify v0.4.9 assets`
- [done] `Install and verify the Linux CLI artifact`

# Details

- User requested updating Kodex and publishing a follow-up release after confirming that `/models` returns `gpt-6.1-sol` for `client_version=0.159.2` but not `0.155.1`.
- Scoped product fix: update `KodexCompatibleApiClientVersion` from `0.155.1` to `0.159.2`; do not broaden this release into model-catalog merge semantics or other codex-rs alignment work.
- Inner implementation commit: `13e95dff` (`fix: align Codex API client version with codex-rs`).
- `:openai-client:jvmTest` passed on Java 25 in an isolated Linux worktree after initializing recursive submodules.
- Release target is `v0.4.9`; GitHub reports no existing release with that tag.
- Preserve the user's active Kodex `refactor/spec` worktree; perform release work in isolated release worktrees.
- The signed Kodex version commit is `3008a6de9f178cda7cd63123a0261d75013f772d`.
- The signed BuildKodex version commit is `199725c6854d532fac7f0488e346278406d26cee`.
