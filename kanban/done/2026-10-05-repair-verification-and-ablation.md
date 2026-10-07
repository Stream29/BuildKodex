# Task Tree

- `Delete redundant codec, URL and Patch layers`()
- `Snapshot mock handlers at build`()
- `Reproduce misplaced SSE test dependencies and repair test classpath`()
- `Reproduce bounded-resource risks before changing semantics`()
- `Run centralized utility, component and downstream checks`()
- `Publish actual results and record user acceptance`()

# Details

- Parent: [Reacceptance and authorized repairs](2026-10-05-reaccept-spec-impl-refactor.md).
- 用户接受当前修复并授权分批提交；内层提交 `e89c5094`，本轮未推送。
- Coordinator-owned utility/test fixes; no source overlap with four repair Sessions.
- Removed unused PromptImageCodec, ExternalUrlOpener/HostExternalUrlOpener and PatchApplier/Impl.
- Real PromptImageTransformer carries transform/error guarantees; URL result and Patch DTO/parser/result remain meaningful specs.
- Patch keeps original functional filesystem operation, not a new injected class just to create interface usage.
- Mock build snapshots simple fallback and explicit full handler; removed dead constructor field.
- New simple/full builder-mutation cases verify old client remains independent; default/full precedence unchanged.
- SSE `compileTestKotlinJvm` actually failed on missing server symbols. Moved server dependencies from spec jvmTest to impl jvmTest.
- Utility regression batch succeeded: mock client-test, SSE impl, Patch spec/impl, images spec, images-codec impl.
- Read/write invalid-owner and History dormant-job release tests are reproduction gates: run red before repair, then green. Do not infer unrelated History stress-timeout root cause.
- Build JVM: `/home/stream/.gradle/jdks/eclipse_adoptium-25-amd64-linux.2`; no Daemon was running initially, selected last recorded JVM and reused resulting PID 2424920.
- Central commands use `--max-workers=1`; IDE remains untouched, no other Session builds.
- No commit/push, no unapproved retry/protocol/tool-wire behavior changes.

## Reproduction checkpoints

- SSE impl test compilation failed on missing Ktor server symbols before the dependency fix;
  the six-target utility regression batch then passed. This is actual compilation evidence,
  not an inference from Gradle file placement.
- Wrong-owner mutex unlock reproduced `Write` → `Free` before throwing. Check the non-null
  writer owner before releasing state; the eight mutex and seven safe read/write tests pass.
- Four History lazy leaf release cases actually failed: Message, Plan Update, Request User Input,
  Suggest Subagent Task. Their dormant Job remained attached after `release()`.
  All four now implement the existing internal release hook and cancel that Job.
  No new public wrapper/state, timeout increase or claim about the unrelated stress timeout.
- Central compilation corrected Shell's empty companion declaration so the serializer remains
  a separate public object. The original names and wire descriptor are unchanged.

## Completed central checks

- Final 37 JVM test targets passed; selected XML results total **827 tests, 0 failures/errors/skips**,
  including up-to-date results from earlier successful runs. Process spec is one NO-SOURCE target.
- Six additional consumer targets passed: notification **10**, Runtime Tool **12**, Context Prefix
  **3**, Path Picker VM **10**, filesystem lease **7** tests; Kodex Home impl is NO-SOURCE.
  The combined 43 targets cover **869** tests. CLI JVM main compilation also passed (not a smoke run).
- Mock snapshot **6**, SSE **7**, Mutex/safe read-write **15**, Patch spec/impl **23/21**,
  images **20**, codec **7**, URL **1** tests passed; original functional APIs remain authoritative.
- History VM **37** and View **49** passed, including four formerly failing dormant release cases.
- Hook/MCP burst input genuinely reproduced lost fields and now passes with latest-draft
  synchronous updates; tests cover uninvoked stale/closed and throwing updates.
- Integration JVM test-source compilation passed without running live integration tests.
- Shell/Process/OS JS and Linux x64 main compilation passed. Real JVM pipe/PTY/local MCP fixtures
  also passed; other-platform runtime/linking, CLI/live browser and unclosed audit experiments
  are still explicit limitations.
- Temporary build/reproduction logs are coordinator-owned and removed after recording results.
  Daemon reused and left available; no IDE/user-resource operations, commits or pushes.
