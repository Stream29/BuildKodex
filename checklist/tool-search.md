# Tool Search

- Keep Lucene/fallback search implementation behind `utils:search-index`.
- Do not let tool modules depend on Lucene directly.
- Keep the Stream29 Lucene KMP fork in `Kodex/LuceneKmp` as an SSH submodule and composite build.
- Substitute `org.gnit.lucene-kmp:lucene-kmp-core` with the fork's `:core` project.
- Preserve host Kotlin Logging configuration unless `LUCENEKMP_LOG_LEVEL` contains a supported explicit level.
- Let `tool:tool-search` convert tool metadata into generic search documents, then call the search-index abstraction.
- Use Lucene on JVM/native targets and a lightweight fallback on JS node where Lucene KMP has no published variant.
- Persist discovered schemas in `ClientToolSearchOutput` so the following Responses request can use them through history.
- Preserve complete `ClientToolSearchOutput` items in remote compaction input. Only a future context-window budget trimming path may conditionally clear `tools`.
- Model tool-search `execution` as a second wire discriminator: client call/output types have a non-null `callId` and implement `ToolCall` / `ToolCallOutput`; server call/output types are pure `HistoryItem` and serialize `call_id: null`.
- When the selected model or provider does not support tool search, promote deferred tools to direct exposure.
