# Task Tree

- `Read fixed-model constraints and sealed evidence`() // complete
- `Trace KGP and Gradle acquisition and retention`() // complete
- `Separate confirmed mechanisms from unproved attribution`() // complete
- `Deliver bounded candidates and falsifiable experiments`() // complete

# Details

- **STATICS READY / RUNTIME BENEFIT PENDING.** This read-only investigation is
  complete independently of the main Session's serial cost census. No runtime
  experiment described below was executed or queued.
- Only this new outer report is owned. Production remains the user-attested
  Kodex `5b41be52d2f64a12ea3671bcecb9151fd2d1fa60`; adoption of the
  KGP 2.4.20 / Gradle 9.6.1 candidate remains paused.
- Fixed model: 216 registered projects, 203 KMP modules, 1,344 target instances
  including metadata, 5,026 source sets, unchanged hierarchy, cinterop and API.
  No target/profile/module removal, commonization bypass, ignored variant
  errors, replacement cache/model controller/provider or resource enlargement.
- Read-only SSH inspected existing report ZIP entries and cached KGP sources.
  No HPROF/index read, MAT/OQL/heap query, Gradle, build, test, IDE, JVM/process
  control, lock acquisition, installation, Git operation or network write.
  Private report output was restricted to class/count/size/reference structure;
  no raw messages, object strings, credentials or system properties were returned.
- Context:
  [compatibility/heap gate, lines 245–359](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-08-validate-kodex-toolchain-compatibility.md#L245-L359),
  [fixed-model census, lines 14–39](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-08-measure-fixed-model-build-cost-ablation.md#L14-L39),
  [active resource task](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-02-plan-gradle-model-resource-optimization.md#L1).

## Bottom line

- **Confirmed:** the OOM snapshot contains large *live retained resolution and
  metadata graphs*, not merely expensive configuration object headers.
  Millions of diagnostic candidate/attribute records occur inside the two
  largest configuration-related retained sets.
- **Confirmed in source:** lenient artifact views tolerate failures **after**
  graph selection has constructed diagnostics. Lenient does not mean
  allocation-free selection or disposal of failed graph results.
- **Confirmed in source:** KGP acquires dependencies per source set, stores
  metadata transformations there, and performs both variant-based source
  acquisition and a default-enabled legacy source-query fallback.
- **Confirmed in source:** the broad project substitution enables full graph
  resolution when affected configurations' task dependencies are visited.
  It does not resolve every configuration at registration time. IDEA/KGP also
  explicitly resolves graphs; therefore removing the rule is not a demonstrated
  OOM fix.
- **Unknown:** the fraction of 17,656 exceptions attributable to commonizer,
  source acquisition, or other resolutions; their complete individual inbound
  paths; and any candidate's same-model runtime benefit. No verified supported
  upstream version fix for this particular retention problem was found.

## Existing MAT evidence: mass, not module “weight”

- All following remote URIs refer to **Xiaoxin**, not this host. Report bytes
  remain private; links identify sealed evidence, not permission to upload it.
- [Sealed MAT report archive](file:///home/stream/ACodeSpace/demo/kodex-gradle-rollout-20261007/idea-current-runs/candidate-private-heap-r01/private/gradle-oom_Leak_Suspects.zip):
  `pages/Top_Consumers6.html`, `pages/Class_Histogram7.html`,
  `pages/20.html`, `pages/40.html`, `pages/60.html`.
- The top-consumer **dominator partition** reports:

  | Group | Group objects | Shallow bytes | Retained bytes |
  | --- | ---: | ---: | ---: |
  | `DefaultLegacyConfiguration_Decorated` | 18,902 | 5,443,776 | 1,523,953,384 |
  | `DefaultConfigurationContainer_Decorated` | 217 | 29,512 | 672,297,544 |
  | `DefaultKotlinSourceSet_Decorated` | 5,026 | 522,704 | 501,625,528 |

- `pages/20.html`'s accumulated-object histogram contains **7,059,955
  `AssessedAttribute` / 169,438,920 shallow bytes** and **792,511
  `AssessedCandidate` / 31,700,440 shallow bytes**. `pages/40.html` contains
  another **3,219,497 / 77,267,928 bytes** and **361,382 / 14,455,280 bytes**,
  respectively. These record headers alone total 292,862,568 bytes; referenced
  lists, arrays, attributes, strings and other graph structures add further
  cost. **The whole 2.196 GB configuration-related partition is not thereby
  proved to be diagnostic-only or recoverable.**
- The global histogram's 27,858 count is specifically the **legacy configuration
  class**, not all Gradle configuration roles. Its shallow total is 8,023,104
  bytes. The 17,656 variant-selection exceptions themselves total only 847,488
  shallow bytes; counting just exception headers misses their retained failure
  payloads. KGP's `createResolvable` helper still calls legacy `create`, so the
  class name does not diagnose deprecated/broken dependency semantics. [K8]
- `pages/60.html` also places 26,348 `KotlinProjectStructureMetadata`, 26,348
  `ChooseVisibleSourceSets`, and 74,248 each of `DefaultResolvedArtifactResult`
  and `DefaultResolvedVariantResult` in the source-set retained set.
- A grouped retained set is **not** “one heavy module” or a per-module weight
  ranking. Group counts differ from the global histogram; shared
  objects can be dominated by a container rather than one configuration.
  The source-set group and configuration partition cannot be added again to
  their descendant object sizes. Suspect-detail container retention is
  672,298,168 bytes, 624 bytes above the top partition: use the top partition
  when reporting the existing 64.44%, not mixed query boundaries.

### Live owner paths actually available

- The three suspect-detail reports include merged root-reference trees through
  `java.lang.Thread`, `DefaultGradle_Decorated`,
  `DefaultTaskExecutionGraph.allTasks`, `RegularImmutableList.array`, and
  scheduled tasks. Task classes include `DefaultTask`, cinterop metadata
  transformation/commonizer tasks, `GenerateProjectStructureMetadata`, and
  `ExportKotlinProjectDataTask`.
- Additional branches include `DefaultProjectRegistry.projects`,
  `DefaultProject_Decorated` and configuration containers. The source-set tree
  additionally includes tooling `KotlinExtensionReflection`,
  `sourceSets$delegate` / `SynchronizedLazyImpl`, the multiplatform extension
  and `sourceSetsContainer`; Native compilation reflection is also present.
- Thus the graphs remain reachable from the **ongoing task graph/project/tooling
  invocation**, not merely an abandoned allocation stack. The thread's own
  retained size, 225,252,904 bytes, need not dominate everything it references:
  alternative roots/shared references explain that distinction.
- These are merged paths to the suspect sets, **not a saved field-by-field
  inbound query for every exception**. Connecting a particular exception to a
  particular source-set configuration remains unproved. No new heap query was
  used to fill that gap.
- [Existing failure-type query](file:///home/stream/ACodeSpace/demo/kodex-gradle-rollout-20261007/idea-current-runs/candidate-private-heap-r01/private/gradle-oom_Query.zip)
  contains 50 `NoCompatibleVariantsFailure` records. It is a limited sample,
  not a classification of all 17,656 failures or their byte ownership.

## Actual source trace

### 1. Commonized cinterop and lenient diagnostics

- KGP names a configuration `<sourceSet.name>CInterop`, reuses an existing one
  by name, extends that source set's resolvable metadata configuration, and
  requests commonizer target / `kotlin-commonized-cinterop` / library / klib
  attributes. Only a shared commonizer target gets this view. Missing
  corresponding dependency elements are explicitly expected; the returned
  view is lenient. **Repeated calls for one source set do not inherently create
  a new named configuration each time.** [K1]
- The concrete IDE path is
  `IdeCInteropMetadataDependencyClasspathResolver.resolve`
  → `createCInteropMetadataDependencyClasspathForIde`
  → transformation outputs + that commonized view
  → `resolveCInteropDependencies`, which iterates its files. The same resolver
  supplies shared-source-set file collections as import task dependencies.
  Separately, commonizer-task dependency construction also calls the view.
  This is not an inference from a histogram class name. [K2]
- Gradle graph selection first calls `selectByAttributeMatchingLenient`; if it
  returns null, the ordinary `selectByAttributeMatching` path invokes
  `noCompatibleVariantsFailure`. The handler **eagerly** assesses all candidate
  variants; the assessor creates four classified attribute lists per candidate.
  The describer builds a message and an exception retaining the failure data.
  Artifact-view leniency is a different layer from the selector method bearing
  “Lenient” in its name. [G1] [G2] [G3]
- A resolved configuration keeps `currentResolveState → ResolverResults`.
  Results keep graph/artifact state; graph results keep unresolved dependencies
  and a graph factory whose failure list survives graph serialization.
  An evaluated `DefaultArtifactCollection` also keeps artifact results and
  failures **even when lenient**. A lenient file collection suppresses reporting/
  rethrowing, not initial graph diagnostic construction. These are source-level
  retention routes, not newly observed individual heap edges. Once fully
  resolved, the same configuration reuses its graph; this does not prove repeated
  graph construction on each file access. [G4] [G5] [G6]

### 2. Metadata and source acquisition during IDEA import

- Applicable dependency resolvers run through `resolveDependencies(sourceSet)`.
  `metadataTransformation` is stored in source-set extras; its
  `metadataDependencyResolutions` and visible-source-set map are lazy **stored
  results**, not transient outputs discarded after serialization.
  `IdeVisibleMultiplatformSourceDependencyResolver` and original/transformed
  metadata resolvers force those results. This agrees with the existing
  source-set retained histogram, without proving every stored item redundant.
  [Import entrypoint][K3],
  [stored transformation](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/sources/KotlinSourceSetMetadataTransformation.kt#L17-L33),
  [stored results](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/GranularMetadataTransformation.kt#L217-L223).
- For binaries, the resolver reads `artifacts.failures` and `artifacts.artifacts`
  from a lenient view; platform-like source sets can require detached
  configurations. Explicit access resolves regardless of task-graph substitution
  flags. [K4]
- `IdeSourcesVariantsResolver` groups the source set's binary coordinates,
  creates a **new detached configuration**, applies source-publication
  attributes and iterates lenient artifacts. It does this per invocation/
  source set, not once for the entire build. The detached configuration has its
  own graph; “lenient sources” is not Gradle variant reselection over an existing
  graph. Gradle's detached factory returns the object without registering it in
  the normal container, so the root `configureEach` callback cannot be assumed
  to apply to these detached source configurations. [K5] [G7]
- The factory also registers `IdeArtifactResolutionQuerySourcesResolver`,
  controlled by `kotlin.mpp.import.enableSlowSourcesJarResolver`, default **true**.
  It gets all components from a selected compilation/metadata graph, executes a
  JVM sources query, and attaches matching results. KGP keeps this fallback for
  libraries published before KMP sources variants were available. Both source
  resolvers run at the same normal priority. Thus repeated acquisition has a
  concrete source mechanism; actual invocation multiplicity/retained cost in
  this failed run has not been measured. [K6]

### 3. Broad project substitution and early graph resolution

- Actual root code is **`subprojects`**, not `allprojects`:
  [root lines 3–9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build.gradle.kts#L3-L9).
  Each realized normal configuration gets module
  `org.jetbrains.kotlinx:kotlinx-rpc-utils` → project
  `:rpc-impl-krpc-utils-patch`. Lazy registration is not itself a resolution.
- In `DefaultDependencySubstitutions.using`, a project on either side sets
  `rulesMayAddProjectDependency=true`. `DefaultResolutionStrategy` then returns
  true from `resolveGraphToDetermineTaskDependencies`. When
  `DefaultConfiguration`'s task-dependency result is requested, it chooses full
  `getValue()/resolveGraphIfRequired()` instead of build-dependency-only
  resolution. This flag does not require the requested graph to actually contain
  utils before taking that branch. The branch exists in the original Gradle
  9.5.1 as well as candidate 9.6.1. [G8] [G9]
- KGP attaches resolver build dependencies to `prepareKotlinIdeaImport`, including
  the cinterop file collections and shared-source-set PSM views. The saved task
  graph paths make this a credible **early-acquisition amplifier**, but do not
  prove exactly which extra graphs the root rule resolved in this import. [K7]
- This is graph resolution/configuration observation under the project model
  lock, **not automatic activation of dependency lockfiles**.
  `activateDependencyLocking` is a separate operation. Later explicit IDE
  resolution still occurs without the root flag; roots already held by both
  paths may change only timing, not eventual retention. [G4] [G8]
- **No-root-substitution changes patch responsibility.** If its graph reaches
  utils it can consume the upstream implementation instead of the Native patch.
  Static declared kRPC consumers include contract/in-memory/client, but
  downstream transitives and test/runtime/metadata graphs also matter.
  Applying a convention only where the RPC plugin is declared is insufficient.
  Preserve the patch's [Native unique-name contract, lines 8–10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/build.gradle.kts#L8-L10).
- The main no-resolution census can measure strategy/action registration cost.
  It cannot prove the early-resolution branch's cost, graph equivalence, patch
  routing or Sync benefit. An unchanged census also cannot falsify a
  resolution-only improvement.

## Upstream fix/version assessment

- [KT-87790](https://youtrack.jetbrains.com/projects/KT/issues/KT-87790/Kotlin-2.4.0-increases-Gradle-sync-times)
  is a concrete KGP 2.4.0 Sync regression fixed in the 2.4.20 line.
  [PR 6887](https://github.com/JetBrains/kotlin/pull/6887/files) changes
  source-set visibility/host-specific computation reuse. The candidate sources
  already contain that task-execution cache; its MAT top-consumer entry is
  82,859,184 bytes / 1.98%, not the dominant configuration partition.
  It is not a missing fix to reapply, nor proof this OOM is repaired.
- [Gradle 35318](https://github.com/gradle/gradle/issues/35318) addresses typed
  diagnostic attributes during configuration-cache deserialization;
  [36284](https://github.com/gradle/gradle/issues/36284) addresses failure-message
  access. They are not demonstrated retained-memory fixes for this Sync.
- Gradle 9.7.0's [failure handler](https://github.com/gradle/gradle/blob/v9.7.0/platforms/software/dependency-management/src/main/java/org/gradle/internal/component/resolution/failure/ResolutionFailureHandler.java)
  still assesses candidates, and its [exception base](https://github.com/gradle/gradle/blob/v9.7.0/platforms/software/dependency-management/src/main/java/org/gradle/internal/component/resolution/failure/exception/AbstractResolutionFailureException.java)
  still retains failure data. No “upgrade and the payload disappears” claim is
  supported. This limited source/issue search is not proof that no relevant
  upstream issue exists.

## Minimal candidates and exact future proof

- **Smallest ready-to-express control:** in an isolated copy only, set
  `kotlin.mpp.import.enableSlowSourcesJarResolver=false`. Keep variant-based
  sources acquisition, binary resolution, all targets and commonization.
  This tests only redundant **legacy** acquisition, not the dominant commonizer
  diagnostic mechanism. Source attachment equivalence is a required gate:
  dependencies still needing the fallback make global adoption unacceptable.
  The flag's existence/default is confirmed; its effective value in the sealed
  invocation was not audited through private property dumps. If the frozen
  baseline already disables it, reject this control as a no-op before another
  GUI run. This is an upstream implementation switch, not a demonstrated
  public support guarantee or safe production default. [K6]
- **Smallest own-build refactor candidate:** move the existing exact utils rule
  to project-local convention consumers determined from the complete resolved
  reachability matrix. Do not replace it with direct API dependencies, drop
  transitives, or remove it globally. The complete consumer/configuration matrix
  is not established by the static scan, so a production-ready placement list
  cannot honestly be supplied yet.
- **Small correctness-preserving upstream cleanup, not an OOM solution:** an
  early return in either sources resolver when its binary request collection
  is empty avoids a query that cannot attach anything. It needs upstream review;
  the eligible invocation count and savings are unknown. No KGP rebuild/fork was
  created, and tiny empty-request savings must not be sold as multi-GB benefit.

| Future experiment, not run | One changed factor | Evidence required / falsification |
| --- | --- | --- |
| Existing main configuration census | Broad substitution registration absent in diagnostic copy | Construction counts/heap only. Never accept its dependency semantics or use it as a Sync benchmark. |
| Legacy source-query A/B | Only the KGP fallback flag | Same full genuine import; compare source artifact/root sets and navigation. Missing sources rejects adoption; unchanged pressure/outcome rejects an OOM-benefit claim. |
| Local-substitution A/B | Only exact rule placement | Every graph that selects utils must still select the patch project/expected Native artifacts; graph/task edges equal. Compare earlier resolved graphs and retained diagnostics where separately authorized, not just callback counts. |
| Joint confirmation, only after individual benefit | Combine surviving factors | Genuine initial import plus three warm imports, indexing/smart and platform navigation; no regression in compilation, DI/test behaviour, RPC/Native patch semantics or publication. |

- Runtime comparisons must use the **same** source/toolchain line within each
  pair: accepted 5b41be52 for the main census; the frozen failed
  2.4.20/9.6.1 candidate for candidate importer diagnosis. Never compare an
  accepted old-tool census against a new-tool GUI as a one-factor result.
- Hold IDEA 3 GiB / Gradle 4 GiB / Kotlin 2 GiB, workers 1, installed IDE/JDK,
  compiler mode, source/model inputs and cache seed constant. Use separately
  cloned identical seeds and fresh private project/IDE state for initial-import
  comparisons; do not compare the earlier changed-seed timings causally.
  Separate cold preparation, initial import and warm samples.
- Collect actual import success/failure and native-import/smart wall time,
  GC/RSS and model/artifact equality. A CLI model can remain equal while full
  import fails: registration equality says nothing about acquired resolution
  payloads. The existing no-probe import failed after 1,068.261 s; it proves
  the extra model probe was unnecessary, not a module-specific allocation cause.
- For routing equivalence, compare the structured tuple
  `(consumer project, configuration/source set, requested coordinate,
  selected component/variant, artifact identity, task dependency)` for every
  graph reaching utils, across main/test compile/runtime/metadata and the
  existing platforms. For cost attribution, distinguish import-task dependency
  preparation from subsequent binary/metadata and source acquisition using
  existing lifecycle/build-operation evidence; do not add a probe that resolves
  extra configurations. Preserve expected optional cinterop failures, and
  compare their diagnostic class/count/retained bytes only if separately
  authorized. No complete tuple census exists in this investigation.
- Any additional heap evidence requires separate authorization, the main
  resource owner releasing the heavy lock, and class/count/structural summaries
  at comparable lifecycle points. Do not launch a concurrent query/import or
  wait for the census to mark this static investigation complete. A lower object
  count without lower retained pressure or a passing same-budget import is not
  the requested benefit.

## Large-cost / no-benefit exclusions

- Do not repeat unchanged 18-minute OOM imports, raise memory, change targets,
  disable commonization/native, or globally hide variant failures.
- Do not infer “broken library” from commonizer's expected optional elements,
  republish ordinary dependencies, or use allocation stacks as retained owners.
- Do not turn off all source downloads: KGP labels its general
  `kotlin.mpp.idea.gradle.download.sources.enabled` gate internal/testing and
  warns about metadata-source lazy support. It also changes navigation
  behaviour, unlike proving a redundant fallback unnecessary. [K3]
- Do not blindly replace commonizer configurations or sources graphs with
  `withVariantReselection()`. The Gradle selector can avoid a no-match exception
  on this path, but reselection does **not** traverse dependencies or
  `available-at` edges of the reselected variant. Same target counts do not
  establish equivalent cinterop/source artifact closure. This is a larger
  upstream contract change, not a safe one-line fix. [G10]
  The candidate-tag [no-match reselection branch](https://github.com/gradle/gradle/blob/v9.6.1/platforms/software/dependency-management/src/main/java/org/gradle/api/internal/artifacts/ivyservice/resolveengine/artifact/VariantResolvingArtifactSet.java#L139-L155)
  confirms the allocation distinction; the current API guide explains the
  graph-closure limitation, not a version-upgrade recommendation.
- Do not migrate buildSrc, enable IP, rewrite providers, or add diagnostic
  deduplication caches/model controllers to address this snapshot. Those have
  larger compatibility/invalidation responsibilities and no demonstrated
  benefit for the retained failure records. Existing native configuration/task
  result ownership must be understood before adding another owner.
- Removing publishing/DI/test attachments remains the main's diagnostic cost
  ablation, not a permitted production deletion or this report's OOM fix.

## Primary-source index

- KGP references below were read from the candidate's
  [cached `gradle96` source archive](file:///home/stream/ACodeSpace/demo/kodex-gradle-rollout-20261007/idea-current-runs/candidate-private-heap-r01/gradle-home/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-gradle-plugin/2.4.20/e8842d20bf1382f835652944a7fd2f21083a9b7e/kotlin-gradle-plugin-2.4.20-gradle96-sources.jar).
  Entry line ranges are given in the links to corresponding upstream sources.
  Gradle links pin the candidate tag, except explicitly identified original/
  follow-up versions. No private values were used as public citations.

[K1]: https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/targets/native/internal/CInteropCommonizerConfigurations.kt#L81-L120
[K2]: https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeCInteropMetadataDependencyClasspathResolver.kt#L20-L36
[K3]: https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImportImpl.kt#L32-L101
[K4]: https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeBinaryDependencyResolver.kt#L140-L341
[K5]: https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeSourcesVariantsResolver.kt#L24-L65
[K6]: https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImportFactory.kt#L135-L149
[K7]: https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/IdeMultiplatformImportImpl.kt#L103-L118
[K8]: https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/utils/configurations.kt#L28-L62
[G1]: https://github.com/gradle/gradle/blob/v9.6.1/platforms/software/dependency-management/src/main/java/org/gradle/internal/component/model/GraphVariantSelector.java#L80-L103
[G2]: https://github.com/gradle/gradle/blob/v9.6.1/platforms/software/dependency-management/src/main/java/org/gradle/internal/component/resolution/failure/ResolutionFailureHandler.java#L184-L195
[G3]: https://github.com/gradle/gradle/blob/v9.6.1/platforms/software/dependency-management/src/main/java/org/gradle/internal/component/resolution/failure/ResolutionCandidateAssessor.java#L85-L173
[G4]: https://github.com/gradle/gradle/blob/v9.6.1/platforms/software/dependency-management/src/main/java/org/gradle/api/internal/artifacts/configurations/DefaultConfiguration.java#L611-L689
[G5]: https://github.com/gradle/gradle/blob/v9.6.1/platforms/software/dependency-management/src/main/java/org/gradle/api/internal/artifacts/configurations/DefaultArtifactCollection.java#L59-L70
[G6]: https://github.com/gradle/gradle/blob/v9.6.1/platforms/software/dependency-management/src/main/java/org/gradle/api/internal/artifacts/configurations/ResolutionBackedFileCollection.java#L56-L87
[G7]: https://github.com/gradle/gradle/blob/v9.6.1/platforms/software/dependency-management/src/main/java/org/gradle/api/internal/artifacts/configurations/DefaultConfigurationContainer.java#L180-L205
[G8]: https://github.com/gradle/gradle/blob/v9.6.1/platforms/software/dependency-management/src/main/java/org/gradle/api/internal/artifacts/ivyservice/resolutionstrategy/DefaultResolutionStrategy.java#L255-L261
[G9]: https://github.com/gradle/gradle/blob/v9.6.1/platforms/software/dependency-management/src/main/java/org/gradle/api/internal/artifacts/ivyservice/dependencysubstitution/DefaultDependencySubstitutions.java#L260-L279
[G10]: https://docs.gradle.org/current/userguide/artifact_views.html#sec:artifact-views-variant-reselection

- Additional exact trace locations:
  [cinterop classpath, lines 23–45](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/targets/native/internal/CInteropMetadataDependencyClasspath.kt#L23-L45);
  [cinterop file iteration, lines 17–24](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/resolveCinteropDependency.kt#L17-L24);
  [commonizer view caller, lines 184–190](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/targets/native/internal/CInteropCommonizerTask.kt#L184-L190).
- Metadata storage:
  [source-set extras, lines 17–33](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/sources/KotlinSourceSetMetadataTransformation.kt#L17-L33);
  [transformation results, lines 217–223](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/mpp/GranularMetadataTransformation.kt#L217-L223);
  [visible project sources, lines 26–67](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeVisibleMultiplatformSourceDependencyResolver.kt#L26-L67);
  [lazy artifact/failure handles, lines 27–60](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/utils/LazyResolvedConfigurationWithArtifacts.kt#L27-L60).
- Legacy source query:
  [reason and acquisition, lines 25–62](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/ide/dependencyResolvers/IdeArtifactResolutionQuerySourcesResolver.kt#L25-L62);
  [default true, lines 399–400](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/PropertiesProvider.kt#L399-L400);
  [property name, line 799](https://github.com/JetBrains/kotlin/blob/v2.4.20/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/PropertiesProvider.kt#L799).
- Gradle diagnostic ownership:
  [describer, lines 56–61](https://github.com/gradle/gradle/blob/v9.6.1/platforms/software/dependency-management/src/main/java/org/gradle/internal/component/resolution/failure/describer/NoCompatibleVariantsFailureDescriber.java#L56-L61);
  [exception failure field, lines 55–65](https://github.com/gradle/gradle/blob/v9.6.1/platforms/software/dependency-management/src/main/java/org/gradle/internal/component/resolution/failure/exception/AbstractResolutionFailureException.java#L55-L65);
  [graph result fields, lines 38–57](https://github.com/gradle/gradle/blob/v9.6.1/platforms/software/dependency-management/src/main/java/org/gradle/api/internal/artifacts/ivyservice/resolveengine/graph/results/DefaultVisitedGraphResults.java#L38-L57);
  [graph factory failure list, lines 359–387](https://github.com/gradle/gradle/blob/v9.6.1/platforms/software/dependency-management/src/main/java/org/gradle/api/internal/artifacts/ivyservice/resolveengine/result/StreamingResolutionResultBuilder.java#L359-L387).
- Original version:
  [Gradle 9.5.1 strategy](https://github.com/gradle/gradle/blob/v9.5.1/platforms/software/dependency-management/src/main/java/org/gradle/api/internal/artifacts/ivyservice/resolutionstrategy/DefaultResolutionStrategy.java);
  [Gradle 9.5.1 configuration task-dependency provider](https://github.com/gradle/gradle/blob/v9.5.1/platforms/software/dependency-management/src/main/java/org/gradle/api/internal/artifacts/configurations/DefaultConfiguration.java).
