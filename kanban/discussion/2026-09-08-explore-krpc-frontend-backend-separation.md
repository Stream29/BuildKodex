# Task Tree

- 探索通过 kRPC 实现 Kodex 前后端分离
  - [done] 核对当前进程边界与 kRPC 官方能力
  - [done] 确认同机共享、断连续跑与多端操作方向
  - [done] 记录设置拆分、侧栏临时化与零新增 DTO 目标
  - 逐项确认现有 contract 到 RPC 的映射与状态归属
  - 明确设置文件边界与迁移行为
  - 明确后端启动、连接与资源释放语义
  - 确认平台约束与后续验证范围

# Details

- 状态：Discussion；用户授权持续讨论、调研和更新本任务文档，不进入 Planning/Executable，不改依赖、代码或既有架构约定，不制作原型或创建 Git commit。
- 用户背景：项目曾放弃前后端分离，目前每启动一个 Kodex 实例就同时启动前端与后端；希望重新探索仅经 RPC 通信、利用跨 RPC Flow 支持状态和流式更新的架构。
- 本轮未独立核实当年放弃方案的具体原因；不把现有模块分层等同于曾有可运行的独立 RPC 后端。
- 记录方式：一边讨论一边更新本文，使其逐步成为完善的迁移指南；区分已确认方向、待确认接口和未验证事实，不把文档完善或讨论结论当作实施授权。
- 写入范围仅为本任务；不修改 Draft、时间戳任务、其他并行工作或架构 checklist。未来落地既有约定的变更仍需明确授权。

## 已确认的迁移方向

### 共享后端与客户端

- 首先覆盖同机、同用户共享后端，避免每个 CLI 实例启动独立后端；跨机器不作为当前必要范围。
- 前端退出或连接中断不取消后端已接受的任务，任务继续运行；显式 Stop 与断开连接是不同操作。
- 多个 CLI 可以同时操作同一个 Session，不采用同一 Session 独占连接或单控制端限制。
- 以上不等于确认草稿同步、共享导航或协同编辑；具体状态归属在接口映射时讨论。

### 复用现有模型与 contract

- 用户希望采用经典前后端范式，通信方式改为 RPC；优先沿用已有分层、领域划分、命名、方法及数据模型，而不是重新设计业务协议。
- 以零新增 DTO 为设计目标；不要为已有值类型再建立 `RpcXxxDto`、转换层或平行数据模型。
- 优先讨论现有 contract 的远程调用版本，尽量复用方法名、参数和返回值类型；只为明确的跨进程限制讨论必要适配。
- 用户预期大部分现有设计可以直接复用；这是迁移目标，不是已经证明全部类型可直接序列化或 contract 可原样跨进程。
- 不把此前助手提出的 `HistoryRead`、`HistoryPage`、`HistoryUpdate`、`PendingInput` 等新类型作为方案基础；其必要性未被证明，用户已要求回到现有模型。
- 此前 `SessionRpc` / `AgentRpc` 接口示例未获确认；服务命名、划分、寻址及具体签名均不因示例出现而定案。

### 设置与侧栏

- 前端与后端各自持有一个设置文件；文件名、位置、字段划分与旧配置处理方式待讨论。
- 侧栏宽度从持久化设置中移除，成为纯前端临时值；应用启动时按四分之一宽度初始化，不持久化后续调整。
- 用户希望对外 breaking change 主要集中在上述配置变化，不因引入 RPC 主动改变其他业务语义。
- 当前实现仍在 `KodexGlobalSettings.sidebars` 中持久化左右宽度，默认各 28 列；本文记录的是迁移方向，不代表已修改实现或旧约定：
  `Kodex/app/shared/settings/contract/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/KodexGlobalSettings.kt:33-72`、
  [全局设置](../../checklist/global-settings.md)。

## 当前代码边界

- CLI 调用 `KodexApplication.openDefault()` 后直接把 ViewModel 交给 Mosaic，退出时调用 `application.shutdown()`：
  `Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt:30-52`。
- Application 持有 DI 图、Agent dependencies、auth、MCP、hook、scope 与 Home handle，并负责整体关闭；目前不是独立于终端的共享后端：
  `Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt:91-152`。
- 已有 contract/viewmodel/view 分层与值类型是主要复用基础；以下是需要核对的进程内表达，不构成另建 DTO 层的理由：
  - Session 直接持有 `rootAgent` 对象，fork 参数也包含 Agent ViewModel：
    `Kodex/app/contract/session/src/commonMain/kotlin/io/github/stream29/kodex/app/session/contract/PersistedSessionViewModel.kt:13-34`。
  - History 暴露带方法与 child 对象的窗口、嵌套 `SharedFlow`、`StateFlow` 和 `LazyListState`：
    `Kodex/app/contract/history/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/AgentHistoryViewModel.kt:20-41,73-106`。
  - Message loading state 还携带进程内 `Job`：
    `Kodex/app/contract/history/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/MessageHistoryItemViewModel.kt:20-31`。
- 现行约定只支持单一 Mosaic frontend，History 滚动/展开等状态由 ViewModel 持有；迁移时需明确对应本地状态和远程访问部分，不预先决定整套 ViewModel 必须搬到哪一侧。参见
  [Session/ViewModel 边界](../../checklist/cli-session-view-models.md)与
  [状态与懒 History](../../checklist/cli-view-model-state.md)。

## kRPC 官方资料核对（2026-09-08）

- GitHub 当前 Latest 为 `0.10.3`，`0.11.0-grpc-189` 标为 Pre-release；后者是另一条含 gRPC 的预览发布，不应因版本号更大就默认选用。库本身仍未稳定，Latest 不等于稳定性承诺。
  [Releases](https://github.com/Kotlin/kotlinx-rpc/releases)、
  [Versions](https://kotlin.github.io/kotlinx-rpc/versions.html)。
- kRPC 支持 RPC 参数与返回值中的 `Flow`，包括双向流；但明确不支持直接暴露 `StateFlow` / `SharedFlow`。服务端返回流必须是非 suspend 方法的顶层 `Flow`，不能照搬现有“状态对象内嵌流”的返回结构。
  [Features](https://kotlin.github.io/kotlinx-rpc/features.html)。
- 提供按调用的背压缓冲，`perCallBufferSize` 统计消息条数而非字节；不等于长历史、大工具结果或慢客户端的内存问题自动解决。
  [Configuration](https://kotlin.github.io/kotlinx-rpc/configuration.html)。
- 官方 Ktor 集成使用 WebSocket；也提供可自定义的 `KrpcTransport`。是否采用 Ktor/WebSocket 尚未决定。
  [Ktor](https://kotlin.github.io/kotlinx-rpc/krpc-ktor.html)、
  [Transport](https://kotlin.github.io/kotlinx-rpc/transport.html)。
- 官方版本表列出 Kotlin `2.4.0` 支持，与项目 Kotlin 版本一致；项目同时使用 Ktor `3.5.0`、coroutines/serialization `1.11.0`，完整依赖组合仍需验证：
  `Kodex/gradle/libs.versions.toml:1-9`。
  [Versions](https://kotlin.github.io/kotlinx-rpc/versions.html)。
- kRPC 平台表列有项目使用的 `linuxX64`、`linuxArm64`、`macosArm64`、`mingwX64`，平台覆盖具备基础；仍需核对所选发布的实际 artifact、客户端/服务器引擎及 WebSocket 能力，不能把模块支持等同于完整运行验证：
  `Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt:22-31`。
  [Platforms](https://kotlin.github.io/kotlinx-rpc/platforms.html)。

## RPC 交界面：待逐项确认

- 下一步以“现有 contract → 对应 RPC 签名”为讨论单位，记录复用类型、目标寻址及本地保留部分，不先设计新的服务数据模型。
- 下表是核对入口，不是已批准的签名或状态归属；尚未完成逐方法迁移清单。

| 现有入口 | 复用基础 | 尚需确认 |
| --- | --- | --- |
| Agent `settings`、`execution`、`tokenCount` | `KodexAgentSettings`、`AgentExecutionState`、`Long?` | `StateFlow` 属性如何映射为返回普通 `Flow` 的方法；首次值与订阅生命周期 |
| Agent `submit`、`updateModel`、`cancel` 等命令 | 原方法名、`ContentItem`、`OpenAiModelId` 等参数类型 | 接收者如何跨进程定位；哪些方法需改为 suspend；返回时的完成语义 |
| Session `rootAgent`、`fork` | 现有 Session identity、fork 语义与返回 index | child 访问与对象参数的远程表示；不预设通用远程对象注册表 |
| History 窗口、条目与输出流 | 既有 history 数据、`ResponsesStreamEvent`、storage index 与 generation | 有界读取、失效与流订阅如何复用现有接口；不引入平行历史模型 |
| Composer、用户输入与建议 Session 确认 | 既有 draft、call id、revision、结果类型 | 本地编辑与后端提交的分界；多端操作如何复用现有校验 |
| Application、settings、auth、MCP 与宿主能力 | 既有领域 contract 和模型 | 哪些操作留在前端，哪些经 RPC；前端不得通过直读后端文件绕过 RPC |

- Agent 方法与状态入口：
  `Kodex/app/contract/agent/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/AgentViewModel.kt:32-122`。
- 候选适配：为已有值类型补齐必要的序列化支持，而非复制 DTO；为属性提供 Flow 方法，而非直接传输 `StateFlow` / `SharedFlow`；这些仍需逐项核实。
- 嵌套输出流优先研究复用 `Flow<ResponsesStreamEvent>` 的暴露方式；流身份、切换和订阅衔接尚未确定，不把拆出一个方法视为已解决全部语义。
- `Job`、renderer 布局与滚动实例不作为跨线数据；对应状态与操作如何保留原 contract，继续结合具体接口讨论。
- 如发现零新增 DTO 目标存在具体障碍，先记录涉及的原类型、限制及最小替代方案并与用户讨论，不直接新增类型。

FIXME:

## 其余待定边界

- 设置：前后端文件位置与字段归属、旧配置是否迁移或忽略、侧栏默认宽度的具体初始化基准。
- 后端：按需或显式启动、发现与避免重复启动、是否按 Home 隔离、空闲退出条件。
- 生命周期：连接/订阅取消的资源释放、已接受任务的后端所有者、关闭 tab 与关闭 Session 的区别、显式后端 shutdown。
- 多客户端：复用后端写入串行化与现有校验；草稿归属、同时提交或回答的结果语义待确认，不预先引入控制权、协同编辑或通用仲裁系统。
- Flow：首次值与后续更新衔接、流切换、重连恢复、命令回执丢失及慢客户端影响；不预设持久事件日志、任意位置重放或全局事件总线。
- 宿主：终端通知、路径选择、文件/图片访问、登录回调分别由哪侧执行；保持后端凭据不向前端泄露。
- 兼容性：前后端版本匹配方式、所选发布与现有依赖组合、四个 Native 目标的实际传输与运行能力。

## 后续验证候选范围与当前证据

- 当前判断：具备 RPC 命令与 Flow 订阅的技术基础；现有模型是迁移起点，零新增 DTO 与项目级可行性尚未通过实现验证。
- 候选验证：现有类型序列化与接口生成、目标 Native artifact/引擎及连接、单 Session 打开/提交/Stop、流式输出、History 按需读取与失效。
- 再按已确认需求验证同一 Session 的多端操作、断连后任务继续、重连恢复、慢客户端和资源释放。
- 上述只是待讨论的验证范围；未授权添加依赖、制作原型、编译或运行跨进程测试，也未进行性能测量。
- 迁移指南随讨论逐项补全；未决项保留为问题，只有用户明确授权后才推进阶段或落实代码及架构约定。
