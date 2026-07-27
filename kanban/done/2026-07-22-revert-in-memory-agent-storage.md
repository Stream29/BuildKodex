# Task Tree

- [done] 回退InMemoryCodexAgentStorage重构
  - [done] 盘点`MutableInMemoryStorageSnapshot`、`SafeRw`及相关修改范围
  - [done] 确定直接存储entries状态的最小回退边界
  - [done] 回退`InMemoryCodexAgentStorage`相关修改
    - [done] 删除storage级snapshot和`SafeRw`
    - [done] 让每个`InMemoryIndexVersioned`直接持有`MutableList<IndexedValue<T>>`
    - [done] 仅用writer mutex串行化`set`和`revert`
    - [done] 保留repository handle、`release()`和操作级补偿改动
    - [done] 删除in-memory模块的`utils-read-write-mutex`依赖
  - [done] 恢复并运行相关测试
    - [done] 保留现有补偿、revert和fork测试
    - [done] 运行in-memory的JVM、JS和Linux Native测试
    - [done] 运行agent-state JVM回归测试

# Details

- 已完成回退并通过目标平台测试。
- `IndexVersioned`以已发布的逻辑index提供快照；reader捕获稳定上界后直接读取entries。
- 每条timeline直接持有`MutableList<IndexedValue<T>>`。读-读和读-写不加锁，也不复制entries。
- 每条timeline仅通过writer mutex串行化`set`和`revert`，并原地追加或移除后缀。
- 回退范围限于`CodexLite/agent-storage/in-memory/src/commonMain/kotlin/io/github/stream29/codex/lite/agentstorage/inmemory/InMemoryCodexAgentStorage.kt`和`CodexLite/agent-storage/in-memory/build.gradle.kts`。测试只在必要时调整。
- 不回退repository接入、`CodexAgentStorageHandle`、`release()`或操作级补偿。
- 不修改`IndexVersioned`契约、filesystem实现、`SafeRw`工具模块或其他并行任务。
- 实施后使用GraalVM JDK 21运行`:agent-storage-in-memory:jvmTest`、`:agent-storage-in-memory:jsNodeTest`、`:agent-storage-in-memory:linuxX64Test`和`:agent-state-impl:jvmTest`。
