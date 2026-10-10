# Task Tree

- `Reject metadata-reselection shortcuts with actual project fixtures`() // complete
- `Compare original and deferred-copy file/task selection`() // 24 real groups + cold fixtures pass
- `Measure graph retention with unchanged selected dependencies`() // repeated headless controls pass
- `Build the typed IDE-only upstream source experiment and regressions`() // full V3 build +19 tests pass
- `Check complete model, representative attachments, functions and real GUI`() // combination passes; Json navigation remains open
- `Require independent review of source/build regressions`() // complete; no universal copy-equivalence claim
- **`Retain isolated evidence without adopting KGP modifications`()** // suspended; user rejects maintenance

# Details

- Latest decision: KGP modification/maintenance is rejected. Preserve this
  experiment and its regressions as evidence only; do not publish or select
  the source-built artifact for production. The official-toolchain OOM gate
  remains open in the parent task.
- Parent: [full import repair](2026-10-08-repair-full-model-idea-import-oom.md).
- Continues the user's isolated full KGP source/build experiment. No remote
  fork, package publishing, production switch or target changes. Production
  remains clean `5b41be52`; complete model and Gradle4GiB stay unchanged.
- Private root: `~/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009`
  on Xiaoxin. Builds/GUI/MAT are serialized under the existing heavy lock.
- Metadata variant reselection is rejected:24 real comparisons pass but
  genuine project fixtures leak a mac-only API dependency, then a Native
  implementation dependency despite metadata-visibility filtering.
- Deferred copies of the original declaration pass both counterexamples.
  Cold fixture task dependencies discover the actual artifact producers
  without explicit producer dependencies, before files are read. Fixture
  payloads prove selection/task ordering, not compiled Native symbols.
- Actual deferred-copy comparison:24 groups' canonical files and task paths
  match the original. Its initial receipt accidentally read the older result;
  separately checked the actual new JSON rather than relying on that receipt.
- Headless original R02/R03 and detached R01/R02 each query406 actual
  Main/Test configuration graphs. Original post-GC heap increments are
  333,587,264 /333,441,640 bytes; detached increments are
  61,505,208 /61,464,992 bytes. Selected identities, failure counts and failure
  types match. All406 weakly observed copies are collectable.
  Explicit GC is a diagnostic, not GUI timing or a complete-model peak saving.
- Original R01 fails before configuration because of an incorrectly named
  uploaded init script; preserve that failure. Successful controls use the
  correct unchanged script and fresh owned JVMs.
- Source candidate uses the original KGP CInterop boundary only for IDE
  queries. Regular build resolution is unchanged. V3 returns plain selected
  files and delegates work nodes without keeping detached configurations or
  artifact views.
  Dependency-locking configurations fall back to the original named view.
- Copy identity differs: custom name-sensitive resolution callbacks and
  dependency mutations remain an explicit review/adoption gate. This is an
  isolated candidate, not a claim of universal Gradle configuration equivalence.
- Preserve earlier parser fix, all targets/source resolvers, ABI, cinterop and
  tests. Full official upstream build is mandatory; no partial or edited JAR.
  Use a separate private `cinterop-maven-local` so prior control/patch artifacts
  and their receipts remain immutable. Private heaps/credentials never exported.
- Full-source R01 compiles the implementation, then rejects the new test's
  deprecated `tasks.create` under the unchanged Werror (207.752s). Preserve
  this failure. V2 changes only that test call to `tasks.register(...).get()`;
  both implementation hashes stay unchanged. No warning suppression or budget
  change. Original frozen review input remains intact for the reviewer.
- V2 full build succeeds in85.001s with16 tests and a sealed private artifact.
  Independent [review](2026-10-10-review-kgp-cinterop-graph-lifetime.md) identifies
  lost transform/work-node dependencies in the public Tasks-only projection.
  Actual cold collection-directory fixture reproduces empty output /failure;
  original named-view control succeeds. This candidate is not used for GUI.
- V3 visits the original work-dependency graph directly, without a TaskSet
  projection or a stored copy/view. The same cold transformed fixture succeeds
  in8.551s. It also runs original declaration actions before copying and
  rechecks late locking/direct consistency per operation. Three added focused
  tests supplement the original three; complete source build remains mandatory.
  Arbitrary name-sensitive incoming listeners are still an adoption gate.
- V3 complete official build succeeds in221.804s with19 tests, all failures /
  errors /skips zero; user MavenLocal remains absent. Sealed Gradle813 SHA256
  `348fc6b9b2b3c6d4be73b388e652faa04062b3696c8ee1ef3b912b7791f40683`.
  Prior parser and V2 artifacts/publications are retained separately.
- Real consumer model gate succeeds in101.181s with216 projects /1,344
  targets /5,026 source sets and complete equivalence after the sole diagnostic
  task exclusion. Exact selected plugin hash and actual loaded helper verified.
- Real source importer succeeds in18.594s. All four existing JVM/common/
  Native/Linux groups' full binary records and source archive SHA/entries match
  the previous aligned parser-patch result. Dedicated Native cache is cloned
  instead of continuing to use the GUI seed as a writable experiment home.
- Independent [V3 review](2026-10-10-recheck-cinterop-transform-and-declaration-actions.md)
  finds no new static blocker in the requested delta; withholds production
  adoption and universal copy-equivalence claims. Full GUI /product tests /
  navigation remain unverified. Real GUI R16 starts with the original4GiB,
  complete project, same R09 seed and unchanged safety thresholds.
- R16 stops at the unchanged host-memory guard, without captured heap OOM
  or completed import. Two external resolves start69ms apart against the same
  project; two owned4GiB Gradle daemons have identical inspected JVM option
  digests. Prior R09/R10/R11/R12 first imports show only one start, so this
  overlap does not explain away their original OOM /retention evidence.
- Release only stopped R16's duplicated caches/Native cache; preserve source,
  logs, receipts and wrapper. Diagnostic R17 uses the same preconfigured SDK
  table/link and removes redundant playback SDK-reset commands, with explicit
  XML assertions. This is a harness-only attempt to remove overlapping refresh
  setup, not a proved unique cause or a suppression of auto-import/indexing.
  Original four imports, complete model, heap and safety guard remain.
- R17 checks the preconfigured SDK without resetting it; only one external
  resolve starts. It still stops at the original host-memory guard before
  import success. This does not prove a heap OOM or that eliminating overlapping
  setup solves the actual remaining problem. Production adoption stays blocked.
- R18 is diagnostic-only: capture a private no-requested-GC heap during the
  single-import V3 path, then inspect retained owners after process cleanup.
  No additional ordinary repeat until the remaining owner is understood.
  R16/R17 duplicate caches are released; R15's raw private heap is losslessly
  archived and read-back SHA-verified before removing only its raw duplicate.
- R18 obtains a complete4,411,357,780-byte private `-all` heap without a
  requested GC. Actual Gradle action completes, but the host guard still
  stops import; there is no captured heap OOM or external import success.
  MAT analysis runs only after owned GUI/build processes are cleaned.
- Its legacy-configuration retained group is176,185,768 bytes versus about
 700MiB in R15. Factory and source-set ownership also changes. These are
  different-phase heaps, so do not claim a controlled total/peak reduction
  by subtracting groups or adding overlapping retained sizes.
- Both later runs retain an idle compiler process around1.1GiB RSS.
  Continue with the [private idle-lifetime experiment](2026-10-10-experiment-idle-compiler-lifecycle.md),
  preserving all budgets, targets and safety guards; do not kill it during
  measurement to manufacture success.
