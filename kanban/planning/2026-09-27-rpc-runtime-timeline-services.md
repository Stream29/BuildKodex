# Task Tree

- **`Await scoped implementation authorization`()**
- `Adapt six timelines and Session settings CAS`()
- `Project runtime state and bind current output streams`()
- `Connect backend-owned Agent commands and automatic titles`()
- `Verify real RPC lifecycle and cache interactions`()
- `Record evidence and complete this batch`()

# Details

## 位置与前置

- 实施批次 P05；父任务：[实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。
- 状态：Planning；依赖 [P01](2026-09-27-rpc-session-settings-cas.md)、[P02](2026-09-27-rpc-backend-session-lifecycle.md)、[P03](2026-09-27-rpc-backend-settings-models-mcp.md)、[P04](2026-09-27-rpc-backend-auth-oauth-usage.md)。
- 交付：六条 timeline 与完整 AgentRuntimeRpc 的真实后端实现；优先沿独立服务适配模块组织，不把业务代码塞进 transport。
- 规范：[分批实施边界](../../checklist/rpc-architecture.md#分批实施边界)、[timeline](../../checklist/rpc-timeline.md)、[AgentRuntime](../../checklist/rpc-agent-runtime.md)、[错误与订阅](../../checklist/rpc-state.md)。

## 实施范围

- 复用后端只读缓存视图，发布各自 nonce/latestIndex、六种查询；失活新访问/旧上游使用 SessionNotActive，nonce 不匹配使用 CacheNonceMismatch。
- SettingsTimelineRpc 委托 P01，其他五条只读；不新建索引真源、后端查询锁或缓存实现，保留最终一致性。
- getState/GetFlow 投影原状态，为原当前输出实例维护随机 nonce；currentFlow 校验并捕获同一实例，直接 replay+collect，不保存旧流 registry。
- 接 storageUri、Agent 全局 latestIndex、runningTurn、pendingSteer、shell 状态及命令；shell completed 变化不依赖成员变化。
- resume/forcedCompact 等委托 P02 的后端所有权并等待原 operation；Stop 只取消处理时当前 turn；手动压缩不补消费、不自动 resume。
- appendUserMessage、completeToolCall、clearPending、revertHistory 保留原准入/提交与异常；历史写入在实际边界核对 index timeline cacheNonce。
- 将自动标题从前端迁入 Runtime 子协程，append 成功后触发；沿原资格，使用同锁 settings CAS 和目标字段冲突规则，不补生成、不因标题失败否定消息提交。
- 入口：[契约](../../Kodex/rpc/contract)、[AgentState](../../Kodex/agent-state/impl)、[AgentRuntime](../../Kodex/agent-runtime/impl)、[原前端运行编排](../../Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt)。

## 测试载体

- “内存 RPC”只描述 transport，不要求后端存储也使用 InMemoryKodexSessionRepository。
- 原 [内存 Session timeline](../../Kodex/agent-session/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/inmemory/InMemoryKodexSessionRepository.kt#L263-L284)仅实现 MutableIndexVersioned，没有 CachedIndexVersioned 元数据；不能强转为 ObservableKodexAgentStorage 或用固定 nonce 补齐测试。
- 本批真实 timeline 往返使用隔离临时目录中的文件 repository，其 Session 已使用原缓存视图。只测协议/客户端边界时可复用明确的测试视图，但不得称为生产存储集成验证。
- 不为测试方便额外改造内存存储或两侧缓存算法；P02 的一般生命周期测试仍可使用原内存 repository。

## 验收

- 用真实内存 RPC、RestoringRpcClient 与 rpcCachedIndexVersioned 验证六条原值，包含 TokenCountSnapshot、null/空列表区别、稀疏范围、回退与重建。
- 旧 nonce 查询失败、迟到结果不能污染新缓存；普通只读竞争可最终收敛，不新增单查询快照保证。
- 新流完整 replay、同流 nonce 稳定、换流/结束旧 nonce 失败；取消收集不 Stop，Compacting 没有输出流。
- 请求等待取消后接受任务继续；显式 Stop 与释放槽位、TTL 失活结束全部旧订阅，不误关闭连接。
- 标题名称冲突放弃、其他字段变化可重试且不重跑模型；消息失败不启动标题，标题失败不撤回消息。
- 原工具完成、回退、压缩、steer/shell 行为回归；JVM/Native 实际运行并检查四种已知异常还原。

## 交接

- P06 复用本批命令做批量创建/运行交接；P07 接停止观察事件；P08 使用这些服务构建前端视图。
- 本批可独立注册 Runtime/timeline 做测试，不提前启动未完整的 GlobalRpc 或切换 CLI。
