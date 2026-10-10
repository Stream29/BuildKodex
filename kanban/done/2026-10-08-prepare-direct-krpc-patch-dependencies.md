# PREPARED READY v2 — preparation archived, runtime gates pending

# Task Tree

- `Load accepted-source constraints and prior kRPC audits`() // complete
- `Inspect real producer declarations, metadata and Main/Test closure`() // complete
- `Prepare minimal isolated dependency overlay with source guards`() // complete
- `Deliver exact edges, hashes and coordinator-owned gates`() // complete

# Details

## Scope and readiness

- **READY for the coordinator's static review and central graph gate only.**
  No present-source static counterexample found on the inspected RPC paths.
  This is not resolved-graph, binary ABI, source-navigation or OOM acceptance.
- Parent: [full-model import repair](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-repair-full-model-idea-import-oom.md#L1).
  User-attested clean production source remains
  `5b41be52d2f64a12ea3671bcecb9151fd2d1fa60`; no Git certification attempted.
- Prior [cost/GUI outcome](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-08-measure-fixed-model-build-cost-ablation.md#L64)
  and [17-consumer audit](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-08-recheck-project-local-krpc-cost-candidate.md#L48)
  loaded. Prior project-local substitution saved 62.006 MiB in its bounded
  configuration census but still failed genuine GUI import. It is not this fix.
- Owned output is only this new outer executable record and
  [isolated overlay](file:///tmp/kodex-direct-krpc-candidate-20261008/).
  All files created with `apply_patch`. No production changes, Gradle/build/test,
  IDE/device/process-control, Git, heavy-lock acquisition or network write.
  Read-only HTTPS GETs inspected public Maven metadata; no external bytes saved.
- Five build files plus two hash manifests; four files differ from production,
  the fifth is the unchanged patch build guard. This is an overlay, not a full
  runnable checkout. Apply only over the coordinator's isolated accepted-source
  copy, never over production or the earlier 17-plugin candidate.
- [Candidate hashes](file:///tmp/kodex-direct-krpc-candidate-20261008/SHA256SUMS#L1)
  cover all five build files. [Baseline guards](file:///tmp/kodex-direct-krpc-candidate-20261008/BASELINE.sha256#L1)
  cover original builds and all 12 real patch Main/Test source files.
  Those source files are reused from accepted source, not regenerated.

## Exact changes and producer edges

| Overlay file | Change |
| --- | --- |
| [root](file:///tmp/kodex-direct-krpc-candidate-20261008/build.gradle.kts#L1) | Remove the entire `subprojects/configurations.configureEach/dependencySubstitution` block; replace the now-obsolete rule comment with the direct-boundary explanation. |
| [contract](file:///tmp/kodex-direct-krpc-candidate-20261008/rpc/spec/contract/build.gradle.kts#L19) | Exclude utils on commonMain core and commonTest client/server/JSON; add commonMain patch API. |
| [client](file:///tmp/kodex-direct-krpc-candidate-20261008/rpc/impl/client/build.gradle.kts#L10) | Exclude utils on commonMain core; obtain patch API through existing `api(:rpc-spec-contract)`. |
| [in-memory](file:///tmp/kodex-direct-krpc-candidate-20261008/rpc/impl/in-memory/build.gradle.kts#L11) | Exclude utils on commonMain core/client/server/JSON; add commonMain patch API. |
| [patch](file:///tmp/kodex-direct-krpc-candidate-20261008/rpc/impl/krpc-utils-patch/build.gradle.kts#L1) | Byte-identical guard: Native module name, group, coroutines API and atomicfu implementation unchanged. |

- **Nine** explicit exclusions, each exactly
  `group = "org.jetbrains.kotlinx", module = "kotlinx-rpc-utils"`.
  Not `kotlinx-rpc-krpc-utils`, a group wildcard, a configuration-wide exclusion,
  or `isTransitive = false`.
- **Two** added commonMain API project edges:
  `:rpc-spec-contract -> :rpc-impl-krpc-utils-patch` and
  `:rpc-impl-in-memory -> :rpc-impl-krpc-utils-patch`.
  Client's existing API edge to contract supplies its patch; no third redundant edge.
- No source declarations, catalog pins, target/source-set/cinterop definitions,
  original base plugins, serializer/compiler plugins or wire protocol changed.
  No new plugin, helper, Provider or 17/203-project routing registry.
- Final declarations use `libs.…get().copy()` before configuration. Pinned
  KGP 2.4.0 `KotlinDependencyHandler` accepts a String or `T : Dependency`
  with a configure block, not a catalog Provider. Gradle 9.5.1's
  `DefaultMinimalDependency` rejects mutation; its `copy()` deliberately creates
  the mutable form. These nine local copies preserve the catalog coordinates/
  version constraints and avoid mutating shared catalog values.
- The first, uncompiled Provider-with-configure draft was a concrete static DSL
  counterexample and was corrected before final handoff. Its three producer
  hashes are superseded by the current `SHA256SUMS`; no build was attempted.
- Exclusions are carried on the real API/runtime library dependency boundaries,
  not separately repeated downstream. Contract's own Test library declarations
  also need exclusions: a Test dependency is not rescued by excluding Main core.

## Real metadata and exclusion semantics

- Inspected pinned 0.10.3 publication metadata, not the current local RPC fork
  (which is a different source/version line).
  [Core root metadata](https://repo.maven.apache.org/maven2/org/jetbrains/kotlinx/kotlinx-rpc-core/0.10.3/kotlinx-rpc-core-0.10.3.module)
  requests utils in common metadata. Its JVM API/runtime and all four existing
  Native API target records likewise request **root `kotlinx-rpc-utils`**, not
  suffixed utility target IDs.
- Roots delegate platform variants with `available-at` to `*-jvm`,
  `*-linuxx64`, `*-linuxarm64`, `*-macosarm64`, `*-mingwx64`.
  [Utils root](https://repo.maven.apache.org/maven2/org/jetbrains/kotlinx/kotlinx-rpc-utils/0.10.3/kotlinx-rpc-utils-0.10.3.module)
  delegates similarly. An exclusion can stop the utils root edge before
  following its utility-target redirect; excluding only a suffixed utility
  would leave common metadata and other platform routes intact.
- Cached Native client/server -> krpc-core/logging; krpc-core -> core,
  serialization-core/logging; JSON -> serialization-core.
  Logging and serialization-core also request utils by its **root ID**.
  Excluding only a direct core declaration would leave those alternate routes.
  Each of the nine real library declarations therefore carries the exclusion.
- Inspected Gradle 9.5.1's cached official `NodeState.java`:
  lines 505–515 filter outgoing edges, 646–670 check the edge's target module ID,
  786–803 propagate exclusion filters through external/delegated variants.
  Parser lines 227–229 represent `available-at` as an external dependency.
  [Pinned upstream NodeState](https://github.com/gradle/gradle/blob/v9.5.1/platforms/software/dependency-management/src/main/java/org/gradle/api/internal/artifacts/ivyservice/resolveengine/graph/builder/NodeState.java#L646-L670).
- Inference from those sources: root-ID exclusions fit the inspected published
  RPC paths, including the producer's initial target redirect. An unrelated
  unexcluded incoming path or direct `kotlinx-rpc-utils-jvm` request is not fixed
  by them; central all-component checking must detect such paths.
- Published attributes inspected:
  common = library / kotlin-metadata / platform common / non-jvm;
  JVM = library / java-api or java-runtime / jar / platform jvm / standard-jvm;
  Native = library / kotlin-api / platform native / non-jvm / exact native target.
  macosArm64 also publishes target kotlin-metadata; sources use documentation /
  sources and platform-specific runtime usage. Do not silently force one usage.
- Patch's unchanged `kodex.kmp-cli` -> `configureHostTargets()` supplies JVM,
  linuxX64, linuxArm64, macosArm64, mingwX64, with normal project variants.
  Existing generated patch PSM lists JVM API/runtime and those four Native API
  variants, plus commonMain/nativeMain/linuxMain with atomicfu and coroutines.
  This cached PSM corroborates structure but is **not** a fresh candidate
  attribute, artifact or task-edge receipt; no fresh model was acquired.
- The wider model keeps all other existing targets, including JS/Wasm/iOS
  elsewhere; no new patch platforms or profiles are invented to hide errors.

Pinned root metadata SHA-256 receipts (GET bytes, not resolved Gradle graphs):

| Module (org.jetbrains.kotlinx, 0.10.3) | SHA-256 |
| --- | --- |
| kotlinx-rpc-core | `219d3862625d95434d1770e7e2cee13077fa7317cb76db35317db619d83c2cef` |
| kotlinx-rpc-krpc-client | `90a06e91ed596307b8472ab98655b6033d5d7d49356d12f6648a0678509ac558` |
| kotlinx-rpc-krpc-server | `de200ec81d12552fe16cea015886132461b031b37dab68057e4b95046a41e433` |
| kotlinx-rpc-krpc-serialization-json | `3086605ec5d7a98ffecbdba71bcd29f7fffa3a7ffe32f0d93235ed6add984d21` |
| kotlinx-rpc-utils | `964d2187a3c5436a07a8fae202e5ff4b9ea60975e5e03982cf3c70be258178dd` |
| kotlinx-rpc-utils-jvm | `3f07596fbc6cfd9ff6d38f2163d8c05079f38b88a87651b7841bf01db5c9b85b` |

- A broader sequential metadata GET scan stopped on a TLS EOF after core's five
  targets and krpc-core root; missing reads were not counted as verified.
  Cached linuxArm64/macosArm64/mingwX64 RPC-chain records were read separately.
  All-platform central resolution remains mandatory.

## Independently recomputed Main/Test consumers

- Parsed explicit settings tree: 203 module scripts, 867 literal API/implementation
  project declarations across Main/Test, named and mosaic source sets; no
  nonliteral project edge found in that scan. Library aliases agree with the
  three roots; compiler-plugin aliases are not library roots.
- Conservative full-edge reverse closure is 17, independently matching the
  prior audit. Also recomputed own Main/runtime and own Test projections:
  dependencies contribute their **Main**, not their Test dependencies.
  Their union still yields 17. Main/Test unions across platforms are conservative
  declaration projections, not resolved per-platform classpaths.
- All 17 obtain a patched producer on the applicable Main or Test dependency
  route. Test-only consumers must not be reported as patched Main consumers.

| Project | Main reaches RPC root | Own Test reaches RPC root |
| --- | --- | --- |
| :rpc-spec-contract | yes | yes, including own external Test roots |
| :rpc-impl-client | yes | yes |
| :rpc-impl-in-memory | yes | yes |
| :rpc-impl-server | yes | yes |
| :app-impl-rpc | yes | yes |
| :app-test-support-rpc | yes | yes |
| :app-impl-session | yes | yes |
| :app-impl-application | yes | yes |
| :app-cli | yes | yes |
| :app-component-new-session-impl-viewmodel | yes | yes |
| :app-impl-view | no | yes |
| :app-component-agent-impl-view | no | yes |
| :app-component-session-sidebar-impl-view | no | yes |
| :app-component-settings-impl-view | no | yes |
| :app-component-settings-impl-viewmodel | no | yes |
| :hook-impl-notification | no | yes |
| :integration-test | no | yes |

- API availability is preserved through contract and in-memory's API exports.
  A consumer's implementation edge supplies producer API for its compilation
  and producer runtime transitives at runtime; it does not make its own API
  export all implementation dependencies. No claim of globally exposing utils.

## Public KMP/API and binary pitfalls

- Patch exports the **existing actual declarations**, not copied/new business
  contracts. Core's pinned source contains utility opt-in annotations on
  public descriptor operations/types (`ExperimentalRpcApi`, `InternalRpcApi`);
  its JVM API metadata explicitly depends on utils. Public krpc classes also
  reference the map/thread-local utilities and deferred helpers.
  Replacing the binary module therefore needs the same actual utility FQCNs.
- `api(project(...))` makes patch utility declarations visibly available at the
  original producer roots; it can change dependency/publication metadata and
  exposes the patch's real public utilities, but does not add Kotlin declarations.
  This is an intentional vendored binary-replacement edge from spec to the
  existing patch impl, not a new domain spec/impl split or an empty wrapper spec.
- Compared pinned utils source archives against all existing common/native files:
  only `SynchronizedHashMap` differs (already-accepted locked collection
  snapshots plus opt-in/header). JVM differences are existing private-class
  opt-in and deprecation suppression; JVM concurrent-map behavior remains real.
  Pinned root source archive SHA-256:
  `c4f27ee816d1d56a9df25731608ec8cc133136df3ec98207533509ead3d22147`.
- Native unique name stays **`org.jetbrains.kotlinx:utils`**,
  [patch build:8–10](file:///tmp/kodex-direct-krpc-candidate-20261008/rpc/impl/krpc-utils-patch/build.gradle.kts#L8-L10).
  Project ID/group is not a replacement for KLIB unique name. Binary KLIBs
  must link to those real symbols; identical FQCN alone cannot certify all
  compiled metadata/inline/expect-actual ABI.
- Never allow patch plus original root/suffixed utils artifacts on one Native
  or JVM library graph: duplicate KLIB identities or JVM classes are blockers.
  Gradle does not infer replacement/capability conflict from our project edge.
- Patch retains actual `api(coroutines.core)` and `implementation(atomicfu:0.29.0)`,
  Native map/worker actuals, JVM `java.util.concurrent.ConcurrentHashMap`, and
  its real snapshot test. Boundary cores already export coroutines; no coroutine
  or atomicfu transitive dependency is dropped and no owner/lifecycle code changes.

## Compiler classpath and coordinator gates

- Original global rule covered normal compiler-plugin configurations too.
  Main-library API project edges **do not feed** `kotlinCompilerPluginClasspath`.
  Cached actual RPC Gradle-plugin source requests CLI compiler artifact
  `kotlinx-rpc-compiler-plugin-cli:2.4.0-0.10.3`.
- Read all four pinned compiler `.module` records: CLI runtime -> k2/common/backend,
  k2/backend runtime -> common, common -> Kotlin stdlib; none requests utils.
  All four cached compiler jars also contain zero `.class` entries under
  `kotlinx/rpc/internal/utils/`. This is bounded archive/metadata evidence,
  not proof of actual full plugin classpaths or relocated embedded classes.
  Compare actual compiler classpaths (including
  Native variants and other plugins) across all applicable projects.
- If actual compiler graph reaches utils under the original rule, report its
  exact requested/selected/artifact route as a counterexample before heavy tests;
  this overlay does not claim to patch compiler classpaths via Main dependencies.
- **Central gate first**, owned by coordinator under its existing heavy lock:
  compare 216 projects / 1344 targets / 5026 source sets, unchanged hierarchy,
  targets/cinterop and baseline pins. Configuration/task-edge differences are
  observations to explain, not grounds to delete model elements.
- Compare **all** Main/Test compile/runtime/common metadata/platform components,
  including omitted projects/external transitives, compiler classpaths and
  actual source attachments. The old 68 JVM/Linux component-ID inventories
  are insufficient; selected variants, attributes, artifacts, dependency edges
  and preparation task dependencies must be captured.
- Expected delta: two explicit project edges plus nine propagated exclusions,
  original utility module request removed rather than a substitution selection
  reason. Compare semantic patch artifact/symbol coverage, not demand literal
  dependency-edge equality that the candidate intentionally changes.
- Preserve original real dependency variant/failure records. Optional shared
  Native cinterop views may legitimately retain NoMatching/
  NoCompatibleVariants/ModuleVersionResolveException records. Record owner,
  requested attrs, candidate variants and outcome before/after; do not suppress
  them globally or treat every Native view failure as a broken required binary.
- Require no original utils root/target library artifacts on every applicable
  consumer graph, same real patch code/platforms, no lost non-utils transitives,
  equivalent sources/navigation, and valid compiler artifacts before proceeding.
- Only then: coordinator's actual JVM/RPC/serializer tests, Native patch/link
  regressions and genuine full GUI import/index/navigation at fixed 4 GiB.
  Preserve expected failures and reject any required lost variant/source/ABI.
- **No candidate memory/time/GUI benefit measured or claimed.** Removing
  module-to-project substitution avoids that rule's early task-discovery
  trigger in principle; explicit IDE resolution still occurs. A graph-correct
  candidate that still OOMs is a failed remedy, not an accepted repair.
- Preparation finishes independently; no resources remain held. Candidate/hash
  files are retained as the requested handoff, not abandoned temporary scratch.
