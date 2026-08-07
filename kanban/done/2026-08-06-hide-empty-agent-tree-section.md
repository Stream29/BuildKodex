# Task Tree

- [done] Hide empty Agent tree section
  - [done] Confirm empty-section behavior
  - [done] Conditionally render tree section
  - [done] Add focused rendering coverage
  - [done] Run application validation

# Details

- Apply the same empty-section behavior as `Shell sessions`.
- When there are no visible agents, omit both the `Agent tree` title and its empty list area.
- Keep the existing title and list ordering whenever agents are visible.
- Gate the title and Agent tree `LazyColumn` on the same non-empty visible-agent list.
- Cover the empty expanded-sidebar rendering directly.
- `:app-cli-application:jvmTest` passed with temporary, fully restored workarounds for the repository's known Mosaic JVM facade, JDK 22 binding, and missing animation dependency issues.
- IntelliJ inspection and focused IDE build passed.
- Diff whitespace checks passed.
