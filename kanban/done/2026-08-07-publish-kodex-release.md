# Task Tree

- [done] Publish the next Kodex GitHub release
  - [done] Verify local and remote repository state
  - [done] Confirm the release version and source fix
  - [done] Prepare the `v0.2.0` release commit
    - [done] Change both project version declarations to `0.2.0`
    - [done] Change the Kodex MCP client version to `0.2.0`
    - [done] Add the missing `LuceneKmp` gitlink at `8af2c20`
    - [done] Have the user commit and push the release changes
  - [done] Package supported native artifacts
    - [done] Rebuild Linux x64, Linux ARM64, and Windows x64 in an isolated checkout
    - [done] Build macOS ARM64 on the remote MacBook
    - [done] Recreate archives and a SHA-256 checksum manifest
  - [done] Publish the `v0.2.0` GitHub release and assets
  - [done] Verify the tag, release metadata, and assets

# Details

- The user requested checking whether local work is pushed, following the previous release process, and publishing a new version.
- The previous `0.1.0` release packaged Linux x64, Linux ARM64, macOS ARM64, Windows x64, and a SHA-256 checksum manifest.
- At initial inspection, `BuildKodex/main` and `Kodex/main` were both synchronized with `origin/main`.
- Both repositories initially contained stale staged additions whose files were absent from the working tree and whose net diff against `HEAD` was empty.
- `Kodex/LuceneKmp` commit `8af2c20` was already pushed to `Stream29/lucene-kmp`, but `Kodex/main` initially lacked its gitlink even though `.gitmodules` and `settings.gradle.kts` referenced it.
- The user selected `v0.2.0` and requested adding the missing gitlink before release.
- The prepared release changes preserved the pre-existing stale index entries, and the user committed only the three modified release files plus `LuceneKmp`.
- Build and publish only after the release commit is pushed so every artifact is tied to the exact tagged source.
- The pushed release source is `444f4d3f738c3ec67f07cecf49f0e1ccce0c6ed4`; its two release commits contain only the three version edits and the `LuceneKmp` gitlink.
- The initial three local release link tasks passed; Gradle discarded the configuration-cache entry because of existing Mosaic task serialization problems, but the build itself completed successfully.
- The isolated MacBook build passed at the same release commit; its temporary checkout and remote artifact were removed after transfer.
- The initial four archive layouts, target architectures, and checksum manifest verified locally.
- The pre-publication clean-state guard detected new user changes made during the local builds. The first publication attempt was stopped before creating a tag or release, and the three local targets were rebuilt from an isolated checkout so their source is unambiguous.
- At the user's direction, the clean MacBook checkout rebuilt Linux x64, Linux ARM64, and Windows x64 successfully. The transferred archives were normalized on Linux to remove macOS extended metadata, and all final archives and checksums verify.
- Published release: `https://github.com/Stream29/Kodex/releases/tag/v0.2.0`.
- The `v0.2.0` tag points to `444f4d3f738c3ec67f07cecf49f0e1ccce0c6ed4`.
- The release is neither a draft nor a prerelease and is the latest release.
- GitHub reports all five assets as uploaded; their API SHA-256 digests and sizes match the local files.
- Final archive checksums:
  - Linux x64: `0008a74b2257d8013c159abd83c166411eaadd6c85513c9c1c9ffc4bb41a232b`
  - Linux ARM64: `3f342a30f4c3faf65bb66d5726bb342022f9367cf10d533535fef9bd48c8a61e`
  - macOS ARM64: `0f3059d0197bf599cb551aa6f403007fbee805bf3479f5dc596f26514c2ed5ff`
  - Windows x64: `5bacb23f7393f892f7218ce8ed7f1f0aa967f59d2ac6faff12c401caa3b24dcc`
