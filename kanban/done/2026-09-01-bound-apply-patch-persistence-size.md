# Task Tree

- [done] 让 `apply_patch` 持久化大小只随解析后 patch 增长
  - [done] 确认目标文件快照的写入链路
  - [done] 量化现有 Session 的存储放大
  - [done] 确认以 parsed diff 作为历史展示事实源
  - [done] 审查实施与 migration 阻塞
  - [done] 收窄 patch 执行结果
    - [done] 从 `PatchApplyResult` 删除完整 delta
    - [done] 只保留实际 affected paths
    - [done] 删除持有文件内容的结果模型
    - [done] 保持 add、delete、update 和 move 应用语义
  - [done] 收窄稳定 patch 事件
    - [done] 保持 `patch_tool_event` 与 success/failure 类型名
    - [done] 保持 parsed `Patch` 和 affected paths
    - [done] 兼容解码带旧 `apply_result.delta` 的事件
    - [done] 保持模型上下文投影不变
  - [done] 改用 parsed diff 渲染历史
    - [done] 成功事件从 `diff.hunks` 生成 target 和正文
    - [done] 失败事件继续显示 parsed diff 和原因
    - [done] 删除 old/new content 的二次 diff 路径
    - [done] 更新 affected history 与 patch view tests
  - [done] 建立持久化大小回归保护
    - [done] 用固定 patch 修改递增的大文件
    - [done] 断言事件大小不随目标文件增长
    - [done] 覆盖 delete、overwrite 和 move overwrite
    - [done] 验证 stable JSON 不含完整文件快照
  - [done] 清理已有稳定事件
    - [done] 等待 [Kodex Home 版本化与启动迁移](../done/2026-09-01-version-kodex-home-for-startup-migrations.md)
    - [done] 为 migration 提供取消安全的原子 record 替换
    - [done] 在 `0.3.3` 登记 future migration
    - [done] 与同版本其他 migration steps 确定固定顺序
    - [done] 快速识别含旧 delta 的 patch records
    - [done] 原子删除 `result.apply_result.delta`
    - [done] 保留 parsed diff、affected paths 和其他 records
    - [done] 验证 migration 可重入并回收实际空间
  - [done] 更新稳定模型维护决策
    - [done] 明确 patch result 禁止保存目标文件快照
    - [done] 明确成功历史由持久化 parsed diff 渲染
  - [done] 完成验证
    - [done] 运行 patch utility 和 tool tests
    - [done] 运行 clean-model serialization tests
    - [done] 运行 history ViewModel 和 patch view tests
    - [done] 运行 migration fixtures 和重入测试
    - [done] 运行受影响 Gradle checks
    - [done] 完成静态与格式验证

# Details

- 状态：实施与验证完成，任务归档。
- 用户已确定：
  - 持久化 parsed diff，不保存完整旧文件或新文件。
  - 非 migration 部分可以先推进。
  - 历史数据通过版本化启动 migration 清理。
  - Migration 使用已经完成的 Home migration 基建。

## 实施结果

- `PatchApplyResult` 只保留 affected paths；完整 old/new snapshots 和相关结果模型已删除。
- `StablePatchToolEvent` 保留 parsed patch、success/failure 和 slim apply result。
- 成功历史与标题统一从 persisted parsed diff 投影。
- `0.3.3` future migration 顺序扫描 root Session 的 `work` timeline：
  - 先读取短前缀筛选 patch records。
  - 只对 legacy success records 删除 `result.apply_result.delta`。
  - 使用 sibling temporary、flush、atomic move 和不可取消 cleanup 替换 record。
  - 保留 slim、failure、non-patch、未知文件和 legacy `subagents/`。
- 稳定模型维护规则已禁止持久化 apply 前后完整文件快照。

## 验证结果

- `utils-patch`、`tool-apply-patch`、`agent-storage-clean-models`、
  `agent-storage-filesystem-layout`、`app-migration-impl` 和
  `app-viewmodel-history` 的受影响 checks 通过。
- `app-view-patch` 的 presentation tests 通过；完整 check 仍有 4 个 Mosaic
  交互测试失败，已在未修改的 clean HEAD 独立复现。
- Stable serialization 覆盖 production `OpenAiJsonCodec` 对 legacy delta 的兼容读取。
- Migration fixtures 覆盖 mixed records、重入、空间回收、stale temporary、
  malformed matching record 和未知文件保留。
- 基于隔壁 scoped filesystem API 重新编译并运行跨 JVM、JS 和 Linux Native 测试。
- 2,000 个 scan records 与 50 个 2 MiB legacy records 的本机 probe：
  - scan 约 11,701 records/s。
  - rewrite 约 353 MiB/s。
  - rewrite peak heap growth 约 103 MiB。
- `git diff --check` 通过。
- 未再次运行会修改 dirty workspace 的 IDEA CLI；IDEA lint/formatter 因已确认的
  `inspect -changes` headless unshelve 故障改为编译、测试和 diff whitespace 检查。

## 阻塞审查

- 非 migration 的模型、tool、history 和大小回归改造没有未决设计阻塞；取得 executable
  授权后可以先实施。
- Slim writer 可以先开发和测试，但不能先于 Home 版本保护单独发布：
  - 新 reader 可忽略旧 `apply_result.delta`。
  - 旧 reader 无法读取缺少 required `delta` 的新事件。
- 历史清理的 Home 版本表、write lease、启动接入和 release activation 已由依赖任务完成。
- Migration 已补齐安全 record rewrite：
  - `writeRecord` 当前直接截断目标 record：
    `Kodex/agent-storage/filesystem-layout/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystemlayout/FileSystemStorageLayout.kt:78`。
    Migration 需要临时 sibling、flush、atomic replace 和取消后清理组成的安全 rewrite。
  - 全局 migration 表要求每个目标 release 只有一个入口；本任务在 `0.3.3` 唯一 entry
    中固定 patch cleanup step。
- Migration 吞吐是完成门槛而不是当前设计阻塞：
  - 本机需处理约 6.2 GB 旧 patch records。
  - 每个 Session 顺序 rewrite 可把峰值内存和临时磁盘限制在单个 record。
  - 发布前必须 benchmark 独占启动迁移耗时；不能因总量较大改成无版本保护的后台 rewrite。
- 当前本机 bloated patch events 都位于 root Session 的 `work` timeline；legacy `subagents/`
  未发现 patch events。Migration 继续遵守 Home 规则，不修改 legacy `subagents/`。

## 问题与目标

- `PatchFileSystem` 对 update 构造完整 `originalContent` 和 `newContent`，并把两者放进
  `PatchFileChange.Update`：
  `Kodex/utils/patch/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/PatchFileSystem.kt:91`。
- `StablePatchToolEvent.Success` 原样持久化整个 `PatchApplyResult`：
  `Kodex/agent-storage/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/work/StablePatchToolEvent.kt:59`。
- 当前历史 UI 再用完整 old/new content 计算展示 diff：
  `Kodex/app/view/patch/src/commonMain/kotlin/io/github/stream29/kodex/cli/patch/PatchPresentation.kt:181`。
- 模型上下文只使用 `diff.patch` 和固定成功文本，不使用完整文件内容：
  `Kodex/agent-storage/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/work/StablePatchToolEvent.kt:34`。
- 本机调查时约 9,388 个 patch events 占用约 6.19 GB；保留事件中 parsed diff
  而删除大型 result 后估算约为 53.7 MB 加少量结果元数据。
- 修复后的单个稳定事件必须满足
  `O(serialized parsed patch + path metadata + fixed outcome metadata)`，不得随未出现在 patch
  中的目标文件内容增长。

## 目标模型

- 继续使用现有 `Patch`：
  - `patch` 保留模型生成的规范化 raw input。
  - `hunks` 保留 add、delete、update、move 和结构化 chunks。
  - raw input 与 parsed hunks 是常数倍重复，但仍只随 patch 增长。
- 将 `PatchApplyResult` 收窄为只包含 `PatchAffectedPaths`。
- 删除 `PatchDelta`、`PatchChange` 和 `PatchFileChange`。
- `Patch.applyToFileSystem` 在执行期仍可读取旧内容并构造新内容，但完成写入后不得把两者放入返回值。
- `StablePatchToolExecutionResult.Success` 继续保存 slim `apply_result`：
  - 保留现有 JSON 层级和 `success` discriminator。
  - `apply_result` 只包含 `affected_paths`。
  - 旧 `apply_result.delta` 由宽松 codec 忽略。
- `Failure` 继续只保存原因。
- `toResponseHistoryItems()` 保持当前协议投影，不改变模型可见 call/output。

## 展示语义

- 成功和 pending 事件共享 parsed patch 的文件与 chunk 投影。
- 成功事件仍显示完成状态，但正文表示成功应用的输入 patch，不再声称是从完整文件快照推导的
  exact filesystem delta。
- Target 使用 `diff.hunks.path`，保持当前 delta change source-path 语义；move 详情继续显示
  source 与 destination。
- Delete 只显示 patch 中存在的删除路径，不永久保存被删除文件正文。
- Add overwrite 和 move overwrite 不保存被覆盖内容；本任务也不新增与 patch 大小无关的审计快照。
- 本任务不引入独立 patch blob、压缩快照、内容寻址存储或历史回滚能力。

## 兼容与 migration

- 生产 codec 已设置 `ignoreUnknownKeys = true`：
  `Kodex/openai/json-codec/src/commonMain/kotlin/io/github/stream29/kodex/openai/jsoncodec/OpenAiJsonCodec.kt:5`。
- 新 reader 必须覆盖从旧成功事件解码并忽略 `apply_result.delta` 的 fixture。
- 新 writer 不再产生 `delta`；旧 reader 无法读取缺少 required delta 的新事件，因此 slim schema
  只能在 Home 版本保护与对应 release migration 一起发布。
- Migration 登记为 `0.3.3` future entry；当前 `0.3.2` 构建不激活，下一次 patch release
  version bump 自动启用。
- Migration 只处理 `work/*.json` 中满足以下条件的 records：
  - 顶层 `type == "patch_tool_event"`。
  - `result.type == "success"`。
  - `result.apply_result` 含 `delta`。
- Rewrite 只删除 `delta`，保留其他 JSON 语义。
- 不匹配 records 不解码完整业务模型，也不重写。
- 每个 Session 内顺序处理命中的大型 records，避免并发物化多个完整文件快照。
- 使用 migration 基建提供的安全 record rewrite；单个 record 成功替换后才算完成。
- 重跑时已无 `delta` 的 record 直接跳过。
- Migration 正常完成条件是所有当前 root Session 的 `work` timeline 都不再含旧 delta。
- 不在 migration 基建完成前增加临时 Python 清理脚本，也不修改本机 `~/.kodex`。

## 预计修改范围

- Patch 执行与模型：
  - `Kodex/utils/patch/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/PatchModels.kt`
  - `Kodex/utils/patch/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/PatchFileSystem.kt`
  - 对应 common tests
- 稳定事件与工具：
  - `Kodex/agent-storage/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/work/StablePatchToolEvent.kt`
  - `Kodex/tool/apply-patch/src/commonMain/kotlin/io/github/stream29/kodex/tool/applypatch/ApplyPatchToolClient.kt`
  - `Kodex/tool/apply-patch/src/commonMain/kotlin/io/github/stream29/kodex/tool/applypatch/ApplyPatchTools.kt`
  - 对应 serialization、tool 和 runtime tests
- History：
  - `Kodex/app/view/patch/src/commonMain/kotlin/io/github/stream29/kodex/cli/patch/PatchPresentation.kt`
  - `Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/HistoryItemHeaderFactory.kt`
  - 对应 ViewModel、presentation 和 Mosaic tests
- Migration：
  - 使用 `Kodex/app/migration/` 最终建立的版本化 migration 路径与 fixture 结构
  - 不提前修改当前活跃 migration task 的实现文件
- 维护决策：
  - `checklist/clean-model-rust-alignment.md`

## 验证边界

- Patch filesystem tests 必须从真实临时 filesystem 验证最终文件内容，不用 full-content result
  代替 filesystem 断言。
- Serialization tests 覆盖：
  - 新 success JSON 只有 parsed diff、affected paths 和 outcome。
  - 旧 success JSON 可由生产 codec 解码。
  - 新事件 round trip。
  - 模型历史投影保持不变。
- Size regression 使用相同短 patch 分别修改小文件和多 MB 文件：
  - 两个 stable JSON 的大小只允许 fixed metadata 差异。
  - JSON 中不得出现目标文件的未修改 sentinel 内容。
- Delete、add overwrite 和 move overwrite fixture 使用远大于 patch 的旧内容，验证 stable event
  不保留旧内容。
- Migration tests 使用真实临时 Home，覆盖 mixed old/new/non-patch records、重复执行、取消边界、
  malformed matching record 和未知文件保留。
- Migration benchmark 使用多 MB patch records 和大型 `work` directory，记录扫描吞吐、rewrite
  吞吐与峰值内存，不设置固定 CI 时间阈值。
- 实施时先完成非 migration 分支；整体任务在 migration、维护决策和全部验证完成前不得归档。
