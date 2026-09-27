# Task Tree

- **`Await scoped implementation authorization`()**
- `Publish backend Stop notifications without control hooks`()
- `Implement one frontend notification consumer and command executor`()
- `Verify overflow matching timeout and process cleanup`()
- `Assemble complete backend services for pre-CLI integration`()
- `Record evidence and complete this batch`()

# Details

## 位置与前置

- 实施批次 P07；父任务：[实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。
- 状态：Planning；依赖 [P03](2026-09-27-rpc-backend-settings-models-mcp.md)、[P04](2026-09-27-rpc-backend-auth-oauth-usage.md)、[P05](2026-09-27-rpc-runtime-timeline-services.md)、[P06](2026-09-27-rpc-session-catalog-management.md)，复用现有 Notification、NotificationHook 与前端 store。
- 交付：通知发布/执行器，以及前端批次开始前的完整后端装配检查点；配置编辑归 P09，生产 CLI 选择归 P11。
- 规范：[分批实施边界](../../checklist/rpc-architecture.md#分批实施边界)、[通知与 Hooks](../../checklist/rpc-settings.md#frontend-notification-only)、[投递](../../checklist/rpc-settings.md#通知投递)、[进程清理](../../checklist/coroutine-resource-lifecycle.md)。

## 实施范围

- 在真实停止位置产生 Notification.Stop 四分支；宿主请求同类聚合，UnhandledError 只包含已接受运行的最终非取消失败。
- 后端源 replay=0、extraBufferCapacity=64、DROP_OLDEST、tryEmit；全局源不随单 Session TTL 结束，不为无订阅者保留事件。
- 新后端路径不执行旧控制型 Hook，也不等待前端结果；旧链在 P11 切换时退出，迁移期间不在同一路径双发/双执行。
- 前端只建立一个全局 collect，按配置快照、types 和列表顺序串行执行；直接以完整 Notification JSON 写 stdin，写完关闭。
- 命令使用前端启动 cwd 和本机默认 shell；单条10秒，输出不写 TUI、不解析控制结果，失败本地诊断后继续。
- 超时/取消请求终止并等待进程清理；取消前端 scope 向上传播，不排空积压、不恢复旧 Stop 续跑。
- 入口：[原 Hook 实现](../../Kodex/hook/impl)、[运行 Hook 装饰器](../../Kodex/agent-runtime/decorator/turn-hook)、[通知模型](../../Kodex/rpc/models)、[ShellClient](../../Kodex/utils/shell-client)。

## 前端接入前的后端装配

- 本批补齐最后的通知切面后，汇合 P03/P04/P06 的业务组件，实现完整 GlobalRpc 并注册八个服务；创建可供隔离测试及 P11 复用的装配入口。
- 所有方法必须真实委托，无未实现占位或假成功；mock 限于外部模型/供应商等依赖，不把业务适配本身替换为固定结果。
- 复用单内存连接和公共客户端保护，使用隔离 Home/假凭据/测试命令；此时不启动默认 CLI、切换旧加载器或激活拆分 migration。
- 这是 P08/P09 真 RPC 测试的前置，不是新增运行特性；P11 不再首次发现组件无法组合。

## 验收

- 四分支触发、混合宿主请求及 payload 顺序正确；可恢复重试、辅助标题失败、普通操作错误和主动取消不误发 UnhandledError。
- 无订阅/重订阅无 replay；慢消费者溢出允许丢旧事件，但不阻塞 Agent 或新增无界任务。
- 单 Hook 多 types 只执行一次；每通知配置快照固定，命令串行；直接 JSON 而非插值到 shell。
- 进程启动交接取消、输入挂起、非零退出、超时、清理失败和宿主退出均释放资源；普通失败不打断后续通知。
- 真 RPC 通知往返及 JVM/Native fixture；仅运行隔离、无外部副作用的测试命令，不执行用户 Hook。
- 完整后端编译并通过全局状态、创建/保活/目录、认证/用量/MCP 命令及通知的代表性真实 RPC 往返；八服务均注册且方法无占位。各领域细粒度回归仍归原批次。

## 交接

- P08/P09 复用完整后端测试入口，P09 提供配置编辑；P11 在 CLI 生命周期只装配一次，并移除旧控制 Hook 的生产入口和依赖。
