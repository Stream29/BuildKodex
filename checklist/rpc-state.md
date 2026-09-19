# RPC 状态与本地投影

- 按 [RPC 迁移适用范围](rpc-architecture.md#适用范围)使用本文件。

## Get 与 GetFlow

- 将可订阅状态暴露为挂起 Get 与非挂起、返回顶层普通 `Flow<T>` 的 GetFlow；不直接跨线传 StateFlow、SharedFlow 或嵌套流对象。
- Get 只用于获取真实 initialValue；失败时不伪造默认值或暴露未初始化 handle。
- 先 Get 初始化本地状态，再订阅包含当前值的 GetFlow；不丢弃流首值，也不用迟到的 Get 覆盖流更新。
- 在 `rpc/client` 使用 `CoroutineScope.rpcStateIn(get, getFlow)` 组合上述顺序：Get 在调用协程中完成，GetFlow 通过库内 `stateIn(scope, Eagerly, initialValue)` 共享，返回不承诺远端订阅已经绑定。
- 初始化工厂不创建独立 scope；上游随调用方提供的 owner 存活，不因 UI 暂时无人 collect 而停止同步。Get/取流表达式失败直接抛出，后续上游失败遵循 owner 的协程异常规则，不自行重试或重读 Get。
- 初始化后的读取、状态更新与 CAS 重试依赖 Flow，不用 Get 轮询或刷新。
- 本地 StateFlow 表达最近收到的值；允许相等性合并及跳过中间状态，不把它当作无损事件日志。
- 多个本地观察者共享同一状态投影；不同状态源不承诺联合原子快照。
- 按需数据查询不是上述初始化 Get：[timeline 查询](rpc-timeline.md)、[Session 目录](rpc-session.md#目录与运行标记)、[Codex MCP 读取](rpc-mcp.md#codex-配置读取)按各自规则调用。
- Session 当前 settings 使用[版本化设置读取](rpc-settings.md#rpc-读写)，不要求每个派生 StateFlow 都有一套独立当前值 Get/Flow；订阅 timeline 元数据后按需取值不违反初始化 Get 的限制。

## 全量 CAS

- 可编辑 settings 使用完整 `expect/update` 和挂起 CAS，不提供无条件 set 或按字段 RPC。
- 在后端实际写入边界内完成值相等比较与替换，保留业务校验、持久化和运行副作用；不能只 CAS 镜像 StateFlow。
- 匹配返回 true；不匹配返回 false 且不写入；当前值、expect、update 均相等时成功但不产生变化。
- 使用值相等而非对象身份；不为检测 A→B→A 额外添加 settings revision。
- 校验、持久化、传输失败不伪装成 false；丢失回执不能证明未写入，不盲目重放结果未知的更新。
- CAS 成功不保证本地 `.value` 已同步，不用回执中的旧结果覆盖后续已收到的状态。
- CAS false 后优先使用订阅已收到的最新值重算；不要求必须观察到不同值才能再次比较，状态合并后可能没有新的可见值。
- 无订阅进展时允许经过可取消的等待后重试最新本地快照，并限制持续失败时的重试频率；不对同一快照紧循环，不用 Get 轮询或重新订阅驱动重试。
- 仅比较失败进入该重试路径；上游失效、SessionNotActive、取消及结果未知的失败交给各自处理，不在已失效投影上持续重试。
- timeline 支撑的 settings 投影随已有 cacheNonce/尾索引通知查询对应值；保留元数据进展，不增加当前值轮询，也不强制等待设置值本身变化。

## Settings 的字段冲突

- settings 仍以完整 expect/update 做原子 CAS；一次编辑固定其目标字段及初始观察值，不将 CAS false 直接等同于该编辑冲突。
- CAS false 后按上述规则使用最新本地快照；若目标字段仍等于该次编辑的初始值，则保留最新其他字段、重新构造完整 update 并重试。没有观察到变化时也可按有节制重试规则再次比较，不据 false 本身推断目标字段已改变。
- 任一目标字段已改变时，视为真正冲突，停止该次编辑的自动重试；不能把基准重置为新值后继续覆盖，也不盲目重放旧全量 update。
- 手动改名与自动标题都遵循此规则，目标字段为 threadName；其他字段变化可重试，名称变化则放弃该次改名。
- CAS false 表示本次替换未提交，不是先写入再回滚；真正冲突不撤销另一方已经提交的结果。
- 本规则是 settings 编辑层的语义，不改变 RPC 的 Boolean 返回值，不扩展到 pendingSteer 等其他 CAS；通用 update 扩展本身不保证该字段冲突策略。

## SuspendMutableStateFlow

- 用客户端本地 `SuspendMutableStateFlow<T> : StateFlow<T>` 封装只读投影与挂起 `compareAndSet(expect, update): Boolean`。
- 不将该 handle 标为 `@Rpc` 或作为返回值传输；读取委托库内 StateFlow，不手写内部状态流机制。
- 在 `rpc/client` 使用 `StateFlow<T>.asSuspendMutableStateFlow(compareAndSet)` 包装已初始化、由调用方持有的投影；不创建收集协程或额外可变状态，CAS 只委托挂起回调。
- 不提供 setter、set、emit、tryEmit 或全局订阅人数。
- `update`、`getAndUpdate`、`updateAndGet` 扩展通过 CAS 派生；transform 留在前端，允许多次执行，不携带不可重复副作用。
- 派生操作的返回值取自成功那次 CAS 的 expect/update，不从可能滞后的 `.value` 猜测。
- 三个扩展每次 CAS false 后固定可取消地等待50ms，再读取最新本地值重算；成功立即返回，异常直接传播，不增加 Get 轮询、错误重试或配置字段。
- 每次尝试前检查协程取消；源订阅初始化、失效处理及 CAS 准入由投影拥有者负责，此封装不替代 Session 恢复或 settings 字段冲突判断。

## 取消与业务错误

- 取消继续使用 CancellationException 的协程传播，不把它包装成业务失败值；不依赖取消异常的具体子类、对象身份或远端清理与本地等待同步完成。
- 需要前端按类型解析的非取消失败，通过显式可序列化联合类型表达，只纳入该业务契约已知、含义确定的错误分支。
- 不给错误联合添加 Exception、Throwable、Unknown、Other 或通用 message 兜底，不从异常类名或人类可读文案猜测业务分支。
- 未知异常仍使调用或上游 Flow 失败，留在非类型化失败通道；不吞掉、不伪装成成功或某个已知错误，前端不依赖其原始异常类型。
- 仅将实际识别的已知失败转换为对应值，不把所有 Throwable/Exception 批量转换成联合类型；取消不得被一般错误处理吞并。
- 已知异常联合通过应用层显式序列化和还原；只给自定义异常加 Serializable、直接交给 kRPC 的异常机制，不作为跨平台类型恢复方案。
- 逐项审查需要业务错误分支的调用及 Flow 绑定，复用已存在且语义匹配的结果类型；不为所有成功返回机械套统一 Result，不因本规则改变 CAS false 等既定业务结果。
- Flow 的已知失败如何编码、是否终止及前端如何恢复须在该接口明确；本地 stateIn 的上游失败由其拥有者处理，不假设 UI StateFlow 订阅者自动收到异常。

## 已知错误的 message 协议

- 已知异常继承 RemoteException，其 message 自动返回异常联合的 JSON；前端通过 kotlinx.serialization 直接还原本地已知异常，不依赖 kRPC 保留原类型或手写类名分派。
- message 首字符为 `{` 时进入协议解码，不添加专用文本前缀；非该首字符的异常不尝试解码。
- 编解码统一使用 RemoteException 的 serializer，复用库的多态 type 判别与序列化类名，不按具体子类编码而丢失判别字段；不设置未知类型的默认映射。
- 保持协议与 helper 简洁，只传错误标识及该分支必要的值，不复制任意异常的属性、栈或 cause。
- 解码完整 JSON，不从 toString 或嵌套 cause 搜索载荷；`{` 只是解码入口，不是可信身份标记。非法载荷或未知分支保留原失败，不用解析异常替换原异常，也不误判为已知错误。
- CancellationException 原样传播；helper 不负责重试、结果缓存或状态恢复。
- 成功返回保持原类型，取流仍返回普通 Flow；在 RPC 上游还原已知错误，不转换前端下游消费者自己的异常。

## 异常还原 helper

- `utils/rpc-exception` 提供 `inline fun <R> restoreRemoteException(block: () -> R): R`；R 由 block 推断，调用无需显式类型参数。
- 所有需类型化恢复的已知远端错误集中为 Serializable sealed RemoteException 的封闭子类，放在该模块 commonMain 的同一 package；不留 open 子类供外部扩展，不手写类型分派或每异常族编码函数。
- 基类 message 为 final 计算属性，不持有 backing field，直接调用 `Json.encodeToString<RemoteException>(this)`；该属性不参与值序列化，不把 Throwable 的 message/cause/栈纳入载荷，避免递归编码。
- 使用默认 Json 固定解码 RemoteException；block 只执行一次，取消原样抛出，非法或未知载荷保留原 Throwable。
- inline block 可在挂起调用方中挂起；仅包住创建 Flow 的表达式不能捕获后续收集异常，流式还原应放在上游 catch 中。
- 模块承担已知远端错误契约，不依赖 RPC 服务接口或后端实现，不封装 kRPC client 或额外提供载体转换函数；发送方直接抛出已知子类，接收方统一恢复，各 RPC 仍分别定义其允许的失败分支。
- 首个已知分支为 NoMatchException：取当前输出流时已无活跃流或 nonce 不匹配。其他错误逐项审查后加入，取消和未知异常不属于该联合。
- SessionNotActive 表示依赖 live Session 的访问发生时 Session 未活跃；按[显式激活准入](rpc-session.md#保活与回收)处理，不合并进输出流 NoMatchException，也不将其解释为 Session 已删除。
- SessionNotFound 表示后端确认指定持久化 Session 不存在；保活、管理命令与 delete 的差异按 [Session 管理](rpc-session.md#管理命令)处理，不把 owner 未命中或存储/传输失败转换成不存在。
- CacheNonceMismatch 表示 timeline 查询或历史命令携带的缓存标记不匹配；无额外字段，读取与命令的恢复差异遵循 [RPC timeline](rpc-timeline.md)，helper 不自动重试或替换标记，不与 currentFlow 的 NoMatchException 混用。

## 公共客户端适配

- `rpc/client` 提供 `RestoringRpcClient(delegate: RpcClient)`，统一包装公开 call 与 callServerStreaming，不逐项复制业务服务接口。
- 在创建服务代理前包装 client，再从包装后的 client 调用 withService；不会追溯替换已由原始 client 创建的代理。
- 普通调用通过 restoreRemoteException 恢复已知错误；流同时覆盖创建表达式立即失败和后续上游失败，按下节添加零容量收集边界。
- 取消及未知错误不重新分类，下游消费者自己的异常不进入远程异常解码；不增加重试、fallback、错误结果 DTO 或业务错误分派表。
- 适配器不拥有或关闭连接，不负责 StateFlow、CAS、Session 激活/重订阅及缓存；内存连接模块仍可独立返回原始 client。

## 订阅释放与重连

- 在公共客户端适配处紧接原始 RPC Flow 使用 `buffer(0)`（等价于 `buffer(Channel.RENDEZVOUS)`），再在上游 catch 中调用 restoreRemoteException；前端处理与本地状态投影放在其后，不要求各 UI 调用点重复添加保护。
- 零容量通道用于分离远程收集子协程；前端下游普通异常通过结构化取消终止上游，让 kRPC 走请求取消路径，前端仍保留自己的失败，不把普通异常改写成业务错误或 CancellationException。
- 此边界不提供 replay、不启用溢出丢弃、不改变各源的投递规则；取消后不保证交付仍在途的元素，本地收集结束不作为服务端清理已同步完成的回执。
- Flow 保护属于客户端接入，utils/rpc-exception 仍只负责已知错误编解码；不为该保护新增 unsubscribe RPC、关闭共享连接或修改框架。
- 前端释放状态时取消真正的远程上游收集；不能只丢掉引用或停止本地 UI collect。
- 正常退订复用 kRPC 取消传播，不新增 unsubscribe RPC，也不为关闭一个 tab 关闭共享 RPC client。
- 后端清理订阅自有资源，不因单个 collector 取消而关闭共享 Session 或 MCP client；遵守[资源生命周期](coroutine-resource-lifecycle.md)。
- [Session 失活](rpc-session.md#保活与回收)以 SessionNotActive 结束已有 RPC 上游；仍需要该状态的前端先保活成功再重订阅，不把失败当空值或静默停止更新。
- 本地 stateIn 不自动向 UI 订阅者传播上游异常；客户端上游拥有者负责还原失活异常和恢复需要的订阅，不仅在创建 Flow 的方法调用外包 helper。
- 后续网络部署断线后由前端恢复连接并重建仍需要的订阅，不要求后端恢复旧连接中的 RPC 调用；不将网络重连列为一期内存 RPC 的必做能力。
- 后续重订阅取得当前状态并核对缓存，不要求重放断网期间的每次状态变化。
- 将前端主动关闭与连接故障区分，避免关闭后再次拉起订阅；状态恢复不自动重放有副作用命令。
