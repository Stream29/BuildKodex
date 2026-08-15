# Task Tree

- [done] Release Kodex v0.2.4
  - [done] Confirm version and publication scope
  - [done] Verify clean synchronized repositories
  - [done] Prepare and push version commits
  - [done] Build and verify four CLI archives
  - [done] Prepare checksums and release notes
  - [done] Publish and verify GitHub Release
  - [done] Clean temporary release files

# Details

- Publish a CLI-only GitHub Release for `v0.2.4`.
- Use the remote MacBook as the canonical builder, staging host, verifier, and publisher.
- Keep release builds on Java 25 with configuration cache disabled.
- Only the two path-limited signed `chore: bump version` commits are authorized.
- Both `main` branches match `origin/main`; recursive submodules are clean.
- `v0.2.4` has no existing tag or GitHub Release.
- Signed Kodex version commit: `819855350625392593ac88d3cd01abb90e6b141c`.
- Signed BuildKodex gitlink commit: `7a4aafb678ad26e47d11627a85e2b9228831f76e`.
- MacBook Java 25 build completed all four CLI release targets.
- All archives contain one correctly named binary with the expected format, architecture, and Unix mode.
- SHA-256 verification passed for all four archives.
- Nine concise release-note items cover all eleven newly completed tasks.
- Published `https://github.com/Stream29/Kodex/releases/tag/v0.2.4`.
- The remote tag targets the exact Kodex release commit.
- The published release is neither a draft nor a prerelease.
- All five downloaded assets match the staged names, sizes, and SHA-256 digests.
- Stopped the MacBook Gradle daemon and removed the detached checkout, temporary publisher, credentials, logs, extracted files, and downloaded verification assets.
- Retained only the final release assets under the MacBook `Kodex/out/`.
