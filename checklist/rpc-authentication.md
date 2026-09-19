# RPC 认证与 OAuth

- 按 [RPC 迁移适用范围](rpc-architecture.md#适用范围)使用本文件；统一交互不等于统一 OpenAI/MCP 的内部协议实现。

## Codex 与 Kodex 来源

- 两种来源均由后端完整参与加载、续期与回存，并支持 login/relogin/remove；不将 Codex 来源只读限制带入迁移实现。
- authSource 选择凭据来源，不选择是否负责续期；Codex 回存共享 auth.json，Kodex 回存私有 auth.yml，保持各自格式兼容。
- 一期沿用 Codex 本地文件兼容范围，只管理固定 ~/.codex/auth.json；不新增 keyring、auto 存储选择或跨进程 ephemeral 凭据支持，不为此读取 Codex 配置。login/remove 只承诺作用于该文件来源，不代表管理所有 Codex 登录存储。
- 续期前重读并核对来源/账号，采用已有更新或续期，再回存对应来源并发布状态；不复制同一凭据到另一文件长期独立续期。
- 复用原认证 store、登录客户端与格式解码，不为两个来源先建立泛化 provider 框架。
- 正常加载与续期不依赖前端定时调用；不新增凭据 CAS 或仅为维持可用性增加手动 reload RPC。
- 迁移后的 OpenAI 设置页移除两种来源的手动 Reload 入口，不新增 reloadAuthentication RPC，也不把摘要 Get 改为重新加载凭据；后端原 reload 能力及自动维护逻辑不因此删除或改写。
- GlobalRpc.getAuthentication/getAuthenticationFlow 返回当前选中来源的 SettingsAuthenticationState 摘要，不发送 OpenAiAuthState 原始 token。
- 来源切换用 settings CAS；登录目标在发起时明确绑定，登录完成不隐式切换 authSource。
- relogin 成功取得并提交新凭据后再替换旧值；失败或取消不预先清空旧凭据。
- removeAuthentication(source) 只移除目标来源本地凭据，不更改其他来源、settings 或执行远程 revoke；不存在时成功。
- 防止后端自己的迟到登录/续期恢复已移除凭据；不承诺控制独立 Codex 实例或立即清除其缓存。

## 续期回存失败

- 两种文件来源均沿用先持久化、再发布新凭据的顺序；取得刷新响应不等于已经完成本地提交。
- 写入失败按原普通异常和后台维护路径处理，不为此增加已知远端异常、待回存队列或未保存凭据模式，也不专门保留本次结果重试保存。
- 后续维护先重新读取来源；失败不触发删除凭据或强行回写旧快照。文件异常不证明写入完全未发生，也不保证旧 refresh token 后续仍可续期。

## 统一登录交互

- 在 GlobalRpc 使用 startOAuthLogin(target, redirectUri)、completeOAuthLogin(attemptId, callbackUrl)、cancelOAuthLogin(attemptId)。
- OAuthTarget 显式区分 OpenAi(source) 与 Mcp(serverName)；OAuthAuthorization 仅携带正 attemptId 与非空 url，放在 rpc/models。
- 前端先启动 loopback listener，传实际 redirect URI，再打开授权 URL，将收到的完整 callback URL 经 RPC 提交。
- 浏览器交互复用[外部 URL 能力](external-url.md)；后端不打开浏览器或监听前端端口。
- 后端绑定目标、redirect URI、OAuth state、私有 PKCE verifier 与目标身份；完成/取消不再接受可漂移的目标参数。
- attemptId 在后端生命周期内跨两类目标唯一且不复用，不作为持久身份；保留目标原有登录并发限制。
- MCP redirect URI 必须匹配配置中的 client；后端执行发现/注册，不能借 start 隐式重写配置身份。
- 授权 URL 和 callback URL 只作临时交互数据，不记录日志、持久化或通过全局 effect 流分发给其他前端。

## 待回调登录的过期

- 准备成功且尚未接受有效回调的 attempt 仅保留有限时间；过期释放临时登录状态及其并发占位，旧回调不得继续交换凭据。
- 一期等待上限固定为 10 分钟：后端从准备完成时计时，前端从取得授权信息后限制自己的回调等待；不要求两侧精确同步，不新增配置、expiresAt 字段或过期通知 RPC。
- 接受回调与过期在同一 attempt 的状态转换边界协调；已接受有效 code 后退出待回调阶段，不用等待回调的期限取消后端交换或保存。
- 过期不删除已有凭据、不切换 authSource，也不承诺回滚 MCP 已完成的客户端注册或配置准备写入。
- 以过期兜底未交付句柄的清理，不新增前端 ack、登录查询/恢复 RPC 或持久化 attempt；用户等待过久须重新发起登录。
- 前端持有 attemptId 时仍按原取消规则释放，不以最终过期替代正常取消与 listener 关闭。
- 前端登录交互完成、等待超时或主动取消时关闭 listener；退出未完成交互且有 ID 时，在有限清理中尝试 cancelOAuthLogin。超时提示用户手动重试，不自动重开浏览器或重发 start。
- 此期限只限制待回调阶段，不作为供应商 code/token 有效期保证，也不重新定义准备请求或已接受交换/保存的超时。

## 提交、取消与退出

- 后端将 callback URL 作为数据解析，不请求或跳转该 URL；先校验 pending attempt、地址、state 与响应结构。
- 畸形或不匹配的响应不消费有效 attempt；匹配 state 的 OAuth error 终结本次登录并报告失败，未知或已消费 attempt 拒绝提交。
- code 至多接受一次；接受后的交换/保存由后端持有，取消 RPC 等待或断连不等于显式取消登录。
- complete 成功表示凭据已提交；MCP 是否连接另看运行状态，不把浏览器打开成功当作认证完成。
- 回调交付前仍依赖前端，不承诺前端消失后授权交互能独立完成。
- cancel 只作用于 exact attempt；未知或已结束为 no-op，迟到取消不影响新登录，也不回滚已提交凭据。
- 不承诺 cancel 返回时异步清理全部结束；MCP 准备阶段可能持久化注册身份，取消不回滚这些准备写入。
- MCP 提交仍检查服务器/OAuth 身份有效；logoutMcpServer(name) 保留服务器、写回 Uninitialized 并关闭认证连接，要求无活跃登录，不执行远程 revoke。
- 保留 removeAuthentication 与 logoutMcpServer 两种退出语义，不把它们统一成按 attempt 注销。
- 丢失 start/complete/remove 回复可能已产生效果，不盲目重放；认证摘要只跟随选中来源，不作为其他目标某次登录的完成回执。
