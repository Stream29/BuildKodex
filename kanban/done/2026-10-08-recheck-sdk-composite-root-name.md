# REVIEW READY

Scoped independent review is complete. Publication and consumer gates remain
owned by the [maintenance task](2026-10-08-maintain-mcp-sdk-composite-root-name.md);
the original static verdict below is preserved.

# Task Tree

- `Review the exact SDK maintenance settings delta`()
- `Verify unchanged module identities and the source-composite admission boundary`()
- `Return a scoped commit and controlled-publication verdict`()

# Details

## R — scoped verdict and evidence

- Independent READONLY review; only this report was written. No builds/tests,
  IDE/device/process control, credentials, network writes or Git commands used.
- Parent: [maintenance scope:16–28](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-maintain-mcp-sdk-composite-root-name.md#L16).
  READY for the approved one-line design, conditional commit after B2 gates;
  not runtime acceptance or permission to bypass controlled publication.
- Exact reviewed [snapshot:1](file:///tmp/kodex-sdk-root-name-review-20261008/settings.gradle.kts#L1):
  SHA256 `237c2bab2a05251ca22b7beff8bdce4f18157de5630e830210ce25139c130fd0`.
  Current [original settings:1](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/settings.gradle.kts#L1) matches it.
- Read [baseline archive](file:///tmp/kodex-sdk-53f71761-source-gate.tar) in memory:
  PAX commit `53f717661712d618cd4822d1760392e903989ead`;
  baseline settings SHA256 `61597baf4880a3108061e92a8d3f31d1e7431e50aa8cd650caf67e0fadeec906`.
  Compared all 428 archived regular files: none missing; only settings differs,
  solely `rootProject.name = "kotlin-sdk"` → `"kotlin-mcp-sdk-fork"`.
- Actual Gradle 9.5.1 `RootProjectAccessorSourceGenerator` bytecode was inspected
  with `javap` in the [local dependency-management JAR](file:///home/stream/.gradle/wrapper/dists/gradle-9.5.1-bin/edjwkjtznjkrmzcbzas0sagxt/gradle-9.5.1/lib/plugins/gradle-dependency-management-9.5.1.jar).
  It emits an accessor for the descriptor's own root name, then its children;
  names normalize separators to camel case. Old root and internal `:kotlin-sdk`
  both emit `getKotlinSdk()`. New root emits `getKotlinMcpSdkFork()`;
  [internal project:26](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/settings.gradle.kts#L26) still emits `getKotlinSdk()`.
- Minimum ablation, static: reverting only line 1 restores that collision;
  changing only outer `includeBuild.name` does not change the nested descriptor.
  No accessor-disable workaround, internal-module rename or API rename is needed.
- Three library substitutions retain `io.modelcontextprotocol:kotlin-sdk-{core,client,server}`:
  [group:10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/gradle.properties#L10),
  [module IDs:22–24](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/settings.gradle.kts#L22).
  All API/source/target declarations are byte-unchanged; build/root identity changes intentionally.
- Actual [three-module publication contract:24–30](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/contract.py#L24) is unchanged.
  [Recipe:14–20](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/publish.init.gradle#L14) already sets exactly this root name;
  the source fix makes that operation idempotent, not a new publication recipe.
  [Root script:1–5,25–29](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/build.gradle.kts#L1) does not publish Maven components and fixes its Dokka title.
  [Module POM name:14](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/buildSrc/src/main/kotlin/mcp.publishing.gradle.kts#L14)
  and [Dokka source links:16–19](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/buildSrc/src/main/kotlin/mcp.dokka.gradle.kts#L16) use each project's name.
  Root-level name-derived metadata can change; published child names/links do not.
  New package identity still enters via [startup version:262](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/pipeline.py#L262)
  and generated [LIB_VERSION:19–28](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/kotlin-sdk-core/build.gradle.kts#L19), so new payload bytes can legitimately differ.

## B1 — confirmed blockers

- None confirmed in the exact one-line delta. Pending execution is not a PASS.

## B2 — coordinator-owned admission and publication gates

- [Actual source composite:31–37](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-maintain-mcp-sdk-composite-root-name.md#L31):
  coordinator reports actual `includeBuild("KotlinMcpSdk")`, no name override,
  `help` SUCCESS in 35s. Mandatory [SDK apiCheck:15](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/AGENTS.md#L15)
  and actual stdio/HTTP JVM/JS checks are running, not reviewer-certified passes.
  Use `:KotlinMcpSdk:apiCheck`; the retained root-name-based selector failure is not hidden.
- After these pass, commit only the settings line on `kodex-submodule`, not upstream/main;
  update the product gitlink on main first, retaining accepted `0.14.0-kodex.53f717661712`.
  [Identity:39–74](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/pipeline.py#L39) requires that gitlink and derives a NEW commit-suffixed package version.
  Never republish old `53f717…` with changed source/recipe; byte equivalence is not assumed.
- Main is currently UNPROTECTED per coordinator: use reviewed manual dispatch only.
  Preserve [existing event gate:54–58](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/.github/workflows/fork-packages-mcp.yml#L54);
  genuine Linux/macOS/Windows producer and JVM/JS/Native consumer gates, then
  [writer/remote byte verification:213–246](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/.github/workflows/fork-packages-mcp.yml#L213), must finish before catalog adoption.

## U — unverified / outside this verdict

- Accepted `c0ba86cc` binary evidence (39 artifact/9 source hashes, 132 JVM/JS tests,
  Native link/PTY) belongs to the old baseline, not this source-composite runtime.
- No new SDK commit/gitlink/package bytes reviewed yet. Root wrapper 9.5.1/KGP 2.4.0
  stay unchanged; full-source Mosaic/Lucene compatibility and real GUI/Sync remain gates.
  No whole-203-project, navigation, IDE/resource improvement or final publication PASS.
