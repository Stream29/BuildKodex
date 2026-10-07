# Task Tree

- `Extend only the SDK publication closure for its real Node consumers`()
- `Add offline closure fixtures and a real JS consumer compile gate`()
- `Hand off the delta for centralized validation and independent review`()
- `Complete the scoped publication/Node validation and hand off consumer acceptance`()

# Details

- Parent: [Fork package CI implementation](2026-10-07-implement-fork-package-ci.md).
- Scoped completion: user-approved SDK JS closure is published and independently
  reviewed. [Actual three-host run 37607943182](https://github.com/Stream29/Kodex/actions/runs/37607943182)
  passed producer, merge and all consumer gates, including real Linux Node
  execution. The writer's historical failed status is unchanged; the subsequent
  read-only original-bundle audit verified all 453 remote files byte-exact.
  Audit digest: `3c53d6c85c578c9735ff68389a69e6ec176b4d245b60d763693c43a354af586d`.
  No repair PUT/DELETE or changed-recipe republish was performed.
- Earlier worker “not run/pending” bullets below are retained handoff history.
  Actual Kodex JS/default-consumer integration remains in the
  [separate consumer task](2026-10-07-consume-verified-fork-packages-by-default.md).
- 用户已批准 MCP SDK 额外保留 JS；原发布实现交接是五目标基线。
  这条增量只修正 SDK 的真实发布闭包，不扩大其他 fork 或 Wasm/iOS 支持面。
- Own `scripts/fork-packages/` SDK target-aware contract, task list, JS smoke and
  focused tests/README, plus the MCP workflow only if its host stages need changes.
  Preserve all unrelated original workflow permissions and publisher behavior.
- SDK JS requires real core/client/server JS payloads, root redirects, module/POM
  dependencies and source attachments. Do not advertise a target merely by adding
  a metadata name or test stub. Compile a JS consumer using the actual SDK API;
  runtime-only environments must not silently replace the declared JS gate.
- No Gradle/test/IDE/runtime or resource operations by the worker. Coordinator
  alone validates remotely, deploys workflows and performs Git operations.

## Integration handoff

- **INDEPENDENT IMPLEMENTATION READY / VALIDATION PENDING — Session 552.**
- Worker used read-only source/archive inspection and `apply_patch` only for edits.
  No tests, Gradle/Node/IDE/runtime/resource operations, network writes, Git commands,
  commits/push, branches, credentials, fork-source/provider edits or review-report edits.
- Changed Kodex paths: `scripts/fork-packages/{contract.py,pipeline.py,smoke.py,
  publisher.py,publish.init.gradle,test_publication.py,README.md}` and
  `.github/workflows/fork-packages-mcp.yml`, plus this task handoff.
- MCP only: targets `jvm/linuxX64/linuxArm64/macosArm64/mingwX64/js`.
  macOS owns `root/jvm/macosArm64/js`; Linux owns `linuxX64/linuxArm64`;
  Windows owns `mingwX64`. Non-SDK target/task closure stays unchanged.
- For each `:kotlin-sdk-core`, `:kotlin-sdk-client`, `:kotlin-sdk-server`:
  - Mac: `publishKotlinMultiplatformPublicationToForkStagingRepository`,
    `publishJvmPublicationToForkStagingRepository`,
    `publishMacosArm64PublicationToForkStagingRepository`,
    `publishJsPublicationToForkStagingRepository`.
  - Linux: `publishLinuxX64PublicationToForkStagingRepository`,
    `publishLinuxArm64PublicationToForkStagingRepository`.
  - Windows: `publishMingwX64PublicationToForkStagingRepository`.
- Total SDK publication tasks: 21 (Mac 12, Linux 6, Windows 3).
- Linux SDK smoke keeps `verifyForkJvm`, `runDebugExecutableLinuxX64`,
  `compileKotlinLinuxArm64`; adds `compileKotlinJs`, `jsNodeDevelopmentRun`.
- Contract extension binds per-fork target/host closure into guard identity
  and manifests, preserves SDK root JS redirects, and requires real JS KLIB,
  sources, POM/GMM dependency closure. No fake publication generation.
- Actual read-only evidence: current SDK JS/main KLIB has
  `builtins_platform=JS`, `compiler_version=2.4.0`, no `native_targets`;
  SDK Mac Native/main has `builtins_platform=NATIVE`,
  `native_targets=macos_arm64`, `compiler_version=2.4.0`.
  Bundled stdlib/test KLIBs can instead contain `2.4.0-dev-8449`.
  Do not infer a library's compiler identity from its Maven version or stdlib:
  the implementation reads compiler archive version resources separately
  for JS and Native and compares publication manifests against those.
- Existing immutable versions retain the same names; the coordinator confirms
  they have not been published. Old five-target bundles must fail the new SDK
  contract; immutable writer behavior/security and old review snapshots stay intact.
- `publisher.py` delta is strict receipt gate matching only; HTTP upload/preflight,
  immutable semantics and original loopback writer tests are unchanged.
- SDK standalone init recipe applies the already approved isolated root-name
  correction to `kotlin-mcp-sdk-fork`, preventing the `getKotlinSdk()` collision.
  It does not add a fork lock, source-provider layer or edit fork source files.
- Linux Node setup is standard `actions/setup-node@v4`, pinned `22.14.0`;
  provided host Node must be at least `22.14.0`. KGP `NodeJsEnvSpec` uses the
  provided executable, download disabled. Missing Node cannot skip JS compilation.
- JS consumer uses actual accepted SDK types/constructors: coroutine stdio
  source/sink/error, `Client`, `Implementation`, `JSONRPCNotification`, `Server`,
  `ServerOptions`, `ServerCapabilities`, `StdioServerTransport`, generated
  `LIB_VERSION`, and `StreamableHttpClientTransport` borrowing ordinary
  `HttpClient { install(SSE) }` with `ktor-client-js:3.5.1`.
  Node performs real SDK stdio send/flush/cleanup; no process/HTTP integration
  claim. JVM and Native probes/gates remain intact.
- Offline tests written for three JS modules' real structural payload/source
  requirements, root available-at/attributes, core POM/GMM dependency versions,
  compiler/platform distinction/CRLF, SDK Mac authority, target-bound guard identity,
  task-matrix drift, provided Node minimum and receipt refusal without JS compile.
  Fixtures are explicitly structural, never production publication payloads.
- Coordinator entry points:
  - [Contract/host closure](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/contract.py#L24).
  - [Task sets/compiler archive evidence](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/pipeline.py#L108).
  - [Real JS consumer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/smoke.py#L70).
  - [Offline SDK fixtures](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/test_publication.py#L324).
  - [Commands and full host gates](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/scripts/fork-packages/README.md#L58).
- Central command, from an authorized isolated checkout:
  `PYTHONDONTWRITEBYTECODE=1 python -m unittest discover -s scripts/fork-packages -p 'test_*.py' -v`.
- Remaining gates (not run): offline tests/YAML validation; SDK original wrapper
  and real Mac root/JVM/Mac/JS generation; Linux/Windows Native publication;
  compiler archive locations/resource identity on all hosts; real JS link/Node
  lifecycle and Ktor default engine; original three-host probes and immutable
  remote writer tests/full CI; actual Kodex JS/default-consumer acceptance.
  All remote build/CI/Git/consumer/version/credential operations remain coordinator-owned.
