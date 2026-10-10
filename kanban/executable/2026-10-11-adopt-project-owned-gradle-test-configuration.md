# Task Tree

- `Respect the decision not to modify or maintain KGP`() // complete
- **`Apply the reviewed explicit test-owner convention`()**
- `Verify the official toolchain and representative functions on Xiaoxin`()
- `Commit product configuration separately from evidence and board updates`()

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
  only related files; preserve unrelated user edits. Commit locally, not push.
- Do not adopt experiment-only 10-second Gradle idle timeout or GC parameters
  as universal defaults. Their development trade-offs need a separate gate.
- The successful isolated OOM combination included KGP source corrections.
  Official-binary and unmodified-source controls still OOM with the same test
  ownership/lifecycle policy. This batch is not an accepted OOM repair.
- Existing [OOM repair](2026-10-08-repair-full-model-idea-import-oom.md) and
  [combination evidence](2026-10-10-verify-minimum-oom-repair-combination.md)
  remain open. Preserve the inherited common Json navigation failure too.
- Validate only on Xiaoxin under the heavy-resource lock, using explicit
  available daemon JVM and private credentials removed after execution.
- Exact candidate:
  `/tmp/kodex-explicit-test-owner-candidate-20261009`;
  manifest SHA256
  `f7c30b94dbec4ed5bd459246d1f0c767c27cb6a1ca0f50a36316a6aedbbfa5c5`.
  Baseline143 hashes match accepted product `5b41be52`.
- [Preparation](../done/2026-10-09-prepare-explicit-test-owner-convention-candidate.md)
  and [independent review](../done/2026-10-09-recheck-explicit-test-owner-convention.md)
  preserve the original static/runtime distinctions.
