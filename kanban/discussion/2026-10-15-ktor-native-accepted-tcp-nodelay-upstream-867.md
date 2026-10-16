# Task Tree

- `Read contribution scope and candidate patch without changing mainline resources`()
- `Review current upstream rules, branches, releases, and duplicate reports`()
- `Review socket boundaries and identify submission validation gaps`()
- `Record the user's main-branch contribution decision`()
- `Prepare the contribution in the user's primary fork and draft the English PR`()
- `Push the user's commits to the fork on explicit request`()
- `Open and prefill the GitHub PR form without submitting it`()
- `Diagnose the submitted PR's failing title check`()
- `Prepare a YouTrack draft after rechecking duplicates`()
- `Confirm the user's created ticket and update the PR title`()
- `Diagnose platform test failures and correct the enabled-option assertion`()
- `Commit and push the reviewed assertion fix on explicit authorization`()
- **`Await upstream CI and human review`()**

# Details

## Ownership and authorization

- Session867 owns only the Ktor upstream contribution discussion.
  Session787 retains dependency builds, Kodex adoption, and real TUI validation.
- User-owned Draft, mainline task/checklists, product main, and Session787 workspaces
  remain unchanged. This Session did not connect to MacBook, run Gradle, create a
  fork or commit, or submit an Issue or PR.
- Heavy builds must wait for coordination with Session787; never alter its remote
  validation root, caches, processes, or shared device lock.
- User's follow-up selects the existing local fork and a new branch from upstream
  `main`. The initial GitHub-only route is superseded: after the PR title check
  failed, the user agreed a YouTrack ticket is needed.
- User explicitly requires the primary fork directory, not an independent worktree.
  Work directly in `/home/stream/ACodeSpace/fork/ktor`.
- The user created the fix and test commits, then explicitly requested push and
  a prefilled PR form for personal editing. The push is complete and the user
  subsequently submitted PR5949. Do not overwrite the user's edited PR body.
- The user reviewed/edited the draft and created
  [KTOR-10005](https://youtrack.jetbrains.com/issue/KTOR-10005).
  On the user's follow-up, this Session changed only the PR title to include it.

## Confirmed upstream state — 2026-10-15

- Read local Ktor `AGENTS.md`, `CONTRIBUTING.md`, Code of Conduct pointer,
  PR template, and Issue template; rechecked current official release-branch guidance.
- Public `git ls-remote` returned:
  - `release/3.x`: `69465106362ee7957ca549889cfde6d77dfe7bdd`.
  - `main`: `4016bdfe32256ff82723af7259e294908e48be75`.
  - `3.6.0` tag: `111c5807d0a9004c8e0f17ae01e12a71aae56a29`.
- Official latest Release resolves to
  [3.6.0](https://github.com/ktorio/ktor/releases/tag/3.6.0), not 3.5.1.
  Read-only source checks found the same missing accepted-socket TCP_NODELAY setup
  in [3.6.0](https://github.com/ktorio/ktor/blob/3.6.0/ktor-network/posix/src/io/ktor/network/sockets/TCPServerSocketNative.kt#L62-L73),
  [release/3.x](https://github.com/ktorio/ktor/blob/release/3.x/ktor-network/posix/src/io/ktor/network/sockets/TCPServerSocketNative.kt),
  and [main](https://github.com/ktorio/ktor/blob/main/ktor-network/posix/src/io/ktor/network/sockets/TCPServerSocketNative.kt).
  This is source inspection, not runtime reproduction on those revisions.
- [Contribution rules](https://github.com/ktorio/ktor/blob/release/3.x/CONTRIBUTING.md#branching-strategy)
  direct bug fixes to `release/*`; [agent guidance](https://github.com/ktorio/ktor/blob/release/3.x/AGENTS.md#release-branches-and-api-policy)
  identifies `release/3.x` for v3 patches. The user subsequently chose `main`
  for this contribution, independently from Session787's 3.5.0 product repair.
  Prepare against `main`; handle any maintainer-requested retargeting later.
- [Issue template](https://github.com/ktorio/ktor/blob/release/3.x/.github/ISSUE_TEMPLATE/issue_report.md)
  redirects reports to YouTrack. A real YouTrack ticket is now needed for PR5949;
  do not open a redundant GitHub Issue or invent a KTOR identifier.
  Use English and the PR template's
  Subsystem / Motivation / Solution sections.
- Current contribution rules require latest-release reproduction and covered code,
  JVM plus affected-platform tests, formatting/linting and ABI checks.
  They also treat relied-upon behavior changes as potentially breaking:
  explain alignment with JVM rather than promise automatic patch acceptance.

## Duplicate review

- Public GitHub issue/PR search used `repo:ktorio/ktor` with `TCP_NODELAY`
  (5 results), `Nagle` (2), `noDelay` (7), and `native socket latency` (1);
  responses reported `incomplete_results=false`. None identified the same fix.
- Public YouTrack API search used `project: KTOR` with `TCP_NODELAY` (8),
  `Nagle` (1), `noDelay` (8), and `"native" "latency"` (3).
  No same-root-cause report was identified; this is not a guarantee of absence.
- Inspected likely neighbors:
  [KTOR-4989](https://youtrack.jetbrains.com/issue/KTOR-4989) concerns JVM client
  connection timeout; [KTOR-413](https://youtrack.jetbrains.com/issue/KTOR-413)
  concerns JVM Windows connection resets;
  [KTOR-9891](https://youtrack.jetbrains.com/issue/KTOR-9891) concerns Native
  lock contention at high concurrency. These do not cover accepted TCP socket defaults.
- [PR4345](https://github.com/ktorio/ktor/pull/4345) is NodeJS CIO preparation,
  not this Native accept repair.
- Some GitHub metadata endpoints returned TLS/403 errors; branch heads were
  obtained using public Git transport and latest Release via the official page.
  Repeat duplicate and branch checks immediately before any publication.

## Candidate patch review — static only

- Mainline candidate was read only:
  [accept implementation](file:///home/stream/ACodeSpace/local/ktor-native-nodelay-787/ktor-network/posix/src/io/ktor/network/sockets/TCPServerSocketNative.kt#L61-L82)
  adds 10 production lines; [regression](file:///home/stream/ACodeSpace/local/ktor-native-nodelay-787/ktor-network/nix/test/io/ktor/network/sockets/tests/TcpSocketTestNix.kt#L19-L57)
  adds 41 lines including the import. `git diff --check -- ktor-network` passed;
  this does not establish compilation, formatting-task, or test success.
- Keep the production change local to `accept`: INET guard, `IntVar(1)`,
  `IPPROTO_TCP`, `TCP_NODELAY`, existing `ktor_setsockopt`, then `.check()`.
  No helper abstraction, client-options expansion, or public signature change is needed.
- The guard matches the existing
  [JVM accepted-socket policy](file:///home/stream/ACodeSpace/local/ktor-native-nodelay-787/ktor-network/jvm/src/io/ktor/network/sockets/ServerSocketImpl.kt#L46-L56).
  Both Native IPv4 and IPv6 map to `InetSocketAddress`; Unix-domain maps separately:
  [address conversion](file:///home/stream/ACodeSpace/local/ktor-native-nodelay-787/ktor-network/posix/src/io/ktor/network/util/NativeSocketAddress.kt#L61-L64).
  Therefore the proposed guard covers both IP families and excludes Unix-domain
  by source reasoning; only IPv4 is exercised by the new candidate test.
- Setting/checking the option stays before socket construction within
  [buildOrCloseSocket](file:///home/stream/ACodeSpace/local/ktor-native-nodelay-787/ktor-network/posix/src/io/ktor/network/selector/SelectorManager.kt#L43-L54).
  An option failure retains shutdown/descriptor-close/rethrow semantics before selector
  registration. Failure cleanup has not been fault-injection tested.
- [Windows wrapper](file:///home/stream/ACodeSpace/local/ktor-native-nodelay-787/ktor-network/windows/src/io/ktor/network/util/SocketUtilsWindows.kt#L242-L267)
  already handles socket/option pointer and length conversion.
  [Kotlin 2.3.21 MinGW posix definitions](https://github.com/JetBrains/kotlin/blob/v2.3.21/kotlin-native/platformLibs/src/platform/mingw/posix.def#L25-L66)
  include Winsock headers. This supports the portability approach, but does not
  prove the newly used constants compile on every Native target.
- Production code is in the shared `posix` source set, while the new test is `nix`:
  Windows does not receive this regression. Retain an explicit Windows compile/CI gate.
- Existing test style uses real selectors and direct descriptors; the new test
  follows that approach and closes accepted/client/server sockets in finally blocks.
  Before submission, review connect/accept failure and cancellation ownership;
  do not introduce a broad cleanup refactor for this patch.
- Public/protected declarations and ABI files are untouched by this candidate.
  ABI preservation is a static expectation; `checkKotlinAbi` is still pending.

## Validation and disclosure gates

- Session787 handoff reports Linux production-byte causal ablation:
  240 synthetic messages: 18.682 s → 0.716 s;
  1,000: 20.746 s → 0.587 s when only accepted TCP_NODELAY is injected.
  A 1 ms curl-poll control did not improve timing.
  [Evidence and limitations](file:///home/stream/ACodeSpace/local/kodex-history-loading-787/README.md#L1-L49).
- Diagnostic interposition is not a compiled source repair. At handoff, mainline
  `red-native-02` was still in build preparation with no red/green result.
  This Session did not poll or interfere with that build.
- Before PR readiness: obtain mainline completed evidence; independently reproduce
  on the current latest release and validate the chosen main-branch revision,
  scheduling Mac work only after coordination.
- Cover IPv4 and IPv6 accepted flag values; run existing Unix-domain echo coverage;
  review configuration-failure cleanup; obtain affected Native compilation/tests,
  JVM tests, assemble, formatting/lint and ABI checks with explicit task outcomes.
  Do not claim all-OS runtime reproduction from one platform's evidence.
- Publish a minimal Ktor-only bind/connect/accept/getsockopt reproducer or unit test.
  Omit private paths, session identifiers, credentials, raw logs and full product
  fixture artifacts. Any timing description must retain platform/version and
  diagnostic-ablation caveats.
- No product upgrade, dependency publication, Release, real backend restart,
  history pagination change, or subscription-semantic change is within this task.

## Primary fork contribution workspace

- Existing fork: `origin=git@github.com:Stream29/ktor.git`,
  `upstream=git@github.com:ktorio/ktor.git`.
  [Primary workspace](file:///home/stream/ACodeSpace/fork/ktor) now checks out
  `fix/native-accepted-tcp-nodelay-867`.
  The prior `sse-allow-missing-content-type` branch and its commits are retained;
  the pre-existing `.idea/vcs.xml` modification remains uncommitted and untouched.
- Contribution branch: `fix/native-accepted-tcp-nodelay-867`;
  base: `4016bdfe32256ff82723af7259e294908e48be75` from current upstream `main`.
  No upstream feature-branch push is intended; any authorized push must target the fork.
- Per the user's correction, migrated both candidate files to the primary fork,
  checked file bytes and full scoped diff for exact equality, and removed this
  Session's temporary worktree without force after clearing only its copied patch.
  `git worktree list` now contains only the primary fork.
- The user's `.idea/vcs.xml` SHA256 remained
  `b83386d08d29654307382c435f4d2477780fbe936c35f09b8734a9b7b8ba68ae`
  across branch switching and migration.
- IDE MCP now confirms the Ktor fork is open. No IDE project, sync, window,
  or build was opened or started by this Session.
- Re-read exact-main guidance, including newly referenced
  [AI policy](file:///home/stream/ACodeSpace/fork/ktor/AI_POLICY.md#L5-L38).
  The policy requires human review and tool-assistance disclosure before submission.
  It prohibits AI-authored conversational comments/reviews: the user must handle
  substantive maintainer discussion; this agent may work on code and descriptions.
- This main baseline uses Kotlin 2.3.21 and Gradle **9.7.1**, unlike the mainline
  3.5.0 validation's Gradle 9.4.1. Use this branch's own wrapper/configuration;
  do not copy or alter the other Session's cache or build environment.
- Prepared candidate: 2 files, 43 added lines, no deletions.
  The 10 production lines preserve the mainline patch's logic.
  The 33 test lines add separate IPv4/IPv6 tests with one private assertion helper.
  Nested `use` blocks close server, client, and accepted connection on failures.
  Connect completes against the already-listening socket before `accept`, avoiding
  an async accept child whose socket ownership needs separate cancellation handling.
- [Production candidate](file:///home/stream/ACodeSpace/fork/ktor/ktor-network/posix/src/io/ktor/network/sockets/TCPServerSocketNative.kt#L62-L83);
  [IPv4/IPv6 candidate tests](file:///home/stream/ACodeSpace/fork/ktor/ktor-network/nix/test/io/ktor/network/sockets/tests/TcpSocketTestNix.kt#L19-L50).
  `git diff --check` passed; no build/test/lint/ABI task was run.
  The tests remain `nix`-only; Windows compilation/coverage is still a gate.

## Push and browser handoff

- User-authored commits:
  - `c0aafe150f40da339e90616763bf4a5f37b7615f`:
    `fix(network): align native TCP_NODELAY option to JVM`.
  - `32a0a7b5705f41ecdfde4bac4d1d0a15f74703dc`:
    `test(network): align native TCP_NODELAY option to JVM`.
- Explicit push:
  `git push --set-upstream origin HEAD:refs/heads/fix/native-accepted-tcp-nodelay-867`.
  Successful new branch on `Stream29/ktor`; local tracking now uses origin's
  same-name branch, not upstream/main. No force push or commit rewrite.
- Opened a new browser tab for the
  [PR creation form](https://github.com/ktorio/ktor/compare/main...Stream29:ktor:fix/native-accepted-tcp-nodelay-867?expand=1).
  Verified base `ktorio/ktor:main`, head
  `Stream29/ktor:fix/native-accepted-tcp-nodelay-867`, automatic mergeability,
  and 2 commits / 2 files / 43 additions / 0 deletions.
- Filled title and description only. Browser tab ID56 is handed to the user;
  the agent did not click Create pull request. Do not close or change that tab.
- Validation remains unverified on this main-based branch; initial form text
  explicitly states this and discloses AI assistance. No private evidence paths,
  credentials, or application-session data were put in the form.

## PR form initial text — snapshot before user edits

Title: **fix(network): align accepted Native TCP_NODELAY defaults with JVM**

**Subsystem**

`ktor-network`, Kotlin/Native server sockets.

**Motivation**

The JVM implementation enables `TCP_NODELAY` for accepted INET sockets, while the
Native implementation only configures them as non-blocking. This leaves different
defaults across the two implementations and can add latency to workloads with
repeated small writes through interaction between Nagle's algorithm and delayed
acknowledgments.

**Solution**

- Enable `TCP_NODELAY` on accepted Native IPv4/IPv6 TCP sockets, matching the existing JVM behavior.
- Keep the `InetSocketAddress` guard so Unix-domain sockets are not given TCP-specific options.
- Configure the option inside the existing `buildOrCloseSocket` block, preserving descriptor cleanup if configuration fails.
- Add IPv4 and IPv6 loopback tests that establish real connections and use `getsockopt` to assert that the accepted socket has `TCP_NODELAY = 1`.

Client socket options and public API signatures are unchanged.

**Validation**

The regression tests have been added but have not yet been run on this main-based
branch. Native/JVM tests, Unix-domain regression coverage, affected-target
compilation, formatting/linting, and ABI checks remain to be verified. No
cross-platform test pass is claimed.

**AI assistance**

AI assistance was used to prepare the code, tests, and initial PR description, in
accordance with the [AI Usage Policy](https://github.com/ktorio/ktor/blob/main/AI_POLICY.md).

## PR5949 title-check failure and YouTrack draft

- User submitted [PR5949](https://github.com/ktorio/ktor/pull/5949) after editing
  the initial form. Do not treat the local initial text as the current public body.
- Read the actual failed
  [check log](https://github.com/ktorio/ktor/actions/runs/38719668120/job/116370834407):
  `Validate PR Title / Check YouTrack issue ID` failed because the title does not
  begin with `KTOR-<number>`. This is not a compiler/test failure.
  At the inspected status snapshot, Native Linux/Windows/macOS, ABI, style,
  and other TeamCity checks were pending; wrapper validation passed.
- The initial review missed the mandatory workflow
  [title check](file:///home/stream/ACodeSpace/fork/ktor/.github/workflows/pr-title.yml#L23-L45).
  It accepts `KTOR-1234: ...` or `KTOR-1234 ...` and verifies the issue exists.
  `chore:` is exempt only for maintenance changes; do not misclassify this fix.
  PR title edits trigger the workflow, so no new source commit is needed.
- Repeated public YouTrack searches for TCP_NODELAY and Nagle found no same-root-cause
  issue. Searching 5949 returned unrelated KTOR-5949, not an existing issue for
  GitHub PR5949. The new-form similarity panel also reported no similar tickets.
- Prepared
  [YouTrack draft](https://youtrack.jetbrains.com/newIssue?project=KTOR&draftId=25-8695616)
  in new browser tab ID57, project Ktor, type Bug, platform Native.
  Title: **Native accepted TCP sockets do not enable TCP_NODELAY, unlike JVM**.
  This Session did not click Create; the user later created KTOR-10005.
- Draft includes the Native/JVM source comparison, Linux 3.5.0 diagnostic ablation,
  explicit limits on latest-release/runtime evidence, the minimal Ktor-only
  IPv4/IPv6 regression, PR5949, and tool-assistance disclosure.
  After the user's request to shorten it, removed repeated expected behavior,
  low-level test narration, and commit/cherry-pick instructions.
  It contains no private paths or credentials.
- Confirmed the user-created ticket's summary and PR5949 link through the public
  YouTrack API. Updated and reread the PR title:
  `KTOR-10005: Align accepted Native TCP_NODELAY defaults with JVM`.
  The user's PR body and source commits were not modified.
- `gh pr edit` failed on deprecated Projects Classic GraphQL metadata; the
  title-only REST PATCH succeeded. After an initial TLS timeout, REST polling
  confirmed the new [Validate PR Title run](https://github.com/ktorio/ktor/actions/runs/38757618186)
  completed successfully. This does not establish other CI jobs passed.
- Preserve the user's PR prose. Do not post AI-authored conversational comments.

## PR5949 CI assertion follow-up — 2026-10-16

- Session883 received the user's request to repair this PR. Scope is the shared
  IPv4/IPv6 regression assertion in the primary fork's existing contribution branch.
- [macOS build 453305](https://ktor.teamcity.com/buildConfiguration/Ktor_KtorMatrixNativeMacOSArm64/453305)
  reports both new tests failing with `Expected <1>, actual <4>`; the same tests
  passed in [Linux build 453304](https://ktor.teamcity.com/buildConfiguration/Ktor_KtorMatrixNativeLinuxX64/453304).
- Apple XNU returns `t_flags & TF_NODELAY` for this option and defines that flag
  as `0x00004`: [getter](https://github.com/apple-oss-distributions/xnu/blob/main/bsd/netinet/tcp_usrreq.c#L2667-L2671),
  [flag](https://github.com/apple-oss-distributions/xnu/blob/main/bsd/netinet/tcp_var.h#L407).
  Changed the assertion to `assertNotEquals(0, value.value)` with a short Darwin
  explanation; the `getsockopt` success check and production code remain unchanged.
- The initial PR/YouTrack drafts below retain their original wording as historical
  snapshots; the enabled-option contract is now nonzero, not necessarily 1.
  Do not overwrite the user's published prose or post generated comments.
- Native/JVM tests, assemble, formatting and lint tasks have not been rerun for
  this edit. The local active Gradle daemon is 9.6.1, whereas this fork requires
  9.7.1; there is no compatible daemon to reuse. No competing build was started.
- `git diff --check` passed. The scoped diff changes only the enabled-option
  assertion and its explanatory comment; this is not a test-execution result.
- The user reviewed the message, explicitly selected omission of the issue ID,
  then authorized commit and push. Commit `69cecc35185978501db2322ad8f3594191e04208`,
  `test: fix macOS TCP_NODELAY assertion`, changes only the test file.
  Pushed without force to `origin/fix/native-accepted-tcp-nodelay-867`;
  the pre-existing `.idea/vcs.xml` change remains untouched and uncommitted.
  Await the updated PR's CI results; no post-fix test success is claimed.

## Condensed YouTrack draft

Title: **Native accepted TCP sockets do not enable TCP_NODELAY, unlike JVM**

### Problem

Native [accepted TCP sockets](https://github.com/ktorio/ktor/blob/4016bdfe32256ff82723af7259e294908e48be75/ktor-network/posix/src/io/ktor/network/sockets/TCPServerSocketNative.kt#L62-L73)
do not enable `TCP_NODELAY`, unlike the
[JVM implementation](https://github.com/ktorio/ktor/blob/4016bdfe32256ff82723af7259e294908e48be75/ktor-network/jvm/src/io/ktor/network/sockets/ServerSocketImpl.kt#L46-L56).
Repeated small writes can therefore incur latency from Nagle's algorithm and delayed acknowledgments.

On Ktor 3.5.0, Linux x64 Native (CIO + WebSockets), enabling this option only on
accepted connections reduced synthetic application startup from about 18.7 s to 0.7 s.
This was a diagnostic experiment, not a patched-source test.

### Reproducer and proposed fix

[PR #5949](https://github.com/ktorio/ktor/pull/5949) includes
[IPv4/IPv6 loopback tests](https://github.com/Stream29/ktor/blob/32a0a7b5705f41ecdfde4bac4d1d0a15f74703dc/ktor-network/nix/test/io/ktor/network/sockets/tests/TcpSocketTestNix.kt#L19-L49)
that bind, connect, accept, and use `getsockopt` to assert that the accepted socket's
`TCP_NODELAY` value is 1.

The fix aligns Native with JVM: enable the option for INET sockets only,
excluding Unix-domain sockets, inside the existing close-on-failure boundary.

### Validation

The omission is also present in 3.6.0 by source inspection.
Latest-release runtime reproduction and this main-based branch's test results
remain unverified here.

**AI assistance:** AI tools assisted with the candidate code, tests, and this description.
