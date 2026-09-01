---
name: release-kodex
description: "Release the CLI-only Kodex from the BuildKodex repository. Use when preparing, publishing, or verifying a Kodex version, including the specially authorized `chore: bump version` commits, MacBook-first builds, four native CLI archives, checksums, completed-task release notes, and GitHub Release."
---

# Release Host Policy

- Load the `remote-macbook` skill and use `ssh stream@macbook` as the primary release builder, staging host, verifier, and publisher.
- Run every host-independent build and every supported cross-target build on the remote MacBook first; do not default to the local Linux host merely because it is available.
- Use another clean builder only when a task is host-bound or demonstrably unsupported on macOS. Treat a normal build failure as a release blocker, not as permission to switch hosts.
- Keep the MacBook copy of the detached release checkout and staged assets canonical throughout verification and publication.
- Use Java 25 for every Gradle daemon, set `JAVA_HOME` explicitly, and keep `--no-configuration-cache` on release builds.

# Prepare Release Commits

- Confirm the release version `X.Y.Z` and whether the user wants preparation only or a published release.
- Fetch BuildKodex and `Kodex/`, require both `main` branches to match `origin/main`, check recursive submodules, reject uncommitted product changes, and confirm that tag `vX.Y.Z` and its GitHub Release do not exist.
- Change the version to `X.Y.Z` in `Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt`, `Kodex/buildSrc/src/main/kotlin/kodex.kmp-shared.gradle.kts`, and `Kodex/mcp/impl/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt`.
- Create a signed, path-limited `chore: bump version` commit in `Kodex/` containing only those three files; this is an explicit exception to the normal no-commit rule.
- Update the BuildKodex `Kodex` gitlink and create a signed, path-limited `chore: bump version` commit containing only that gitlink; this is the second and final commit allowed by the exception.
- Do not use this exception for any other commit, amend, rebase, squash, or force-push.
- Stop after reporting both commit IDs when the user requested preparation only.
- When publication was requested, push `Kodex/main` first and BuildKodex `main` second, then verify both remote commits.

# Verify Home Migrations

- Read the previous published application version and the proposed `X.Y.Z`; verify `app/migration/impl` generates its current `MigrationVersion` from the same Gradle `project.version`.
- List registry entries satisfying `previousVersion < toVersion <= X.Y.Z`; an entry is required only when that release changes persisted Home data.
- Require each newly activated entry to have its frozen version-specific source, old codec when needed, fixture, target-layout assertions, interrupted-run reentry test, and cross-version upgrade coverage.
- Compare every migration, old codec, and fixture activated by or before the previous release against that release commit; reject any modification, move, deletion, `toVersion` reuse, or registry reordering.
- Run the Home baseline, version selection, newly activated migration, read/write lease, and isolated-Home startup tests before building release assets.

# Build Release Assets

- Record the pushed Kodex commit and create a fresh recursive clone under `~/ACodeSpace/local/` on the MacBook, checked out detached at exactly that commit.
- Treat the release as CLI-only; do not look for Desktop modules, tasks, archives, or installers.
- On the MacBook, first run all four CLI tasks: `:app-cli:linkReleaseExecutableLinuxX64`, `:app-cli:linkReleaseExecutableLinuxArm64`, `:app-cli:linkReleaseExecutableMingwX64`, and `:app-cli:linkReleaseExecutableMacosArm64`.
- Move only a CLI target that is demonstrably unsupported on macOS to a clean compatible builder at the same detached commit, then return the executable to the MacBook.
- Perform all renaming, archive creation, checksums, and final publication on the MacBook.

# Stage and Verify Assets

- Package the four CLI executables as `kodex-X.Y.Z-linux-x64.tar.gz`, `kodex-X.Y.Z-linux-arm64.tar.gz`, `kodex-X.Y.Z-macos-arm64.tar.gz`, and `kodex-X.Y.Z-windows-x64.zip`, with only `kodex` or `kodex.exe` at each archive root.
- Extract every CLI archive and verify its single entry, Unix executable mode where applicable, binary format, and target architecture.
- Create `kodex-X.Y.Z-SHA256SUMS.txt` on the MacBook for all four CLI archives and verify every digest before publication.

# Prepare Release Notes

- Use the previous published release's BuildKodex version-bump commit as the exclusive start and the current BuildKodex version-bump commit as the inclusive end of the release-note range. Use the repository root for the first release.
- Enumerate every task newly present under `kanban/done/` in that range, including tasks moved there from another kanban state; exclude tasks that were already done and only modified during the range.
- Read every discovered task at the current release commit instead of deriving release notes from filenames alone.
- Write an unordered Markdown list whose items are very short sentences describing the completed changes.
- Cover every discovered task, but combine related tasks and reorder the resulting items when that produces a clearer summary; do not require one item per task.
- Supply the list through `gh release create --notes` together with `--generate-notes` so it precedes the generated notes and retains the Full Changelog link. Do not publish a description containing only the Full Changelog link.
- Check the final task-to-item coverage and rendered release description before publication.

# Publish and Clean Up

- Re-fetch and recheck the commit, tag, release, asset list, and checksums immediately before publication.
- Create `vX.Y.Z` with `gh release create`, `--target` set to the exact Kodex commit, `--generate-notes`, the prepared summary supplied through `--notes`, and all five assets: four CLI archives and the checksum file.
- Verify that the remote tag points to the release commit, the release is published and not a prerelease, and all five GitHub asset names, sizes, and SHA-256 digests match the MacBook staging files.
- Remove every local and remote temporary checkout, staging directory, log, and transferred intermediate from all builders, but keep the final assets under `Kodex/out/` on the MacBook.
