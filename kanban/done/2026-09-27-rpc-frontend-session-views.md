# Task Tree

- `Receive P07 backend assembly handoff`()
- `Build frontend Session views and owned subscriptions`()
- `Connect history current output and settings projections`()
- `Migrate submission tool interaction and catalog consumers`()
- `Verify tab disposal recovery and delayed replies`()
- `Record evidence and complete this batch`()

# Details

## 位置与前置

- 实施批次 P08；父任务：[实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。已获顺序实施授权，等待前批完成。
- 状态：Done；依赖 [P05](../done/2026-09-27-rpc-runtime-timeline-services.md)、[P06](../done/2026-09-27-rpc-session-catalog-management.md) 及 [P07 完整后端装配](../done/2026-09-27-rpc-notification-hooks.md#前端接入前的后端装配)，复用 rpc/client 已完成组件。
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

- 原 Application 工厂（e6154848 的 Application.kt L354–395）按实体参数构建旧 ViewModel，旧 contract/renderer 消费 execution；[当前工厂](../../Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt)已由 P11 替换。本批隔离实施时，不能仅以“不选择新工厂”保证共享签名变更后旧 CLI 仍可编译。
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

## 实施与验证

- 新增 `app/viewmodel/rpc` 的 Session 注册、绑定、输出、历史消费者、草稿和目录视图；原历史与工具 ViewModel 增加只读数据/回调入口，旧 CLI 签名保留到 P11。前端未获得后端实体。
- 22 项真实内存 RPC 测试在 JVM/Native 通过，覆盖共享、20 秒续期、失活重建、关闭竞争、丢回执、输出 nonce/NoMatch、历史确认、shell、秘密回答及单次批量创建。
- 四目标库/测试源码编译、旧 CLI JVM 编译通过；历史模块两端各33项、Agent 模块两端各12项回归最终通过。macOS 按既有依赖条件跳过。
- 旧 Native 历史索引测试在组合构建中出现已知超时，单独原样重跑通过；没有据此修改旧测试或算法。新增确认测试曾错误假定 nonce 与 state 同时到达，已改为等待各自真实状态。
- RPC contract 未改；生产工厂尚未选择新路径。P09 接设置与认证，P11 统一替换共享 UI 签名/原实体构造与释放路径。
