# RPC Session 边界

- 按 [RPC 迁移适用范围](rpc-architecture.md#适用范围)使用本文件；本地 tab、共享运行与后端资源驻留是三个不同范围。

## Repository 实体与前端视图

- 真实 repository 由后端创建、持有并释放；Session、runtime、存储及其资源所有者构成完整的后端生命周期，不挂在前端 tab、ViewModel 或单次 RPC 等待的 scope 下。
- 后端目录查询与 Session 业务服务复用同一 repository；不由各前端工厂分别创建实体，也不把真实 repository 引用直接注入前端，即使一期处于同一进程。
- 前端仅通过 RPC 构建 repository/Session 的本地视图，持有值、缓存、订阅与命令入口；前端自己的 scope 只管理这些本地资源，不代表远端 Job 或所有权。
- 前端释放视图遵循[创建与前端关闭](#创建与前端关闭)，后端实体按[保活与回收](#保活与回收)及宿主生命周期释放；不将原实体的 close/cancel 转交路径机械保留在视图中。
- 目录 running/isActive 来自这份后端真实 repository，而不是客户端视图是否存在；不跨 RPC 返回 repository、Session 实例、CoroutineScope 或可写存储句柄。

## 目录与运行标记

- GlobalRpc.getSessionCatalog(includeArchived) 返回轻量 SessionCatalogEntry 列表，不提供目录 Flow 或独立运行标记订阅。
- 当前只考虑单用户且 Session 总量不大的场景，接受每次全量读取符合筛选条件的目录；这不是面向任意规模的查询设计。
- 未来扩大使用规模时必须将 catalog 查询接口改为分页；本期不提前添加 cursor、pageSize、分页结果模型或相关基础设施。
- 前端在打开、重新打开目录或切换 archived 时查询；保留自己的加载/错误、选择、筛选与结果快照。
- 每次查询取得当时目录，不永久复用陈旧 repository 盘点；失败不能当作空列表。
- 排除 archived 时先过滤再读取其元数据；按最近活动时间降序、Session index 降序排列，保留未初始化标题/活动时间的 null。
- 条目增加后端提供的运行布尔值；不能从本地 tab 打开、保活或缓存驻留推导正在运行。
- SessionCatalogEntry.running 直接取 root Agent 的统一 runningTurn 是否非空，包含普通运行和手动压缩，不包含历史操作或前端等待；不依赖 AgentExecutionState，不据此推导辅助任务的资源寿命或 TTL 回收准入。
- 目录另含 isActive，表示查询时后端 Session owner 已加载且存活；与 running 分开，空闲的已加载 Session 可以 isActive=true、running=false。
- 活跃标记来自 repository 原有的打开对象登记与生命周期，不由前端 tab 推导，不为查询目录调用 open，也不新增独立的活跃状态 RPC。
- 沿用 repository.open 对同一活跃 Session 返回同一实例的约定；不为重复取得该实例新增 getActive。isActive 是目录快照，不替代实际访问的活跃准入或保活。
- 前端依据 running 生成 spinner 动画；只表示查询时的运行状态，不保证目录保持打开期间实时变化。
- 不为读取目录或判断两个标记加载全部历史 Session 的重型 runtime；isActive/running 均不作为持久化事实写盘。

## 菜单日期

- Session catalog 条目携带 createdAt 与 updatedAt，使用可空 Instant；Sessions view 的菜单直接展示当次目录快照，不在打开右键菜单时再请求日期、订阅 timeline 或激活 Session。
- SessionCatalogEntry 只保留 updatedAt 这一更新时间字段，不并列传输 lastActivityAt 或增加兼容别名；后端可沿用 repository 的内部字段名进行映射。
- 目录 createdAt 保留原 timestamp.getExact(0) 语义，缺少 0 时为空；updatedAt 为 timestamp 最新记录，与原 lastActivityAt 同义，排序及相对时间展示语义不变。
- 日期随目录查询一起刷新；不为右键菜单另加日期 RPC、目录 Flow 或独立刷新调用。
- 已打开的持久化 tab 右键菜单复用 TimestampTimelineRpc：createdAt 按原 ceilToIndex(0) 后精确读取，updatedAt 读取最新记录；遵守原 cacheNonce 与活跃准入。
- 保留[菜单时间展示](context-menu-timestamps.md)的格式、空值隐藏和 tab 每次打开读取规则；不把 tab 与目录的取值、刷新时点差异统一掉。

## Session 编号

- 保留原 Session index 分配规则，允许删除后复用空号；不新增持久化分配水位、编号迁移或永久身份机制。
- RPC 继续使用原 Int sessionIndex，按当前编号寻址，不额外添加 Session nonce 或身份 DTO。
- cacheNonce 校验 timeline 缓存及带标记的历史操作，currentFlow nonce 校验当前输出流；不把它们解释为所有 RPC 都携带的 Session 身份校验。
- 本次重构按单用户使用场景接受编号复用，不增加跨删除/重建的意图隔离；保留各操作已有的活跃、缓存标记和业务准入校验。

## 管理命令

- 后端确认指定持久化 Session 不存在时使用无字段 RemoteException 分支 SessionNotFound；不将未命中 live owner、权限或存储/传输失败解释为不存在。
- live 访问可先返回 SessionNotActive，不为每个属性读取额外查磁盘；前端保活确认 SessionNotFound 后停止该绑定的自动保活/重订阅并提示缺失，不隐式新建。
- SessionNotFound 只描述检查时不存在，不保证该编号以后不会复用；已有上游因 owner 关闭使用 SessionNotActive，不预判持久化删除是否成功。缺失 shell ID 不属于该分支。

- archiveSession/unarchiveSession 只修改现存 Session 的归档标记，同方向重复操作幂等；缺失不隐式创建。
- 归档不等于 Stop、删除、关闭 owner 或关闭前端 tab；取消归档不隐式打开 runtime。
- forkSession 执行完整 Session fork 并返回新 index；保留初始化、非运行、fork capability、标题处理与失败目标清理。
- 完整 fork 不修改源归档状态或前端导航，不包含按历史边界 fork。
- 按边界 fork 使用 GlobalRpc.forkSessionHistory，遵守 [历史操作](rpc-timeline.md#历史操作)的缓存标记与边界约束。
- deleteSession 先关闭匹配共享 owner 再删除持久数据；成功 true、不存在 false、操作失败抛错。
- 删除影响全部使用者；确认弹窗及各前端导航留前端，不把删除解释成关闭调用方 tab。
- 操作后需新目录时重新 Get；不增加目录推送，也不因丢失回复盲目重复 fork 等命令。

## 创建与前端关闭

- 空白 NewSession 保持前端草稿，不因打开 tab 创建持久 Session。
- GlobalRpc.createSession(initialSettings: KodexAgentSettings) 返回新 Session index；使用前端草稿设置并由后端替换初始标题、校验和初始化，不传首条消息。
- 当前 RPC 创建范围不支持自定义初始标题；后端按分配的 index 初始化为 `Session <index>`，不增加默认名开关或用空串约定区分命名意图。
- 将创建/初始化与首条消息提交分开；创建成功而提交失败时保留 Session 和可重试内容，不自动删除新 Session。
- 创建本身失败的清理与之后提交失败分开处理；不以传输失败推断创建未发生。
- 前端 tab 关闭只停止自己的保活、观察和本地状态，不提供 closeSession RPC。
- 前端保持自己的 tab 顺序、选择与草稿，不把旧 Application 的导航和 registry.release 一起映射成后端命令。

## 建议子任务批量创建

- 用户确认、批次配置与调用时机由前端决定；后端提供专属批量创建 RPC 执行批次，不要求前端逐个组合创建和命名请求。
- 专属批次同时由后端负责首条消息提交和运行启动，不让前端逐个 append/resume；不等待各孩子的模型运行结束。
- 参数直接使用任务列表和原 KodexAgentSettings，不传只有 tasks 的外层 SuggestSubagentTaskArgs，也不为 RPC 保留旧前端 SuggestedSessionConfiguration 投影。
- GlobalRpc.createSuggestedSessions(tasks: List<SuggestedSubagentTask>, initialSettings: KodexAgentSettings) 返回 List<CreatedSuggestedSession>；每个孩子以任务 name 覆盖初始 threadName，并按原新建校验与初始化独立生成 turn/window 身份。
- 结果按输入任务顺序携带 sessionIndex 与原 SuggestedSessionMeta；meta 是创建时的 URI/name，不是活跃租约、最新标题或执行结果，不要求前端逐项查询才能构造父工具结果。
- 成功返回表示创建与启动任务已交接给后端，不保证首条消息或模型执行成功；父工具 completeToolCall 仍由前端另行提交，不把父提交与孩子执行合成原子操作。
- 创建失败使用普通未知异常通道，不增加专属已知错误分支或部分成功协议；后端尽力减少副作用，不承诺整批回滚、清理必成功或异常后没有新 Session。
- 异常或丢失回执不自动重放整批；取消前端等待不撤销后端已接受的工作，不为此增加批次持久记录或结果恢复机制。
- 该入口服务于已确认的子任务批次，不由模型建议或通知事件自动触发；前端交互驱动不意味着存储操作在前端执行。
- 复用后端现有创建和设置能力，不用远程 ViewModel 或通用 RPC 批处理框架；普通 createSession 的独立创建语义不因此改为批量事务。
- 导航、打开及选择 tab 仍归前端，后端不持有前端页面或弹窗状态。

## 保活与回收

- 使用 GlobalRpc.keepSessionAlive(sessionIndex): Unit 进行激活与周期保活；成功返回前须使对应 Session 已活跃，不能只更新期限再异步加载。它不是排他访问许可或取消归档，缺失 Session 时失败而非创建。
- 前端通过 keepSessionAlive 续期；后端在 runningTurn 存续期间自行续期同一 TTL，包含普通运行、手动压缩及其取消清理阶段，不要求前端仍在等待或保持连接。
- 普通读写、订阅、pendingSteer 和 shell 登记项不续期；runningTurn 释放后停止自保活，不因完成再授予另一份保留期。不采用长时间挂起的 open 调用。
- 任一前端停止保活不能撤销其他前端已续出的期限，不增加 close、acquire/release 或另一套客户端 lease 协议。
- 前端先等待 keepSessionAlive 成功，再访问 latestIndex 等依赖 live Session 的属性；普通访问不再隐式加载未活跃的 owner。处理访问时 Session 未活跃，通过已知 RemoteException 分支 SessionNotActive 拒绝。
- 保活成功只表示该次激活已完成，不保证任意后续时刻仍活跃；后端在实际访问/新订阅绑定时校验，前端不能将之前的回执当作永久访问许可。
- TTL 是 Session 驻留与自动失活的唯一判据；runningTurn 通过续期避免运行中自动卸载，不以另一套 busy、引用计数或队列非空条件否决过期。
- 到期卸载不等待 shell 被消费；关闭所属进程资源，放弃尚未读取的尾输出和临时登记。残留 pendingSteer 不阻止卸载，按运行逻辑问题排查，不为它增加持久化或独立保留机制。
- 到期决定失活，实际关闭仍须维护写入一致性、取消清理和同一 Session owner 的互斥；这些同步要求不构成额外保活来源或长期驻留条件。
- 宿主存活时，前端观察结束不直接取消已接受任务；运行存续由 runningTurn 自保活保障，不把所有辅助工作都当作永久驻留理由。整个 CLI 退出按[一期宿主生命周期](rpc-architecture.md#一期部署与交付)清理。
- Session 安全卸载为不活跃时，后端以 SessionNotActive 结束其已有 RPC 上游并释放对应收集；不保留失活 Session 的订阅，也不让旧流静默挂起。
- 后端适配层将业务失活转换成该失败，不依赖 SharedFlow 自动完成或 kRPC 自动识别 Session 状态。仍需要该 Session 的前端先重新保活，成功后重建订阅并核对 cacheNonce；已关闭的界面不自动激活。
- 订阅本身不阻止 TTL 回收，也不为结束观察发送 Stop；需要继续运行的 turn 由后端自保活，与前端订阅独立。
- 后续独立网络部署的断网恢复遵循 [RPC 状态生命周期](rpc-state.md#订阅释放与重连)；不作为一期内存 RPC 的接入要求。

## 保活参数与范围

- 一期固定 Session TTL 为 60 秒，前端 keepSessionAlive 与后端 runningTurn 自保活间隔均为 20 秒；不增加配置字段、到期时间 DTO 或批量保活 RPC。
- 前端为每个已打开的持久化 Session 保活，包括未选中的 tab；按 Session index 复用一次循环，新建草稿和目录条目不保活。
- 首次等待激活成功后再初始化访问；同一 Session 的续期调用不重叠。关闭 tab 只停止自己的周期调用，不撤销已续出的期限。
- 后端接受并占用 runningTurn 时立即续期，再由独立于模型 delta、工具返回和前端 collect 的周期任务续期；清理结束、槽位释放后停止。内部只续对应存活 owner，不通过 RPC 重新激活旧 Session。
- 两侧更新后端维护的同一截止时间，以单调时间计算，新续期不缩短已有期限；激活完成时给足 60 秒，不让加载时间消耗刚建立的有效期，不使用前端传入时间。
- 全部续期停止后，沿最后一次成功续期的剩余期限失活，不从任务完成、shell 完成或普通读写另起算。
- 到期与续期、关闭与重建须协调；60/20 秒不是严格调度保证，也不承诺 OS 资源恰好在第 60 秒全部释放。
