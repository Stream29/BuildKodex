# Task Tree

- `Receive scheduled execution authorization`()
- `Verify every required backend and frontend component is complete`()
- `Resolve the independent application-version activation authorization`()
- `Compose one backend frontend and in-memory connection`()
- `Switch production factories settings readers and migration activation together`()
- `Remove superseded production ownership and control-hook paths`()
- `Verify startup failure and shutdown in isolated CLI runs`()
- `Record evidence and complete this batch`()

# Details

## 位置与前置

- 实施批次 P11；父任务：[实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。已获顺序实施授权，等待前批完成及生产切换门禁。
- 状态：Done；版本激活、生产接线及 Native 父级取消验收均已完成。P01–P10 已完成，交给 P12 做整体回归。
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

## 激活前核对

- [P10](../done/2026-09-27-rpc-settings-split-migration.md) 已准备并验证0.4.7迁移；当前两处Gradle版本及MCP客户端版本仍为0.4.6，生成Home版本也为0.4.6。
- 后端八服务、前端视图/编辑器及通知执行器均已有隔离验证；生产Application/renderer仍为旧路径，未提前删除旧加载器或启用迁移。
- 依照本计划原有的独立授权条款，需要明确允许本重构分支的版本激活，不能把“连续实现”解释为发布、提交或 main 操作许可。最小待确认范围是同批调整三个版本值到0.4.7并切换生产入口；仍不提交、不发布、不升级真实Home。
- 这不是RPC契约缺口；无需修改方法/模型。确认前不绕过coordinator、增加第二个当前版本常量、双读文件或提前开放新加载器。
- 用户已明确选择“允许并继续”：仅在refactor/rpc调整三处版本到0.4.7，与生产入口和文件读取同步切换，然后执行P12；不提交、推送、发布或升级真实Home，不修改main。

## 已实施的生产切换

- 三处源码版本统一0.4.7；Main 先准备 Home，再进入 `withKodexApplication` 的单后端、八服务、JSON 连接与前端 scope。迁移与两侧 store 同时启用，旧加载器不再进入 CLI。
- Session registry/draft、设置编辑器、Application 导航与 renderer 已替换实体入口；rootAgent 随 binding 可空重建，运行按钮从 AgentStateValue/running 派生。原 execution 聚合、ApplicationKoin、前端 Runtime/配置 store 桥已移除。
- 关闭 tab 只释放本地视图；Hooks 只保留一个前端通知消费者。认证页面无 Reload，两来源均经 RPC 登录/移除，MCP 导入只进行值 CAS。
- `app/test-support/rpc` 只在 test source set 使用完整后端及内存 RPC，外部模型/登录 mock；旧 frontend fake repository 测试改为真实代理、纯只读 History fixture 和 exact popup/nav 测试。
- 保留旧单文件 store 供 P10 codec 对照及旧文件权限回归，不由生产组合根调用；冻结历史 migration 未改。保留原领域 Hook 接口/测试，生产后端只注入 NoOpKodexHooks。
- 后端清理按顺序尝试全部资源并保留主失败，补充 renderer/close 双失败与正常返回后 close 失败验证。JVM debug stack recovery 会复制普通异常，suppressed 检查使用携带额外字段的真实测试异常，普通失败仍独立验证类型/message。

## 本轮验证与未完成项

- JVM 前端/renderer 九模块回归已通过；旧 history index 与 bounded window 各出现一次超时，未改算法，原样复跑通过。新 Docs history fixture 改为等待异步 RPC 视图完成，而不是假设命令返回即已渲染。
- Linux x64 CLI debug 编译/链接通过；链接任务的 Mosaic 旧 cklib task 不兼容 configuration cache，显式 `--no-configuration-cache` 后通过，没有修改插件或停止共享 Daemon。
- 两次独占测试 PTY：全新隔离 HOME、带普通字段/旧 Hook 的0.4.6 HOME，均进入真实 CLI，升级后显示指定模型；Ctrl-C 正常退出0，终端恢复、Home read lease 移除。两侧文件生成、旧源最后删除、旧 Hook 未启用，未访问真实用户 Home/凭据。
- 完整组合根测试覆盖五个启动阶段、取消、正常返回、清理失败、自定义准备根与0.4.6升级后的实际新字段读取；日志/generated artifacts 的默认进程路径未改。
- 首次多个 Native test binary 并行链接耗尽4GiB Daemon 堆；保留错误记录，Daemon 自行退出，使用同一 Temurin25 和 `--max-workers=1` 复验，不扩大资源预算。
- Native 还暴露测试目录相对/绝对路径断言差异，已规范化测试 fixture 或比较 resolve 后的路径；这不是生产目录行为修改。
- Native 首轮未通过：串行执行期间 Application 的父级取消触发下面的框架故障；当时未以 JVM 或 CLI smoke 代替该验证。解决及重跑结果见下节。

### 修复前可完成的检查

- JVM 实际报告：session8、new-session3、settings7、application24、agent12、history33、view-settings22、view-history50、view-application99，共258项通过。另 rpc-in-memory18项 JVM 通过，含新增并发取消测试；不是将 Native 挂起计作成功。
- CLI 的 JVM/Linux x64/Linux ARM64/Windows x64 源码编译通过；Linux x64 debug executable 链接与上述 smoke 通过；Application 的 ARM64/Windows 测试源码编译通过。macOS CLI 在 Linux 主机按既有 cinterop 限制跳过，未宣称链接其他平台或执行跨平台二进制。
- 命令采用 Temurin25、既有 Gradle Daemon，最终跨目标调用为 `--no-configuration-cache --max-workers=2 --continue` 加 `:app-cli:compileKotlin{Jvm,LinuxArm64,MingwX64,MacosArm64}`、`:app-viewmodel-application:compileTestKotlin{LinuxArm64,MingwX64}` 及相关 JVM test；Native测试串行调用仍失败，不因编译通过撤销阻塞。
- 新 tab 名称测试也改为先等待本地派生 name Flow，不把 rename 返回当作投影已同步。修改后的 renderer99项已通过；生产状态没有加入乐观回写。
- 两仓仍在 refactor/rpc；HEAD、main 和 gitlink 未变化，未提交/推送。`rpc/contract`、`rpc/models`、`utils/rpc-exception` 对本次基线无变更；复用跨线领域模型未暗改。清理临时 PTY/Home、Native abort 遗留目录及临时构建日志；错误证据保留在本节、Gradle test reports 和新增回归中。

## Native 取消阻塞与修复

- 实际栈：`CompletionHandlerException` → `ConcurrentModificationException` → `KrpcClient` 完成回调遍历 `requestChannels.values`；取消过程中其他请求正在移除登记，Native 测试进程因此退出。
- 已核对本地0.10.3 source jar：Native 使用 SynchronizedHashMap；`values` 仅在取得 live collection 时加锁，遍历已在锁外，`entries` 同样先释放锁再映射。JVM 使用不同实现，不能从 JVM 通过推导 Native 安全。
- 上游 [0.10.3 SynchronizedHashMap](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/utils/src/commonMain/kotlin/kotlinx/rpc/internal/utils/map/SynchronizedHashMap.kt) 与核对时 main 仍有该实现；[KRPC-548 的修复](https://github.com/Kotlin/kotlinx-rpc/pull/660)只调整测试 dispatcher，不是本次 live collection 的修复，不能据此直接升级版本或宣称已解决。
- 在原 `rpc/in-memory` 测试加入 Default dispatcher、32个 active Flow、父级取消重复32轮的真实往返回归；JVM通过，Native 没在60秒协程预算内结束，等待约两分钟后仅终止该测试进程。该挂起与 Application 的具体异常分别记录，不把未退出的 probe 伪称为第二次同栈复现。
- 用户批准在 `~/ACodeSpace/fork/kotlinx-rpc` 保留未来上游贡献的未提交修复分支。Kodex 独立 `rpc/krpc-utils-patch` 模块复用0.10.3 utils API，仅将 Native `SynchronizedHashMap` 的 entries/keys/values 在锁内复制；对 `kotlinx-rpc-utils` 做项目依赖替换，不引入整个 fork 的 `includeBuild`，不修改其它 kRPC 模块或冻结 RPC contract。
- Native KLIB `unique_name` 必须与0.10.3二进制引用的 `org.jetbrains.kotlinx:utils` 一致；初次链接报未绑定注解符号，设置模块名后成功。`dependencyInsight` 确认 Linux Native 的传递依赖解析为本地项目，补丁自身 JVM/Native 各1项快照测试通过。
- 真实 `rpc-in-memory` JVM/Native 各18项测试通过，含此前未结束的并发取消回归；Application Native24项通过，父级取消无 `ConcurrentModificationException`。首次重跑中14项失败源自测试 fixture 在 Native 对尚不存在的临时目录调用 `resolve`，修为创建目录所需的原始 Path；余下一项路径比较改为比较规范化路径后，Native24项通过。JVM原回归先前通过，P12继续覆盖两端。
- 重新链接 Linux x64 CLI debug executable 成功。新的独占 PTY/隔离 HOME 实际启动，Home 从0.3.2迁移到0.4.7、进入界面，Ctrl-C 退出0并恢复终端；临时 Home 已删除。此项不替代 P12 的矩阵或其他平台实际执行。
- 上游 fork、Kodex重构分支均未提交或推送；未读取真实 Home 或凭据，未调整0.10.3以外的协议组件。
