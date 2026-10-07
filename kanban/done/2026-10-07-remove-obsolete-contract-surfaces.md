# Task Tree

- `Trace compatibility-only consumers and normative document gaps`()
- `Remove obsolete Session Settings API and retarget actual callers`()
- `Ablate unused helpers and correct authoritative KDoc`()
- `Publish deletion and test-preservation evidence`()
- `Receive authorization to seal and commit the scoped repair`()

# Details

- 协调者：真实 Path/Patch spec 已接管，三个无职责项目退役、项目总数 203；
  SafeRw 实际缓存调用保留。Session Settings VM 14 / View 8、Patch spec 23 /
  impl 21 项 JVM 通过。独立消融复审支持真实归属，不把 test marker 删除
  当业务变更；[最终统一证据](2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)。

- 用户授权修复后重新审查；[主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md)，
  基线 `6b7129fa`。[Settings B1-C](../done/2026-10-07-final-audit-settings-components.md)
  是闲置 API，不是第二套真实 VM。
- 独占 `app/component/session-settings` 全树与旧重载/effect 的纯测试调用方适配、
  `utils/{read-write-mutex,host-test-support,images,images-codec,patch}` 的确定闲置
  helper/API KDoc。主工程构建文件删除/依赖移动先 handoff，不跨线写 host Gradle。
- 删除 Rename compatibility effect/channel/producer/getter 和 obsolete constructor
  overload；真实调用直接 typed dependencies、selection.picker。保留当前 rename/
  cwd child、revision/cancel/FIFO 和全部真实 UI/源测试，不为了删测试改语义。
- 测试 consumer 只做该 API 构造/字段拼写替换，不能改前端线的新行为逻辑；
  与前端线会交叉的文件先列 handoff 由协调者统一适配。
- SafeRw 若确认仅测试使用且无真实承诺则消融并保留 actual Mutex 的测试。
  fixture 转发层若导出唯一必要 dependency 则直接 retarget 同一 spec，不生成接口；
  Gradle 删除与实际测试消费者清单交主协调者。
- Codec/transformer/filesystem glue 的真实异常、资源 borrow、Patch FS 操作 KDoc
  准确补齐，不恢复伪 Codec/PatchApplier 接口。实际操作如必须迁移契约先报告
  真实 API 与循环检查，不能仅为配对发明 interface。
- 不改 Home provider（协调者）、process/curl（平台线）、storage（State 线）。
  不运行 Gradle、提交/推送/IDE；写自己 handoff，中央验证/独立复验待执行。

## Early integration handoff — Session 530

- Cross-lane fixture files: `app/impl/rpc/.../RpcSettingsTest.kt`,
  `app/component/settings/impl/viewmodel/.../{SettingsViewModelTest,WorkingDirectoryOwnershipTest}.kt`,
  `app/component/settings/impl/view/.../{SessionRenameAdapterTest,SettingsPopupInputTest}.kt`.
  This lane changes only constructor/import syntax and picker access in these files;
  frontend behavior assertions and logic remain owned by the frontend lane.
- Removed public API: `SessionSettingsEffect` (including `RenameSession`),
  `SessionSettingsViewModel.effects`, `SessionWorkingDirectoryPicker.viewModel`,
  and the source/models/ownerScope factory overload. Keep rename `.viewModel`;
  picker consumers use `.selection.picker`. Production Application already uses the typed factory.
- Main-only Gradle handoff: retarget OpenAI client `commonTest` dependency from
  `:utils-host-test-support-impl` to `:utils-host-test-support-spec`, then delete
  `utils/host-test-support/impl/build.gradle.kts` (the forwarding project has no sources).
  Preserve MockEngine dependency and `OpenAiLoginClientTest`.
- SafeRw is **not unused** in the current source: `agent-session/impl/filesystem`
  `CachedAgentStorage.kt` imports it and `CachedIndexVersionedImpl.indexes` uses it.
  Retain helper and all helper/Mutex tests; original audit remains unchanged.
- Patch placement: the real `Patch.applyToFileSystem(root, fileSystem)` uses
  `CoroutineFileSystem` already API-exported by patch spec. Its default argument uses
  `SystemCoroutineFileSystem` from IO impl; that is the only placement obstacle.
  Moving the whole function would require spec → IO impl; splitting explicit-FS/default
  overloads changes the API and needs review first. No move or fake interface in this lane.
  Existing explicit dependencies are patch impl → patch spec + IO impl,
  patch spec → IO spec, IO impl → IO spec, IO spec → external IO/coroutines.
  No reverse dependency to patch and no cycle in this local closure; the obstacle is
  forbidden spec → impl direction, not a demonstrated cycle. Patch contract relocation
  is stopped for coordinator review; its current function and tests remain untouched.
- Settings/image implementation is ready; central tests and independent review remain pending.

## Implementation handoff — READY, centralized tests pending

- Inner HEAD checked read-only: `6b7129fa0e719dde8f88a0034b9d572679984a03`.
  All edits used `apply_patch`; no Gradle, IDE, device/window operation, commit, push
  or branch switch. No temporary files or acquired resources require cleanup.
- Removed exactly the Session Settings compatibility API listed above, from the original
  spec and sole implementation. The typed factory, exact target/source, child identity,
  revision admission, cancel-on-close FIFO, frozen-cwd reporting and close behavior
  are retained. Production renderer is byte-identical to the baseline.
- All old factory callers in the current Kotlin source use `SessionSettingsDependencies`;
  picker getter consumers use `selection.picker`. Cross-lane edits are syntax-only.
  `RpcSettingsTest` also contains a frontend-owned `RpcSessionDraft` constructor change;
  that is not this lane's behavior work and was not overwritten.
- Preserved all 14 ViewModel test declarations and all 7 renderer test declarations
  (parameterized cases still expand at runtime), and 3 spec tests. The former effect test
  is now `public close disposes the exact rename child without dispatching a rename`:
  retains exact-child closure, null handle and no rename assertions; removes only
  buffered notification collection/assertion. Only the two obsolete effect DTO invariant
  assertions were removed from spec tests.
- Image spec KDoc now attaches actual throwing conditions to MIME/header/data-URL
  guards, dimension validation and each `toPromptImage` overload. Transformer documents
  per-call resource ownership, non-universal error normalization and observed cancellation,
  not prompt interruption of synchronous codec work. No image algorithm or codec body changed.
- Codec IO KDoc records bounded reads, header-only inspection, borrowed filesystem/
  transformer, per-call handle closure, original failure propagation, result-metadata checks
  and raw replace/append writes without container validation or rollback.
  Missing documentation is not evidence of runtime image corruption.
- Added 8 source regression cases: 3 pure image cases (defensive copies, skipped transform/
  metadata rejection, propagated failures), and 5 IO cases (header/transform-required errors,
  codec failure/cancellation, bounded-read delegation, actual replace/append, write failure/
  cancellation). The bound test verifies delegation and exception identity, **not**
  a 1 GiB real-input overflow experiment or native codec resource-release acceptance.
- SafeRw's genuine consumer is present even in the pinned baseline:
  [CachedIndexVersionedImpl.indexes](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt#L137).
  Whole read-write-mutex tree is byte-identical; all 8 Mutex and 7 SafeRw test declarations remain.
- 在作者首次 handoff 时 Patch 仍未移动，normative placement 门槛当时未闭合。
  协调者后来直接移动
  [borrowed-FS API](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/PatchFileSystem.kt#L19)
  到 spec，仅保留 impl 中真实 system-FS 默认 overload；原 FQCN、算法和
  parser/matcher/real-FS 断言保留，没有复制第二算法或新增 PatchApplier。
- Static checks: scoped `git diff --check` clean; obsolete Settings type/channel/old
  constructor searches empty; actual production renderer, Mutex and Patch baseline byte
  comparisons equal. These are source checks, not compilation or runtime passes.
- Coordinator queue: perform the host-test Gradle handoff above; run Session Settings
  spec/VM/Mosaic, Settings root/ownership/rename/popup and RPC settings regressions,
  images spec + codec IO + actual host transformer suites, preserved Mutex/Patch suites
  and affected CLI/Integration/native compile checks. Independent re-review remains required.
- Original Settings/other-utils/platform audit reports were not edited. Keep this task
  executable until the placement decision, build integration, central results and review
  are recorded; do not archive it as a full contract-closure pass.
