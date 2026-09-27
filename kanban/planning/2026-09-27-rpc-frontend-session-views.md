# Task Tree

- **`Await scoped implementation authorization`()**
- `Build frontend Session views and owned subscriptions`()
- `Connect history current output and settings projections`()
- `Migrate submission tool interaction and catalog consumers`()
- `Verify tab disposal recovery and delayed replies`()
- `Record evidence and complete this batch`()

# Details

## 位置与前置

- 实施批次 P08；父任务：[实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。
- 状态：Planning；依赖 [P05](2026-09-27-rpc-runtime-timeline-services.md)、[P06](2026-09-27-rpc-session-catalog-management.md) 及 [P07 完整后端装配](2026-09-27-rpc-notification-hooks.md#前端接入前的后端装配)，复用 rpc/client 已完成组件。
- 交付：Session/Agent 本地 RPC 视图及现有业务消费者适配；生产工厂整体替换留 P11，不把真实 repository 注入前端。
- 规范：[分批实施边界](../../checklist/rpc-architecture.md#分批实施边界)、[Session](../../checklist/rpc-session.md)、[状态](../../checklist/rpc-state.md)、[timeline](../../checklist/rpc-timeline.md)、[运行投影](../../checklist/rpc-agent-runtime.md#前端执行投影)。

## 实施范围

- 本地 owner 管理订阅、缓存与保活；所有已打开持久化 tab 按 index 共享20秒循环，先等待激活，再初始化视图；草稿/目录不保活。
- 组合六条 rpcCachedIndexVersioned、全局 latestIndex、state、runningTurn、pendingSteer、shell；保持只读与原值类型，不复制后端实体接口。
- 独立收集 state 与 currentFlow；同 nonce 共享完整 replay，换流/结束取消旧上游；Compacting 只展示状态。
- 接历史窗口、token-count/上下文展示、标题/plan/当前 settings 与 tab 日期；不将不同流拼成强原子快照。
- 原 AgentExecutionState 退出迁移路径，控件按真实原语派生；本地操作等待不冒充后端 running。
- 替换提交编排：create、append、resume 分开；运行中 CAS 排队；工具草稿留前端，批量创建走单一 RPC 后独立提交父工具结果。
- 目录只全量 Get，菜单读条目日期；归档/删除/fork/历史目标确认走既有命令，不恢复原 repository 工厂/临时实体访问。
- 失活使旧绑定失效，仍存活视图重新保活并重建；SessionNotFound 停止缺失绑定恢复；主动关闭不再激活，不重放未知结果写命令。
- 入口：[Session ViewModel](../../Kodex/app/viewmodel/session)、[Agent ViewModel](../../Kodex/app/viewmodel/agent)、[历史 ViewModel](../../Kodex/app/viewmodel/history)、[Application ViewModel](../../Kodex/app/viewmodel/application)、[RPC client](../../Kodex/rpc/client)。

## 分批编译边界

- 原 [Application 工厂](../../Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L354-L395)仍按实体参数构建现有 ViewModel；旧 contract/renderer 还消费 execution。仅说“不选择新工厂”不能保证直接修改共享签名后旧 CLI 仍可编译。
- 本批交付可在隔离装配中使用的新视图/工厂及必要消费者适配；删除旧共享签名、旧 execution 消费和旧实体构造路径的破坏性步骤集中到 P11。
- 允许仅供旧入口使用的短期签名适配，不给新视图注入真实实体，不要求新路径伪造 AgentExecutionState；避免复制整套 ViewModel/renderer 或引入用户可见切换开关。
- 每个阶段都编译仍使用旧入口的 Application/CLI。不能独立保持编译的消费者替换与其旧入口删除一并交给 P11，不将编译失败留给后批“自然修好”。

## 验收

- 空白草稿不创建 Session；已创建后的提交失败保留实体/内容，迟到回执不覆盖新订阅状态。
- 多 tab/后台 tab、同 index 多观察只共享必要订阅/保活；关闭一个不关闭后端或共享 client。
- 状态切换时输出 replay 不串流，历史追加/回退/owner 重建后旧缓存和迟到结果被拒绝。
- 失活恢复与页面关闭竞争不复活已关视图；异步 stateIn 错误由 owner 处理，不让 UI 永久持有无效绑定。
- shell 完成但仍登记可见；用户回答、secret answer 展示和建议子任务确认沿原约束。
- 用真实内存 RPC 与 mock 运行测试前端行为；更新旧“关闭 tab 必关 repository”的预期，仍验证本地释放完成。

## 交接

- 当前 settings 投影和 CAS 入口供 P09 复用；P11 切换全部生产工厂，禁止只切释放方法而保留直接实体依赖。
