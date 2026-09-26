# Token-count Timeline

- Preserve the timeline's context-count snapshot semantics while adding response usage and request diagnostics.
- Add optional turn-state settings with an absent-value default; do not batch-rewrite old settings solely to add this field.
- Migrate old integer token-count records to structured legacy records through the existing Home migration registry; do not maintain integer compatibility in the ordinary runtime decoder.
- Preserve each old total, including zero, without inventing input/output/cache/reasoning breakdowns or inferring a reset event solely from its value.
- Keep missing usage details distinct from explicitly reported zero.
- Keep collection best-effort: record decodable provider usage and available diagnostics, without requiring a record for every request or synthesizing records solely for missing usage, transport failure or cancellation.
- Preserve record indexes, timestamps and latest pointers during this format conversion; never treat `latest.json` as a token-count record.
- Follow [Kodex Home](kodex-home.md) for migration ownership, versioning and failure semantics: in-place deterministic conversion, recognizable target records skipped on rerun, malformed or ambiguous data blocks opening, and no added backup/staging/journal/transaction framework.
