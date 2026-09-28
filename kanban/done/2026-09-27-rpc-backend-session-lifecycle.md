# Task Tree

- `Receive the P01 handoff and review lifecycle ownership`()
- `Implement repository ownership and accepted-operation isolation`()`
- `Separate accepted operations from RPC waiters`()
- `Connect Session renewal expiry and deactivation`()
- `Verify owner cleanup and repository reuse`()
- `Record evidence and complete this batch`()

# Details

## 位置与前置

- 实施批次 P02；父任务：[实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。已获顺序实施授权，P01 已通过验收。
- 状态：Done；已通过隔离宿主与真实文件 repository 验证，不代表生产 CLI 已切换。
- 交付：后端 Session 所有权、已接受操作与等待者隔离、TTL 和失活信号；不提前替换现行 CLI 工厂。
- 规范：[分批实施边界](../../checklist/rpc-architecture.md#分批实施边界)、[实体与视图](../../checklist/rpc-session.md#repository-实体与前端视图)、[保活规则](../../checklist/rpc-session.md#保活与回收)、[参数](../../checklist/rpc-session.md#保活参数与范围)、[资源生命周期](../../checklist/coroutine-resource-lifecycle.md)。

## 实施范围

- 后端 scope 持有同一真实 repository，目录与业务服务共用它；复用原幂等 open 和登记信息，不增加第二份活跃真源、getActive 或 closeSession RPC。
- 显式保活完成激活后才返回；普通 live 访问只访问已活跃 owner，不因读取、订阅或目录快照而打开/续期。
- 已接受命令挂在所属后端/Session 生命周期，RPC 等待者只等待结果；保留完整 operation 等待语义和原运行准入，不改成启动即返回。
- runningTurn 接受时立即续期，存续及取消清理中每20秒续期；唯一期限为60秒，使用单调时间，激活完成给足期限。
- 只有前端保活和 runningTurn 续期；标题、历史命令、steer、shell、普通访问不增加驻留条件。到期、删除与重建协调原写入和清理。
- 暴露给后续适配的失活信号必须能让 Session 旧上游以 SessionNotActive 结束；不能只依赖 SharedFlow 自动完成或让旧流绑定新 owner。
- 宿主退出取消全部 owner，等待资源收尾；不改变原 shell/steer 消费、压缩或磁盘租约算法。
- 入口：[Session 契约](../../Kodex/agent-session/contract)、[文件 repository](../../Kodex/agent-session/filesystem)、[内存 repository](../../Kodex/agent-session/in-memory)、[运行组合](../../Kodex/agent-runtime/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt)。

## 验收

- 同 index 并发/重复保活复用同实例；失活关闭完成后重建，不出现两个占用同一资源的 owner。
- 取消等待与关闭前端观察不停止已接受运行；Stop 取消当前 turn，runningTurn 在实际清理结束后释放。
- 无前端长运行超过 TTL 仍存活；停止全部续期后沿最后期限回收，不从任务完成重新计时。
- 未消费 steer、运行/已完成 shell 均不阻止到期；清理释放进程、文件 lease 和子协程。
- 过期/续期、删除/激活、初始化失败、宿主取消等交错均有确定性测试；采用测试时钟，不靠长时间 sleep。
- JVM/本机 Native 实测，并保留原两种 repository 回归；不操作真实 Session 或 Home。

## 交接

- P05、P06 只复用此生命周期，不各建 owner 管理器；P08 持有前端视图而非实体。
- P11 才移交旧 CLI 的释放责任；不允许先去掉旧释放路径而尚无新回收机制。

## 启动核对

- P01 已验收，领域 CAS 通过原 runtime 委托可用；本批不重复增加比较接口，不修改冻结 RPC contract。
- 已核对两种 repository 的 `openRoots`、幂等 `open`、目录 isActive 与关闭路径；真实 repository 必须在后端 scope 内创建，不能从原前端工厂借来或再登记一份活跃真源。
- 运行槽位仍登记实际执行 Job；接入时必须让它属于后端/Session，而非 RPC 调用方。等待者取消与 Stop 是不同入口，取消清理结束前 runningTurn 仍需续期。
- 已在 [rpc/server](../../Kodex/rpc/server) 实现 scoped BackendSessionHost；真实 repository 在后端 scope 创建，接受操作独立于等待者，续期元数据仅引用原 owner，不增加目录真源。
- 到期/删除先取消并等待接受操作，再关闭原 Session，使操作收尾仍可访问存储；失活信号绑定原 owner，不把旧订阅接到重建实例。宿主退出等待子资源收尾。

## 验证记录

- 新增17项测试，覆盖并发激活、60/20秒时序、运行/取消清理自续期、普通操作不续期、等待者取消、手动压缩、实际测试 shell、删除与重建、初始化失败及真实文件租约释放。
- JVM/Linux x64 Native 各17项新测试实际通过；原文件/内存 repository 回归目标通过，报告分别37/20项。四目标库及测试源码编译通过，旧 CLI JVM 编译通过；macOS 沿原 cinterop 条件跳过。
- 首轮测试修正：跨协程异常不能用对象身份断言；文件 I/O 测试改用独立测试单调时钟，避免调度器自动推进触发非预期 TTL；测试进程输入等待使用受控关闭。临时诊断已移除，未修改原存储或 shell 算法。
- 未接入 RPC 服务、CLI、迁移或真实 Home；契约未改。后续按既定授权连续推进 P03，不重新等待批次确认。
