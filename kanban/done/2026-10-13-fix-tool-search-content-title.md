# Task Tree

- `Confirm the misplaced content with the user`()
- `Move search inputs into expanded content and update regression tests`()
- `Validate the scoped changes and report coverage`()

# Details

- User confirmed that input parameters, not returned definitions, were
  incorrectly displayed as the outer title.
- Restore a short outer summary for local and hosted Tool Search. Put the
  raw name and complete recorded inputs in the expanded body's first line.
- Preserve independent disclosure of returned definitions and existing
  pending/streaming behavior.
- Update the confirmed search display checklist and completed-history header
  projection alongside the renderer.
- Checks: collapsed headers exclude parameters; expanded content retains
  long/multiline local queries, hosted arbitrary JSON and meaningful zero/false;
  invalid inputs keep their error; returned definitions remain independently
  expandable with namespace wrappers.
- User authorized starting one temporary Gradle daemon because no matching
  daemon was running locally or on `remote-macbook`.
- Remote JBR 25 validation passed under the existing
  `kodex-gradle-research-445/device-heavy.lock`:
  - History View JVM: 96 tests, no failures/errors/skips.
  - History ViewModel JVM: 62 tests, no failures/errors/skips.
  - Local/remote SHA-256 comparison of changed source files: identical.
  - Scoped whitespace and updated document-link checks: passed.
- Logs, JUnit XML and source hashes are retained under
  [out/tool-search-811/](../../out/tool-search-811/).
- Removed the isolated `~/ACodeSpace/demo/kodex-tool-search-811/` remote copy
  and stopped only this task's idle Gradle/Kotlin daemons (91245/91278).
- Native tests and a live CLI smoke test were not run.
- User authorized scoped commits and pushes on 2026-10-13. Publication is
  outside this request.
- Kodex fix and regression tests were committed as `daa15ea7` and pushed to
  `origin/main`; the parent repository records this submodule revision.
