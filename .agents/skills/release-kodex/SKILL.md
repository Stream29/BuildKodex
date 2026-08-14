---
name: release-kodex
description: "Release Kodex from the BuildKodex repository. Use when preparing, publishing, or verifying a Kodex version, including the specially authorized `chore: bump version` commits, MacBook-first builds, four native CLI archives, the universal Desktop shadow JAR, Desktop installers, checksums, and GitHub Release."
---

# Release Host Policy

- Load the `remote-macbook` skill and use `ssh stream@macbook` as the primary release builder, staging host, verifier, and publisher.
- Run every host-independent build and every supported cross-target build on the remote MacBook first; do not default to the local Linux host merely because it is available.
- Use another clean builder only when a task is host-bound or demonstrably unsupported on macOS. Treat a normal build failure as a release blocker, not as permission to switch hosts.
- Build the macOS DMG on the MacBook, the Linux x64 DEB on a Linux x64 builder, and the Windows x64 MSI on a Windows x64 builder. Run only the minimum host-bound or proven-unsupported task on each secondary builder and return its artifact to the MacBook staging directory.
- Keep the MacBook copy of the detached release checkout and staged assets canonical throughout verification and publication.
- Use Java 25 for every Gradle daemon, set `JAVA_HOME` explicitly, keep `--no-configuration-cache` on release builds, and confirm that Desktop tasks select JetBrains Runtime 25.

# Prepare Release Commits

- Confirm the release version `X.Y.Z` and whether the user wants preparation only or a published release.
- Fetch BuildKodex and `Kodex/`, require both `main` branches to match `origin/main`, check recursive submodules, reject uncommitted product changes, and confirm that tag `vX.Y.Z` and its GitHub Release do not exist.
- Change the version to `X.Y.Z` in `Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt`, `Kodex/buildSrc/src/main/kotlin/kodex.kmp-shared.gradle.kts`, and `Kodex/mcp/impl/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt`.
- Create a signed, path-limited `chore: bump version` commit in `Kodex/` containing only those three files; this is an explicit exception to the normal no-commit rule.
- Update the BuildKodex `Kodex` gitlink and create a signed, path-limited `chore: bump version` commit containing only that gitlink; this is the second and final commit allowed by the exception.
- Do not use this exception for any other commit, amend, rebase, squash, or force-push.
- Stop after reporting both commit IDs when the user requested preparation only.
- When publication was requested, push `Kodex/main` first and BuildKodex `main` second, then verify both remote commits.

# Build Release Assets

- Record the pushed Kodex commit and create a fresh recursive clone under `~/ACodeSpace/local/` on the MacBook, checked out detached at exactly that commit.
- Run `:app-desktop:test` on the MacBook before building release assets.
- On the MacBook, first run all four CLI tasks: `:app-cli-executable:linkReleaseExecutableLinuxX64`, `:app-cli-executable:linkReleaseExecutableLinuxArm64`, `:app-cli-executable:linkReleaseExecutableMingwX64`, and `:app-cli-executable:linkReleaseExecutableMacosArm64`.
- Move only a CLI target that is demonstrably unsupported on macOS to a clean compatible builder at the same detached commit, then return the executable to the MacBook.
- Require `:app-desktop:shadowJar` and run it on the MacBook. Block the release if the task is absent or if the JAR is not universal; do not substitute `packageReleaseUberJarForCurrentOS`, which contains only the current host runtime.
- Require the shadow JAR to contain the application entry point and the Linux x64, macOS arm64, and Windows x64 Desktop runtimes.
- Run `:app-desktop:packageReleaseDmg` on the MacBook.
- Run `:app-desktop:packageReleaseDeb` on the clean Linux x64 builder and `:app-desktop:packageReleaseMsi` on the clean Windows x64 builder, both at the exact release commit.
- Return the DEB and MSI to the MacBook and perform all renaming, archive creation, checksums, and final publication there.

# Stage and Verify Assets

- Package the four CLI executables as `kodex-X.Y.Z-linux-x64.tar.gz`, `kodex-X.Y.Z-linux-arm64.tar.gz`, `kodex-X.Y.Z-macos-arm64.tar.gz`, and `kodex-X.Y.Z-windows-x64.zip`, with only `kodex` or `kodex.exe` at each archive root.
- Stage the Desktop assets as `kodex-X.Y.Z-desktop-all.jar`, `kodex-X.Y.Z-desktop-linux-x64.deb`, `kodex-X.Y.Z-desktop-macos-arm64.dmg`, and `kodex-X.Y.Z-desktop-windows-x64.msi`.
- Extract every CLI archive and verify its single entry, executable mode, and architecture.
- Inspect the shadow JAR manifest and entries, verify all three platform runtimes are present, and run a macOS smoke test with Java 25.
- Verify the DMG, DEB, and MSI with platform-native tooling, including package version, target architecture, and contained application identity.
- Create `kodex-X.Y.Z-SHA256SUMS.txt` on the MacBook for all eight payload assets and verify every digest before publication.

# Publish and Clean Up

- Re-fetch and recheck the commit, tag, release, asset list, and checksums immediately before publication.
- Create `vX.Y.Z` with `gh release create`, `--target` set to the exact Kodex commit, `--generate-notes`, and all nine assets: four CLI archives, four Desktop assets, and the checksum file.
- Verify that the remote tag points to the release commit, the release is published and not a prerelease, and all nine GitHub asset names, sizes, and SHA-256 digests match the MacBook staging files.
- Remove every local and remote temporary checkout, staging directory, log, and transferred intermediate from all builders, but keep the final assets under `Kodex/out/` on the MacBook.
