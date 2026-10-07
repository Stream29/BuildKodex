# REVIEW READY — independent static review

- **Verdict: no confirmed B1; proceed with controlled consumer validation only.**
- **B2: no precise small defect established in these four files.**
- **U: real resolution, model, compiler/ABI, runtime and sources gates remain open; not commit approval.**

# Task Tree

- `Verify the fixed four-file consumer integration`()
- `Review real repository routing, credentials and target dependency closure`()
- `Return scoped integration readiness and required runtime gates`()

# Details

- Parent: [Default binary consumer](2026-10-07-consume-verified-fork-packages-by-default.md);
  taskmaster: [Gradle development experience](../executable/2026-10-02-rescue-gradle-development-experience.md).
- Session 561; independent reviewer, not implementer. Only this report changed.
  No builds/tests, Python, IDE, device access, process control, credentials access,
  network operations or Git commands. No waiting for centrally running gates.
- Reviewed AGENTS, associated workflow/document/Gradle skills and relevant JVM SOP.
- Fixed candidate: [four-file snapshot](file:///tmp/kodex-default-binary-consumer-review-20261008).
  Rechecked manifest and all four file hashes twice: **all match**.
  `SHA256SUMS`: `863ceb96715a0d856daf9ea05904fefddcd8ab7850e484040e0be66cb9ac7f4f`.

| Candidate file | Verified SHA256 |
| --- | --- |
| `settings.gradle.kts` | `78fbc45a63250ce92cbce9d87fc21f0c1e5031248c60a3b0d8335000db0ed074` |
| `README.md` | `36b4f1ac075f03ba2348e2948e886e15485d6e3fb88b62a16e07fe6e4647b55a` |
| `gradle/libs.versions.toml` | `19cec0692ea3fb492d03858ce656be40e6d2507c02189d9f4b7445c23351bd8a` |
| `docs/gradle-development.md` | `33633f4760cde9049c1827451c0f952703f289ba555a34700011048abbf03260` |

- Base identity `49d78d99069104baa777ac5fd8a7db569fab8e8b` and unchanged
  wrapper/KGP/model/kRPC rules are supplied baseline facts, not newly extracted
  through Git. Non-candidate working-tree files were caller context only;
  unrelated dirty code is neither reviewed nor attributed to this candidate.
- Original-file links below map to the same numbered lines in the frozen
  [settings](file:///tmp/kodex-default-binary-consumer-review-20261008/settings.gradle.kts#L1),
  [catalog](file:///tmp/kodex-default-binary-consumer-review-20261008/gradle/libs.versions.toml#L1),
  [README](file:///tmp/kodex-default-binary-consumer-review-20261008/README.md#L13) and
  [development document](file:///tmp/kodex-default-binary-consumer-review-20261008/docs/gradle-development.md#L1).

## R — responsibilities and static result

- **Routing is ordinary and narrowly scoped.** Three exact fork groups are
  exclusive inside `dependencyResolutionManagement`; Google/Central remain for
  other groups. Plugin repositories are unchanged and separate. No default
  fork `includeBuild`, Lucene-core substitution, Maven Local, SDK special route,
  source fallback, second lock or dirty-source publisher appears in the candidate.
  [Settings L3–44](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L3).
- **PREFER_PROJECT is a conditional boundary, not itself a confirmed blocker.**
  With the supplied baseline's absence of explicit dependency repositories in
  production consumer projects, settings owns their fork resolution. A future
  repository added to a consuming project can displace that policy; verify actual
  repositories at resolution, rather than claiming universal authority.
  [Settings L15–43](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L15).
- **The known Node route does not establish fork fallback.** Cached official
  KGP 2.4.0 sources show root application, a non-transitive detached Node
  configuration and temporary Ivy content restricted to `org.nodejs:node`.
  The supplied root script has no fork dependency. Root repository presence
  alone does not propagate project repositories into fork-consuming subprojects.
  Do not switch to PREFER_SETTINGS without a real Node compatibility test.
  Cache source entries: `NodeJsRootPluginApplier.kt` L48–72,
  `NodeJsPluginApplier.kt` L44–49, `AbstractSetupTask.kt` L93–106/L131–148,
  `NodeJsSetupTask.kt` L24–30, in this
  [official source JAR](file:///home/stream/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-gradle-plugin/2.4.0/8cc8a31f5f34bbace6166c7ad0187724c345932f/kotlin-gradle-plugin-2.4.0-sources.jar).
- **Credentials stay ordinary and nullable during configuration.** Private
  `gpr.user`/`gpr.key` independently precede CI actor/token; explicit Basic auth
  belongs only to the fork repository. There is no credential provisioning or
  eager credential failure. Successful `help` cannot prove authenticated resolution.
  [Settings L21–32](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L21).
- **Catalog is the sole consumer pin.** Mosaic explicitly preserves upstream
  `0.19.0-SNAPSHOT` before `-kodex.78f94c4c2e13`; SDK and Lucene use
  `0.14.0-kodex.53f717661712` and `10.2.0-alpha14-kodex.549f8afc9570`.
  These match the supplied verified packages; group/artifact aliases remain intact.
  [Catalog L10–23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/gradle/libs.versions.toml#L10),
  [aliases L58–72](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/gradle/libs.versions.toml#L58).
- **Docs are appropriately limited.** Full targets stay unchanged; private
  read-only PAT, finite expiry/revocation and CI read permissions are described.
  Manual source opt-in requires genuine substitutions; SDK root-name repair and
  composite compatibility are explicitly separate gates, not promises that the
  old composite works. No GUI gain is claimed. No wording fix is required.
  [README L13–16](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/README.md#L13),
  [development L5–43/L57–76](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/docs/gradle-development.md#L5).
- **Minimum ablation:** routing/authentication, canonical pins and short onboarding
  each carry a real responsibility. Removing one breaks access, identity or usable
  instructions; no new abstraction/profile/publisher is needed for this batch.

## U — acceptance boundary for the coordinator

- Accept supplied prerequisites without reopening CI reviews: Mosaic run
  `37678581820` SUCCESS, three-host producer/consumer JNI21/FFM25/Native and two
  Windows Native regressions; 803 byte-exact files, manifest
  `c3a84f667256ec29ce70f49d873fbf71511e9417f8209c7cf83bcbfdaa53f88c`.
  Original SDK 453/453 and Lucene 133/133 audits are complete; historical failed
  writers remain failed, with no republish. These are not this reviewer's executions.
- Source-absent root `help` at 77s proves configuration only. Before approval,
  retain actual repository/coordinate/artifact evidence for JVM and Native, plus JS
  where a real consumer requires it; confirm no fork composite or fallback,
  and honest missing-credential/missing-pin resolution failures on uncached inputs.
- Verify actual plugin codeSource/classpath and Native compiler 2.4.0 consuming
  Mosaic's 2.3-built artifacts: representative compile/link and true runtime,
  not producer success or `help`. Preserve the model inventory of 203 build-script
  projects + 12 namespace projects + implicit root, targets/source sets and kRPC patch.
- Exercise existing responsibilities: Mosaic CLI/history rendering and testing,
  MCP stdio/HTTP client and real server tests, Lucene real indexing/query; verify
  matching sources artifacts/attachments. Caller anchors:
  [CLI L12–21](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/build.gradle.kts#L12),
  [MCP L7–18](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/build.gradle.kts#L7),
  [Lucene L10–25](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/impl/build.gradle.kts#L10),
  [integration tests L46–60](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/build.gradle.kts#L46).
- No forced FakeReleaseManager/widget-spec redesign. Actual IDE navigation/Sync
  benefits require later real IDE evidence; profile, build-logic, SDK root-name and
  KGP/Gradle 9.6.1 batches remain separate. Centrally running gates need not finish
  for this static review to be ready.
