# Task Tree

- **`Await scoped implementation authorization`()**
- `Build catalog snapshots from the shared backend repository`()
- `Adapt Session creation archive fork and deletion`()
- `Connect suggested-task batch creation and execution handoff`()
- `Verify persistence lifecycle and unknown-result boundaries`()
- `Record evidence and complete this batch`()

# Details

## 位置与前置

- 实施批次 P06；父任务：[实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。
- 状态：Planning；依赖 [P02](2026-09-27-rpc-backend-session-lifecycle.md) 和 [P05](2026-09-27-rpc-runtime-timeline-services.md)。
- 交付：GlobalRpc 的九项 Session 能力，包括保活、目录、创建/批量创建、归档、两类 fork 和删除。
- 规范：[分批实施边界](../../checklist/rpc-architecture.md#分批实施边界)、[Session](../../checklist/rpc-session.md)、[历史操作](../../checklist/rpc-timeline.md#历史操作)、[turn-state](../../checklist/codex-turn-state.md)。

## 实施范围

- catalog 每次盘点同一后端 repository，先过滤 archived，再取元数据并沿原顺序排序；补上 createdAt 实际填充，updatedAt/running/isActive 来自后端当次事实。
- 不激活全部目录；保持单用户、小总量、全量查询边界，不加入分页或目录推送。
- keepSessionAlive 委托 P02；createSession 保留初始化校验与 `Session <index>` 命名，不同时提交首条消息。
- 归档只改标记；delete 关闭目标 owner 后删数据，缺失 false；其他需要确认不存在的管理操作用 SessionNotFound。
- 完整 fork 与边界 fork 复用原复制/准入/失败目标清理；边界标记在实际源快照边界校验，生成新当前身份而不改写继承历史。
- 批量创建使用原 tasks+initialSettings，后端命名、提交并交接运行，按输入顺序返回 index+meta；不等待孩子运行结束。
- 未知创建失败仅尽力减少副作用；不新增整批回滚、部分结果协议、自动重放或父工具/孩子原子事务。
- 入口：[原 Session 管理](../../Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt)、[目录模型](../../Kodex/app/contract/session-catalog)、[文件 repository](../../Kodex/agent-session/filesystem)、[GlobalRpc](../../Kodex/rpc/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/GlobalRpc.kt)。

## 验收

- 空库、归档过滤、缺省日期、缺 timestamp[0]、排序、创建后刷新，以及空闲/运行/失活目录标记；读取不打开历史 Session。
- 创建成功后消息失败仍保留 Session；创建本身失败沿原清理，丢失回执不产生自动重复调用。
- 两种 fork 的当前身份与历史内容、源不变、非法边界/旧 nonce 拒绝；编号允许复用。
- 批次返回顺序和 meta 正确，无前端逐项补 URI 查询；取消等待不撤销已接受工作，父 completeToolCall 仍独立。
- 删除/TTL/重建交错释放匹配 owner，不关闭其他 Session；旧订阅因失活结束，不虚报持久化删除成功。
- 管理原语可使用隔离文件及内存 repository 做 JVM/Native 测试；与 timeline/cache 联动时采用 P05 的隔离文件 fixture，不假定内存存储已有可观察元数据。
- 本批验证真实业务委托；完整 GlobalRpc 的 Session 方法往返归 P07 后端装配验收，不等待 P11，也不为提前测试填充其他方法的假成功实现。

## 交接

- P07 先汇合本批及全局其他组件，P08 再使用完整服务替换旧 NewSession 物化、tab/目录管理与子任务确认编排；P11 启用生产路径。
