# Task Tree

- [done] Package Kodex 0.1.0 for native platforms
  - [done] Confirm version and native executable tasks
  - [done] Change the project version to `0.1.0`
  - Build release executables on Linux
    - [done] Build Linux x64
    - [done] Build Linux ARM64
    - [done] Build Windows x64
  - [done] Build the macOS ARM64 release executable remotely
  - [done] Verify the four resulting artifacts

# Details

- Requested targets: macOS ARM64, Windows x64 (MinGW), Linux x64, and Linux ARM64.
- The macOS target must be built on the remote MacBook.
- The remote's existing project worktree has unrelated uncommitted changes, so the macOS build will use an isolated copy under `~/ACodeSpace/local/`.
- Updated both project-coordinate version declarations from `0.1.0-SNAPSHOT` to `0.1.0`.
- Local builds passed: `:cli-app:linkReleaseExecutableLinuxX64`, `:cli-app:linkReleaseExecutableLinuxArm64`, and `:cli-app:linkReleaseExecutableMingwX64`.
- The isolated MacBook build passed: `:cli-app:linkReleaseExecutableMacosArm64`.
- Release archives and their checksum manifest are in `Kodex/out/`:
  - `kodex-0.1.0-linux-x64.tar.gz`
  - `kodex-0.1.0-linux-arm64.tar.gz`
  - `kodex-0.1.0-macos-arm64.tar.gz`
  - `kodex-0.1.0-windows-x64.zip`
  - `kodex-0.1.0-SHA256SUMS.txt`
- Verified archive entries, checksums, and target architectures. Removed the temporary local and remote build directories after collecting the artifacts.
