# Task Tree

- `Verify the feature tip and preserve both lines of work`() // local and remote tip equal
- `Prepare a non-fast-forward merge without changing the Gradle route`() // automatic, no conflicts; signed commit retained
- `Validate merged protocol, HTTP, RPC and UI fixtures on Xiaoxin`() // R03 passes
- `Record the signed merge and outer pointer with honest acceptance status`() // complete integration inputs; committed together

# Details

- User explicitly requests merging `feat/ultra-fast` into current main.
- Product main: `f47f4198`; feature tip: `d85e0e61`; common baseline:
  `5b41be52`. Feature contains `2399a6ea` (Ultrafast) and `d85e0e61`
  (ProMax). Preserve both commits and the new explicit test-owner convention.
- Scope: the existing reviewed subscription service-tier and plan-value work.
  No KGP modification, toolchain upgrade, target change, new provider/RPC
  contract, release, push or paid Ultrafast call.
- [Feature task](../executable/2026-10-09-support-openai-ultrafast.md) preserves contract
  approvals and the fact that the branch was pushed before runtime acceptance.
- Inner main is clean; the feature worktree is clean at inspection. Outer
  user-owned edits and unrelated notes remain untouched.
- Validate on Xiaoxin only under the existing heavy lock, with official KGP,
  explicit available daemon JVM, private package credentials removed afterward.
  Use an isolated merged-source snapshot, not overwrite earlier control inputs.
- Protocol/HTTP loopback, settings persistence, RPC round trips, notification
  warnings and Mosaic/UI are targeted gates. Fixtures do not establish real
  Ultrafast entitlement or speed; no live-model request is intended.
- Current full-native-link resource and GUI/navigation blockers remain separate.
  Do not disguise the previous resource stops as successful release validation.
- Retain the original feature worktree unless its owning session's release is
  established; a clean worktree alone is not permission to reclaim its resource.

## Frozen merge inputs

- Remote branch query confirms full tip
  `d85e0e6179faaac08f8adb1da46a002669291f47`.
- Proposed merge tree: `1a37f993389b144afde65eec7bcfa8c73866008e`;
  38 changed feature files, no build-script or Gradle overlap.
- Isolated source: 1312 files; manifest SHA256
  `90708186c34f552f19873e3f5d8652cbb87de36a3b1ed417457b0cf0cc9a9112`.
- Private remote root:
  `~/ACodeSpace/demo/kodex-ultrafast-main-integration-20261011/`.
  Prior OOM control project inputs remain untouched.
- Initial gate requests all 17 JVM fixture owners from the feature plan,
  actual CLI JVM compile, protocol Native tests and changed client/RPC Native
  compilation. Official-plugin provenance rejects both experimental classes.
- Explicit warm private cache reuse; no empty-cache benchmark claim. No
  credential-requiring live integration is requested.

## Actual runtime findings

- R01: 260 tests execute, one failure. The real 32-column submenu clips
  `support` from `access/model support`. Preserve the failing frame and source
  inputs; split the visible hint into short complete lines, retaining the
  same width/height, pre-opt-in assertions and atomic selection check.
- R02: the same executed 260 tests have no failures, but the next authentication
  renderer test does not compile. New nullable public `planType` cannot be smart
  cast across modules. Capture it once in a local value; keep both nonnull and
  missing-plan assertions, including ProMax. This is a test correction, not an
  auth/entitlement behavior change.
- Both failed gates retain logs/results. The private cleanup helper does not
  automatically release one detached daemon; verify each recorded PID/start/
  executable/private cwd and stop only it via pidfd. Fix the diagnostic's
  original-project-to-private-daemon-directory transition guard; no global stop
  or shared cache/process reclamation.

## Accepted integration result

- Final tree `e1bda61102a7f25ad302b10d3e1dd9e38dd87485`; 1312-file manifest
  SHA256 `49e5b718da6419772cb6392321650cebf4125646adeaaaa3f79a2595583e7fa9`.
- R03 succeeds in177.754s: **694 actual JVM/Linux tests**, zero failures/errors/
  skips, CLI JVM compile and affected client/RPC Native compilation. Test
  tasks are forced; compilation may reuse unchanged outputs. Official KGP
  provenance and source hashes are checked. Cleanup completes automatically.
- Signed merge `c92fdc59` retains parents `f47f4198` and `d85e0e61`.
  Original feature commits remain reachable. Only the clipping fix and nullable
  test correction supplement the automatic feature merge.
- No main push, release, new complete native CLI link/smoke or paid call.
  Previous resource stops and GUI/navigation blockers are not closed.
- Inner main and feature worktree are clean. Keep the latter and the feature
  branch for its original owning session; do not delete them automatically.
