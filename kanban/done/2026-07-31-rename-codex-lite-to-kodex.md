# Task Tree

- [done] Rename Codex Lite to Kodex
  - [done] Confirm rename mapping and semantic boundary
  - [done] Rename source packages and project-owned symbols
  - [done] Rename project configuration and documentation
  - [done] Rename inner and outer repositories, including GitHub
  - [done] Run focused validation and record baseline limitations

# Details

- Renamed `BuildCodexLite` to `BuildKodex`.
- Renamed `CodexLite` and the `Codex Lite` product name to `Kodex`.
- Renamed package root `io.github.stream29.codex.lite` to `io.github.stream29.kodex`.
- Renamed project-owned `Codex` symbols to `Kodex`.
- Preserved explicit OpenAI Codex CLI, wire protocol, metadata, and `CODEX_HOME` compatibility names.
- Preserved pre-existing uncommitted `AD` entries without restoring or rewriting them.
- Renamed GitHub repositories to `Stream29/BuildKodex` and `Stream29/Kodex`, then updated both local `origin` URLs.
- Passed `projects`, `kotlinStoreYarnLock`, and `:cli-app:compileKotlinLinuxX64`.
- `:agent-runtime-decorator-tool:jvmTest` and Mosaic's JDK 22 compile failure reproduce at the original HEAD, so they are baseline limitations rather than rename regressions.
