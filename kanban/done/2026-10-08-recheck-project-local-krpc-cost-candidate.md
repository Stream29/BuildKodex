# REVIEW READY — independent static review, 2026-10-08

# Task Tree
- `Review exact candidate rules, consumer closure and measurement claims`() // complete
- `Report structural blockers and remaining runtime gates`() // complete

# Details
## R — scoped verdict and evidence
- **Static scope sound; suitable for the controlled private GUI gate, not production acceptance.**
  Reviewed only the exact 19-file `/tmp/kodex-local-krpc-review-20261008` snapshot.
  Production `5b41be52` identity/cleanliness is user-supplied, not independently Git-certified.
- Parent: [fixed-model census and acceptance boundary](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-08-measure-fixed-model-build-cost-ablation.md#L12-L39).
  Project-local work was already approved [before the version experiment](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-02-plan-gradle-model-resource-optimization.md#L64-L71);
  this fixed-baseline candidate does not authorize a whole build-logic migration or bypass the paused upgrade.
- Diffed all existing counterparts: root rule removed, one eight-line convention added, plugin applied in 17 scripts.
  No target declaration, source hierarchy, Kotlin declaration, compiler/package pin,
  publishing/TestBalloon/Koin attachment, or B9 export deletion is in this diff.
- **Minimal ablation:** the rule body is necessary for patch routing; registration placement is the measured change.
  Restoring root registration removes that ablation. No empty wrapper, consumer Registry,
  Provider/Resolver/Manager or additional ownership framework is introduced.

### Rule identity, receivers and Native contract
- [Original root:3–9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build.gradle.kts#L3-L9)
  is `subprojects`, not `allprojects`; [candidate root:1](file:///tmp/kodex-local-krpc-review-20261008/build.gradle.kts#L1)
  removes that callback. [Actual convention:2–8](file:///tmp/kodex-local-krpc-review-20261008/buildSrc/src/main/kotlin/kodex.krpc-utils-patch.gradle.kts#L2-L8)
  preserves the exact module selector, project selector and original `.because`.
- Inside `dependencySubstitution`, `project(String)` is the implicit
  `DependencySubstitutions` selector, not `Project.project(String)` returning a Project.
  No explicit receiver redirects it; the separately reported actual Kotlin-DSL compilation passes.
- Each consumer applies it before its base plugin. `configurations.configureEach`
  covers existing and subsequently created normal configurations without resolving them.
  It is not restricted to JVM or Main; Test/Native/metadata normal configurations retain the rule.
  Detached configurations are not newly covered, just as with the original callback.
- Patch project itself has neither the new plugin nor a kRPC library dependency:
  no new self-dependency. Its [Native unique name:8–10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/build.gradle.kts#L8-L10)
  remains `org.jetbrains.kotlinx:utils`; compiler/core/transport/protocol modules are not substituted.
- Actual [Native factory:10–13](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/src/nativeMain/kotlin/kotlinx/rpc/internal/utils/map/ConcurrentHashMap.native.kt#L10-L13)
  uses the [locked collection snapshots:52–61](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/src/commonMain/kotlin/kotlinx/rpc/internal/utils/map/SynchronizedHashMap.kt#L52-L61).
  The real [snapshot test:15–41](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/src/commonTest/kotlin/kotlinx/rpc/internal/utils/map/SynchronizedHashMapTest.kt#L15-L41) is unchanged, not executed by this reviewer.

### Independently recomputed consumer closure
- Parsed the explicit settings tree from [settings:93 onward](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L93),
  obtaining **203 module scripts**. Scanned all **867 literal Main/Test project-edge declarations**,
  including mosaic/named/platform sets; no nonliteral `project(...)` route was found.
  Recursed backwards through both `api` and `implementation`, not only exported compile edges.
- Only three direct runtime-library roots were found using aliases **and direct coordinate strings**.
  Catalog [library aliases:35–38](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/gradle/libs.versions.toml#L35-L38)
  are distinct from the [compiler-plugin alias:93](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/gradle/libs.versions.toml#L93).
  BuildSrc dependency helpers and Mosaic/LuceneKmp/KotlinMcpSdk build/catalog sources reveal no extra declared kRPC root.
- Fixed-point reverse closure is **exactly 17**, matching all snapshot plugin applications.
  Table gives original witness lines; every linked candidate applies the plugin at line 2.
  This is a conservative Main/Test union, not proof that dependency Test edges propagate as published runtime edges.

| Consumer / candidate application | Original direct-root or reverse-edge witness |
| --- | --- |
| [rpc-spec-contract](file:///tmp/kodex-local-krpc-review-20261008/rpc/spec/contract/build.gradle.kts#L2) | [core and test kRPC libraries:19–26](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/build.gradle.kts#L19-L26) |
| [rpc-impl-client](file:///tmp/kodex-local-krpc-review-20261008/rpc/impl/client/build.gradle.kts#L2) | [core:10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/client/build.gradle.kts#L10) |
| [rpc-impl-in-memory](file:///tmp/kodex-local-krpc-review-20261008/rpc/impl/in-memory/build.gradle.kts#L2) | [core/client/server/json:11–14](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/in-memory/build.gradle.kts#L11-L14) |
| [rpc-impl-server](file:///tmp/kodex-local-krpc-review-20261008/rpc/impl/server/build.gradle.kts#L2) | [contract:25](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/build.gradle.kts#L25) |
| [app-impl-rpc](file:///tmp/kodex-local-krpc-review-20261008/app/impl/rpc/build.gradle.kts#L2) | [client:27](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/build.gradle.kts#L27) |
| [app-test-support-rpc](file:///tmp/kodex-local-krpc-review-20261008/app/test-support/rpc/build.gradle.kts#L2) | [app RPC:8](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/test-support/rpc/build.gradle.kts#L8) |
| [app-impl-session](file:///tmp/kodex-local-krpc-review-20261008/app/impl/session/build.gradle.kts#L2) | [app RPC:8](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/session/build.gradle.kts#L8) |
| [app-impl-view](file:///tmp/kodex-local-krpc-review-20261008/app/impl/view/build.gradle.kts#L2) | [session:15](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/build.gradle.kts#L15) |
| [app-impl-application](file:///tmp/kodex-local-krpc-review-20261008/app/impl/application/build.gradle.kts#L2) | [in-memory:25](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/build.gradle.kts#L25) |
| [app-cli](file:///tmp/kodex-local-krpc-review-20261008/app/cli/build.gradle.kts#L2) | [view:14](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/build.gradle.kts#L14) |
| [agent impl-view](file:///tmp/kodex-local-krpc-review-20261008/app/component/agent/impl/view/build.gradle.kts#L2) | [mosaicTest in-memory:30](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/impl/view/build.gradle.kts#L30) |
| [new-session impl-viewmodel](file:///tmp/kodex-local-krpc-review-20261008/app/component/new-session/impl/viewmodel/build.gradle.kts#L2) | [app RPC:8](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/viewmodel/build.gradle.kts#L8) |
| [session-sidebar impl-view](file:///tmp/kodex-local-krpc-review-20261008/app/component/session-sidebar/impl/view/build.gradle.kts#L2) | [test support:19](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-sidebar/impl/view/build.gradle.kts#L19) |
| [settings impl-view](file:///tmp/kodex-local-krpc-review-20261008/app/component/settings/impl/view/build.gradle.kts#L2) | [test support:29](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/impl/view/build.gradle.kts#L29) |
| [settings impl-viewmodel](file:///tmp/kodex-local-krpc-review-20261008/app/component/settings/impl/viewmodel/build.gradle.kts#L2) | [test support:14](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/impl/viewmodel/build.gradle.kts#L14) |
| [hook-impl-notification](file:///tmp/kodex-local-krpc-review-20261008/hook/impl/notification/build.gradle.kts#L2) | [commonTest in-memory:16](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/notification/build.gradle.kts#L16) |
| [integration-test](file:///tmp/kodex-local-krpc-review-20261008/integration-test/build.gradle.kts#L2) | [server:50](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/build.gradle.kts#L50) |

- Artifact contracts are real source definitions, not renamed or empty interfaces:
  [GlobalRpc:31–37](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/GlobalRpc.kt#L31-L37),
  [withInMemoryRpc:37–40](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/rpc/inmemory/WithInMemoryRpc.kt#L37-L40),
  [RestoringRpcClient:24–32](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/rpc/client/RestoringRpcClient.kt#L24-L32).

### Measured-benefit interpretation
- Coordinator/user checkpoint: actual convention compiled; four output copies of 68 JVM/Linux
  Main/Test compile-resolution component inventories are byte-equal to baseline;
  **216 projects / 1344 targets / 5026 source sets / 68010 configurations** unchanged. Reviewer did not read remote result bytes.
- Important precision: [routing probe:16–27](file:///tmp/kodex-scoped-substitution-resolution-20261008.init.gradle#L16-L27)
  stores sorted **component IDs**, not full dependency edges, variants, artifacts or task dependencies.
  Byte equality supports those inventories, not a complete all-platform dependency/artifact certification.
- Supplied warm summary **720,728,592 → 655,710,168 bytes** gives **65,018,424 bytes = 62.006 MiB / 9.021%**.
  Three warm samples per variant, fresh daemon per variant, one cold sample excluded;
  reverse baseline uses the same init scripts/tasks [pilot:52–79,94–110](file:///tmp/kodex-project-local-rule-pilot-20261008.py#L52-L110).
  [Probe:25–35](file:///tmp/kodex-build-cost-census-20261008.init.gradle#L25-L35) supports a coarse post-GC used-heap observation.
  Accept this bounded pre-routing observation; it is not dominator attribution, peak RSS, an OOM repair or proven wall-time/IDE gain.
- Separate root-filter+B9 diagnostic: **253 JVM + 1 patch Native + 5 spec Native tests** reportedly passed.
  They were not executed on this actual convention-only candidate; its own evidence is the 68-route comparisons and pending GUI.
  No-publish (~21.5 MiB/5774 tasks), no-Koin (~0.3 MiB), no-Testplugin (~3 MiB) are excluded diagnostics, not bundled wins.

## B1 — confirmed blockers
- **None found in this exact static candidate.** No missing current declared consumer,
  wrong selector receiver, target change, self-dependency or altered Native unique name was demonstrated.

## B2 — concrete coordinator-owned gates
- **Maintenance guard:** require recomputation of direct external roots + full Main/Test reverse closure
  whenever a dependency/module changes, and check every resulting consumer applies the convention.
  Negative canary: a new test-only consumer without the plugin must be detected.
  This is not currently automated; it is a maintenance gate, not an invented present-source bug or a request for a production 17-entry Registry.
- Before adoption, complete requested/selected component+variant/artifact/task-edge coverage for
  consumer Main/Test compile/runtime/metadata across existing platforms, including actual library source attachment.
  Include omitted-project external transitive routes; component-ID checks alone cannot exclude them.
- Complete the exact candidate's controlled private 4 GiB genuine GUI/import/index/navigation gate,
  then convention-form RPC/serializer/Native patch regressions; preserve fixed targets and baseline toolchain.

## U — still unverified / completion boundary
- No full GUI result or complete all-platform consumer matrix here; no measured IDEA memory/speed acceptance.
  Static fork/declaration inspection cannot certify every pinned external binary's transitive metadata.
  Headless launch exit 126 was reportedly corrected by invoking the original non-executable shell via bash, not a product bug.
- Loaded parent census, independent fanout/retention reports and applicable skills; wrote only this report.
  No Python execution, Gradle/build/test/GUI/IDE/network/device/process-control/MAT/Git operations.
  No temporary files or persistent resources acquired; review finishes independently without awaiting other workers.
  **Do not approve the whole Gradle development-experience task at this CLI/static stage.**
