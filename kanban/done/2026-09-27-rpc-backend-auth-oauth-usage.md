# Task Tree

- `Receive the P03 handoff`()
- `Adapt both file credential sources under backend ownership`()
- `Implement bound OAuth attempts and pending-callback expiry`()
- `Connect account usage and explicit-credit reset`()
- `Verify cancellation late results and credential isolation`()
- `Record evidence and complete this batch`()

# Details

## 位置与前置

- 实施批次 P04；父任务：[实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。已获顺序实施授权，P03 已完成。
- 状态：Done；依赖 [P03](2026-09-27-rpc-backend-settings-models-mcp.md) 的后端 settings/MCP 写入接线，完整服务装配仍归 P07。
- 交付：GlobalRpc 的认证、统一 OAuth、usage、reset 及 logoutMcpServer 业务组件；不实现前端 listener。
- 规范：[分批实施边界](../../checklist/rpc-architecture.md#分批实施边界)、[认证](../../checklist/rpc-authentication.md)、[账号用量](../../checklist/rpc-account-usage.md)、[MCP](../../checklist/rpc-mcp.md)。

## 实施范围

- 两种来源均由后端加载、续期、回存、登录和移除，复用原格式/store；Codex 仅固定 auth.json，不增加 keyring/auto。
- 保持先保存再发布及原回存失败路径；拒绝后端自己的迟到续期/登录恢复已移除凭据，不增加待回存队列。
- getAuthentication/GetFlow 只返回选中来源摘要，凭据不跨线；登录目标与 authSource 切换独立。
- 统一 start/complete/cancelOAuthLogin，后端保存目标、redirect/state/PKCE 和不复用的 attemptId；只解析 callback URL，不请求它。
- 待回调阶段10分钟过期；接受 code 至多一次，接受后的交换/保存归后端，不随 RPC 等待取消。精确取消与 logout/remove 保留各自准入。
- 接 usage Get/Flow、refreshAccountUsage 与指定 creditId 的 consumeUsageReset；原私有 attempt/供应商幂等值不跨线。
- 不打开浏览器/前端回调端口，不新增手动 Reload 或登录状态恢复 RPC，不改变既定业务错误模型。
- 入口：[认证 store](../../Kodex/app/shared/auth/filesystem)、[用量 store](../../Kodex/openai/account-usage/impl)、[MCP manager](../../Kodex/mcp/impl)、[GlobalRpc](../../Kodex/rpc/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/GlobalRpc.kt)。

## 验收

- 源切换、重登录失败、删除与刷新/保存竞争：不污染另一来源或恢复旧凭据，摘要不泄漏 token。
- 错误 state/地址不消费有效 attempt；匹配 OAuth error 结束；重复 code、未知 attempt、迟到取消均按契约处理。
- start 结果未交付时过期释放占位；回调接受与过期竞争；接受后的交换不被待回调期限误取消。
- MCP 身份修改后拒绝旧提交；cancel 回执不当成清理完成，logout 保留原登录准入。
- usage previous 不跨账号；只消费指定券，不自动换券；已有确定消费结果不被后续刷新失败抹掉。
- 使用 mock provider、测试时钟和隔离假凭据目录，覆盖 JVM/Native；不执行真实登录、refresh、reset 或读取用户认证。

## 交接

- 与 P03 的 settings/MCP 组件对接后，在 P07 装配真实 client、model catalog 和 usage 的共享资源图；不能保留对旧 Application/旧全局设置 store 的反向依赖。
- P09 实现前端 listener、浏览器、取消及选券 UI；P07 先在隔离宿主注册完整服务供测试，P11 才选择生产入口。
- 不以认证组件测试代替真实 CLI 集成或供应商端有效性验证。

## 实现与验证

- 后端 [双文件凭据 store](../../Kodex/app/shared/auth/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStore.kt) 复用原格式、JWT 投影与刷新时间规则，独立维护两种来源；提交/删除共享写锁与登录 fence，成功保存后才发布。Codex 回存保留原 auth mode，不新增 keyring。
- [统一 OAuth](../../Kodex/rpc/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendOAuth.kt) 保存后端私有 PKCE/目标/回调身份，以唯一 ID 管理准备、等待、接受和清理；错误地址/state 不消耗 attempt，匹配 error 终结，10分钟只限制待回调阶段。
- 原 MCP OAuth 的发现/注册/交换抽出无 listener 的准备入口；manager 复用原持久化准入，后台 callback 不发布前端 effect。删除取消旧交换 Job，提交额外核对 URL，清理在 NonCancellable 中释放原登录占位。
- [账号状态](../../Kodex/rpc/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendAccountState.kt) 输出原摘要，usage 子 scope 属于后端；指定 credit 的私有 attempt 不跨线。新 owned usage 工厂保留已取得的确定结果，旧 CLI 工厂行为不切换。
- 新增28项测试（文件来源10项、OAuth14项、usage4项），使用假凭据、mock provider、隔离目录和可控期限 barrier；不读取真实认证、不执行真实登录或 reset。
- 四目标库与测试源码编译及旧 CLI JVM 编译通过；四模块 JVM99项、Native89项通过，其中 server 两端各48项。macOS 沿原 cinterop 条件跳过；Native/CLI 混合构建成功但既有 Mosaic 配置缓存问题导致缓存丢弃，未顺带修改构建框架。
- 初轮修正新增代码的 Ktor dependency alias、显式类型/Unit 警告、enum 断言及测试初始化维护竞争；通过可控首次维护 barrier 确定测试顺序，不以 sleep 回避。未改变 RPC contract、凭据传输边界或生产启动路径。
- server 与 auth filesystem 的 JVM/Native 测试已强制重跑通过（server 各48项，auth JVM30项/Native28项）；重复运行继续使用隔离目录，没有触碰真实凭据。
