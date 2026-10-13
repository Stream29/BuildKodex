# Task Tree

- `Respect the decision not to modify or maintain KGP`() // complete
- `Apply the reviewed explicit test-owner convention`() // complete; 144 files match the fixed overlay
- `Verify official KGP representative functions on Xiaoxin`() // 353 tests pass
- `Commit product configuration and guidance separately`() // signed commits; subsequently pushed for release
- `Verify later exact-source release links and isolated startup`() // all four MacBook builds and Mac/Linux smoke pass
- `Keep parent OOM/navigation blockers open`() // configuration scope complete, not an OOM repair

# Details

- User authorizes adopting project-owned improvements and batch commits, then
  explicitly rejects modifying or maintaining KGP because of maintenance cost.
- Keep official KGP2.4.0, Gradle9.5.1 and all existing targets, source hierarchy,
  cinterop, API, warnings policy and maximum heaps. No experimental repository,
  Kotlin fork, patched plugin publication or production toolchain switch.
- Batch one: the independently reviewed 144-file test-owner overlay. Apply the
  existing framework and runtime together to 140 actual test owners; remove
  both from 63 empty owners. New test owners must explicitly apply the convention.
- Batch two: developer guidance and this adoption's evidence/board state. Stage
  only related files; preserve unrelated user edits. Original authorization
  covered local commits; later release authorization separately permits pushes.
- Do not adopt experiment-only 10-second Gradle idle timeout or GC parameters
  as universal defaults. Their development trade-offs need a separate gate.
- The successful isolated OOM combination included KGP source corrections.
  Official-binary and unmodified-source controls still OOM with the same test
  ownership/lifecycle policy. This batch is not an accepted OOM repair.
- Existing [OOM repair](../done/2026-10-08-repair-full-model-idea-import-oom.md) and
  [combination evidence](../done/2026-10-10-verify-minimum-oom-repair-combination.md)
  are now closed as stopped research, not successful remedies. Preserve the inherited common Json navigation failure too.
- Original development validation uses Xiaoxin under the heavy-resource lock,
  with explicit available daemon JVM and private credentials removed afterward.
  The later separately authorized release uses MacBook-first gates.
- Exact candidate:
  `/tmp/kodex-explicit-test-owner-candidate-20261009`;
  manifest SHA256
  `f7c30b94dbec4ed5bd459246d1f0c767c27cb6a1ca0f50a36316a6aedbbfa5c5`.
  Baseline143 hashes match accepted product `5b41be52`.
- [Preparation](../done/2026-10-09-prepare-explicit-test-owner-convention-candidate.md)
  and [independent review](../done/2026-10-09-recheck-explicit-test-owner-convention.md)
  preserve the original static/runtime distinctions.

## Original Xiaoxin gate

- Entire product-source manifest: 1307 files,
  SHA256 `2e36c07f3be30b9744cb13ef94ab3844e5d75058ff59cbfe8a942fb8aab840ba`.
  The separate developer-document change is not a compilation input.
- Reuse the stopped private R36 Gradle/Native cache for R40's unchanged official
  KGP project. This is an explicit warm-cache validation, not a cold benchmark.
- First gate fails before tests: the provenance writer follows canonical
  Gradle-home location and tries a nonexistent evidence directory. Its cache
  alias also defeats the old cleanup helper's path restriction. Stop only the
  recorded owned daemon by verified PID/start/executable/private cwd; cleanup
  completes, credentials and alias are removed. No product defect is inferred.
- R02 uses the real cache path, an explicit evidence destination and an exact
  two-root cleanup boundary. Representative functions and offline integration
  total **353 tests, zero failures/errors/skips**. The following native link is
  stopped by the original resource reserve, not accepted as a successful build.
- R03 requests only the native link after completed-phase process cleanup, with
  original maximum heaps and resource thresholds. Its first preflight refuses
  below4GiB free disk, before starting processes or staging credentials.
  Release only finished R36's regenerable private IDE `system` after validating
  the latest owned cleanup; preserve all source/artifacts/model/log receipts.
  R03 then proceeds with the unchanged official plugin and source manifest,
  but is also stopped by the original resource reserve after388.439s. No
  successful new link or native startup is claimed. The first combined run's
  resource stop takes377.178s; its353 completed test results remain valid.
- Both runs use unchanged maximum heaps and thresholds; no idle/GC override.
  This does not establish a link regression caused by the convention. No
  same-condition baseline link counterfactual has been run; retain that limit
  rather than blaming the product or lowering the guard.
- Owned process cleanup is verified. Copied credentials are removed; no
  Gradle/Kotlin daemon remains and no CLI is installed. Evidence and source
  manifests remain private on Xiaoxin; no tokens/heaps enter the commits.
- Preserve first failure and second-run logs under Xiaoxin:
  `~/ACodeSpace/demo/kodex-kgp-producer-experiment-20261009/gui-runs/baseline-stock-kgp-test-owner-idle-r40/`.

## Original commit batches

- Outer `7cab8ad`: research/control evidence and explicit rejection of KGP
  maintenance. Pending adoption status was recorded honestly.
- Product `44aac502`: atomic 144-file test convention/owner configuration.
- Product `f47f4198`: developer guidance in a separate commit. Both product
  commits are signed and local; no push or release.
- Official KGP/buildSrc dependency input, wrapper, catalog, targets, business
  sources, maximum heaps and project resource properties remain unchanged.
- Broader OOM and navigation tasks stay open; this card covers only the
  project-owned configuration batch, not the rejected source-patch route.

## Later release gate closes the configuration batch

- User subsequently authorized main pushes and v0.4.11 publication after gates.
  Product `f22629c3` includes the exact configuration and guidance, the accepted
  Ultrafast merge and the three version-only declarations.
- [Release gate](2026-10-11-release-kodex-0-4-11.md) uses a fresh
  detached recursive checkout on MacBook, official KGP2.4.0/Gradle9.5.1/Java25,
  original4GiB heap and full targets. Official KGP JAR SHA256 is
  `ca5903a236a19a54a883a8695958d8516f9e39cc333bd1e3077f92ae38271cd6`.
- All four full release links succeed in16m27s;429 JVM and73 Mac Home tests
  pass. Archived Mac arm64 and Linux x64 bytes pass real PTY fresh-Home and
  v0.4.10→v0.4.11 startup, draft input and normal exit, with no lease residue.
- Earlier Xiaoxin stops remain failures at their original resource boundary;
  this is later cross-host acceptance, not a rewritten result or causal proof.
- Close only the project-owned configuration batch. Full IDE OOM, Json
  navigation, development resource policy and unexecuted platform/fault cases
  stay with their existing parent tasks. No experimental KGP is adopted.
