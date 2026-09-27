# Task Tree

- **`Await scoped implementation authorization`()**
- `Add Session settings comparison at the existing write boundary`()
- `Verify response-header and settings write interleavings`()
- `Run affected JVM and Native regressions`()
- `Record evidence and complete this batch`()

# Details

## 位置与前置

- 实施批次 P01；父任务：[RPC 重构实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。
- 状态：Planning，尚未开始源码实施；依赖已提交的契约与 AgentState 基线，不依赖新的 RPC 服务。
- 交付：Session settings 的后端实际 CAS 原语，供 P05 委托；不重复实现已有 BackendSettingsStore 文件 CAS。
- 规范：[分批实施边界](../../checklist/rpc-architecture.md#分批实施边界)、[settings CAS](../../checklist/rpc-timeline.md#读取边界)、[写入串行化](../../checklist/agent-state-mutation-serialization.md)、[字段冲突](../../checklist/rpc-state.md#settings-的字段冲突)。

## 实施范围

- 在原 AgentState 的私有写入锁内读取当前完整设置、比较 expect，再按原路径追加 update；不在 RPC 层先比较再无条件 updateSettings。
- 保留运行中编辑设置的原准入：当前请求沿用已捕获快照，新设置供后续请求使用；不借用拒绝 RequestResponse 的 modify 改变该行为。
- mismatch 返回 false；同值成功不追加 settings/timestamp；合法提交继续发布原全局 latestIndex 和 timeline 元数据。
- 校验、I/O、取消不伪装成 false；内部 turn-state 等字段的合法性沿原领域约束处理，不为完整比较增加 revision 或忽略字段。
- 不公开 Mutex，不修改通用缓存/文件提交算法，不顺带改 plan 或工具完成的事务语义。
- 入口：[AgentState 实现](../../Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt)、[状态契约目录](../../Kodex/agent-state/contract)、[SettingsTimelineRpc](../../Kodex/rpc/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/SettingsTimelineRpc.kt)。

## 验收

- 旧 expect、相等但不同实例、同值更新及正常追加；比较失败/同值时历史文件与索引不变。
- 响应头先提交 turn-state 后旧 expect 失败；反向交错也不丢失最新值；不通过服务层专有锁绕过后台写入。
- RequestResponse 期间 CAS 不重置运行状态；已缓存 exact 旧记录不被原地覆盖。
- 写入失败、等待锁时取消与提交中取消保留原状态恢复/文件事务保证。
- 实际运行相关 JVM 与本机 Native 测试，并编译受影响目标；记录平台跳过，不用旧报告代替本批结果。

## 交接

- P05 使用此原语接 SettingsTimelineRpc，并迁入自动标题 CAS；P09 验证前端字段重试与回执不回写。
- 完成后更新父任务进度，移入 done 并修正引用；本任务不启动 CLI、迁移 Home 或自动提交。
