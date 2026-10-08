# Task Tree

- `Read the scoped instructions and actual fork build inputs`() // complete
- `Prepare the five minimum candidate inputs without fork edits`() // complete
- `Identify API, Node, composite and publication version gates`() // complete
- `Seal and statically review the exact candidate handoff`() // complete

# Details

- Parent: [toolchain compatibility](../executable/2026-10-08-validate-kodex-toolchain-compatibility.md).
  Canonical user decision is a Kodex-only KGP2.4.20/Gradle9.6.1 compatibility
  line; fork adaptation belongs to `kodex-submodule`, not upstream cooperation
  branches. Failure stops migration for the user's decision.
- Preparation only: write candidate files under
  `file:///tmp/kodex-fork-toolchain-candidates-20261008/` and this task handoff.
  No production/fork edits, Git operations, builds, tests, devices or publication.
- Accepted source predecessors: Mosaic78f94c4c, SDK51ca6e02, Lucene549f8afc.
  Preserve original groups/artifacts/upstream versions, product source and API
  dumps, all full targets and SDK JS exception. No plugin/library upgrades
  beyond the authorized KGP line.
- Inventory actual KGP/Compose/serialization/convention version references.
  Prepare catalog2.4.20 changes and Gradle9.6.1 URL with official checksum
  `9c0f7faeeb306cb14e4279a3e084ca6b596894089a0638e68a07c945a32c9e14`.
  Keep wrapper/runtime resource policies separate. Generated wrapper assets
  are coordinator-owned standard-task inputs, not hand-authored by this lane.
- Existing `smoke.py` selects the real producing KGP from sealed host receipts.
  Verify that it still exercises the new actual compiler rather than an older
  guessed consumer. Do not change the publisher or old immutable versions.
- Report independent literal versions, known API/plugin constraints and exact
  bounded compile/API/source-composite gates. Do not add broad adapters or mute
  warnings to make an unexecuted upgrade look compatible.
- Coordinator alone validates, reviews, commits maintenance branches and new
  gitlinks, then runs three-host CI. Default package pins remain the already
  verified versions until complete new artifact validation succeeds.

## Preparation handoff

- **EARLY READY — PREPARED / VALIDATION PENDING.** Worker execution remains
  preparation-only; no tests/builds (including Python unittest), IDE, devices,
  process-control, Git operations, credentials or network writes.
- Accepted root main `5b41be52` is immutable. The separate isolated root candidate's
  JVM110/Node12/Native64, complete Linux CLI and private PTY passes are
  coordinator-reported; GUI and all-three-fork source mode remain open.
- [Candidate README](file:///tmp/kodex-fork-toolchain-candidates-20261008/README.md#L1-L48)
  and [exact manifest](file:///tmp/kodex-fork-toolchain-candidates-20261008/MANIFEST.tsv#L1-L6)
  are ready for asynchronous coordinator reading. Five proposed inputs only:
  three catalogs (Mosaic2.3.21→2.4.20, SDK/Lucene2.4.0→2.4.20), SDK checksum
  addition and Lucene9.5.1→9.6.1 URL/checksum. Mosaic wrapper already matches.
- Official checksum GET independently matched the specified9.6.1 SHA256.
  No generated wrapper JAR/scripts authored; coordinator's standard `wrapper`
  regeneration and full generated-file review remain required.
- Mosaic uses catalog-backed real buildscript KGP/Compose plus included
  build-support `compileOnly` KGP; SDK buildSrc already uses catalog-backed
  implementation KGP/serialization; Lucene direct plugins use catalog aliases.
  Root's independent KGP/Koin literals belong to its separate approved candidate.
- Poko0.22.1, SDK BCV0.18.1/atomicfu0.33.0, root TestBalloon and all runtime
  dependencies stay pinned. Poko's published2.3.21/2.4.0-Beta2 compatibility
  does not establish2.4.20. No speculative compatibility fix is prepared.
- SDK API tasks are qualified per core/client/server module, not root aggregate.
  Actual BCV JVM checks inspect compiled classes against existing dumps; they
  are not KGP compiler-integrated common API validation. SDK KLIB checking is
  disabled by default, while Mosaic explicitly enables it. K2 dump differences
  must stop for review, never be accepted by rewriting golden API files.
- SDK JS and Wasm stores remain original and **PENDING** standard generated
  updates if an actual normal gate fails. Root's approved lock cannot be copied.
  No scoped source registers a literal `commonApi`/`fork-native-gates`; any
  coordinator-only shortcut needs a real task graph and cannot certify JS locks.
- Existing smoke chooses real KGP from sealed host receipts; archive evidence
  and KLIB compiler-version validators are parameterized, not old-version
  constants. New2.4.20 cache/archive/payload evidence is still a real host gate.
- Coordinator must provide manual HTTPS source checkouts of **all three** exact
  predecessors, with current root moduleTree and genuine SDK51 root fix.
  Old temporary source settings contain superseded recursive discovery/name
  override and must not be reused. Actual task selectors/host publication
  closure and Node inputs are in the candidate README.
- Local Lucene cooperation branch `fix/host-logging-configuration` was not
  checked out or changed; coordinator explicitly switches existing
  `kodex-submodule` at accepted549f8afc. All source/API/wire/targets/groups/
  artifacts/base versions, SDK JS closure and Windows resize regression remain.
- Any concrete failure suspends migration for a user decision. No Werror,
  lock check, target or artifact-shape bypass; no provider/plugin facade.
  New maintenance commits/identities, multihost CI, complete remote-byte audit
  and eventual package adoption belong exclusively to the coordinator.

## Static completion

- Exact candidate tree contains eight files: five proposed build inputs,
  README, baseline manifest and [hash seal](file:///tmp/kodex-fork-toolchain-candidates-20261008/SHA256SUMS#L1-L7).
  Seal SHA256:
  `3bb66b268393cd8a06d79a6b829f47fd9239fd3b2915021912553bbbd85ec089`.
- Read-only `sha256sum -c` verified all seven entries. Each catalog's entire
  final diff contains only its one Kotlin version line; wrapper diffs contain
  only the specified checksum/URL inputs. Baseline hashes remain unchanged.
- Whole-document/change/ablation review completed against actual sources and
  the preparation boundary. No incidental temporary files or owned resources
  were created; the requested handoff directory is intentionally retained.
- Preparation script and coordinator handoff are complete. This preparation-only
  task is archived; parent validation and adoption remain open.
- **PREPARED / VALIDATION PENDING — independent preparation result only.**
