# Task Tree

- `Await first-wave Session approval`()
- `Load parent contract and inspect backend creation consumers`()
- `Implement cwd-only creation and backend defaults capture`()
- `Update business protocol checkpoint and targeted tests`()
- `Review scoped diff and report evidence`()
- `Receive parent acceptance`()

# Details

## 父级最终验收

- 已完成范围审查、JVM 测试与 macOS Arm64 编译；跨 Native frontend/JVM backend 的真实本地 RPC 也通过。记录见父任务 §6.7–6.10，任务移入 done。
- 后文“未执行/待验收”为 worker 交付时点；最终结果以父任务实际运行记录为准。

## 上下文与授权

- 父任务：[singleton New Session](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-18-singleton-new-session-navigation.md#L1)，重点第 1、2.5、6 节；用户授权 subagent 实现，主会话验收。
- 工作根：`/home/stream/ACodeSpace/push/BuildKodex`；产品源码在 `Kodex/`。开始加载根指导、buildkodex-change、kanban/planning/ask-user 及相关技能；读取 `kanban/Draft.md`、父任务及当前 RPC/Daemon checklist。
- 本任务是第一波 A，可与 B/C 并行；仅写下面范围和本文件。不得修改其他子任务、父任务、checklist、既存 `gradle.properties` 改动；不提交/推送、不派生 worker、不启动 Gradle/Native/IDE Sync 或控制设备。

## 唯一写范围

- `Kodex/rpc/client/**`、`Kodex/rpc/server/**` 的本任务源码、KDoc、直接依赖及测试。
- `Kodex/app/backend-daemon/src/specMain/kotlin/io/github/stream29/kodex/backend/BackendEndpoint.kt` 的业务协议标识/KDoc。
- `Kodex/app/backend-daemon`、`Kodex/app/backend-host` 的针对性协议兼容测试；不改其无关生产行为。
- frontend 调用者归 B/D/F：不得顺手修改；在报告列出遗漏。

## 实现接口与约束

- `GlobalRpc.createSession(cwd: Path): Int` 替换完整 Agent settings 参数，显式复用 `PathAsStringSerializer`；检查 kRPC 生成输入与依赖，不能假设参数的自定义 serializer 自动可见。
- 在后端实际创建边界一次读取其当前 `BackendSettings.newSession`，合并请求 cwd、构造初始 settings，再沿用现有校验、初始化、标题 `Session <index>` 和失败清理。必要的 typed supplier/依赖由真实后端 composition root 接入，不新增默认值缓存或 Get 轮询。
- 不提供旧签名转发、双协议兼容、create-and-submit 事务或恢复重试；不传首条消息。不为前端配置更新额外加 barrier。
- `createSuggestedSessions(tasks, initialSettings)` 和 fork 仍使用原显式/源设置，不能随全局默认值改写。
- `BackendBusinessProtocol` 从 `0.4.15` 更新为 `0.5.0`（应用当前已是 0.5.0），control protocol 保持 1；不改应用版本、历史 migration 或发布文件。

## 本任务检查

- 编写真实 JSON/kRPC 参数编解码测试，至少含 Unicode/空格 cwd。
- 编写后端默认值变更后创建的四字段/cwd、标题及既有 Session 隔离测试；模拟前端没有传任何模型字段，避免测试只比较同一份旧输入。
- 更新所有范围内旧 `createSession` 测试：需要特定模型配置时先更新后端默认值，再调用 cwd-only 接口；不能删断言或硬编码跳过校验。
- 保留创建失败清理及结果未知不重放的覆盖；确认建议子任务显式设置不变。
- 编写旧 business protocol 拒绝、新协议可用、management 协议不变的针对性测试，不停止/替换真实后端。
- 只运行静态检查；将拟跑 `:rpc-client:jvmTest`、`:rpc-server:jvmTest`、backend-daemon/host 测试交给主会话统一执行。不得把“测试已编写”记为“通过”。

## 交付报告（worker 填写）

- Session ID：953；[Session storage](file:///home/stream/.kodex/sessions/953)。
- 状态：A 范围实现与静态复核完成，等待父会话验收；没有运行编译或测试，没有移动 done。
- 工作树边界：未写父任务、兄弟任务、checklist、前端或其他生产文件；未写既存 `Kodex/gradle.properties`。B/C 的并发改动仍在共享工作树中，不能把全仓 diff 统计视为 A 的交付。
- IDEA 只读进程检查显示打开的是 PersonalSkills；未操作 IDE、GUI、远端、账号、模型或构建资源。没有创建临时文件、后台进程或 worker；未提交/推送。

### 实际文件与接口/依赖

- [rpc/client/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/client/build.gradle.kts#L42)：`specMain` 显式 `api(:utils-kotlinx-io-serialization)`，让公开协议类型注解及生成代码能取得既有 serializer 和 Path。
- [GlobalRpc.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/client/src/specMain/kotlin/io/github/stream29/kodex/rpc/contract/GlobalRpc.kt#L278)：唯一普通创建签名为 `createSession(cwd: @Serializable(with = PathAsStringSerializer::class) Path): Int`；KDoc 说明后端真源、无 barrier、默认标题及创建/提交失败边界。
- [BackendGlobalRpc.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalRpc.kt#L61)：直接委托 cwd，不读前端配置、不加兼容重载。
- [BackendSessionManagement.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionManagement.kt#L29)：构造增加必填 typed supplier `() -> KodexNewSessionSettings`。普通创建进入 host-owned operation 和现有 `commands` 锁后，只调用 supplier 一次，捕获不可变四字段，再结合请求 cwd 构造 `KodexAgentSettings`，沿用原 `create`、初始化与失败清理。
- [BackendServices.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L81)：真实后端 composition root 接入 `{ store.settings.value.newSession }`，不建默认值缓存、轮询或同步等待。
- [BackendEndpoint.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/backend-daemon/src/specMain/kotlin/io/github/stream29/kodex/backend/BackendEndpoint.kt#L5)：仅业务协议/KDoc 改为 `0.5.0`；management 保持 `1`，未改应用版本和历史迁移。
- `createSuggestedSessions(tasks, initialSettings)`、fork、内部 `create(initialSettings, name)` 及创建后的独立 append/resume 接口均未改形状或语义；未新增恢复、自动重试、双签名或 create-and-submit。

### 测试文件与用例（已编写，未执行）

- 新增 [SessionCreationValuesTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/client/src/commonTest/kotlin/io/github/stream29/kodex/rpc/contract/SessionCreationValuesTest.kt#L18)：
  - 检查实际生成的 `GlobalRpc` descriptor 仅有一个必填 cwd 参数，type-use annotation 指向 `PathAsStringSerializer`。
  - 用真实 JSON 编解码 Unicode、空格、引号及相对目录，断言 wire 值是字符串。
  - 拒绝 null 和旧完整 settings 对象，而不是给 Path 注册另一套 DTO/serializer。
- 修改 [BackendServicesTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendServicesTest.kt#L35)：
  - 新用例通过真实 in-memory JSON/kRPC、真实 `GlobalRpc` 和 `SettingsTimelineRpc` 创建并读取。
  - 故意保留旧前端 snapshot，由后端完成默认值 CAS 后不刷新前端、不等待订阅，直接 cwd-only 创建；断言四字段、cwd、标题、独立 turn/window、未提交/未运行及既有 Session 不变。
  - 所有原创建调用迁移 cwd；共享 fixture 在后端 defaults 中指定 `test-model`，不削减旧断言。
- 修改 [BackendSessionManagementTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionManagementTest.kt#L44)：
  - 默认值变化后下次创建使用新四字段；supplier 总调用数等于创建数，既有 Session 不变。
  - 初始化暂停期间改变默认值并取消 caller，已接受创建仍使用原来一次捕获的四字段，且保留原取消/结果未知覆盖。
  - fork 与建议批次传入“读取全局默认值即失败”的 supplier；建议批次继续使用显式模型配置。
  - 保留创建失败删除预留目标、后续提交失败不删除已建实体、目录/归档/历史 fork 等既有断言。
- 修改 [BackendStartupTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendStartupTest.kt#L24)：先在后端 defaults 指定模型，再 cwd-only 创建，保留 repository 懒加载和重开断言。
- 修改 [BackendTitleRpcTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendTitleRpcTest.kt#L43)：迁移三处 cwd-only 调用，自动标题资格、迟到输出与显式 rename/history 覆盖均保留。
- 新增 [BackendProtocolTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/backend-daemon/src/commonTest/kotlin/io/github/stream29/kodex/backend/BackendProtocolTest.kt#L7)：固定业务 `0.5.0`、拒绝旧 checkpoint 等同、management `1`，endpoint 默认协议不依赖 applicationVersion。
- 新增 [BackendProtocolCompatibilityTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/backend-host/src/commonTest/kotlin/io/github/stream29/kodex/backend/host/BackendProtocolCompatibilityTest.kt#L22)：
  - 使用现有临时 Home/离线 `withHost` fixture 模拟旧 `0.4.15` discovery record；bootstrap 和业务连接前置校验拒绝，但真实 management status 仍成功，同一 PID/instance/锁与宿主存活；finally 恢复 record。
  - 恢复新协议后 bootstrap/真实业务 RPC 可用；applicationVersion 不同也可访问，management 仍使用 `1`。
  - 这是协议前置拒绝与当前离线宿主的兼容测试，不声称启动过旧版本二进制或运行过真实账号。

### 静态检查与未跑门禁

- 已执行 scoped `git -C Kodex diff --check -- rpc/client rpc/server app/backend-daemon/src/specMain/kotlin/io/github/stream29/kodex/backend/BackendEndpoint.kt`，退出 0、无输出；三个新增测试文件另用 `grep -nE '[[:blank:]]+$'` 检查，无命中。
- 已全文复核 A 的 tracked diff 与三个新增测试文件；生产只改普通创建路径和指定协议常量，修改文件清单如上。
- 已用 `rg` 检查 `rpc/client` / `rpc/server`：旧普通创建 settings 签名、`createSession(KodexAgentSettings...)` 调用和旧 management 构造均无残留；全仓 management 构造仅在已迁移的 production composition root 与本测试中。
- 已对照本地 kRPC 0.10.3 source artifact 的 `RpcParameter` / `RpcType` / `RpcTypeKrpc`，以及本地 compiler-plugin 的 type-use `@Serializable` 接线；不只依赖默认的 Path serializer 推断。此为源码静态依据，不是生成/编译成功证据。
- 未执行任何 Gradle、Native、IDE Sync、单元测试或网络/进程运行测试。父会话需集中运行 `:rpc-client:jvmTest`、`:rpc-server:jvmTest`、`:app-backend-daemon:jvmTest`、`:app-backend-host:jvmTest`，并核实 Native 编译；测试名称/构建结果尚无通过声明。

### 范围外遗漏与父会话接手项

- 以下是复核时仍存在的旧签名消费者；B/C 正在并发迁移的文件不视为他们已交付失败，父会话需在各波交还后重新搜索：
  - B 范围：[UltrafastRpcIntegrationTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/UltrafastRpcIntegrationTest.kt#L70) 的 70、115、143 行。B 已并发迁移其他 app/rpc 调用，本轮 A 未写。
  - 波次 2：[ExactTargetCommandTest.kt:297](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/ExactTargetCommandTest.kt#L297)。
  - 波次 3/integration：[ApplicationRootBoundaryTest.kt:182](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/ApplicationRootBoundaryTest.kt#L182)、同文件 186 行。
  - 需父会话另行指派：[ComposerHostRepairTest.kt:463](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/ComposerHostRepairTest.kt#L463)，不在 A 写范围。
  - **backend-host 编译前置缺口**：[BackendHostTest.kt:143](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/backend-host/src/commonTest/kotlin/io/github/stream29/kodex/backend/host/BackendHostTest.kt#L143)、[同文件:196](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/backend-host/src/commonTest/kotlin/io/github/stream29/kodex/backend/host/BackendHostTest.kt#L196)、[BackendReconnectTest.kt:93](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/backend-host/src/jvmTest/kotlin/io/github/stream29/kodex/backend/host/BackendReconnectTest.kt#L93)。它们属于 turn/Hook/reconnect 行为而非本任务获准的针对性协议测试；A 不越界迁移。父会话需先设置后端默认模型再改 cwd-only，才能编译该模块测试。
- [backend-daemon checklist](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/backend-daemon.md#L150) 仍写 Home/business checkpoint 均为 `0.4.15`；请父会话仅将业务 checkpoint 更新为 `0.5.0`，保留 Home migration 和 management `1`。A 未写 checklist。
- 实现本身无需新增权限或改变冻结接口；上述测试消费者与规范同步是父级集成前置，不能为临时编译添加兼容重载或删断言。
- worker 交付时父会话尚未验收；最新父级状态见下方检查点，Task Tree 继续停在加粗 `Await parent acceptance` 等待运行门禁。

### 父级检查点

- 主会话已完成本波静态审查，允许下游 Application 集成；代码和测试尚未经构建执行，不是最终验收。
- 四处额外旧创建消费者与 backend-daemon checklist 已由主会话接手迁移，记录见父任务 §6.6。
- 主会话已实际执行 `:rpc-client:jvmTest :rpc-server:jvmTest`：151 + 142 = 293 项，0 失败、0 跳过；随后 `:app-backend-daemon:jvmTest` 17 项通过。日志/XML 见父任务 §6.7。包括新增 Path descriptor/编解码、真实后端默认值和协议常量用例；backend-host 兼容连接和 Native 门禁尚未执行，因此任务仍未最终完成。
