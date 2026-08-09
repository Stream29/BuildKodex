---
name: release-kodex
description: "Release Kodex from the BuildKodex repository. Use when preparing, publishing, or verifying a Kodex version, including the specially authorized `chore: bump version` commits, four native builds, packaging, checksums, and GitHub Release."
---

- Confirm the release version `X.Y.Z` and whether the user wants preparation only or a published release.
- Fetch BuildKodex and `Kodex/`, require both `main` branches to match `origin/main`, check recursive submodules, reject uncommitted product changes, and confirm that tag `vX.Y.Z` and its GitHub Release do not exist.
- Change the version to `X.Y.Z` in `Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt`, `Kodex/buildSrc/src/main/kotlin/kodex.kmp-shared.gradle.kts`, and `Kodex/mcp/impl/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt`.
- Create a signed, path-limited `chore: bump version` commit in `Kodex/` containing only those three files; this is an explicit exception to the normal no-commit rule.
- Update the BuildKodex `Kodex` gitlink and create a signed, path-limited `chore: bump version` commit containing only that gitlink; this is the second and final commit allowed by the exception.
- Do not use this exception for any other commit, amend, rebase, squash, or force-push.
- Stop after reporting both commit IDs when the user requested preparation only; when publication was requested, push `Kodex/main` first and BuildKodex `main` second, then verify both remote commits.
- Record the pushed Kodex commit and build every artifact from a fresh recursive clone checked out detached at exactly that commit.
- Detect the running Gradle daemon JVM, set `JAVA_HOME` explicitly, and on Linux run `:app-cli-application:linkReleaseExecutableLinuxX64`, `:app-cli-application:linkReleaseExecutableLinuxArm64`, and `:app-cli-application:linkReleaseExecutableMingwX64` with `--no-configuration-cache`.
- In a clean temporary clone under `~/ACodeSpace/local/` on `ssh stream@macbook`, check out the same commit and run `:app-cli-application:linkReleaseExecutableMacosArm64` with `--no-configuration-cache`.
- Package the four executables as `kodex-X.Y.Z-linux-x64.tar.gz`, `kodex-X.Y.Z-linux-arm64.tar.gz`, `kodex-X.Y.Z-macos-arm64.tar.gz`, and `kodex-X.Y.Z-windows-x64.zip`, with only `kodex` or `kodex.exe` at each archive root.
- Create `kodex-X.Y.Z-SHA256SUMS.txt`, verify it with `sha256sum -c`, extract every archive, and verify its single entry, executable mode, and architecture with `file`.
- Re-fetch and recheck the commit, tag, release, assets, and checksums, then create `vX.Y.Z` with `gh release create`, `--target` set to the exact Kodex commit, `--generate-notes`, and all five assets.
- Verify that the remote tag points to the release commit, the release is published and not a prerelease, and all five GitHub asset names, sizes, and SHA-256 digests match the local files.
- Remove every local and remote temporary checkout, staging directory, log, and transferred intermediate, but keep the final assets under `Kodex/out/`.
