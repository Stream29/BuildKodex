# REVIEW READY — bounded diagnostic review, adoption blocked

# Task Tree

- `Verify the fixed actual dependency overlay and Native source guards`() // complete
- `Review exclusions, API visibility and all-platform comparison limits`() // complete
- `Report diagnostic readiness and remaining adoption gates`() // complete

# Details

## R — recommendation and authority

- **Controlled coordinator-owned GUI diagnosis is ready; product adoption is not.**
  No confirmed static defect found in the fixed overlay; not full IDE/OOM acceptance.
- Independent reviewer, not Session582 preparer/coordinator. Only this report changed.
  Read-only files/hashes, public Maven GETs and Xiaoxin static receipts; no Python execution,
  builds/tests/IDE/CI, process control, Git, credentials or private properties/heap reads.
- Loaded [parent repair](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L157), [completed preparation](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-08-prepare-direct-krpc-patch-dependencies.md#L1) and relevant skills/spec-impl checklist.
- Source authority: [five-file SHA256 manifest](file:///tmp/kodex-direct-krpc-candidate-20261008/SHA256SUMS#L1), not the mutable remote project.
  Independently verified all five overlay hashes and [all17 baseline guards](file:///tmp/kodex-direct-krpc-candidate-20261008/BASELINE.sha256#L1): five builds +12 actual Main/Test sources.
  Clean production `5b41be52` is user-attested; guard matches are not Git/history certification.

## Before/after actual boundaries and call chains

- Before: each subproject configuration installed external RPC → utils root request → patch substitution ([baseline root:3–10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build.gradle.kts#L3)).
  After: the entire global rule is gone ([fixed root:1–3](file:///tmp/kodex-direct-krpc-candidate-20261008/build.gradle.kts#L1)); no new routing registry/cache/resolver/wrapper.
- Contract Main core + own Test client/server/JSON: [before:19–26](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/build.gradle.kts#L19) → [four excluded declarations + patch API:19–34](file:///tmp/kodex-direct-krpc-candidate-20261008/rpc/spec/contract/build.gradle.kts#L19).
- Client Main core: [before:10–13](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/client/build.gradle.kts#L10) → [excluded core:10–12; existing contract API:15](file:///tmp/kodex-direct-krpc-candidate-20261008/rpc/impl/client/build.gradle.kts#L10).
  Main patch route is client → contract → patch; own Test → in-memory also retains patch.
- In-memory Main core/client/server/JSON: [before:11–14](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/in-memory/build.gradle.kts#L11) → [four excluded declarations + patch API:11–22](file:///tmp/kodex-direct-krpc-candidate-20261008/rpc/impl/in-memory/build.gradle.kts#L11).
- Exactly nine `org.jetbrains.kotlinx:kotlinx-rpc-utils` exclusions and two added commonMain API project edges.
  `.get().copy()` configures separate mutable dependencies, not shared catalog objects; `project(...)`/dependency-project access is ordinary Gradle/KGP API in the existing container, not a new facade.
- External route: client/server → krpc-core → core/serialization-core/logging → utils; JSON → serialization-core → utils.
  Main-only core exclusion cannot cover contract's own Test roots; exclusions must follow every real declaration, including Test producers.
- [Pinned core root metadata](https://repo.maven.apache.org/maven2/org/jetbrains/kotlinx/kotlinx-rpc-core/0.10.3/kotlinx-rpc-core-0.10.3.module) delegates target variants via `available-at`; inspected JVM/four Native core records request **root** utils.
  macosArm64 used cached publication metadata after GET TLS failure, not a successful fresh GET.
  [Gradle9.5.1 NodeState:646–670,786–803](https://github.com/gradle/gradle/blob/v9.5.1/platforms/software/dependency-management/src/main/java/org/gradle/api/internal/artifacts/ivyservice/resolveengine/graph/builder/NodeState.java#L646-L670) filters module IDs and propagates exclusions through delegated variants.
  Inference: root exclusions fit these paths; independent unexcluded/direct suffixed requests are not automatically covered.
- Real patch group/FQCNs, coroutines API/atomicfu and Native unique name `org.jetbrains.kotlinx:utils` unchanged ([patch:5–16](file:///tmp/kodex-direct-krpc-candidate-20261008/rpc/impl/krpc-utils-patch/build.gradle.kts#L5)).
  [CLI convention:9–12](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-cli.gradle.kts#L9) → [host targets:22–32](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt#L22) keeps JVM/linuxX64/linuxArm64/macosArm64/mingwX64; no target/profile/budget switch.
- Runtime chain stays precompiled kRPC map factory → [Native actual:10–13](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/src/nativeMain/kotlin/kotlinx/rpc/internal/utils/map/ConcurrentHashMap.native.kt#L10) → actual `SynchronizedHashMap`.
  [Collection copies:52–61](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/src/commonMain/kotlin/kotlinx/rpc/internal/utils/map/SynchronizedHashMap.kt#L52) stay inside the lock; `withKeys` runs under lock. No source implementation changed.
- **Scoped framework exception**, not a general reversal of [spec/impl direction:9–11](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/spec-impl-module-boundaries.md#L9):
  contract → vendored utils impl supplies actual upstream binary/API declarations, including public opt-in annotations.
  Inspected [pinned core source archive](file:///home/stream/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlinx/kotlinx-rpc-core/0.10.3/948f7e746444a7226e08cf3b0a8868853d34f34b/kotlinx-rpc-core-0.10.3-sources.jar), entry `commonMain/kotlinx/rpc/descriptor/RpcServiceDescriptor.kt:10–42`, publicly references `ExperimentalRpcApi`.
  Empty DDD interfaces cannot replace those symbols; no normative business contract/ViewModel moved.
- API exports intentionally change future dependency/publication metadata and expose existing utilities.
  Current delivery is CLI, not a promise to publish these Maven product libraries; `maven-publish` conventions do not make metadata unchanged.
  No Kotlin declaration, RPC wire/serialization shape or protobuf API renamed.

## Actual coordinator evidence and limits

- Following URIs identify remote **Xiaoxin static receipts**, not local files or reviewer-executed builds.
  [Owner-task adapter:13–50](file:///home/stream/ACodeSpace/demo/kodex-import-sources-repair-20261008/direct-routing-r03.init.gradle#L13) resolves each owner's configurations in its own task; root aggregates task dependencies.
- [Comparison:1–6](file:///home/stream/ACodeSpace/demo/kodex-import-sources-repair-20261008/direct-routing-comparison-r03.json#L1):17 owners /300 graphs, selected component IDs/variant names/attributes/capabilities equal; zero original root/suffixed utils components.
  Independently counted records and hashed both full graph JSONs: identical SHA256 `3bd38b259b7640876e277dbf3ddc7c70f89774275152934945208a40bb59f918`.
- Scope: JVM Main/Test **compile**, four Native Main/Test compile KLIBs, common/native Main/Test metadata,
  actual compiler-plugin configurations on three RPC producers and generic Native compiler classpath on all17.
  Not all runtime configurations, requested labels, artifact SHA/type-safe source/library symbols, full source navigation or all203 IDE modules.
- [Functional receipt:1–14](file:///home/stream/ACodeSpace/demo/kodex-import-sources-repair-20261008/direct-functional-receipt.json#L1): exit0; XML totals240 JVM +1 linuxX64, zero failures/errors.
  Static log confirms four requested ordinary test tasks executed without `FROM-CACHE`/`UP-TO-DATE`, plus actual contract/client/in-memory Linux compilations.
  Patch compilation/link and CLI JVM compilation reused cache; not proof all241 were freshly executed against freshly rebuilt/relinked artifacts.
- Control41.425s → direct15.358s are ordered cached graph probes, **not benchmark, memory gain or full GUI speed claim**.
  R01 wrong input/init path and R02 cross-project exclusive-lock failure are harness failures, not product defects; R03 fixes task ownership.

## B1 — confirmed blockers, not inferred overlay bugs

- No confirmed fixed-overlay defect found in inspected scope. **Adoption blocked**: successful complete r05 GUI/index/navigation and fresh required link/regression acceptance are not established here.
- Preserved [r04 outcome:143–155](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L143): actual OOM after approximately300.736s with earlier V2 helper + installation sources.
  That combination failed; two earlier no-OOM timeouts were not fixes. R04 does not show this different direct overlay fails or succeeds.

## B2 — specific conditional reproducers / minimum ablation

- No B2 defect reproduced against this overlay. Static regression control, **not executed**: restore unexcluded original utils while retaining patch API.
  Both supply identical Native unique name/JVM classes; duplicate-original graph detection must reject this before fresh link/GUI.
- Remove patch API instead and restore original utils: Native factory returns upstream live collections.
  [Pinned utils archive](file:///home/stream/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlinx/kotlinx-rpc-utils/0.10.3/c1a776a604be6d7efe272a162804fb9109f6fcbe/kotlinx-rpc-utils-0.10.3-sources.jar), entry `commonMain/kotlinx/rpc/internal/utils/map/SynchronizedHashMap.kt:49–56`, copies entries outside lock and returns live keys/values.
  [Existing snapshot reproducer:15–39](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/src/commonTest/kotlin/kotlinx/rpc/internal/utils/map/SynchronizedHashMapTest.kt#L15) captures accessors/iterators, mutates map, requires old snapshots: original implementation violates this contract.
  Deleting the actual utility replacement is not a legitimate simplification; preserve collection-iteration safety.
- Removing contract's Test exclusions leaves alternate original routes; removing patch API without original utils leaves required public utility symbols absent.
  These are isolated coordinator controls, not proposals to edit production or introduce switches.

## U — unresolved acceptance and current diagnostic

- Parent reports genuine `baseline-direct-patch-r05` uses original KGP source resolvers/defaults enabled, no helper/sources disable, old toolchain/full targets/4GiB ([parent:180–185](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L180)).
  [Static staging:28–45](file:///home/stream/ACodeSpace/demo/kodex-import-sources-repair-20261008/direct-gui-stage.py#L28) adds diagnostic ownership/heap-dump flags beside four changed builds; no private properties/dumps inspected.
- [Installation receipt](file:///home/stream/ACodeSpace/demo/kodex-import-sources-repair-20261008/installed-sources-receipt.json#L1): same-version private **bin** installation,683 original file hashes,10,942 official sources/210 roots; not an official `all` ZIP.
  Receipt facts, not a new reviewer installation verification or product adoption.
- Coordinator owns fresh required link/real RPC/serializer/Native cancellation tests, runtime/binary/source artifact coverage,
  type-safe source/library-symbol checks and full genuine GUI import/index/navigation at fixed budget.
  Preserve missing-variant/source failures; no target deletion, new profiles, resolver/cache replacements or toolchain/budget changes.
- Review complete independently, without waiting for heavy GUI. No resources held or temporary files created; fixed handoff evidence intact.
