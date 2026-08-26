# Task Tree

- [done] 为 filesystem `CachedIndexVersioned` 增加主动 value TTL
  - [done] 封装每条 timeline 的缓存资源边界
    - [done] 从 Agent owner scope 创建私有 child scope
    - [done] 在 child scope 中启动周期 cleanup Job
    - [done] child scope 完成时立即清空 decoded values
  - [done] 增加固定缓存过期语义
    - [done] 保留既有 entry capacity 和 LRU 淘汰
    - [done] 固定使用 60 秒 expire-after-access
    - [done] 固定每 60 秒主动触发 expiration cleanup
    - [done] 将 cleanup 细节封装在 `CachedIndexVersioned` 内
  - [done] 保持并发和存储语义
    - [done] 保留同 key loader 去重
    - [done] 阻止 owner 关闭后的 loader 写回 value
    - [done] 保持 append、revert 和完整稀疏索引行为
  - [done] 补充主动过期回归测试
    - [done] 验证无后续 cache 访问时仍会主动移除过期 value
    - [done] 验证命中会刷新 expire-after-access 时间
    - [done] 验证容量淘汰与 TTL 可以同时工作
    - [done] 验证 owner 取消会停止 cleanup 并清空 value
    - [done] 验证 loader 与关闭竞争不会留下缓存 value
  - [done] 运行相关模块测试和格式检查

# Details

- 状态：`done`。已完成实现、测试和格式检查；未提交 Git。
- 用户已明确将原 History planning 收窄为 decoded value cache 的主动 TTL。原任务中的 History item 状态机、延迟加载、一行骨架、稳定 key 和无全局读取 semaphore 已由后续 History 重构实现，不再纳入本任务。
- 目标是让默认折叠后不再由 ViewModel 持有的巨大 tool event，在 value cache 超过空闲时间后失去最后一个长期强引用并允许 GC。
- 每个 `CachedIndexVersioned` 独立拥有私有 child scope、cleanup Job、expire-after-access 配置和关闭清理。`CachedAgentStorage`、Agent runtime 和 `IndexVersioned` contract 不感知具体淘汰机制。
- 生产配置固定为 60 秒 expire-after-access 和 60 秒 cleanup 周期，不增加 `IndexVersionedCachePolicy` 或用户配置。实际删除发生在最后访问后的约 60–120 秒。
- 保留既有 `valueCacheSize` 入口及默认 1,024-entry 容量。
- cache4k 0.14.0 只在 cache interaction 时执行惰性 expiration。`CachedIndexVersioned` 使用不可能成为 stored index 的负数 key 触发无副作用的周期 interaction，并用回归测试锁定主动清理行为；不向外部暴露 `cleanUp` 或 `invalidate`。
- owner scope 完成时立即 `invalidateAll()`。读取在 delegate loader 返回后再次检查 child scope，防止关闭后的 value 重新进入 cache。
- TTL 只管理 decoded timeline value；完整稀疏索引继续由 Agent storage 生命周期持有。
- 主要实现位置：
  - `Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt`
  - `Kodex/agent-session/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorageTest.kt`
- 验证命令：
  - `./gradlew :agent-session-filesystem:jvmTest --no-daemon`
  - `./gradlew :agent-session-filesystem:linuxX64Test --no-daemon`
- 不改变 History folding/rendering、AgentSession 淘汰、AgentState request projection、全局 cache budget 或 payload 序列化格式。
