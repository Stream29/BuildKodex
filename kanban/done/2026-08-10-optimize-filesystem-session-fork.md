# Task Tree

- 优化 filesystem session fork
  - [done] 核对现有 session fork 与 filesystem storage 路径
  - [done] 确定同格式 filesystem storage 的字节复制边界
  - [done] 确定目标发布、缓存及失败恢复语义
  - [done] 确定跨后端 fallback 与格式兼容条件
  - [done] 确定验证范围与性能验收标准
  - [done] 接入统一 Fork range 与 repository materialization
  - [done] 实现 filesystem 原始文件 fast path
  - [done] 覆盖 prefix、rebase、fallback 与失败清理
  - [done] 完成 JVM、Linux X64 与 IDEA 验证

# Details

- 状态：`done`。
- 本任务是[统一 Fork materialization 批次](2026-08-27-unify-fork-materialization-batch.md)的 filesystem 子任务。
- 当前基线：
  - `Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt:318-335` 先创建并打开空目标，再调用通用 `forkTo`，最后追加 target-only 的 `[fork]` settings change point。
  - `Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/KodexAgentStorage.kt:176-200` 逐一复制六条 timeline。
  - `Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/IndexVersioning.kt:118-125` 对每个 stored index 执行对象级读取与写入。
  - `Kodex/agent-storage/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemIndexVersioned.kt:101-145` 因此会对每条记录执行 JSON 解码、重新编码和原子发布。

## Fast path

- filesystem repository 在目标 AgentSession 打开前 materialize 其直接 `FileSystemAgentStorage`：
  - 目标 cache 只在复制完成后扫描，不允许先建立空索引 cache 再绕过 decorator 写文件。
  - repository 只在六条 timeline 完成并校验后发布新的 `entries` 快照。
  - 任一复制、pointer 重建或后续 caller 初始化失败时删除完整目标节点。
- 同格式 filesystem source 通过内部 raw-storage capability 进入 fast path：
  - 完整 prefix 使用 `fromInclusive = 0`，原样流式复制范围内的数字 JSON 文件。
  - 活动窗口使用非零 `fromInclusive`，文件名按 `sourceIndex - fromInclusive` 重定位。
  - 非 compaction timeline 的 JSON 内容保持逐字节一致。
  - compaction checkpoint 需要重定位 `historyBaseIndex`，非零 range 只对选中的 checkpoint 做对象级转换。
- 每个目标数字文件先写唯一临时文件，完整关闭后再 atomic move 到 `<targetIndex>.json`。
- 不复制 source 的 `latest.json`；每条目标 timeline 从实际复制结果重建 pointer。
- 不复制 `lock.json`、`subagents/`、`archive.mark`、source identity、临时文件、未知文件或 descendants。
- `[fork] <boundary title>` 和 Subagent child settings 都是 materialization 后的产品语义写入，不属于 raw copy。

## Compatibility 与 fallback

- 当前格式没有独立 manifest/version；fast path 的兼容门槛是当前进程可识别的 `FileSystemAgentStorage` 数字 JSON layout 与六条已知 timeline。
- 不为本优化增加 storage format version 或迁移已有 Session。
- 非 filesystem source、无法安全取得 raw backing 的 decorator 或未来不兼容格式使用统一对象级 range copy。
- 对象级 fallback 直接写入尚未打开的目标 storage，仍保留跨后端 Fork。
- source writer 由上层命令串行化；fast path 只读取已捕获 exclusive boundary 前的 append-only 文件。

## 验证

- 使用真实临时目录，不增加 mock storage。
- 完整 prefix：
  - 六条 timeline 的 boundary、稀疏 index 和目标 pointer 正确。
  - 非规范但合法 JSON 的目标字节与 source 完全一致，证明未经过 codec。
  - source boundary 后文件、descendants、archive marker 和 lock 不进入目标。
- 活动窗口：
  - 文件名从零重定位，checkpoint 前文件不存在。
  - checkpoint 除 `historyBaseIndex` 外的 payload、prefix 和 window lineage 不变。
  - 目标打开后的 cache、`latestIndex` 与模型可见历史正确。
- Fallback 与恢复：
  - 覆盖 in-memory → filesystem 和 filesystem → in-memory。
  - 注入真实 filesystem delegate 的受控 I/O 失败，验证临时文件、目标目录和 inventory 均被清理。
  - 比较相同 fixture 的对象级与 fast-path 结果；性能验收使用“numeric payload 不解码、字节一致、复制文件与字节量受 range 限制”，不使用易波动的 wall-clock 断言。
- 运行 `agent-storage-filesystem`、`agent-session-filesystem` 及共享 contract 的 JVM/Native 验证，并执行 IDEA 检查与两层 `git diff --check`。
