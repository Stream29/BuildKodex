# Task Tree

- 优化 filesystem session fork
  - [done] 核对现有 session fork 与 filesystem storage 路径
  - 确定同格式 filesystem storage 的字节复制边界
  - 确定目标发布、缓存及失败恢复语义
  - 确定跨后端 fallback 与格式兼容条件
  - 确定验证范围与性能验收标准

# Details

- 状态：`await discussion`。
- 当前不进入规划或实现。
- `Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionRepositoryViewModel.kt:137-151` 先创建并打开空目标，再调用通用 `forkTo`，最后追加 target-only 的 `[fork]` 标题 settings change point。
- `Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/KodexAgentStorage.kt:176-199` 逐一复制六条 timeline。
- `Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/IndexVersioning.kt:118-124` 对每个 stored index 执行对象级读取与写入。
- filesystem 实现因此会在 `Kodex/agent-storage/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemIndexVersioned.kt:101-122` 对每条记录执行 JSON 解码、重新编码、临时文件写入和 atomic move，而不是复制原始文件字节。
- 目标在复制前已经打开并建立空索引缓存；直接绕过 decorator 写文件会使缓存失真。候选路线是在 filesystem repository 创建路径中先复制 prefix，再打开目标并建立缓存。
- filesystem fast path 只复制六个 timeline 目录中 index 小于 exclusive boundary 的数字 JSON 文件；不复制 `lock.json`、`subagents/`、source identity 或 descendants。
- `[fork] <boundary title>` 仍是 fork 后的独立语义写入，因此该路径是 timeline prefix 原始字节复制加一次 settings 写入，不是完整目录复制。
- 同 backend、同 storage format 时使用字节级 fast path；跨后端或格式不兼容时保留现有对象级 `forkTo`。
- 文件发布仍需遵守临时目标加 atomic move 的现有可见性边界。`CoroutineFileSystem` 当前没有 copy 原语，但 `CoroutineRawSource.copyTo` 可提供不经 JSON codec 的流式 fallback。
- 原持久化设计明确优先保留跨后端 `forkTo`；当前缺口是没有为同格式 filesystem storage 增加 specialization，而非 fork 语义错误。
