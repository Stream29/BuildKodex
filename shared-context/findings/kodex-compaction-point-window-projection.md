# Kodex compaction-point window projection

- `CleanCompactionPoint` is a data-free marker in the `index` timeline.
- Its lineage fields (`windowNumber`, `firstWindowId`, `previousWindowId`, and
  `windowId`) live in the `KodexAgentSettings` snapshot at the same index.
- For a point at `n`, the active model window is:
  - `(-inf, n)`: `buildCompactionPrefix`, retaining only
    `CompactionRetainedItem` values within the token budget.
  - `[n, index]`: the raw chronological merge of `index` and `work`, including
    `StableContextCompaction`.
- `IndexVersioned` owns `indexes`, `indexesDescending`,
  `valuesWithIndexes`, and `valuesWithIndexesDescending` as `Flow` members.
- Their default traversal fetches exponentially growing index ranges so
  consumers do not implement doubling or enumerate a complete timeline.
- `latestCompactionPointOrNull()` finds the newest marker by descending index
  metadata and stops at the first marker.
