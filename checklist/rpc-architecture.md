# RPC 前后端分离

## 适用范围

- 本文件及链接的 RPC checklist 记录已确认的迁移目标，不表示现行 CLI 已完成迁移。
- 审查或实现 RPC 切面时采用这些目标；维护尚未迁移的进程内实现时，继续遵守对应现行 checklist。
- 不从设计记录推导实施授权；契约落地情况、研究证据、未决项与验证结果留在任务文档。

## 一期部署与交付

- 一期仅重构内部边界，保留单一 CLI 分发和原有使用方式；用户不需要单独启动、连接或管理后端。
- 在同一 CLI 进程内组织隐藏的 frontend/backend，经 in-memory RPC 通信，不启动独立后端进程或 daemon。
- 不把“多个 CLI 共享一个后端”作为一期目标，不增加服务发现、端口配置、跨进程版本协商或网络重连工程。
- CLI 进程退出仍由组合根清理两侧资源；不承诺进程退出后后台任务继续运行。
- 内存承载不取消契约隔离：前后端业务访问仍经过 RPC，不退化为直连业务 ViewModel、共享可变对象或绕过后端的文件访问。
- 一期同时交付[设置拆分、临时侧栏宽度与 notification-only Hooks](rpc-settings.md)、[Session 保活与 TTL 回收](rpc-session.md#保活与回收)、[显式选券 reset](rpc-account-usage.md#重置券选择)；这些调整已获用户接受，不因保留单 CLI 使用方式而后移。

## 业务边界

- 在宿主仍存活时，前端视图关闭或等待取消不等于取消后端已接受的工作；显式 Stop、删除与资源释放分别处理。
- 以保持后端合法状态为并发正确性标准，复用[既有写入准入与串行化](agent-state-mutation-serialization.md)；不增加跨前端意图仲裁。
- 允许状态传播滞后，不要求命令返回时所有前端缓存同步；不得绕过后端校验直接修改存储。
- 前端保留自己的导航、弹窗、编辑草稿和展示状态；不传输 ViewModel、Job、滚动对象或进程内对象身份。
- repository 及其完整资源作用域归后端；前端只持有经 RPC 构建的本地视图，按 [实体与视图边界](rpc-session.md#repository-实体与前端视图)接入，不以同进程共享引用替代 RPC。
- 模型调用由后端执行，包括自动标题等辅助请求；前端不直接访问模型服务。业务操作的触发方式按各自契约确定，不因调用由前端发起而将模型执行移到前端。

## 工作目录选择

- 一期工作目录选择器继续在前端本地解析路径、展开 ~ 并枚举目录，不新增目录浏览 RPC；交互及选择行为沿用[路径选择器](path-picker.md)。
- 该边界依赖一期前后端同进程的文件系统环境；独立部署前重新审查，不据此承诺跨机器或跨操作系统浏览。
- 本地浏览不授权前端绕过 RPC 读取 Session 存储、后端设置或凭据；选择后的配置更新仍走所属状态的既定提交入口。

## 内存连接与宿主生命周期

- 每个 CLI 共用一个 kRPC client/server 和一对有界内存通道；GlobalRpc、AgentRuntimeRpc 及六条 timeline 共用连接，不按 tab、Session 或 Flow 建连接。
- 使用 JSON 编解码协议消息，不直接传输业务对象。协议通道满时挂起发送，不使用通知的溢出丢弃策略；通道容量不代表字节级内存上限。
- CLI 组合根拥有 frontend、backend 和连接资源；后端已接受任务仍归属后端/Session，不绑定前端调用的等待协程。
- 沿用 Home 准备及日志入口，准备后端并注册服务后初始化客户端和前端，再进入 renderer；启动中途失败清理已取得资源，不留下半启动后端。
- renderer 结束或 CLI 取消即结束宿主：停止前端新操作，取消连接双方并解除挂起收发，等待两侧资源清理，最后释放 Home handle。不新增 shutdown RPC，也不等待 runningTurn 自然完成。
- 共享 transport/端点意外终止而连接已不可用时，记录原因并结束宿主，走同一清理路径；一期不自动重建连接、前端或后端，不重放命令。
- 普通调用或 Flow 失败、Session 失活及单次取消不升级为连接死亡；关闭 tab 或取消订阅不关闭共享 client，也不取消后端已接受工作。
- 清理遵循[协程资源生命周期](coroutine-resource-lifecycle.md)，保留主失败，不因等待一侧退出而阻止另一侧取消；测试 fixture 的容量和超时不自动成为生产参数。

## 内存连接模块

- `rpc/in-memory` 提供 `withInMemoryRpc(registerServices, block)`：注册回调使用 RpcServer，block 使用 RpcClient；不依赖业务契约、Session 或 ViewModel。
- client、代理及连接自有资源不逃出使用作用域；正常返回前等待 block 的子协程结束。服务对象的外部资源仍由创建方持有并清理，不依赖框架自动关闭任意对象。
- 当前实现每方向容量16条协议消息，满时挂起；只支持固定 JSON 字符串，不增加用户配置或多格式协商。
- 主动关闭先停监视协程，再取消双方及通道；NonCancellable 收尾等待预算为10秒。它不是业务调用期限，也不承诺强制终止不响应取消的协程或整个进程的退出上限。
- 监视端点终止时同时检查连接作用域，避免外层取消级联尚未到达监视协程时，将正常关闭误报为连接故障。
- 返回的是原始 kRPC client；[异常还原与 Flow 保护](rpc-state.md#订阅释放与重连)由公共客户端适配承接，不混入 transport 或 utils/rpc-exception。

## 后续独立部署

- daemon、独立进程及其他分发/部署形式属于后续工程，不作为一期实现或验收要求。
- 此前同机、同用户共享后端和多前端同时操作同一 Session 的设计保留为后续方向，不据此在一期建立部署与协调设施。
- 独立后端下，前端退出/断连不直接终止后端已接受任务；runningTurn 通过[后端自保活](rpc-session.md#保活与回收)继续运行。该目标与一期整个 CLI 进程退出的生命周期不同，不为所有辅助资源增加永久驻留保证。

## 模块与契约复用

- 将可编译契约放在 `Kodex/rpc/contract`（`:rpc-contract`），必要的新值模型放在并列的 `Kodex/rpc/models`（`:rpc-models`）。
- 优先复用现有领域类型、方法语义和命名，不为已可传输的值复制 DTO、mapper 或平行状态机。
- 只为实际跨线的原值类型补齐必要序列化；不顺带序列化其整个 ViewModel、effect 通道或原始凭据对象。
- 普通泛型父接口统一同构方法；具体 `@Rpc` 接口用具体类型逐项显式 `override` 重声明，不采用泛型 `@Rpc`、抽象类服务或空子接口替代。
- 以 `GlobalRpc` 聚合应用全局能力，不按每个内部 store 或设置子页面建立独立服务。
- 使用按 Session 寻址的六条 timeline；当前设置读取与 CAS 统一在 SettingsTimelineRpc，不保留独立 AgentSettingsRpc。纯展示派生状态由前端从已有值计算，不为每个派生字段新增 RPC。
- 在接入获准前保持契约模块独立，不让现行 CLI/Application 提前依赖或启动新服务。

## 按主题加载

- 状态初始化、全量 CAS、本地抽象与订阅：[RPC 状态](rpc-state.md)。
- 持久化历史的轻量通知与按需读取：[RPC timeline](rpc-timeline.md)。
- 两侧文件、字段归属与通知型 Hooks：[RPC 设置](rpc-settings.md)。
- Codex/Kodex 凭据与统一 OAuth：[RPC 认证](rpc-authentication.md)。
- MCP 运行状态与 Codex 配置导入：[RPC MCP](rpc-mcp.md)。
- 账号用量读取与显式刷新：[RPC 账号用量](rpc-account-usage.md)。
- 目录、运行标记、管理命令和 TTL：[RPC Session](rpc-session.md)。
- Agent 运行态队列与原子更新：[RPC AgentRuntime](rpc-agent-runtime.md)。
