# Task Tree

- **`Await scoped implementation authorization`()**
- `Own the real repository in the backend scope`()
- `Separate accepted operations from RPC waiters`()
- `Connect Session renewal expiry and deactivation`()
- `Verify owner cleanup and repository reuse`()
- `Record evidence and complete this batch`()

# Details

## 位置与前置

- 实施批次 P02；父任务：[实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。
- 状态：Planning；依赖既有 repository、统一 runningTurn 和协程资源原语，可在注入 mock 领域依赖的隔离宿主中验证。
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
