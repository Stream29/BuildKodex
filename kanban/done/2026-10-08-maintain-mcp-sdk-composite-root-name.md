# Task Tree

- `Freeze the accepted binary baseline and unchanged SDK source contracts`()
- `Change only the SDK root name on its kodex-submodule maintenance line`()
- `Validate a real explicit source composite in the isolated Xiaoxin consumer`()
- `Independently review the exact root-name and gitlink delta`()
- `Commit the maintenance change and product gitlink without changing the consumer version`()
- `Publish the new SDK commit through the existing three-host CI gates`()
- `Adopt the new immutable SDK package only after complete byte verification`()

# Details

- Parent: [Gradle development experience](../executable/2026-10-02-rescue-gradle-development-experience.md).
  Prerequisite: [accepted default binary consumer](../done/2026-10-07-consume-verified-fork-packages-by-default.md)
  at signed main `c0ba86cc`.
- User-approved root name is `kotlin-mcp-sdk-fork`, distinct from internal
  `:kotlin-sdk`; no module IDs, protocol/API, targets or versions are changed.
  Original SDK pin `53f717661712d618cd4822d1760392e903989ead` remains the old
  published package identity. Upstream/main cooperation branches stay untouched.
- Do not override `includeBuild.name` to disguise an unfixed fork. Explicit
  source validation includes the actual fork with its new root name, preserving
  ordinary substitution and current project identities; no default source mode.
- Heavy operations use the same Xiaoxin lock and explicit existing Daemon JVM.
  Local user machine remains free of builds/CLI/IDE.
- Root name changes the source commit and immutable package version even though
  library bytes may be equivalent. Keep the old accepted SDK catalog version
  until the new commit's genuine JVM/JS/Native three-host gates and complete
  remote verification pass. Never republish the old suffix under a new recipe.
- Root-wide moduleTree, compatibility versions and build-logic remain distinct
  batches, not included in this maintenance commit.
- Xiaoxin explicit `includeBuild("KotlinMcpSdk")`, with no name override, configures
  successfully in35s. The first API validation used the incorrect task selector
  `:kotlin-mcp-sdk-fork:apiCheck`: an included build's task identity defaults to
  its directory `KotlinMcpSdk`, distinct from its actual root project name.
  That task-not-found failure is retained; rerun uses `:KotlinMcpSdk:apiCheck`
  plus real stdio/HTTP JVM/Node consumers. No production file was changed to
  hide the selector mistake.
- Correct module-qualified API selectors are
  `:KotlinMcpSdk:kotlin-sdk-{core,client,server}:apiCheck`; the root itself has
  no aggregate `apiCheck`. All three JVM API checks and actual stdio/HTTP JVM/Node
  consumers passed in414s (24 tests, no failures). KLIB API checks were skipped
  by the actual validator, not certified here.
- Current three-source composite also configures successfully. Warm old-name
  help controls did not reproduce the earlier baseline accessor error, so these
  are not claimed red tests or unique-root-cause proof.
- A minimal standalone SDK diagnostic explicitly enables its real type-safe
  accessors using the SDK's existing wrapper9.6.1: old name fails with duplicate
  `RootProjectAccessor.getKotlinSdk()`, fixed name passes in100s. The final red
  starts with no available Daemon; final green reuses its exact JVM/PID.
  An earlier standalone attempt incorrectly started9.6.1 while9.5.1 was still
  available: it is retained as a failed resource gate, and only that newly owned
  Daemon was released. Our completed idle9.5.1 was later explicitly released
  before the valid wrapper transition. No user process was stopped.
- [Independent one-file review](2026-10-08-recheck-sdk-composite-root-name.md)
  found no confirmed B1. Actual one-line SHA remains
  `237c2bab2a05251ca22b7beff8bdce4f18157de5630e830210ce25139c130fd0`.
  SDK normal source, diagnostic typed-accessor and API evidence are distinct;
  none certifies real IDE/GUI performance or the root compatibility upgrade.
- Signed maintenance commit `51ca6e0243f82e4bd7a521132a1445a9ff59fc1f`
  (tree `0b4862b5068746470a72cb4fd0f8654e9038ffe7`) is pushed only to
  `kodex-submodule`; upstream and fork main are unchanged. Signed Kodex main
  gitlink commit `320356450230ff695a31c89397b05b0d860746a1` is pushed.
  Catalog still consumes the accepted old SDK `…53f717661712`.
- Reviewed manual SDK publication
  [run37696609442](https://github.com/Stream29/Kodex/actions/runs/37696609442)
  is running against exact main32035645; no product-main push is permitted while
  its publisher gate remains active. The unprotected push run skipped writing,
  as designed. Expected new suffix is `0.14.0-kodex.51ca6e0243f8`.
- The same run is now terminal **SUCCESS**, exact main32035645. All three
  producers and actual host consumers passed; writer checked453 immutable
  remote files. New manifest SHA256
  `aeed44fa3899aca01aa55750cc57399a55507bd3a1b904b9eb8fcad41663f245`.
  Separate read-only registry audit and ordinary root binary-consumer gates
  precede catalog commit; no old version was overwritten.

## Scoped adoption completed

- Separate read-only audit confirmed453/453 exact, zero missing/different/errors.
  Xiaoxin `registry-audit/mcp-root-full-read.json` SHA256
  `ba9aca2c2230729321b74dc4e2fc5b1cb5beffe12da7fc16dda8afac627b0d23`.
- Ordinary source-free root stdio/HTTP JVM/Node, composition, CLI JVM compile
  and complete Linux release link passed in491s. Actual model216, included builds
  empty;39 resolved artifact observations and9 genuine source attachments match
  sealed publication hashes. No catalog/source fallback was added.
- A fresh-resolution observer initially borrowed another project's configuration
  from the root task, correctly rejected by Gradle's exclusive-lock check.
  New acceptance-only observer resolves each configuration in its real owner task;
  exact model/hash checks then passed. Production code was not modified to weaken
  locking, and the failed log remains separate.
- Isolated real CLI rendered, opened Settings by pointer, dismissed with Escape
  and exited0 with no owner files; its temporary Home was removed. Binary SHA256
  `89fd618de81702935e00935e78f1472990768b6f3ad6da7400334714aeec02a3`,
  equal to the prior accepted CLI. The SDK root-name change does not establish
  a new product feature or IDE performance improvement.
- Signed catalog-only commit `2e5c0df4` adopts `0.14.0-kodex.51ca6e0243f8`.
  Original53 package is retained, not overwritten. The next root compatibility
  and genuine IDE gates are distinct and remain open.
