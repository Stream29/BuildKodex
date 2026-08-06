# Task Tree

- [done] Fork and integrate Lucene KMP
  - [done] Confirm upstream and Mosaic integration pattern
  - [done] Create the Stream29 fork and submodule
  - [done] Substitute the published Lucene core dependency
  - [done] Prevent Lucene from overriding host logging
  - [done] Validate dependency resolution and runtime logging

# Details

- Use `nehemiaharchives/lucene-kmp` as the upstream repository.
- Follow the existing `Kodex/Mosaic` SSH submodule and composite-build pattern.
- Keep compatibility with the currently used `10.2.0-alpha14` baseline.
- Add explicit composite-build substitution if Lucene's published coordinates do not match its Gradle project coordinates.
- Preserve host logging when the Lucene environment variable is absent or invalid while retaining explicit environment-based configuration.
- Validate the fork directly and through Kodex's resolved dependency graph and logging reproduction.
- Preserve all pre-existing working-tree changes.
- IntelliJ diagnostics and focused builds passed for the changed source files.
- The focused Android-host regression test passed.
- The Kodex Linux release build compiled `:LuceneKmp:core` and completed successfully.
- An isolated Linux runtime check without `LUCENEKMP_LOG_LEVEL` retained application and Agent runtime logs after Lucene initialized.
- Durable integration and logging rules are recorded in `checklist/tool-search.md`.
