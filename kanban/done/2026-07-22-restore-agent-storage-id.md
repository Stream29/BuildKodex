# Task Tree

- [done] 恢复由 `KodexAgentStorage` 实现派生的稳定身份
  - [done] 审计 `KodexAgentRuntimeId` 的引入动机、生命周期和调用链
  - [done] 明确五条 timeline 约束只限定 AgentStorage 文件夹结构
  - [done] 确定恢复 `KodexAgentStorage.id` 并删除临时 runtime 代际身份
  - [done] 恢复 `KodexAgentStorage.id` contract 与实现
    - [done] filesystem storage 使用规范化 storage 路径
    - [done] in-memory storage 使用对象 identity hash
    - [done] handle、wrapper 和 session node 转发底层 storage id
  - [done] 删除 `KodexAgentRuntimeId` 及 `KodexAgentState` factory 的 `runtimeId` 参数
  - [done] 恢复基于 `storage.id` 的稳定 OpenAI 请求身份投影
    - [done] 使用既定固定 provider 规则生成 wire `thread_id`
    - [done] 使用同一结果与 compaction window number 生成 wire `window_id`
  - [done] 清理 CLI、WebRun、artifact 和 UI 中对 runtime id 的依赖
    - [done] 使用 storage id 表达 storage-backed identity
    - [done] 使用 `ManagedAgent` 对象身份和 ownership 生命周期拒绝 stale callback
  - [done] 更新相关 contract、request projection、lifecycle 测试和设计记录
  - [done] 运行相关格式化、测试和检查

# Details

- 状态：已完成并归档。
- 本任务纠正[Session surface任务](../done/2026-07-22-redesign-session-surface.md)第一版实现中删除 `KodexAgentStorage.id` 并引入 runtime 请求命名空间的错误方向。
- `KodexAgentStorage.id` 是由后端实现派生的运行期属性，不增加 manifest、metadata 文件或第六条 timeline。
- filesystem storage 的规范化路径可跨 reopen 稳定恢复；in-memory storage 不跨进程存在，使用对象 identity hash。
- OpenAI wire identity 沿用[Agent持久化方案](../done/2026-07-21-plan-agent-persistence.md)中从本地 storage id 做固定稳定投影的设计，不使用每次 cold open 随机生成的值。
- 同一 storage reopen 或重建 model、tools、MCP 时保持同一身份；fork、新路径和新内存 storage 获得不同身份。
- delete 后路径复用依赖 close、取消、join 和 UI 状态清理保证旧 runtime 不可继续回调；不为生命周期清理错误增加跨层 generation token。
- `KodexAgentRuntimeId` 已删除；AgentState、CLI、WebRun与artifact统一使用storage-backed identity。
- Linux相关模块测试与IDE增量构建通过。
