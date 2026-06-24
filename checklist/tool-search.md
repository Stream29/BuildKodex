# Tool Search

- Keep Lucene/fallback search implementation behind `utils:search-index`.
- Do not let tool modules depend on Lucene directly.
- Let `tool:tool-search` convert tool metadata into generic search documents, then call the search-index abstraction.
- Use Lucene on JVM/native targets and a lightweight fallback on JS node where Lucene KMP has no published variant.
