# Task Tree

- [done] 缩短 Responses 请求的全局写锁生命周期
  - [done] 重构请求准入、落盘与恢复临界区
  - [done] 覆盖请求期间 settings 写入
  - [done] 覆盖并发非法状态与取消恢复
  - [done] 更新 AgentState 写入串行化决策
  - [done] 运行相关 Gradle 检查

# Details

- 仅修正 `requestResponseApi()` 的锁生命周期。
- 请求开始时在全局写锁内校验并发布 `RequestResponse`，网络和流式读取期间释放锁，每次 storage 写入及最终恢复时重新取得锁。
- `RequestResponse` 状态负责拒绝冲突的会话操作；`updateSettings` 接口升级与 compaction 锁生命周期不在本任务范围。
- 验证以 `agent-state-impl` 的公共测试目标和代码格式检查为主。
- 实现已获用户明确授权。
- `:agent-state-impl:jvmTest` 与 `:agent-state-impl:allTests` 验证通过；当前 Linux 主机按配置跳过 `mingwX64Test`。
- IntelliJ 定向构建与 `git diff --check` 验证通过。
