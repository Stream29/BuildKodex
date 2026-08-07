# Task Tree

- Publish the next Kodex GitHub release
  - [done] Verify local and remote repository state
  - [done] Confirm the release version and source fix
  - Prepare the `v0.2.0` release commit
    - [done] Change both project version declarations to `0.2.0`
    - [done] Change the Kodex MCP client version to `0.2.0`
    - [done] Add the missing `LuceneKmp` gitlink at `8af2c20`
    - Have the user commit and push the release changes
  - Package supported native artifacts
    - Build Linux x64, Linux ARM64, and Windows x64 locally
    - Build macOS ARM64 on the remote MacBook
    - Create archives and a SHA-256 checksum manifest
  - Publish the `v0.2.0` GitHub release and assets
  - Verify the tag, release metadata, and assets

# Details

- The user requested checking whether local work is pushed, following the previous release process, and publishing a new version.
- The previous `0.1.0` release packaged Linux x64, Linux ARM64, macOS ARM64, Windows x64, and a SHA-256 checksum manifest.
- `BuildKodex/main` and `Kodex/main` are both synchronized with `origin/main`.
- Both repositories contain stale staged additions whose files are absent from the working tree and whose net diff against `HEAD` is empty.
- `Kodex/LuceneKmp` commit `8af2c20` is pushed to `Stream29/lucene-kmp`, but `Kodex/main` does not contain a `LuceneKmp` gitlink even though `.gitmodules` and `settings.gradle.kts` reference it.
- The user selected `v0.2.0` and requested adding the missing gitlink before release.
- Preserve the pre-existing stale index entries and have the user commit only the three modified release files plus `LuceneKmp`.
- Build and publish only after the release commit is pushed so every artifact is tied to the exact tagged source.
