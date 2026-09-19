# RPC AgentRuntime

- 按 [RPC 迁移适用范围](rpc-architecture.md#适用范围)使用本文件；AgentRuntimeRpc 通过 sessionIndex 定位现有 Session 的 root Agent，不传运行对象。

## 存储定位

- getStorageUri(sessionIndex): String 返回 root Agent 的原 storage.uri，作为稳定属性按需读取，不增加 URI Flow 或 Session 信息 DTO。
- 从已活跃的后端 owner 读取；未活跃使用 SessionNotActive，不隐式加载或续期，先由前端等待 keepSessionAlive 成功。
- 保留后端返回的定位符，不由前端拼装 Home 路径或假定存储协议；URI 不授权前端绕过 RPC 读写文件，也不替代 owner/cacheNonce 身份。

## 全局存储位置

- getLatestIndex/getLatestIndexFlow 直接暴露原 AgentState 在存储事务完成后发布的全局可见 index；Get 只初始化，Flow 包含当前值与后续更新。
- 不与各条 TimelineRpc 的实际记录尾位置混用，也不由前端合并六条独立通知来重建后端发布时机。
- 历史回退可使位置下降；允许合并中间值，不把该流当作每次提交的事件日志。
- 位置不携带 cacheNonce，也不是跨多次 RPC 查询的快照锁；按捕获的位置查询数据时仍遵守各 timeline 的缓存标记和迟到结果规则，同数字位置不能证明内容未被替换。
- 当前 tokenCount 继续复用 TokenCountTimelineRpc 的可见值查询，不为旧前端 nullable 初值复制一组后端投影接口。
- 读取与订阅不续期 TTL，取消订阅不关闭 Session。

## Pending steer

- 使用 getPendingSteer/getPendingSteerFlow/compareAndSetPendingSteer，复用 List<StableIndexEvent.Steerable>，不新增队列 DTO。
- 队列是后端运行态，不是前端输入草稿或已持久化历史；保持原元素值、顺序和重复项。
- Get 只初始化；Flow 发布当前队列及后续值，包括 runtime 消费并清空队列的变化，遵循 [RPC 状态](rpc-state.md)。
- CAS 操作实际 pendingSteer，与 runtime 的 getAndUpdate 消费共享同一原子值；不另建镜像、不提供无条件 set。
- 比较相等后替换完整队列，不相等返回 false；队列解析或操作失败不伪装成比较失败。
- CAS false 按[全量 CAS 重试规则](rpc-state.md#全量-cas)使用最新本地队列，无新值时也允许有节制重试；不通过 Get 轮询，不绕过原子比较恢复已消费内容。成功不保证前端缓存同步。
- 成功只表示队列更新，不表示模型已消费或内容已写入历史；不隐式 submit、resume 或续期 Session。
- 用空列表清空 pendingSteer 与中断待处理工具调用是不同操作，不将原 clearPending 当成队列清空。
- 取消订阅不清空队列或停止 Agent；不增加队列持久化或后端任务取消协议。
- 压缩期间保持原有入队行为；手动压缩结束不额外消费 pendingSteer，也不因此自动 resume。不借 RPC 重构增加压缩期提交限制或补消费机制。

## 待处理工具清理

- AgentRuntimeRpc.clearPending(sessionIndex): Int 独立于 pendingSteer CAS；沿用原扩展，逐个将待处理工具完成为 user interrupt 并提交结果。
- 等待清理流程结束后返回最后存储 index，无 pending 时返回当前 index；缺失 Session 或操作失败不伪装成成功清理。
- 每个工具完成保留原 completeToolCall 校验及 stable/unstable 原子更新，不把整个循环变成单事务，后续失败不回滚已提交结果。
- 不清空 pendingSteer、不取消或 resume Agent、不续期 TTL；已接受清理由后端 owner 持有，不因调用方取消等待而中止。

## 历史回退

- revertHistory 的参数、缓存标记、准入和前端职责遵守 [历史操作](rpc-timeline.md#历史操作)，不暴露原确认弹窗状态。

## 工具结果提交

- AgentRuntimeRpc.completeToolCall(sessionIndex, completed: StableCleanEvent.CompletedTool): Int 复用原跨 timeline 完成事件，不按工具复制完成 RPC 或 DTO。
- 问题与调用参数来自 state 的 ToolPending；前端保留草稿、选择和确认，提交构造好的原完成事件，不跨线传编辑 ViewModel。
- 后端保留原 ToolPending 准入与 projected callId 校验，原子写入 index/work 结果并移除对应 unstable pending；返回提交 index，失效或已完成调用失败。
- 不自动 resume、不创建建议的 Session、不消费前端草稿或续期 TTL；已接受提交由后端持有，返回不保证前端投影同步。
- 只为实际传输的 CompletedTool 子联合补序列化，持久化继续使用原 index/work 联合；不顺带序列化外层所有 StableCleanEvent。

## 用户消息写入

- AgentRuntimeRpc.appendUserMessage(sessionIndex, content: List<ContentItem>): Int 复用原内容联合，写入一条用户消息并返回提交的存储 index，不新增消息 DTO。
- 保留原原子准入、timestamp 和持久化 turnId 判断，不恢复独立 markNewTurn；前端缓存不能替代后端当前状态校验。
- 运行中输入使用 pendingSteer CAS，不以直接写入绕过准入；不向前端暴露仅供后端使用的 injectHistory。
- 写入成功后由后端触发[自动标题](#自动标题)的资格检查，不等待生成完成；不自动 resume、清空 composer 或续期 TTL。前端分别处理写入和后续运行结果，不因 resume 失败或写入回执丢失自动重复 append。
- 已接受写入由后端 owner 持有，取消前端等待不撤销写入；返回 index 不保证前端订阅已同步，不用它覆盖较新的订阅状态。

## 自动标题

- 模型执行归属遵循[业务边界](rpc-architecture.md#业务边界)；自动标题的模型调用、资格判断、任务和结果回写均保留在后端。
- 后端在 appendUserMessage 成功接受消息后触发自动标题，不新增 startAutomaticTitle RPC，不要求前端重复传入已提交内容或编排标题请求。
- 后端读取 BackendSettings.sessionTitle，复用原首个非空文本、默认名称和一次生成资格判断；图片-only 输入不消耗文本生成机会。
- 建议子任务通过[专属批量创建入口](rpc-session.md#建议子任务批量创建)在后端完成命名准备，再提交首条消息，不要求前端逐项发起命名 CAS；对已有设置的改名仍遵守 CAS 规则。不再显式抑制自动标题，不提供 suppressAutomaticTitle RPC；按现有名称及其他原资格条件判断，接受默认格式名称仍可能被自动标题替换。
- 标题是后端辅助任务，不占用 runningTurn，不隐式 resume 或续期 TTL；前端等待或观察取消不取消已接受的标题任务。
- 标题生成作为 AgentRuntime CoroutineScope 的普通子协程，随该作用域关闭取消；不另建独立生命周期、恢复任务或补生成机制，仍沿用原输入资格与历史回退处理。
- 不等待标题结果才返回消息 index；标题生成失败不撤销消息、不将已提交的消息报告为写入失败。
- 生成的名称通过原 settings 写入边界提交，由 SettingsTimelineRpc 的通知与值查询更新前端当前设置，不另设标题状态流或结果 DTO。
- 手动与自动改名均使用 settings CAS，按[目标字段冲突规则](rpc-state.md#settings-的字段冲突)处理；自动标题遇到名称冲突即放弃该结果，仅其他设置改变时可保留生成结果重建 update 后重试，不重新调用模型。
- 不以“手动改名成功后取消自动任务”作为本规则的正确性保障；先成功提交的名称保留，另一方不能绕过 CAS 覆盖。不新增普通 rename RPC。
- 辅助标题不构成独立的 TTL 回收阻挡条件；Session 到期关闭时仍须取消或收敛其工作，避免迟到结果向已失活 owner 写入。

## Resume

- AgentRuntimeRpc.resume(sessionIndex): Unit 使用挂起方法，沿用原 AgentRuntime.resume 的完整 operation 等待语义，不改为启动后立即返回。
- 正常完成后返回，运行失败不伪装成成功；中间状态通过独立状态流观察，不把 resume 返回值改成任务句柄或新结果 DTO。
- 普通运行与手动压缩共用 runningTurn 槽位和准入，不允许并行占用；保留拒绝重复运行的方向，不因跨 RPC 改成排队执行或合并重复调用。
- RPC 等待时长与运行任务所有权分别处理；已接受任务及调用方等待取消遵循[既定业务边界](rpc-architecture.md#业务边界)，不因挂起等待而将任务绑定到前端视图寿命。

## Running turn

- runningTurn 统一表示后端当前可取消的 Agent 执行任务，包含普通运行与手动压缩；不再为手动压缩保留独立 runtimeOperation 真源或只在前端合并 busy。
- getRunningTurn/getRunningTurnFlow 返回该统一槽位非空的 Boolean 初值和更新，不传 Job；不包含历史操作，不将该值等同于所有资源或操作的存活状态。
- cancelRunningTurn(sessionIndex): Unit 取消后端处理命令时的当前 turn；现存 Session 无 turn 时 no-op，缺失 Session 时失败。
- 不绑定旧观察中的 turn，不排队取消未来 turn；发出取消请求后返回，不等待清理完成。
- 正常完成、失败和取消均在实际清理结束后释放统一槽位；取消请求发出后 runningTurn 可暂时仍非空，退出后再通过 Flow 发布 false。
- runningTurn 存续期间由后端自行续期 Session TTL，保证普通运行和手动压缩不依赖前端继续保活；释放槽位后停止续期，按 [TTL 规则](rpc-session.md#保活与回收)处理，不能以残留 steer/shell 替代运行自保活。
- 保留各操作自身的取消清理，普通运行的 pending-tool interruption 不因共享槽位而机械套到手动压缩；统一生命周期不改变各自操作结果或持久化轮次语义。
- 前端 Stop 统一调用 cancelRunningTurn，可取消普通运行或手动压缩，不新增 cancelExecution；不取消历史操作、不显式清空 pendingSteer、不关闭 Session、不续期 TTL。
- 取消观察仅释放订阅，不触发 cancelRunningTurn。

## 前端执行投影

- 迁移时废除 AgentExecutionState 聚合模型，不提供 getExecution/getExecutionFlow，也不把 phase/capabilities/activityVersion 搬成后端状态。
- 前端按控件需要从 state、runningTurn 等真实原语派生展示与可操作提示，不为复刻旧 DTO 增加平行状态或新的聚合 DTO。
- 前端 ViewModel 的操作 Job 和等待标记不直接定义后端事实；新增后端原语须有独立业务需要，不能只因旧 execution 包含该字段而新增。
- 后端仍在实际操作边界校验合法性；前端派生的按钮状态不是权限、锁或执行成功保证。

## Shell 进程

- getShellSessions/getShellSessionsFlow 发布整体 Map<Int, ShellSessionState>，Get 只初始化；Map key 复用所属 Agent 的进程 ID，状态值只含原 ExecCommandArguments 和 completed 布尔值。
- 后端同时观察 registry 成员及各进程 completed，完成但成员未变也须发布新值；前端只订阅整体状态，不提供逐进程 completed RPC。
- 保留已完成但等待工具读取尾输出的登记项，不能将该表过滤成仅运行列表；移除与 completed=true 是不同状态。
- closeShellSession(sessionIndex, shellSessionId) 解析指定 Agent 当前登记项并捕获实例，缺失失败；不将 ID 视为跨移除/owner 重建永久唯一的对象身份。
- 关闭沿原行为请求终止进程树，不主动移除登记项或消费输出；返回不表示进程已退出，不提前伪造 completed=true。
- 不向前端开放 execCommand/writeStdin 或输出缓冲读取；展示所需的逐进程投影在前端本地派生。
- 取消状态订阅不关闭进程；进程 close 不关闭 Kodex Session、不 Stop Agent、不续期 TTL。
- shell 的运行、登记或未读尾输出都不续期或阻止 Session TTL 关闭；保持登记和最终读取的规则只在所属 Session 仍存续时成立。

## 手动压缩

- AgentRuntimeRpc.forcedCompact(sessionIndex): Int 沿用原 forcedCompact 扩展，固定 Manual/UserRequested/StandaloneTurn，等待压缩 checkpoint 提交后返回 index。
- 保留后端状态准入和失败传播，不自动 resume、不开放内部自动压缩策略参数；压缩占用的 runningTurn 按统一规则自行续期。
- 接受后的压缩由后端 owner 持有并占用统一 runningTurn 槽位；取消前端等待不等于取消压缩，显式 cancelRunningTurn 取消当前压缩。

## 压缩状态

- 手动与自动压缩使用相同的压缩核心，通过 getState/getStateFlow 发布无字段的 Compacting 状态；压缩结束后恢复实际存储对应的状态。
- 手动压缩占用一个 runningTurn，自动压缩是现有 runningTurn 内的步骤，不重复占用槽位或与所属任务并行运行。
- 不向前端发布压缩 SharedFlow，不给 Compacting 增加 nonce，也不将压缩接入 currentFlow；前端仅展示正在压缩的状态。
- 压缩 SSE 的收集、重试、结果验证、checkpoint 提交与清理由后端负责，不依赖前端是否订阅；取消前端状态订阅不取消后端压缩。
- 统一运行机制不抹去 Manual/Auto、UserRequested/ContextLimit、StandaloneTurn/PreTurn/MidTurn 等触发和阶段区别，不改变既有压缩结果的持久化语义。

## 输出流引用

- AgentRuntimeRpc 提供 getState/getStateFlow 与 currentFlow(sessionIndex, nonce: Long)；状态使用 rpc/models 的 AgentStateValue，输出事件复用原 ResponsesStreamEvent。
- 普通 Responses 的 AgentStateValue 保留原状态分支与 ToolPending 的非空原事件列表；五种活跃输出分支以 Long nonce 替换 SharedFlow，Started 不携带流标记。Compacting 仅按[压缩状态](#压缩状态)处理，不提供输出流。
- Long 标记为不透明随机值，不以正负或零表示生命周期，不编码序号、索引或时间。
- 每个 Agent 沿用唯一当前输出流，不建立按编号保存历史流的 registry、旧流保留窗口或流列表。
- 每次创建新的活跃输出流时生成随机标记，并随对应活跃输出状态通过 state Flow 发布；同一流的后续 delta 不更换标记，重试或下一输出项创建的新流重新生成标记。
- 随机标记用于确认当前流身份，不复用持久化 latestIndex 或供应商 output_index，也不为开始流式输出推进存储索引。
- 前端携带状态中的随机标记请求 currentFlow；后端在订阅绑定时确认当前确有活跃流且标记匹配，捕获该同一流实例后提供普通 RPC Flow<ResponsesStreamEvent>。
- 无活跃流或标记不匹配时拒绝订阅，不返回空流冒充成功、不转发新流、不寻找旧流补播；前端依照最新 state 和已提交历史继续展示。
- 无当前活跃流或 nonce 不匹配时使用 NoMatchException；通过[异常 message 协议与恢复 helper](rpc-state.md#已知错误的-message-协议)供前端识别，不依赖 kRPC 原生恢复自定义异常类型。
- 对校验通过并绑定的流，重放其已产生的全部事件，再连续接收后续事件；不承诺已经结束或被替换的流仍可接受新的订阅。
- 校验与取流基于同一状态快照，不能校验旧标记后重新读取当前流而误接重试产生的新流。
- latestIndex 只继续通知已提交历史变化，不作为发现新输出流的唯一入口；不以持久化最终值冒充原 delta 序列 replay。
- 前端需要本地 SharedFlow 时共享同一 RPC 上游并保留该输出的 replay；不为每个本地观察者重复启动 RPC。
- 前端独立收集 state，不在状态处理协程内等待输出流收集结束；同一标记复用本地流，新标记重新建立累积状态，不串接上一次未完成输出。
- 标记过期时依据已收到的最新 state 切换，尚无新状态则等待订阅推进；不轮询初始化 Get、不忙重试旧标记、不因此 resume 或取消 Agent，其他错误不得吞作过期。
- 状态不再指向当前输出、流标记变化或前端界面关闭时，前端取消旧 RPC 上游并释放本地投影，后端沿既有订阅取消路径释放引用；不依赖 SharedFlow 自动完成。
- 不要求前端消费完已放弃的旧输出或观察到所有短暂状态；已完成输出通过持久化历史恢复，不新增全输出流日志。
- 不将独立读取 replayCache 再开始 collect 当作无缝订阅，以免漏失或重复中间事件；复用后端原 SharedFlow 的 replay 加实时收集语义。
