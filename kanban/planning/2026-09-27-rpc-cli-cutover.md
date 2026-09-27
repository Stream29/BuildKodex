# Task Tree

- **`Await scoped implementation authorization`()**
- `Verify every required backend and frontend component is complete`()
- `Compose one backend frontend and in-memory connection`()
- `Switch production factories settings readers and migration activation together`()
- `Remove superseded production ownership and control-hook paths`()
- `Verify startup failure and shutdown in isolated CLI runs`()
- `Record evidence and complete this batch`()

# Details

## 位置与前置

- 实施批次 P11；父任务：[实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。
- 状态：Planning；P01–P10 全部完成后才能开始生产切换，不能以占位返回值满足依赖。
- 交付：实际单 CLI 经内存 RPC 使用完整后端，迁移和新文件读取同步启用，旧直接业务路径退出生产。
- 规范：[分批实施边界](../../checklist/rpc-architecture.md#分批实施边界)、[内存组合根](../../checklist/rpc-architecture.md#内存连接与宿主生命周期)、[文件切换门禁](../../checklist/rpc-settings.md#文件切换门禁)、[实体所有权](../../checklist/rpc-session.md#repository-实体与前端视图)、[Home](../../checklist/kodex-home.md)。

## 实施范围

- 复用 P07 已验证的完整后端装配，在现有 Home/logging 启动框架下注册 GlobalRpc、AgentRuntimeRpc 和六条 timeline；沿用一对 withInMemoryRpc 端点及 RestoringRpcClient，不在本批才首次组合后端组件。
- GlobalRpc 聚合 P03/P04/P06/P07 全部能力，不以 unsupported、空结果或 no-op 隐藏未实施的方法。
- 前端只取得 RPC 代理与本地 store/环境能力；不泄漏 repository、Session、模型 client、MCP manager、原始凭据或后端可写 storage。
- 同批切换 Application/CLI 生产工厂、tab/目录释放责任、两侧文件读取及 P10 migration 激活；按版本 SOP 核验适当目标，不在旧加载器尚存时删除旧源。
- 只建立一个通知消费者；不同时运行旧控制 Hook、旧 settings 真源、旧 execution 聚合或每 tab 实体 repository。
- 将已无消费者的过渡适配、旧生产路径和依赖随替换清理；冻结 migration/历史 codec 必须保留，不借清理删除其他功能。
- 接收 P08/P09 暂缓的破坏性 contract/renderer/工厂替换，一次移除旧 execution、实体参数与旧控件调用，避免中间提交让原 CLI 无法编译。
- 启动失败按已取得资源收尾；renderer 结束/取消或共享连接死亡清理双方和 Home handle，不自动重建连接、不重放命令。
- 入口：[Application](../../Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt)、[CLI Main](../../Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt)、[内存连接](../../Kodex/rpc/in-memory)、[前端 registry](../../Kodex/app/viewmodel/session)、[迁移模块](../../Kodex/app/migration/impl)。

## 验收

- 八个服务完整注册，全部契约方法有真实委托；新前端不再调用原业务实体或后端文件加载器。
- 新 Home、旧 Home 升级与自定义 dataDirectory：迁移后只读新文件，不丢普通配置，不双发 Hook；保持日志/artifact 原目录约束。
- 在 Home、后端、服务注册、前端初始化、renderer 各阶段注入失败，检查连接/协程/进程/租约释放及主错误保留。
- 关闭 tab 不关后端；整个 CLI 退出释放两侧；单次调用/Flow 错误不误判为连接死亡。
- JVM 和本机 Native CLI 编译、链接、隔离运行；不抢占用户 TUI、真实 Home 或其他设备。

## 交接

- 交给 [P12](2026-09-27-rpc-integration-acceptance.md) 做整体回归与完成判定。
- 本批需要实际版本激活时，先核对并遵循发布/提交规则；不自动 push、发 release 或升级用户 Home。
- 激活以实际生成的应用版本覆盖 P10 目标为准；仅切换源码或登记 future entry 不算已经迁移。版本/提交授权未就绪时，保留隔离装配结果，不先开放新生产加载器读取旧 Home。
