# Codex upstream refresh — 2026-09-30

## Source and comparison

- Source repository: [openai/codex](https://github.com/openai/codex)
- Retrieved: 2026-09-30 09:24 UTC
- Previous reference commit: `a86631502d49274cb47208925c7d3dcece032029` (2026-09-21)
- Updated reference commit: `d42056091aded7feb1d88ac7e83972108b2aa478` (2026-09-30)
- Comparison size: 547 commits
- Latest stable release observed during refresh: `rust-v0.159.2`
- The reference submodule continues to track `origin/main`; the outer repository now points at the updated commit.

## Applied Kodex update

- Synchronized `Kodex/openai/model-catalog/impl`'s built-in catalog with the current upstream `models.json`.
- Added GPT-6-Astra, GPT-6.1-Sol, GPT-6-Sol, and GPT-6-Luna.
- Updated GPT-5.6 and Codex Auto Review context windows, compaction hashes, reasoning presets, and Fast-tier descriptions.
- Removed bundled GPT-5.4, GPT-5.4-Mini, and GPT-5.2 entries that are no longer present in the upstream bundled catalog.
- Kept the Kotlin model shape unchanged. The upstream hidden Daybreak entries remain omitted because Kotlin `ModelInfo` has no visibility field. Transport capabilities such as `prefer_websockets`, `use_responses_lite`, and `supports_search_tool` remain outside this focused catalog refresh.

## Review boundary

- The upstream range also contains broad app-server protocol, MCP authorization, WebSocket prewarming, tool-metadata budgeting, retry-advice, and agent orchestration changes.
- Those changes require separate compatibility designs because Kodex currently owns a Kotlin client and an HTTP/SSE transport boundary. They were not copied into the current change.
- No live model request, credential refresh, or real Session migration was performed.

## Validation

- `git -C shared-context/codex status --short --branch`: clean, tracking updated `origin/main`.
- `git -C Kodex diff --check`: passed.
- `git diff --check`: passed.
- `JAVA_HOME=/home/stream/.gradle/jdks/eclipse_adoptium-25-amd64-linux.2 ./gradlew :openai-model-catalog-impl:jvmTest --no-configuration-cache --max-workers=1`: passed.
