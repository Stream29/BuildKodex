# RPC 设置与前端通知

- 按 [RPC 适用范围](rpc-architecture.md#适用范围)使用本文件；当前 CLI 使用本文件的两侧存储与通知型 Hooks，旧单文件 codec 只作为迁移兼容测试依据。

## 文件与字段归属

- 后端自行读取、持有并持久化 `settings.backend.yml`；CLI 自行读取、持有并持久化 `settings.frontend.cli.yml`。
- 在 `rpc/models` 使用 BackendSettings、CliFrontendSettings、CliSidebarSettings，复用嵌套领域类型。
- BackendSettings 包含 authSource、shell、contextSources、newSession、sessionTitle、mcpServers；路径与 shell 按后端环境解释。
- newSession 是各前端共享的新建默认值，不是本地草稿，也不批量改写已有 Session。
- sessionTitle 控制后端自动标题请求，不是前端标题展示偏好。
- CliFrontendSettings 承载本地偏好与通知命令；字段为 newLineKey、sidebars 与 hooks，沿用 ShiftEnter 及左右 HistoryIndex/TerminalSessions 默认值，hooks 默认空列表。
- CliSidebarSettings 仅保存 left/right 内容；宽度为前端临时状态，应用启动时各按应用宽度的四分之一初始化，调整不持久化。
- 将按 Session 版本化的 KodexAgentSettings、本地 NewSession 草稿与 models/认证/usage/MCP 运行状态留在各自范围，不塞进两份全局文件。

## 目录与加载

- 一期两份设置文件都位于当前应用的 dataDirectory，默认采用 KodexHome；自定义 dataDirectory 同时替换两者的根，不另加前端配置目录或文件路径选项。
- 同目录不改变两侧所有权；各侧只读取和更新自己的文件，不建立合并的持久化设置真源。
- 缺文件或字段使用所属侧默认值，未知字段忽略；已知字段非法或 YAML 损坏时报错，不以默认值掩盖错误或覆盖原文件。
- 正常读取不创建、重写设置文件；更新沿用同目录临时文件加原子替换，只发布所属侧文件，不覆盖另一侧。
- 磁盘配置可稀疏；后端完成默认值解析后才发布完整 BackendSettings，RPC 的必填字段不要求用户在 YAML 写满全部配置，也不由前端补后端默认值。
- 普通加载不回退读取旧 settings.yml；旧文件拆分统一经[既有 Home migration](kodex-home.md#migration)完成，不增加第二套迁移入口或修改已发布 migration。

## 文件切换门禁

- 独立 store 和冻结的拆分 migration 可以先在隔离目录准备、验证，不由组件落地自动启用生产读取或旧文件删除。
- 两侧新加载器与拆分 migration 必须在同一次生产切换中生效；不得先删 settings.yml 使旧加载器退回默认值，也不得先用新加载器把尚未迁移的旧 Home 当成空配置。
- 迁移激活以 [Home registry](kodex-home.md#migration-registry-管理)及实际生成的应用版本为准；登记 future entry 不等于当前版本会执行它。目标版本、读取切换或相应授权未就绪时，不启用新生产路径。
- 切换前验证隔离旧 Home 的完整启动链：版本准备、转换和最后删源完成后才打开新 store；不以直接调用转换函数的测试代替该验证。
- 不增加双读、双写或第二套迁移入口作为过渡；普通加载、迁移重入和未知路径保留继续按各自规则处理。

## 旧设置拆分

- 旧 settings.yml 的普通字段按[文件与字段归属](#文件与字段归属)迁入两侧；侧栏只迁移 left/right 内容，不迁移已裁剪的持久宽度。
- 新前端 hooks 初始化为空，不自动转换或启用旧 Hook 命令，不增加旧事件/脚本输入兼容层。
- 两份目标设置确认完成后才删除旧 settings.yml，不保留旧文件或另存副本；未转换的旧 Hook 命令等内容不进入新设置。
- 只完成一侧时不得删除源文件；全部转换与源文件删除完成后，才由既有 Home migration 推进目标版本。

## 拆分迁移重入

- 本节只用于 Home 版本仍需执行拆分 migration 时；正常加载仍允许任一设置文件缺失并使用默认值。
- 旧 settings.yml 存在时，先按冻结迁移规则计算两侧目标并检查全部已有目标；一致目标跳过，只补缺失侧，冲突或损坏时拒绝，不覆盖现场。
- 两侧目标均与旧文件转换结果一致后才删除源文件；删除失败则保留当前目标，下次仍从旧文件计算并核对。
- 旧文件不存在且两侧目标都有效时，识别为目标已就绪，允许继续版本推进；三份文件都不存在时，无旧配置需要转换，不生成默认文件。
- 旧文件不存在但只剩一侧目标，作为迁移歧义拒绝，不猜测补另一侧默认值；需检查的文件损坏时同样拒绝且不推进版本。
- 不承诺跨文件回滚，不增加 journal、backup 或完成标记；按既有 Home 独占迁移入口和版本提交顺序处理，未知 Home 路径不参与转换或清理。

## RPC 读写

- GlobalRpc 的 getSettings/getSettingsFlow/compareAndSetSettings 直接传完整 BackendSettings；不保留独立 GlobalSettings 公开投影。
- BackendSettings 顶层字段全部必填，不由前端本机默认值补后端快照。
- 允许其中完整 MCP 配置及内嵌凭据跨线；不扩大到独立 OpenAI 凭据存储，不记录原始配置或 RPC 载荷。
- 全量 CAS 包括 MCP 凭据；后端续期导致值变化时也参与比较，不忽略字段或回写脱敏假值。
- 前端设置由本地 store 管理，不经 GlobalRpc 同步；页面同时展示两侧设置不构成可写的合并真源。
- SettingsTimelineRpc 提供 persisted Session 的版本化 KodexAgentSettings 读取及当前值 compareAndSet，不接管虚拟草稿或全局 defaults；不保留独立 AgentSettingsRpc。
- 当前设置通过 timeline 的 cacheNonce/latestIndex 通知与值查询重建；初始化元数据之后依赖订阅，不轮询初始化 Get。实际值查询仍是按需数据读取，不另加当前 settings Get/Flow。
- 所有状态与 CAS 遵循 [RPC 状态](rpc-state.md)；实际写入仍须保留 runtime-owned 字段的合法性约束。
- 全局和新建默认值页面的设置写入在前端接受后由应用作用域排队；关闭弹窗只拒绝新编辑、释放页面观察，并排空已接受写入，不以取消页面 Job 撤销后端可能已接受的命令。MCP 配置写入所需的草稿或预览状态在已接受写入完成前保持可用，关闭弹窗不使其自动失效。Session 专属编辑仍须校验其确切活跃绑定，不借全局队列对失效 Session 重放。
- 全局与新建默认值页面的未知操作异常写入应用级本地失败状态，弹窗关闭后仍可在重开时看到；界面只展示不含远端详情的提示并允许确认清除，日志不记录原始配置或凭据。取消不作为业务失败，后续无异常完成的编辑可清除旧提示；不借后端 Notification 流显示前端设置错误。
- models 以 GlobalRpc.getModels/getModelsFlow 返回原 List<ModelInfo>，只读、保留目录顺序，不另设 OpenAiModelCatalogRpc。
- 账号用量作为同一全局服务的独立只读状态，按 [RPC 账号用量](rpc-account-usage.md)处理，不纳入 settings CAS。
- modelOptions 等可由 settings/models 纯计算的展示值留前端；模型目录内部职责沿用[模型目录](model-catalog.md)。

## Frontend notification only

- 通知采用后端生产业务事件、前端消费事件并调用本地 Hook 的模型；不是前端错误提示状态的远程镜像，也不只依赖发起调用者仍在等待的 RPC 回执。
- 通过 GlobalRpc.getNotificationFlow 返回 Flow<Notification>，前端按事件分支分派 Hook，不按旧 notification 的 id/level/message/dismiss 状态建立 RPC。
- 一次订阅覆盖各 persisted root Agent，不依赖前端打开的 tab 或特定 Session 活跃；个别 Session 的 TTL 卸载不结束全局通知流。
- 保留外层 Notification 联合，一期只实现 Notification.Stop；Stop 内按原因区分 AssistantMessage、RequestUserInput、SuggestSubagent、UnhandledError 四个联合分支。其他通知类型未来再增加，不预建占位分支；UnhandledError 不与 Stop 平行。
- AssistantMessage 表示助手消息结束，RequestUserInput 与 SuggestSubagent 区分两类宿主交互等待，UnhandledError 表示未处理错误；前端按类型分派，不解析文本猜停止原因。
- 不生产 UserPromptSubmit、PreToolUse、PostToolUse、PreCompact 或 PostCompact 通知，也不为其预留分支或后端发布入口。
- 这是事件订阅，不提供初值 Get、通知 CAS 或 dismiss RPC；不把事件流包装成只保存最后一个通知的 StateFlow。
- 通知是实时、无 replay 的广播，语义对应 replay=0 的 SharedFlow：订阅者不接收订阅建立前的事件；无订阅者时不为未来订阅者保存通知。
- 重新订阅只接收新事件，不补发断开期间或此前已发生的通知；不建立通知历史、持久队列、确认回执或补播协议。此规则不改变普通模型输出流的完整 replay。
- 新 Hooks 只承担前端本地通知；不实现 backend hooks，不让后端等待前端 Hook 的控制结果。
- 通知成功、失败、超时或前端断连不改变后端任务的准入、结果与完成状态；没有前端时后端仍继续工作。
- 不保留旧 pre_tool_use 阻止执行、user_prompt_submit 注入上下文/阻止流程、stop 续跑等控制语义。
- 不通过“收到通知后自动调用业务 RPC”变相恢复控制 Hook。
- 通知配置和执行归前端，不把旧 hooks 原样复制到 BackendSettings 或新前端模型，不增加后端 Hook CRUD。
- 订阅取消只释放该通知收集，不停止后端 Agent；Hook 输出不回传作为业务执行决策。RPC 调用异常与通知事件各司其职，事件不能把失败调用伪装成成功。

## 通知投递

- 后端通知源使用 replay=0、extraBufferCapacity=64 和 DROP_OLDEST；发布使用 tryEmit，不因等待缓冲空位挂起 Agent。容量为一期固定默认值，不增加设置字段。
- 接受在线慢消费者在缓冲溢出时漏掉积压旧通知；这是尽力投递，不承诺每个在线前端必达，不因漏通知修改业务状态。
- 无订阅者时不保留事件；额外缓冲不向新订阅者补播。tryEmit 成功不是前端收到或 Hook 执行成功的回执。
- 不用每事件无界 launch 或无限队列积压待发送任务，也不增加通知确认、重放或持久化。
- 丢失提醒不表示 Session、历史或待处理问题消失；业务数据仍由对应状态和命令提供。此丢弃策略不适用于普通模型输出的完整 replay。

## 前端本地 unhandled error

- Application 恢复原本的本地 unhandled operation reporter，实际 CLI 将它接到 Root 的错误回调；日志保留原始异常，取消不作为错误报告。
- Catalog/History 等前端操作失败不是后端 Agent Stop，不向 RPC Notification 联合增加分支，也不伪造 Session 标识。
- 本地输入使用 `{"type":"unhandled_error","message":...}` JSON，message 可空；复用现有 `stop_unhandled_error` 配置选择和唯一 Hook 执行器。后端四种 Stop 的 JSON 与含义不变。
- 本地消息 live、无 replay、64 条缓冲、DROP_OLDEST；不逐错误 launch 等待发送，不持久化、不确认执行结果。Application 关闭取消当前执行，不排空积压。
- 两种输入共用配置快照、串行执行、10 秒命令预算与清理；Hook 失败只诊断，不递归报告，也不改变原操作的成功/失败或控制后端。

## 前端 Hook 配置

- CliFrontendSettings.hooks 使用有序 List<NotificationHook>；条目包含 name、types: Set<NotificationHookType>、command。
  真实 Hook 值模型归 `hook/spec/notification`，保留原 rpc.models 包名和序列化形状，只保存到前端文件。
- name 与 command 非空白，name 在 hooks 列表中唯一，作为编辑和诊断标识；保留原字符串，不隐式修剪或改写命令。不新增 UUID、enable、超时或并发配置。
- types 非空，保留四个选择项；序列化值为 stop_assistant_message、stop_request_user_input、stop_suggest_subagent、stop_unhandled_error。最后一项也选择上述前端本地错误，不增加通配或自动选择未来类型的语义。
- hooks 顺序影响执行及设置值相等性，types 的集合顺序不影响匹配；不同命名 Hook 可使用相同命令或重叠类型，不按 command 去重。
- 开始处理每条通知时读取一次本地配置快照，固定本次命令链；之后的配置更新只影响下一条通知。空 hooks 不执行命令。

## 前端 Hook 类型匹配

- 单个前端 Hook 使用 types 选择多个通知类型，不限制为单一 type；同一命令可消费多种 Notification 分支。
- 每条通知匹配其具体分支，只要该类型属于 Hook 的 types 就执行一次该 Hook，不因多类型配置重复执行同一事件。
- 匹配使用 Notification 的具体分支，不解析消息文本或依赖类名反射；配置集合中重复的类型值只算一次选择。

## 命令输入与环境

- stdin 直接使用 `Json.encodeToString<Notification>(notification)` 的 UTF-8 内容，写完关闭 stdin，不额外补换行；同一通知的各 Hook 接收相同 JSON，不另建输入 DTO 或配置包装。
- 四类通知都使用该输入，包括 UnhandledError；模型内容与诊断仅作为 JSON 数据交付，不拼接进 shell command，也不恢复旧错误 Hook 的纯文本输入。
- cwd 固定为前端启动时解析并捕获的目录，不随选中 tab 改变、不重读 live Session，不使用后端 Agent cwd；相对命令路径按该目录解释。
- 使用前端本机的 Shell.default、login=false、tty=false，继承前端进程环境，不增加环境覆盖；不读取 BackendSettings.shell，不提供每条 Hook 的 cwd/shell/environment 字段。
- 不承诺与旧 Hook 配置或脚本输入兼容，不因迁移自动执行旧命令；文件迁移与执行器接入不由值模型自行完成。

## 前端 Hook 执行

- CLI 生命周期内只建立一个全局通知订阅，与 tab、当前 Session 和业务 RPC 等待分离；不在 UI 渲染路径执行命令。
- 使用普通 collect 按接收顺序处理通知，同一通知匹配的命令按配置顺序串行执行；不使用 collectLatest 因新通知取消当前 Hook，不逐事件 launch 或另建积压队列。
- 每条命令固定 10 秒执行预算，包含输入交付与等待完成；一期不增加超时、并发数或重试配置。接受串行慢 Hook 延迟后续提醒及长脚本超时。
- 启动失败、非零退出和超时只记录前端本地诊断，不影响 Agent、不生成新的 Stop.UnhandledError、不自动重试；清理本条后继续后续命令和通知。
- Hook 输出不作为控制结果解析，也不直接写入 TUI 终端；不把命令失败升级为整个通知订阅失败。
- 超时或取消须请求终止进程并完成清理，再启动下一条；不得以协程超时或 close 返回代替进程已退出的确认，接入时验证实际退出等待和异常清理。
- 前端生命周期取消向上传播，取消订阅并清理当前 Hook；不为排空积压通知延长 CLI 退出。关闭单个 tab 不停止全局订阅。
- 后端 64 条缓冲不等于端到端总条数或字节上限；接入时同时核对 RPC 缓冲和进程资源，不用额外无界积压抵消既定丢弃策略。

## Stop 通知载荷

- 在 rpc/models 定义 Serializable sealed Notification 与嵌套 sealed Notification.Stop；四个具体分支均携带 sessionIndex: Int，不新增 Session DTO、通知 ID、缓存标记或旧 Hook 控制字段。
- AssistantMessage 携带本次停止对应的原 StableAssistantMessage，不用任意历史消息冒充本次结果。
- RequestUserInput 携带非空 List<PendingRequestUserInputToolEvent>，SuggestSubagent 携带非空 List<PendingSuggestSubagentTaskToolEvent>；复用原参数、callId、问题/任务分组和顺序。
- 一次宿主等待按类型聚合，每种出现的类型发一条，保留其全部调用；两类同时存在则各发一条，不视为原子批次。不取 first 丢弃请求，也不按单个问题或任务拆分通知。
- UnhandledError 只表示已接受 Agent 运行最终被非取消错误终止，携带可空 message 诊断字符串，不传 Throwable、栈或 cause，也不对该字符串执行 RemoteException 恢复。
- 内部可恢复重试、普通工具失败结果、主动取消、辅助标题失败及目录/设置等无关操作失败不生成 Stop.UnhandledError；相关调用失败和后端日志仍保留，不伪造 Session 标识。
- 事件只描述发生时的事实，不证明当前仍在等待；回答和子任务创建继续依赖已有状态与命令校验，不能把通知当作执行许可。
- 不复制旧 HookTurnContext、整份 settings 或承诺旧脚本的 cwd/model/turn_id 包装兼容；前端按事件业务载荷分派，不以事后重读 live Session 冒充事件时的快照。
