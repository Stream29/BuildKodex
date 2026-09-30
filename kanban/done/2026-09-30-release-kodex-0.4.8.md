# Task Tree

- [done] `Commit the Codex catalog and release-scope records`
- [done] `Create signed Kodex v0.4.8 version bump`
- [done] `Create signed BuildKodex v0.4.8 version bump`
- [done] `Build and package all four CLI archives on MacBook`
- [done] `Publish and verify the GitHub Release`
- [done] `Install and verify the Linux CLI artifact`

# Details

- User authorized the v0.4.8 release and selected inclusion of the current Codex model catalog update.
- Release baseline: the published v0.4.7 marker and release; target tag: `v0.4.8`.
- The signed inner version commit is `b0e68ac920f3b7f116b7302716f69c9f3a14920c`; the signed outer version commit is `5628d795bbfc5afec56d556a8fed9310e61bf359`.
- The MacBook canonical build used Oracle GraalVM Java 25 and `--no-configuration-cache`; migration gates `:app-migration-impl:jvmTest` and `:app-migration-impl:macosArm64Test` passed, followed by all four CLI link tasks in a successful 14m14s build.
- The four archives passed single-entry, executable-mode, format, architecture, and macOS codesign checks. The MacBook checksum file passed verification:
  - Linux x64: `b17791d973ec8ea7e89a060476f998de88dd02d92a5babfd755aa6f61636ba20`
  - Linux arm64: `7406606464852a8f4692445ca841e8c8f46d7eb6383328e48e5a1b7794057791`
  - macOS arm64: `33a1f396e72f0380155f07b575c5247832432499a83ea139d054de415d696e6b`
  - Windows x64: `de76c5aef9eb4a4ba84e2aeb2277454976c1d2446feb8eb06d6a32190a5efe32`
- [Kodex v0.4.8](https://github.com/Stream29/Kodex/releases/tag/v0.4.8) is published, non-draft, non-prerelease, and points to the exact inner release commit. Remote asset names, sizes, and SHA-256 values match the canonical MacBook staging directory `~/ACodeSpace/local/Kodex/out/0.4.8/`.
- The published Linux x64 archive was downloaded and verified on Linux; `Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe` was atomically replaced and `~/.local/bin/kodex-cli` still resolves to it. An already-running Kodex process retains its previous executable until restart.
- The temporary MacBook checkout, build log, staging directory, and download intermediates were removed; only the five final release assets remain under MacBook `Kodex/out/0.4.8/`.
- The separate [codex-rs alignment discussion](../discussion/2026-09-30-align-kodex-with-codex-rs.md) remains an open follow-up and is not an implementation authorization.
- Release notes summarize every completed task newly present after v0.4.7 and exclude this completed release task and the still-open codex-rs discussion task.
