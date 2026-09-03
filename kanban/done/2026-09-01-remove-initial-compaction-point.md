# Task Tree

- [done] 移除 initial Compaction point
  - [done] 删除共享 `initialize()` 的 index `0` marker 写入
  - [done] 删除 in-memory initialized storage 的 index `0` marker
  - [done] 保留 settings、timestamp 和 token count 的 index `0` snapshot
  - [done] 保持真实 compaction marker 与 output 布局
  - [done] 保持旧 storage 的 initial marker 读取兼容
  - [done] 更新初始化、storage、AgentState 与 Session 测试
  - [done] 验证无 marker 时的 message window 与 History projection

# Details

## 已确定需求

- 新 storage 不再向 index timeline 的 index `0` 写入 `CleanCompactionPoint`。
- Settings、timestamp 和 token count 继续在 index `0` 初始化。
- 第一个真实消息或工具事件继续从下一个全局 storage index 开始。
- 真正发生 context compaction 时仍写入 `CleanCompactionPoint`。
- 不迁移或改写已有 storage。
- 读取旧 storage 时继续兼容 index `0` 的 initial marker。
- 用户界面忽略旧 storage 中 index `0` 的 initial marker。

## Implementation plan

- 从 `AgentStorageInitialization.initialize()` 删除初始 index marker 写入及其专用 import。
- 让 `InMemoryKodexAgentStorage(initialSettings)` 的 initialized state 使用空 index
  timeline；`empty()` 维持空 storage 行为。
- 更新依赖初始 marker 或旧 index 偏移的测试断言。
- 保留 `AgentStorageProjection` 对缺失 compaction point 的支持。
- 保留 `appendCompaction()` 写入真实 compaction marker 的逻辑。
- 保留 History 对旧 index `0` marker 的过滤和旧 storage 读取行为。

## Verification

已完成以下验证：

- `:agent-storage-contract-ext:jvmTest`
- `:agent-storage-in-memory:jvmTest`
- `:agent-storage-filesystem:jvmTest`
- `:agent-state-impl:jvmTest`
- `:agent-session-in-memory:jvmTest`
- `:agent-runtime-decorator-compact:jvmTest`
- `:agent-session-filesystem:compileTestKotlinJvm`
- `git diff --check`

`agent-session-filesystem` 的 JVM 测试 runner 在本机此前无法在合理时间内结束，
因此使用测试源码编译作为该模块的验证；相关 filesystem storage tests 已通过。
