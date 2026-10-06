# Task Tree

- `Trace the real State interface, factory, implementation, and exceptions`()
- `Receive review of the State hard-cutover plan`()
- `Move the original contract and observable exception to spec/contract`()
- `Move the actual factory, implementation, and tests to impl/state`()
- `Reconcile KDoc with actual atomic and streaming behavior`()
- `Run the State and real host validation matrix`()
- `Return evidence to the parent task and receive scoped acceptance`()

# Details

## Ownership and scope

- [Parent task](2026-10-06-hard-migrate-agent-state.md)；用户已授权本批硬迁移。
- 直接迁移原 `agentstate.contract.KodexAgentState`、`KodexAgentStateValue`、
  `RequestFinish` 与 extensions；不新增 State dependencies façade、第二套
  command/state、lifecycle handle 或 replay controller。
- 接管原 `agentstate.impl.KodexAgentState` scope 工厂及唯一私有实现；
  工厂显式借用 OpenAiClient、MutableKodexAgentStorage、contextSettings、McpService。
- `KodexAgentStateInvalidTransitionException` 移到 spec/contract，但保留原
  `agentstate.impl` FQCN、构造参数与 currentState 属性；这是原可观察异常的物理
  接管，不是新异常或兼容 alias。构建依赖不能因包名含 impl 继续依赖旧项目。
- public spec 直接声明需要的 clean-model/OpenAI/coroutine 契约依赖；
  不依赖 State impl、工具 impl 或 context-prefix 实现。

## Actual operation matrix

| 操作 | 必须保持的现行边界 |
| --- | --- |
| 工厂 | 加载实际 storage/latest 与稳定历史；失败取消新建子 scope；不关闭借用资源。 |
| modify | 同一非重入写锁，ExternalWrite；无论成功/失败/取消均从实际 storage 恢复可观察 state/index。 |
| requestResponseApi | 一次请求、固定开始快照；OutputItemDone 落盘后释放该输出 flow；最终 NonCancellable 恢复。 |
| compact | 固定压缩输入/元数据；Compacting 逻辑所有权；checkpoint 和 synthetic zero；保留基线锁与设置行为。 |
| injectHistory/appendUserMessage | 顺序稳定 clean events；当前 final/commentary 历史推断 turnId；空 injection 不新增写入。 |
| completeToolCall | 验证 callId；移除对应 unstable，写 stable index/work；允许结果乱序并按实际完成顺序追加。 |
| updateSettings/CAS | 与 State 写锁共用准入；CAS 比较完整最新值，mismatch false/equal update true 不追加；当前请求不改快照。 |
| appendPlanUpdate/clearPending | 原 settings/completion 分步扩展与逐项 user interrupt；不承诺整个扩展一次事务。 |

- RequestFinish：Completed(endTurn=false)→Continue，其余 completed→Finish；
  Failed/无终态→Retryable；Incomplete、传输/存储错误、Cancellation 传播。
- 普通请求 planning/prefix 在 history 前临时投影；不持久化，不进入远程压缩输入；
  null promptCacheKey 取固定 threadId，空 override 保留。
- provider header turn-state、stable/unstable 配对、token diagnostics、window/thread
  身份算法及 remote compaction whole-item retention 保留原所有者，不复制模型。
- 状态流是观测值，不是第二份持久化真源；业务恢复读取真实 storage。
  调用方取消与 owner scope 取消分别按实际代码写入 KDoc，不能承诺取消即回滚。

## KDoc and evidence gates

- 为实际公开操作列出非法状态、工具目标不匹配、缺少初始化、Incomplete、
  delegate I/O 及取消的触发条件；不把任意 delegate Throwable 固化为新异常类。
- 原源码与 SOP 的已知冲突见主计划。`appendPlanUpdate` 文档仍按两步操作；
  `RequestFinish.Retryable` 不是“仅暂时网络故障”的凭空保证。
- 保留 implementation-specific Mutex/SharedFlow/UUID/hash 机制描述于 impl，
  同时让规范描述实际可观察顺序；不借 KDoc 移动重写算法。
- State 原测试整体迁移：
  [Impl tests](../../Kodex/agent-state/impl/state/src/commonTest/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImplTest.kt)、
  [Settings CAS](../../Kodex/agent-state/impl/state/src/commonTest/kotlin/io/github/stream29/kodex/agentstate/impl/SessionSettingsCasTest.kt)、
  context/request projection、hosted search 及 fixture。
- 缺口优先：factory failure/借用资源所有权，inject 用户消息经过 final/commentary 的
  turnId，流式项已落盘后取消、不完整终态诊断、取消等待写锁。
  压缩最新 settings 风险按主计划单列复现，不写成永久兼容保证。

## Execution handoff requirements

- 报告原 FQCN、工厂返回静态类型、唯一实现、旧声明删除及全部原测试位置；
  不以测试绿色替代生产路径证明。
- 同一核心文件由一个实现 owner 修改；另一 helper 工作单元只迁移自己的文件，
  协调者统一合并构建依赖。
- 验证与停止条件以主计划为准；持久化、协议、scope/取消语义改变须单独评审。

## Implementation and validation evidence

- 原接口、State 联合类型、RequestFinish 和扩展直接迁入 spec；原 scope 工厂
  及唯一私有实现直接迁入 `impl/state`。新增可观察异常文件保留原 FQCN/消息；
  除该物理移动与 KDoc/import 外，原 State 实现和契约可执行代码对照一致。
- 原测试完整保留，新增三项 `AgentStateHardCutoverTest`；Xiaoxin Ubuntu
  State JVM 55 项通过。构造失败后 owner 可复用、借用 client/MCP 不关闭、
  final/commentary 的 turnId 推断均由真实 State 路径验证。
- 新测试明确复现已有 compact/CAS 覆盖风险：中途成功写入的标题被旧快照覆盖。
  这是未修复缺陷的证据，不是理想行为规范或“所有一致性问题已解决”。
- Runtime、Session、RPC、History 和 Application 真实消费者矩阵、
  CLI/Integration 编译及平台范围见主任务 `Central validation checkpoint`；
  四个 State Linux x64/JS 主源码目标通过。未运行 live provider 集成。
- 用户已验收迁移范围并授权分批提交推送；本任务归档 done。
  未修改压缩算法，该并发缺陷的行为修复仍需另行明确协议。
