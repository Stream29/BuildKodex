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
- Migration gates `:app-migration-impl:jvmTest` and `:app-migration-impl:macosArm64Test` passed on the MacBook with Java 25.
- All four native CLI link tasks passed on the MacBook; the canonical build completed in 14m15s.
- The published release is [Kodex v0.4.9](https://github.com/Stream29/Kodex/releases/tag/v0.4.9), published and non-prerelease, targeting the exact inner release commit.
- The final archive checksums are:
  - Linux x64: `3a598142e7ba00c58ef83f97a19d3d212dcda22827c0d3bd0c197fba17eb3467`
  - Linux arm64: `8c7fae29c16cf1e4b479be0329e465a9eb0c98cc8edb812c055eb2cf225462f3`
  - macOS arm64: `cb023c2ba781e0d9e40ea8d7dc73f06ee3c30d85aca27ae6d8ed1f1709181bad`
  - Windows x64: `99d151a08f2ea3ff6f0f9d3fd3cf257bd31871d3f501460db4e19f7c716e6883`
- All five remote assets passed checksum verification; each archive contains only its single CLI executable at the archive root.
- The published Linux x64 artifact was atomically installed at `Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`; `~/.local/bin/kodex-cli` resolves to it.
