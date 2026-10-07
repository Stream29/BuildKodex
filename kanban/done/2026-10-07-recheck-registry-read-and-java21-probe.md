# REVIEW READY — Session 557

- **Exact three-file snapshot: publisher deployment HOLD for one signed-URL
  diagnostic B1. Mosaic probe correction: static APPROVED for controlled
  build/smoke-only rerun, with no publisher execution until B1 closes.**
- Independent read-only reviewer, not implementer. This report is ready for
  asynchronous coordinator reading; no runtime or other reviewer was awaited.
- Verdict applies only to manifest digest
  `7e78ed6f3d560865327dc5cf5d72f74e4328c1d1b0251c9b9fa25703cd7ddaa3`.
  The later “Final tiny security delta” note below is preserved, **not reviewed
  or approved**: the user restricted current evidence to the original snapshot.
- No approval of all packages, a default consumer switch, IDEA/model gains,
  or a Kodex release.

# Task Tree

- `Independently review the fixed GET transport and Java21 probe bytes`()
- `Return scoped deployment verdict without executing builds or package writes`()

# Details

- Parent: [Fork CI](2026-10-07-implement-fork-package-ci.md).
- Current immutable three-file snapshot:
  `file:///tmp/kodex-fork-registry-read-review-20261007/`;
  `SHA256SUMS` digest
  `7e78ed6f3d560865327dc5cf5d72f74e4328c1d1b0251c9b9fa25703cd7ddaa3`.
  Before: deployed inner main `dc3b258d52d935b13081d6859401b80fea72d1bf`.
- Third CI round built all three forks on all three hosts. MCP/Lucene binary
  smoke passed; Mosaic's Java21 JNI launcher could not load the Java25-compiled
  probe. Fix compiles Mosaic with Java21 but explicitly runs its FFM gate on
  Java25; no fork source or runtime support is removed.
- Registry reads actually return a redirect to
  `github-registry-files.githubusercontent.com`. The new transport permits only
  bounded GET redirects to this exact HTTPS host, without Maven Authorization;
  PUT redirects remain refused. Signed URLs and credentials must not enter logs.
  Initial Maven 404 is absence; redirected storage 404 is an error.
- Coordinator's independent read-only audit against the original dc3 SDK bundle
  found all 453 expected files exact, including marker/checksums. Original failed
  run status is preserved; no deletion, overwrite or resume occurred.
- Xiaoxin ran 74 offline tests successfully before the final diagnostic-only
  verification-loop change. Recheck final bytes remotely before deployment.
- Final-byte remote rerun: 74 cases passed in 6.706s. Lucene's completed
  read-only audit also found all 133 expected files exact, with no
  missing/different/read-error files. Original failed CI statuses are retained.
- Reviewer writes only this report. No sources, original reports, builds,
  credentials, device operations or package writes. Static approval is not
  actual three-host Java21 success, default consumer acceptance or IDE gain.

## Final tiny security delta

- The initial three-file snapshot above remains immutable. Final bytes are in
  `file:///tmp/kodex-fork-registry-read-final-review-20261007/`, manifest SHA
  `033b6ceb235d9707776a9276b2675bd99e9ac1a975a1c26753b5c73aca17b566`.
- Relative to the first snapshot: reject non-printable/non-ASCII characters in
  redirected URLs before requesting them; sanitize `http.client.HTTPException`
  like transport errors; add a NUL-containing signed-URL counterexample.
  Other smoke/publication semantics are unchanged. Final approval must include
  these bytes, not silently equate the two manifests.

## Independent evidence boundary

- Loaded project AGENTS, build-change, kanban, planning, checklist, ask-user,
  Gradle, document, workspace and toolchain skills; Draft, this task, the CI
  implementation/Packages authorization and previous validator/root-tooling
  review handoffs. Regular product Java25 guidance is unchanged; Java21 here
  is the isolated compatibility probe, not a product toolchain migration.
- Independently checked
  [manifest:1–3](file:///tmp/kodex-fork-registry-read-review-20261007/SHA256SUMS#L1):
  supplied digest matched; `sha256sum -c` matched **3/3**, including recheck.
  - `publisher.py`: `04159ed23f874311a1b453cb7dbc072759c910d585bbf2a90be6c11dac314004`.
  - `smoke.py`: `6942a1be417e8369343bc30f5ed9bf3f4883e8d016b4f336c26e2b4f2d5e36eb`.
  - `test_publication.py`: `da9f717eae7c1a992c25ce522b60d69757c129624085f88d1a7e5e549335da71`.
- BEFORE is immutable Kodex `dc3b258d52d935b13081d6859401b80fea72d1bf`,
  tree `e8c0e12069bf2df6a964215bc9db5d260c93db19`. Read-only in-memory
  decompression verified SHA-1 of the accessed loose commit/tree/blob objects;
  no Git command or mutable implementation file was used.
- Verified BEFORE blobs:
  publisher `476ffb937fd726e794034f3906509cf880c65cdc`;
  smoke `e1b09ed925352e8b765aaf214baf2afd8e0dd760`;
  tests `eff820ea949c82439f97d165a18552f3c6b9dae3`;
  contract `04aad7928920973117bc1925370b5b8992b69b21`;
  pipeline `b2db82ae4b1a0d67c2a4eadcc27082c9446a4f89`;
  Mosaic workflow `71e73c331a31449680f55368d0eb429c994069f5`.
  Product-path links below are **line mappings for these pinned blobs**, not
  claims about dirty worktree bytes. Snapshot links are actual current evidence.
- This is a three-file delta review, not a repeat audit of every fixed file.
  [Previous root-tooling verdict:1](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-recheck-root-tooling-metadata-attachment.md#L1)
  is preserved; later dc3 corrections and real results are separately identified.

## R — registry trust, bytes and failure semantics

- BEFORE publisher:34–56 refused every redirect. Current
  [transport:21–74](file:///tmp/kodex-fork-registry-read-review-20261007/publisher.py#L21)
  retains `NoRedirect` and follows GETs manually: **four requests maximum /
  three followed redirects**, each with a 60-second timeout. PUT follows none.
- [Predicate:55–63](file:///tmp/kodex-fork-registry-read-review-20261007/publisher.py#L55)
  requires parsed HTTPS, exact storage hostname, default/443 port, no username,
  no password and no nonempty fragment. Parsed host case is normalized;
  suffix lookalikes, trailing-dot names, HTTP, other ports and empty userinfo
  are not accepted as that exact host. Invalid port/bracket parsing in this
  manual block fails closed; inherited-handler exceptions need B1 below.
- Production endpoint is fixed. Pinned
  [safe_name:105–109](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/contract.py#L105)
  rejects absolute/traversal/empty segments, backslashes, URL delimiters,
  percent escapes and C0/space in original Maven paths before HTTP.
  `urljoin` resolves Location against the current URL; an initial relative
  Maven redirect is refused, whereas a relative storage hop can remain on the
  already allowed host. Storage paths need not equal the Maven path: exact
  returned bytes, not the signed path's spelling, establish artifact identity.
- [Fresh headers:68](file:///tmp/kodex-fork-registry-read-review-20261007/publisher.py#L68)
  carry only User-Agent on every storage hop. Request header normalization
  cannot restore Basic Authorization: the original header map is not reused.
  No response headers, signed query or HTTP body enter ordinary error messages.
  This is sound for the covered failures, not the complete guarantee in B1.
- [404 scope:51–53](file:///tmp/kodex-fork-registry-read-review-20261007/publisher.py#L51)
  makes only the original Maven GET's 404 absence. Redirected storage 404,
  unsafe Location and hop exhaustion are errors, never “absent” permitting PUT.
- [Read:44–46](file:///tmp/kodex-fork-registry-read-review-20261007/publisher.py#L44)
  preserves the 512-MiB-plus-one bound. There is no automatic gzip decompressor
  or gzip acceptance shortcut; unexpected compressed content fails exact-byte
  comparison. Query escapes/`+` are not decoded and re-encoded by this recipe.
  Do not repair malformed signed URLs by silently rewriting their signatures.
- The real host was coordinator-confirmed by a sanitized private-PAT GET/302,
  not by these fixtures or a reviewer credential operation. GitHub also lists
  it among runner communication domains; that corroborates host ownership,
  **not a universal Maven redirect/status contract**.
  [GitHub runner communication reference](https://docs.github.com/en/actions/reference/runners/github-hosted-runners#communication-requirements-for-github-hosted-runners).
- Actual production call chain remains
  [main:206–209](file:///tmp/kodex-fork-registry-read-review-20261007/publisher.py#L206)
  → receipt equality → `upload_set`/pinned `verify_bundle`
  → [classify:115–127](file:///tmp/kodex-fork-registry-read-review-20261007/publisher.py#L115)
  → per-path reread/PUT → bounded final reread. `upload_set` includes every
  admitted raw artifact, all four raw-file sidecars and manifest/checksums.
  No marker-only, checksum-header-only or subset completion is introduced.
- Existing complete-exact state returns after live-HEAD recheck **without PUT**.
  Existing partial/different state fails before writes. Fresh upload failure
  remains fail-sealed; conditional PUT and ordering do not make Maven atomic.
  No delete, overwrite or resume is required or authorized.
- [Diagnostics:145–187](file:///tmp/kodex-fork-registry-read-review-20261007/publisher.py#L145)
  add operation, validated Maven path and acknowledged PUT count. The final
  explicit loop retains the actual failing path, unlike the old generator's
  local variable. Six attempts and waits are unchanged. `successful PUTs=0`
  is **not absence proof**: a server may accept a PUT before its response fails.
  Preflight read failure is outside the upload catch, but cannot issue a new
  PUT in this invocation; it says nothing about prior remote writes.

## B1 — URL-bearing transport exception escapes sanitization

- **Confirmed static security boundary gap in this exact snapshot**, not a
  claim of observed credential leakage or a failed real rerun.
- Example Location, using only dummy data:
  `https://github-registry-files.githubusercontent.com/file?signature=fixture secret`.
  Its scheme/host/port/userinfo/fragment pass
  [publisher:55–61](file:///tmp/kodex-fork-registry-read-review-20261007/publisher.py#L55).
  Parsing does not reject the embedded query space.
- On the next request, `http.client._validate_path` raises `InvalidURL` with
  the complete selector, including that signed query. `InvalidURL` derives
  from `HTTPException`, **not** `OSError` or `URLError`; urllib's HTTP handler
  wraps only `OSError` here. The production publisher logs Python3.12 usage at
  [original writer log:10–14](file:///tmp/kodex-fork-ci-logs-20261007/mcp-third-publish.log#L10).
  The relevant primary implementations confirm the static path:
  [CPython3.12 HTTP client](https://raw.githubusercontent.com/python/cpython/3.12/Lib/http/client.py),
  [CPython3.12 urllib handler](https://raw.githubusercontent.com/python/cpython/3.12/Lib/urllib/request.py).
- Responsibility is lost at
  [publisher:41–43,73–74](file:///tmp/kodex-fork-registry-read-review-20261007/publisher.py#L41):
  Request construction is outside `try`, and `HTTPException` is not sanitized
  around opening/reading. The inherited redirect handler also parses Location
  before `NoRedirect.redirect_request`; malformed-URL `ValueError` there is
  outside the manual redirect block's `except ValueError`.
- Reachability: `main → publish → classify → get → request` prints the raw
  exception directly during preflight; `publish → read-before-upload/final
  verify → get → request` additionally exposes it through new
  [exception chaining:181–187](file:///tmp/kodex-fork-registry-read-review-20261007/publisher.py#L181).
  Safe operation/path/count does not sanitize an unsafe chained cause.
  Maven Basic auth is still stripped; the demonstrated risk is signed-storage
  URL/query disclosure, not forwarding the Maven token to another host.
- **Minimum correction/ablation:** keep this small transport owner, exact-host
  loop and safe HTTP status diagnostics. Reject malformed raw URL characters
  before requesting without echoing/requoting them; cover construction,
  inherited redirect parsing, open and read with sanitized URL/protocol-error
  handling (`HTTPException` and relevant URL/encoding errors). Use `from None`
  at that transport boundary. Chain only the resulting safe operation/status
  failure, never raw URL-bearing exceptions. No generic HTTP framework,
  provider or second publication owner is needed.
- The newer task note describes a proposed security replacement, but it is
  outside the user-authorized current snapshot. This finding is not a verdict
  on its unseen bytes; final manifest `033b6ceb...` is **unreviewed**, not
  silently equivalent to the reviewed `7e78ed6f...` input.

## R — Mosaic Java21 compilation / Java25 FFM separation

- The original failure is concrete:
  [Linux:401](file:///tmp/kodex-fork-ci-logs-20261007/mosaic-third-linux.log#L401),
  [Windows:377](file:///tmp/kodex-fork-ci-logs-20261007/mosaic-third-windows.log#L377)
  show ProbeKt class version69 rejected by a runtime supporting at most65.
  It failed before JNI could be exercised; it is not proof of broken JNI code.
- BEFORE smoke emitted no compile toolchain and left the ordinary JavaExec
  launcher implicit. Current
  [toolchain:193–195,245–247](file:///tmp/kodex-fork-registry-read-review-20261007/smoke.py#L193)
  adds `jvmToolchain(21)` only for Mosaic, without an explicit overriding
  `jvmTarget`. This has actual KGP2.3.21 support, not merely a DSL-string guess.
- Independently hashed the cached
  [KGP2.3.21 sources archive](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/local-native-20261003/gradle-home/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-gradle-plugin/2.3.21/5121ff4e2e491f2814c387f1b1ff6fc02f56641b/kotlin-gradle-plugin-2.3.21-sources.jar):
  SHA-256 `092c67c3ff8261d61ae46cb6695b0c0dfa65906e21c1d8e09b9670428e26ea0d`.
  Entries read without extraction:
  - `dsl/ToolchainDsl.kt:52–68` applies the Java toolchain spec and launcher.
  - `targets/jvm/KotlinJvmTarget.kt:250–256` wires target compiler options.
  - `tasks/DefaultKotlinJavaToolchain.kt:208–217,249–270` conventions
    `jvmTarget` from that launcher's JDK version.
  These paths are under `org/jetbrains/kotlin/gradle/`. Therefore Java21 target
  inference is statically justified; an explicit target is not a missing B1.
  [Kotlin toolchain semantics](https://kotlinlang.org/docs/gradle-configure-project.html#gradle-java-toolchains-support).
- [JavaExec:225–240,261–267](file:///tmp/kodex-fork-registry-read-review-20261007/smoke.py#L225)
  keeps the Java21 JNI launcher and explicitly selects JAVA_HOME's Java25 for
  FFM; both use the same probe/classpath. Pinned
  [workflow:194–215](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/.github/workflows/fork-packages-mosaic.yml#L194)
  installs21, then25, and passes the saved21 path separately. Toolchain21 must
  not accidentally downgrade the ordinary FFM JavaExec to21.
- [Command:286–294](file:///tmp/kodex-fork-registry-read-review-20261007/smoke.py#L286)
  keeps Gradle JVM explicitly on JAVA_HOME, provides both installation paths
  and disables download for Mosaic. Auto-detection is not explicitly disabled:
  these are supplied paths, not a claim of an exclusive discovery whitelist.
  Actual selected JDK/bytecode/launchers remain a real-CI gate.
- Probe APIs, Native binary declarations/tasks, linked host runtime checks,
  LinuxArm64 compile-only boundary and JNI/FFM gate closure are unchanged.
  Receipt is written only after the real command succeeds; publisher still
  requires all three exact receipts. No runtime/source support was removed.
- **Static probe mismatch CLOSED / controlled build-and-smoke rerun approved.**
  This does not authorize the unchanged workflow's automatic transition into
  the held publisher. Do not dispatch an end-to-end writer-enabled run under
  this limited approval; close B1 first or use a separately scoped no-writer
  validation route within existing authorization.

## B2 / U — focused checks and truthful acceptance

- Five new cases are statically present:
  [generated probe:727–752](file:///tmp/kodex-fork-registry-read-review-20261007/test_publication.py#L727),
  [transport/status cases:1043–1112](file:///tmp/kodex-fork-registry-read-review-20261007/test_publication.py#L1043).
  They cover separate launchers/toolchain commands, stripped auth, unsafe
  hosts/PUT refusal/hop bound, storage404 scope and sanitized HTTP500 context.
  Transport redirect cases mock `opener.open`; HTTP500 uses loopback transport;
  probe generation mocks Gradle and touches a dummy launcher. **None is real
  three-host JNI/FFM or GitHub Packages execution.**
- Coordinator reports **74 offline tests green on Xiaoxin before the last
  explicit verification-loop diagnostic change** in the requested handoff.
  A later coordinator note now reports another 74-case pass in 6.706s; this
  reviewer did not inspect its raw result or independently bind it to an
  approved security snapshot. No reviewer tests were performed. Retain the
  exact remote-tested manifest and the failed-verification path/count check;
  neither reported pass erases the original snapshot's static B1.
- B1's smallest negative control: dummy signed URL with a raw space/NUL/DEL
  or malformed Location must fail without URL/query/token/body in the
  **rendered exception chain**, in preflight and final-read paths. Cover actual
  `http.client.InvalidURL`/URL-parser failure, not only injected HTTPError.
  Valid signed `%2F`, `%3D`, `+`, mixed-case host and explicit443 must preserve
  intended read semantics; no extra redirect hop may regain Authorization.
  These are remote-coordinator checks, not reviewer-run tests.
- Reported third dc3 CI: **nine producer stages and three merges passed**;
  SDK/Lucene all-three-host smoke passed, SDK Node included. Both writer jobs
  failed; Mosaic Java21 JNI failed. Original failed status must remain.
  [Coordinator evidence:133–146](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-implement-fork-package-ci.md#L133).
- Reported read-only SDK audit used the **original dc3 verify/upload_set
  contract plus the revised GET**, comparing **453/453 exact files** including
  manifest and checksums, with no PUT. This supports declaring that retained,
  coherent **existing SDK artifact complete**, despite historical CI failure.
  It does not certify revised writer execution. Lucene was pending in the
  requested handoff; the preserved later coordinator note reports **133/133
  exact** with no missing/different/read-error files. That is later
  coordinator-reported existing-artifact completeness, not an independently
  repeated reviewer audit or revised-CI success. No deletion/resume is required.
- Pinned
  [recipe:78–83](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/pipeline.py#L78)
  hashes every `.py`/`.gradle`, including tests/publisher; pinned
  [verify_bundle:378–381](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/pipeline.py#L378)
  requires current recipe equality. New recipe bytes necessarily change the
  rebuilt manifest even if SDK payload bytes reproduce.
  **Do not run the new verifier against the old bundle and call it corrupt,
  or silently rebuild/resume/overwrite the same SDK version with a new marker.**
  Use its original contract for the retained artifact audit; a new automatic
  same-version run must reject recipe/manifest differences without PUT.
- Mandatory U: actual three-host Mosaic Java21 JNI plus Java25 FFM with selected
  compile/launcher evidence; real bounded GitHub reads; retained SDK/Lucene
  audits tied to their original contracts; any future writer's exact approved
  main/ancestry/receipt and remote-byte gates. Reuse reported evidence only for
  its actual revision; do not reopen a completed original-bundle audit merely
  because the recipe has subsequently changed.
- Default package adoption remains separate: full binary/JS consumer,
  accessors/regressions, model/source navigation and real IDEA acceptance.
  Old coherent SDK binaries may be evaluated as such; their complete audit
  alone does not authorize consumer-pin changes or claim performance gains.

## Final handoff

- **REVIEW READY / original three-file publisher snapshot HOLD for B1 only;
  Mosaic probe static correction APPROVED for controlled no-writer validation.**
  Missing final runtime results are U, not additional static deployment B1.
- Wrote only this owned report using `apply_patch`; preserved the later
  coordinator note and all previous reports. No implementation/other-document
  edits, tests (including Python), build/Gradle/Node, IDE, credential/device
  operations, Git commands, network writes or process controls.
- Read-only hashing/diffs, archive-to-stdout/object decoding, source/log reading
  and public primary-document GETs only. No temporary files, services, windows
  or persistent resources were created; read commands completed.
- A later security snapshot needs its own exact-byte authorization/review.
  This report neither waits for that work nor approves unseen replacements.
