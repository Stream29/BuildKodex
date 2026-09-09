# Task Tree

- [done] 取消前后端 RPC 化并回滚归档
  - [done] 按用户决定停止设计与实施
  - [done] 撤回 RPC 模块、依赖与原模型序列化改动
  - [done] 清理 RPC 模块生成文件
  - [done] 保留历史讨论并移入取消任务归档
  - [done] 检查回滚后的工作区与相关项目验证

# Details

- 状态：`done (cancelled)`；2026-09-09 用户明确决定暂时废止前后端 RPC 化，要求代码回滚及任务归档。这里的 done 表示取消收尾，不表示迁移完成。
- 已撤回本任务创建的 `Kodex/rpc/contract`、`Kodex/rpc/model` 及其模块生成文件，恢复 settings.gradle.kts、libs.versions.toml、agent-context-contract 构建配置和四个原类型序列化注解的原状。
- 回滚后 `git -C Kodex status --short` 为空；没有回退提交、暂存文件或创建 Git commit。并行发布任务和 shared-context 文件未修改。
- 原迁移计划、设置文件拆分、侧栏宽度临时化及 frontend/backend hooks 讨论全部停止；不改变现行 CLI、设置或 Hooks 行为，不修改既有架构 checklist。最后的 frontend hooks 行为问题未获回答，也不再推进。
- 回滚验证：复用原 Temurin 25 Daemon，`:agent-context-contract:compileKotlinJvm` 与设置 contract/filesystem 的 `jvmTest` 成功（27 秒）。两组测试分别为 3 项、16 项，全部通过且无跳过；未运行完整 CLI 或 Native 验证，因为本次撤回独立契约草案，业务源码已恢复到现有 HEAD。
- 归档检查：Kodex 子模块工作区干净、rpc 目录不存在、活动看板无同名任务，未发现需更新的旧任务路径引用；文档结构和空白检查通过。先前 RPC 编译与测试结果仅属于已回滚草案。
- 以下内容仅保留为取消前的历史调研、决策演变与验证记录，不是现行架构指导或继续工作的授权；其中 RPC 源码路径指向已删除的草案。若以后重启，须用户重新授权并重新核对代码与库能力。

## 历史讨论与实现记录（已取消）

- 历史阶段：用户曾明确授权从 Discussion 移入 Planning，并创建可编译的 `Kodex/rpc/contract` 模块，添加必要构建配置与 kRPC 依赖，用契约源码反复审查迁移形状。
- 本阶段仅实施已授权的契约设计与编译检查；不进入 Executable，不实现客户端代理、后端服务、缓存或连接，不改现有业务行为、设置文件或架构 checklist，不创建 Git commit。
- 用户背景：项目曾放弃前后端分离，目前每启动一个 Kodex 实例就同时启动前端与后端；希望重新探索仅经 RPC 通信、利用跨 RPC Flow 支持状态和流式更新的架构。
- 本轮未独立核实当年放弃方案的具体原因；不把现有模块分层等同于曾有可运行的独立 RPC 后端。
- 记录方式：一边讨论一边更新本文，使其逐步成为完善的迁移指南；区分已确认方向、待确认接口和未验证事实，不把文档完善或讨论结论当作实施授权。
- 写入范围为本任务、`Kodex/rpc/contract` 及其必要构建入口和依赖声明；用户追加批准 `Kodex/rpc/model` 与本次 GlobalSettings 所需四个原类型的序列化注解及必要构建改动。不修改 Draft、时间戳任务、其他并行工作或架构 checklist。未来完整迁移仍需明确授权。

## Planning 工作方式

- RPC 契约源码是可编译的设计稿，保持独立，现有 CLI/Application 不依赖它；编译通过不等于接口已最终批准。
- 先沿用已具备序列化支持的 Agent settings 模型审查流订阅与全量 CAS，不为覆盖全部接口而引入 DTO 或修改既有模型。
- 首批接口不等同于完整迁移清单；HistoryIndex、execution 等现有值类型的序列化支持和缓存 owner 仍需逐项审查。
- 用户已要求继续构建其余 RPC；Get/流重试规则已修订，IndexTimelineRpc 已获批，现另行授权同批补齐 work/settings/timestamp/tokenCount/unstable 五条同构 timeline 契约，不修改原模型的序列化声明。
- 除本次五条同构 timeline 批次外，其余 RPC 及必要原模型修改仍逐项审批。首个 HistoryIndexRpc 的整窗方案未获批；不得据此添加原类型序列化注解或启用原模块插件。
- 继续在本文记录原接口映射、未决项、编译结果及审查反馈；后续通过源码迭代收敛，再决定实现计划。
- 用户另行授权：本机无运行中的 Gradle Daemon 时，可使用上一轮的 `/home/stream/.gradle/jdks/eclipse_adoptium-25-amd64-linux.2` 启动 Daemon，后续编译复用同一 JVM；不改变全局 Java 配置或切换设备。

### 首批可编译契约与审查结果

- 模块：`Kodex/rpc/contract/build.gradle.kts:1-18`；Gradle project 为 `:rpc-contract`，复用 `kodex.kmp-cli` 的 JVM 与四个 Native 目标，不引入新的 convention。
- 注册入口：`Kodex/settings.gradle.kts:53`；版本目录新增 kotlinx.rpc `0.10.3`、core artifact 和 compiler plugin alias，不升级既有 Kotlin/Ktor/coroutines/serialization。
- 仅添加 RPC core 与编译所需依赖，不选择 Ktor/WebSocket、序列化格式或 gRPC 预览版。版本及配置依据：
  [官方版本表](https://kotlin.github.io/kotlinx-rpc/versions.html)、
  [0.10.3 官方 common 模块示例](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/samples/ktor-web-app/common/build.gradle.kts)。
- 源码：`Kodex/rpc/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/AgentSettingsRpc.kt:7-43`；Get、GetFlow 与全量 CAS 方向已确认，服务分组和寻址仍是待审查设计稿。

| 原 AgentSettingsViewModel 入口 | 当前 AgentSettingsRpc 草案 |
| --- | --- |
| `settings: StateFlow<KodexAgentSettings>` | `getSettings(sessionIndex)` 读取当前快照；`getSettingsFlow(sessionIndex)` 包含订阅时当前值及后续更新 |
| 各按字段更新方法、`updateModelConfiguration` | 收敛为 `compareAndSet(sessionIndex, expect: KodexAgentSettings, update: KodexAgentSettings): Boolean` |
| `models: StateFlow<List<ModelInfo>>` | 模型目录不属于 settings 值；从本接口移出，后续单独审查其订阅归属，不取消该读取需求 |

- 原类型全部复用，没有新增手写 DTO、mapper 或 serializer；`cwd` 使用 `KodexAgentSettings` 原有的 `PathAsStringSerializer` 声明，不再单独传递 `Path` 参数。
- 用户明确：settings 体积不大，采用完整值读写，并以 CAS 替代无条件覆盖；最新要求恢复独立 Get，用于显式取得 initialValue，其他状态读取同样提供 Get/GetFlow。不保留按字段 RPC，不增加版本 DTO 或合并协议。
- 前端提交本地原快照 `expect` 与编辑后的完整 `update`；后端在同一写入临界区比较当前值并替换，匹配返回 `true`，不匹配返回 `false` 且不写入。其他写入可以发生在前端读取与提交之间，但不能插入后端的比较与替换之间。
- 沿用 `MutableStateFlow.compareAndSet` 的 `equals` 值比较，不比较跨进程对象身份；`expect`、`update` 均等于当前值时返回 `true`，不写入或产生新状态。它不检测 A→B→A 的中间变化，不因此追加 generation/revision。
  [官方 compareAndSet 语义](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-state-flow/compare-and-set.html)。
- 这提供“后端原子 CAS + 前端异步状态投影”，不是可直接实现原生 `MutableStateFlow` 的同步 `.value` setter / `compareAndSet`：RPC 方法需挂起，订阅仍可滞后，成功回执不保证本地 `.value` 已更新。
- 前端未来可用本地纯函数重算 `update` 并重试 CAS，函数无需跨 RPC 传输；重试函数可能执行多次，不应带不可重复副作用。CAS 比较失败后依赖订阅流推进，不调用 Get 刷新；具体等待、取消仍未实现，不能对同一陈旧快照忙循环，也不承诺无冲突或不饥饿。
  [官方 update 语义](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/update.html)。
- 比较失败与调用失败分开：`false` 仅表示值不匹配；校验或传输失败不能伪装成 `false`。回执丢失时是否已写入可能未知，不据此自动当作比较失败重试。
- 现有 `KodexAgentStateImpl.updateSettings` 在 `writeMutex` 内全量写入，但尚无带 `expect` 的入口（`Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt:398-406`）。后续实现必须把比较也放入同一写入边界；在 RPC 层先读后调用原 `updateSettings`，或仅 CAS 一个镜像 StateFlow，均不能证明实际存储更新原子。本轮不改 AgentState 或其 checklist。
- 现有 `KodexAgentSettings` 还包含 `turnId`、`windowNumber`、`plan` 等运行信息（`Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/CompactionModels.kt:44-83`）；实现阶段需核对全量更新的合法性校验，不能把本次接口编译当作任意旧快照均可无条件写入的证明。本轮不拆 DTO，也不实现写入策略。
- 当前候选寻址为 persisted root Agent 的 `sessionIndex`；虚拟 NewSession 与应用 defaults 不在该接口中。服务分组、命名和寻址继续在源码上审查，不由首次编译冻结。
- HistoryIndexWindow/Entry/Detail、AgentExecutionState 等原模型尚未补序列化声明，本轮不修改它们、不另造 DTO；其接口与最小序列化支持是后续审查项。
- 依赖方向保持隔离：现有 CLI、Application、ViewModel 不引用新模块；未手写或接入客户端代理、服务实现、连接和缓存。

FIXME:

### 契约编译验证

- 使用已授权的 Temurin 25 启动 Gradle Daemon，第二轮复用同一 PID/JVM；命令显式设置 `JAVA_HOME` 与 `-Dorg.gradle.java.home` 为上述 Java 25 路径。
- 第一轮 `:rpc-contract:compileKotlinJvm :rpc-contract:compileKotlinLinuxX64` 成功；第二轮以下五项全部成功：
  `compileKotlinJvm`、`compileKotlinLinuxX64`、`compileKotlinLinuxArm64`、`compileKotlinMacosArm64`、`compileKotlinMingwX64`，均使用 `:rpc-contract:` 前缀。
- 修订为 `getSettingsFlow` + `compareAndSet` 后，复用同一 Temurin 25 Daemon 再次运行上述五项，全部编译成功（3 秒）；未运行中间的无条件全量更新草案。CAS 尚无实现，未做原子性、相等性往返或并发运行测试。
- 恢复仅初始化使用的 `getSettings`、限定 CAS 重试依赖订阅后，再次复用同一 Daemon 运行上述五项，全部编译成功（1 秒）；没有初始化或订阅等待的运行验证。
- 验证覆盖 Kotlin/RPC 编译插件、当前依赖组合与该契约的 JVM class / 四个 Native KLIB 产物；不是四平台 executable 链接或运行验证。
- 构建仍报告既有 composite build 的 deprecated Gradle property 与其他模块 macOS cinterop 警告；不在本任务修改那些模块。
- 没有运行单元测试：本批仅接口声明、无业务实现；没有进行序列化往返、RPC 连接、Flow 重连、缓存或性能运行验证，均不属于本次契约编译范围。
- IDEA：首次建模块时 CLI 导航因显示授权失败，已结束那次尝试启动的进程；本次修订发现项目 IDEA 已运行，复用其 CLI 成功导航至契约文件。MCP 工具仍未暴露，未改 IDE/显示配置。

## 已确认的迁移方向

### 共享后端与客户端

- 首先覆盖同机、同用户共享后端，避免每个 CLI 实例启动独立后端；跨机器不作为当前必要范围。
- 前端退出或连接中断不取消后端已接受的任务，任务继续运行；显式 Stop 与断开连接是不同操作。
- 多个 CLI 可以同时操作同一个 Session，不采用同一 Session 独占连接或单控制端限制。
- 系统始终只有一个用户，多前端只是该用户的不同访问入口；不按多用户协作系统设计并发冲突、控制权或状态同步机制。
- 用户明确没有强同步性要求；接受 RPC 状态传播的短暂延迟，不要求命令返回时各前端的本地 `.value` 已同步更新。
- 保留现有业务命令准入、写入串行化及失效校验；单用户不等于异步操作绝不交错，但不因此增加跨端强一致性协议。
- 多端并发的正确性标准是后端数据与状态始终合法，不是最终效果必须符合用户分别在各前端操作时的原始预期；不增加跨前端意图协调要求。
- 用户示例：两个前端同时提交 user message，两条请求都到达后端后，实际效果可能偏离用户预期；只要后端按既有业务规则处理并保持合法状态，该情形可以接受。请求到达不等于两条消息都必须无条件接受，也不规定新的排队、合并或拒绝策略。
- 用户明确现有后端数据结构已建立合法性保证；迁移以复用这份保证为前提，不重新设计并发正确性机制。RPC 命令须进入同一后端的既有业务操作与校验边界，不绕过它们直接改写状态或存储。
- 以上不等于确认草稿同步、共享导航或协同编辑；具体状态归属在接口映射时讨论。

### 复用现有模型与 contract

- 用户希望采用经典前后端范式，通信方式改为 RPC；优先沿用已有分层、领域划分、命名、方法及数据模型，而不是重新设计业务协议。
- 优先复用原值类型，不为已有可传输模型建立 `RpcXxxDto`、转换层或平行数据模型。用户已明确批准例外：将不含前端字段和 MCP 凭据的全局 settings 快照定义在 `rpc/model`，嵌套值仍复用原类型；不将该单项授权扩展到其他模型复制。
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
    `Kodex/app/contract/session/src/commonMain/kotlin/io/github/stream29/kodex/app/session/contract/PersistedSessionViewModel.kt:13-49`。
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

## 经典三层范式的行为参考（待确认）

- 本节回应用户对理想行为的提问，是结合当前需求提出的行为基线，不是已确认的状态归属或实施方案；三层分离本身不强制具体草稿、冲突或恢复策略。
- 前端拥有“我正在怎么看、怎么编辑”：View、展示 ViewModel、导航、弹窗、输入草稿、滚动、展开、连接与加载状态；业务数据副本只是后端状态的只读投影，不成为第二个可独立修改的业务真源。
- 后端拥有“Session 实际是什么、正在做什么”：共享 Session/runtime、命令校验、任务与工具执行、业务状态及持久化编排；同一 Session 不因多一个观察者而多启动一份 Agent 执行。
- 存储拥有已提交的持久化数据与一致性机制，不负责运行 Agent、驱动 UI 或持有活跃 Job；业务数据读写经过后端，前端只直接读写自己的前端设置。
- 这里的“数据库层”可由现有 repository/storage 和文件格式承担，不意味着新增数据库服务、替换存储格式或再增加一条后端到数据库的 RPC。

| 操作或场景 | 候选理想行为 |
| --- | --- |
| 启动第二个 CLI | 新建本地交互状态并连接共享后端，不复制后端运行环境 |
| A、B 打开同一 Session | 各自按需读取和订阅同一业务状态；打开页面不是创建另一份业务 Session |
| A 打字、切 tab、滚动或展开 | 仅改变 A 的本地交互状态，不向 B 同步；如需共享草稿，另作为明确需求讨论 |
| A 提交内容 | 后端校验、接受并持有工作；A、B 均能观察实际业务更新，B 不因更新丢失自己的草稿或视口 |
| A 修改 Session 设置 | 提交完整 expect/update，后端原子比较并替换；不匹配返回 false，订阅者收到实际生效值，不同步 A 的设置弹窗状态 |
| A 关闭 tab 或退出 | 释放 A 的本地状态与订阅，不等同于 Stop、删除 Session 或关闭共享后端 |
| 任一客户端显式 Stop | 请求停止同一后端任务；各客户端观察同一实际结果 |
| 两端同时提交或回答工具请求 | 按既有命令准入、call id 与状态校验处理；只要求后端状态合法，允许结果偏离用户原始预期，不额外协调跨端意图 |
| 断线后重新连接 | 重新取得后端当前事实及所需历史；不从客户端旧状态恢复后端，不盲目重试结果未知的写命令 |
| 后端进程重启 | 持久化数据可按既有恢复能力读取；活跃 Job、连接和内存流不能因三层分离自动恢复 |

- 对 RPC 的含义：远程操作针对业务对象和能力，而非某个终端的当前选中项；客户端可维护本地 child ViewModel，不必把整个 UI 对象树同步到后端。
- 两侧可以共享同一份 Kotlin 值类型定义；共享类型不等于共享对象实例或可变状态，也不要求另建 DTO 层。
- 长任务的提交确认与任务完成分别表达；订阅只负责观察，不因新增 collect 而重新执行提交。具体方法何时返回仍需逐项确认。
- 复用依据：现有写入准入见 `checklist/agent-state-mutation-serialization.md:3-7`；已接受工作不随 renderer 调用取消的约定见 `checklist/cli-session-view-models.md:112-115`。这些是进程内基础，不代表跨进程实现已完成。

FIXME:

## RPC 交界面：待逐项确认

- 下一步以“现有 contract → 对应 RPC 签名”为讨论单位，记录复用类型、目标寻址及本地保留部分，不先设计新的服务数据模型。
- 下表是核对入口，不是已批准的签名或状态归属；尚未完成逐方法迁移清单。

| 现有入口 | 复用基础 | 尚需确认 |
| --- | --- | --- |
| Agent `settings`、`execution`、`tokenCount` | `KodexAgentSettings`、`AgentExecutionState`、`Long?` | `StateFlow` 属性如何映射为返回普通 `Flow` 的方法；首次值与订阅生命周期 |
| Agent `submit`、settings 更新、`cancel` 等命令 | `ContentItem`、`KodexAgentSettings` 等原参数类型；settings 已确认全量 CAS | 接收者如何跨进程定位；哪些方法需改为 suspend；返回时的完成语义 |
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

### Get / GetFlow 与本地 StateFlow

- 用户最新确认：每个跨 RPC 的状态读取提供独立 Get 与 GetFlow，Get 用于取得构造 StateFlow 的 initialValue；覆盖此前省略单次 Get 的草案。只读状态不因此获得 CAS，可编辑状态才另加对应 CAS。
- Get 仅限初始化；初始化后的状态读取和 CAS 重试都依赖订阅流，不将 Get 作为刷新、轮询或比较失败后的读取入口。
- 当前 settings 形状为 `suspend fun getSettings(...): KodexAgentSettings`、`fun getSettingsFlow(...): Flow<KodexAgentSettings>` 与 `suspend fun compareAndSet(..., expect: KodexAgentSettings, update: KodexAgentSettings): Boolean`；原值类型不变，前端仍可暴露原来的 `StateFlow<KodexAgentSettings>`。
- StateFlow 新订阅会收到当前值，后续按相等性合并更新，允许跳过中间状态；它不是逐条不丢的事件日志。
  [StateFlow 官方语义](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/)。
- GetFlow 应订阅后端同一状态源，并包含订阅时当前值，而不是只发送订阅后的变化。例如 Get 得到 A、订阅前状态变为 B 后不再变化，GetFlow 仍须提供 B；不能用 `drop(1)` 把这个衔接保障去掉。
- 初始化基线：先挂起调用 Get 取得真实快照，以其构造本地 StateFlow，再从 GetFlow 持续更新。不伪造默认 settings；Get 失败则初始化失败，不暴露未初始化 handle。GetFlow 首值覆盖两次调用之间的状态变化。
- 技术上，`getSettingsFlow(...).stateIn(scope)` 也能挂起等待流首值并继续同一订阅，并非没有 Get 就绝对无法初始化；但这不替代用户要求的独立快照读取接口。
  [stateIn 官方 API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/state-in.html)。
- Get 与 GetFlow 不是一次原子快照加订阅；先 Get 再订阅且保留流首值即可满足当前弱同步要求，不需要版本 DTO。不得并行初始化后用迟到的 Get 无条件覆盖已收到的流更新。
- 可复用范围：前端 `.value`、`collect`、`combine` 和 UI 观察接口可以继续消费本地 StateFlow；后端保留原有状态源及业务写入，适配集中在远程访问与对象初始化边界。多个本地观察者共享同一状态订阅。
- 已确认的同步要求：前端 `.value` 表示最近收到的值即可，允许命令完成与订阅更新之间存在短暂延迟；不为命令返回后的立即可见性新增确认屏障、版本 DTO 或跨端同步协议。
- 正常连接下状态仍应随订阅更新，不能因初始化衔接错误永久停在旧值；这是基本状态传递正确性，不是新增强同步要求。
- 远程初始化需等待，连接故障不能由普通 StateFlow 值自动表达；`stateIn` 不会自动处理重连，具体策略待讨论。直接写 `MutableStateFlow` 的业务路径仍应复用对应命令，而非由只读 Get/GetFlow 自动远程化。
- 适用前提是原值类型可跨线传输且保留所需值相等语义；包含 ViewModel 引用、`Job`、布局对象或嵌套 `SharedFlow` 的类型仍需逐项处理，不因此引入新 DTO，也不将本方案等同于全部 contract 可原样传输。
- kRPC 签名仍使用普通顶层 `Flow`、非 suspend 流方法；两侧内部使用 StateFlow 不受该签名约束影响。
  [kRPC Features](https://kotlin.github.io/kotlinx-rpc/features.html)。
- 当前结论是只读状态消费侧有望以少量适配复用；首批 RPC 声明已通过编译，但尚未实现或运行本地 StateFlow 适配，不能据此声称整体迁移已验证。

FIXME:

### 客户端 SuspendMutableStateFlow 接口

- 用户授权落下 `SuspendMutableStateFlow` 接口，将状态订阅与 CAS 包装为单一业务抽象；明确不提供 `set`，避免业务误用。`update`、`getAndUpdate`、`updateAndGet` 扩展方向已认可，重试实现仍留待后续。
- 源码位于 `Kodex/rpc/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/SuspendMutableStateFlow.kt:6-29`；只新增接口声明，不新增 DTO、依赖、客户端实现或模块。它是客户端本地 handle，不标记 `@Rpc`，不作为 RPC 返回值。RPC 继续传递普通 Flow 与原有 settings 值，Session 寻址由后续 handle 实现绑定。

```kotlin
interface SuspendMutableStateFlow<T> : StateFlow<T> {
    suspend fun compareAndSet(expect: T, update: T): Boolean
}
```

- `value`、`collect`、`replayCache` 委托本地 StateFlow；挂起工厂通过 Get 取得 initialValue 并建立 GetFlow 订阅，不伪造初始 settings。多个观察者共享这份投影，订阅由已有客户端 owner scope 管理；Get 保持为底层 RPC 能力，本轮不扩展该本地接口。
- 后续提供挂起扩展 `update(transform)`、`getAndUpdate(transform)`、`updateAndGet(transform)`，不提供 `set`；通过 CAS 重试，重新按所取快照计算，变换函数留在客户端，不跨线传递。返回值来自成功那次 CAS 的 expect/update，而非随后读取可能滞后的 `.value`。本轮不新增扩展实现或占位函数。
- 可恢复的是“缓存式 StateFlow 读取 + 远程原子写入”的业务接口，不是原生 MutableStateFlow 的全部同步保证：不提供同步 `.value = ...`、同步 `tryEmit` 或全局订阅人数；`value` 仍是最近收到的值，写入成功不保证本地投影已更新。无需继承 MutableSharedFlow 去模拟这些无当前需求的能力。
- CAS 返回 `false` 后等待订阅流更新，再按收到的快照重算并重试；不得调用 Get 或以重新订阅获取快照的方式替代等待，也不得对同一旧缓存忙循环。若订阅在请求期间已推进，重试可使用该更新，不应从失败回执时刻重新等待下一次更新。
- 后续验证需覆盖失败回执前后更新到达、等待可取消，以及 A→B→A 被相等性合并时的等待活性；具体实现仍待审查，不把接口编译当作重试必然继续的证明，本轮不新增版本 DTO 或重试机制。
- CAS 成功回执不应直接无条件写回本地投影，以免覆盖已到达的后续状态；投影继续跟随权威订阅。传输异常/取消不等于比较失败，不自动重放结果未知的业务变换。
- `StateFlow` 官方不保证第三方继承的未来稳定性；接口继承其只读能力，后续实现优先委托库内 StateFlow 并随依赖升级编译验证，不手写状态流内部机制。不新增通用 RPC 服务或远程对象注册表。
- 核对依据：[MutableStateFlow API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-state-flow/)、[StateFlow API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/)、[updateAndGet API](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/update-and-get.html)。
- 新接口首次编译因继承 StateFlow 需要 opt-in 而被项目 `-Werror` 拒绝；核对本地 coroutines 1.11.0 源码后，仅在本接口添加 `@OptIn(ExperimentalForInheritanceCoroutinesApi::class)`，不降低告警级别或增加全局 opt-in。
- 复用现有 Temurin 25 Daemon，重新运行上述五目标编译全部成功；JVM 字节码确认只继承 StateFlow 并新增挂起 CAS，没有 setter/emit。仅接口声明，未运行单元测试、客户端重试或 RPC 运行验证。

FIXME:

### History timeline 边界审查

- 用户质疑整窗 HistoryIndexRpc：`HistoryIndexWindow` 持有完整 indexes，不宜作为每次更新的跨线状态；提出单独订阅 generation/latestIndex，按需读取 timeline，在前端重建 CachedIndexVersioned。原 getWindow/getWindowFlow 及窗口序列化方案撤回；后续获批的替代契约见 IndexTimelineRpc 一节。
- 已核实：`HistoryIndexViewModelImpl.sync` 追加时执行 `current.indexes + appended`，回退时重读索引并发布新代窗口（`Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexViewModel.kt:115-149`）。直接传完整窗口会反复传输增长的索引列表，不等同于增量通知。
- 修订方向：generation/latestIndex 分别提供仅初始化用 Get 和持续 GetFlow；它们只是轻量失效/追加通知，不携带完整索引列表。timeline 的 `indexesIn`、`getExact`、`valuesIn` 是按需数据读取，不受“状态 Get 仅初始化”限制，不用于刷新 CAS 状态。
- 复用依据：`IndexVersioned<T>` 已有稀疏查询、floor/ceil、inclusive range 与原值读取（`Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/IndexVersioning.kt:21-42`）；`KodexAgentStorage` 有 index/work/settings/timestamp/tokenCount/unstable 六条独立 timeline，全局 latestIndex 是六条尾索引的最大值（同目录 `KodexAgentStorage.kt:17-53`）。
- 最初候选围绕 `index: IndexVersioned<CleanIndexEntry>`：返回已有 `List<Int>`、`CleanIndexEntry?` 或 `List<Pair<Int, CleanIndexEntry>>`，保留范围与 exact 读取语义，增加 Session/generation 参数；无需先序列化 HistoryIndexWindow/Entry/Detail。`CleanIndexEntry` 已声明 Serializable（`Kodex/agent-storage/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/CleanIndexEntry.kt:11-12`）；用户最新提出将相同边界应用于后端各个 CachedIndexVersioned，见下节。
- 前端重建本地只读 timeline 缓存，再生成 HistoryIndexWindow 与展示摘要/详情；同代追加可查询未取得的索引区间，payload 按需加载。generation 变化废弃旧代索引和值缓存，旧代请求不得填入新缓存。完整索引仍可能随历史增长常驻；减少重复跨线传输不等于已经实现有界索引内存。
- 现有 `CachedIndexVersioned` 是 filesystem 模块的 internal 类，构造依赖 `FileSystemIndexVersioned<T>`，实现 `MutableIndexVersioned<T>`；其索引缓存通过自身 set/revert 更新，读取调用 delegate.getUnsafe，并无外部 generation/尾索引订阅入口（`Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt:112-139,148-177,195-286`）。不能原样实例化；后续需逐项审批只读缓存复用/调整，不把此建议当作现有类已支持 RPC。
- 服务端代际必须覆盖对应 timeline 的破坏性替换，且读取在对应写入边界下校验代际；前端仍捕获 owner/generation 防迟到污染。现在的 HistoryIndex generation 是 ViewModel 私有计数，不能直接宣称 storage 层已有可订阅代际。不同流暂时不同步可以接受，但不能以旧代读取结果推进新代缓存。
- 最新候选以每条后端 CachedIndexVersioned 为 generation/latestIndex owner；具体接口、原类调整及初次索引加载范围仍待逐项审批。原始数据的前端展示仍须保留现有隐藏 secret answer 等规则，不因移动投影丢掉展示约束。
- 本轮只核查源码与修订文档；未创建 HistoryIndexRpc、缓存实现或新模型，未补原模型注解，未运行测试或 RPC 原型。

FIXME:

### 每条 CachedIndexVersioned 作为 RPC 读取边界

- 用户提出：后端为各个 CachedIndexVersioned 增加 generation 并暴露 RPC，前端重建本地缓存。评估结论：适合作为持久化数据的读取边界；远程暴露的是只读 IndexVersioned 语义与轻量通知，不是传输缓存对象或开放 MutableIndexVersioned 写入。
- 已有 Session 打开路径为六条 timeline 分别创建缓存，再交给同一 AgentState（`Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt:26-41`、同目录 `FileSystemKodexAgentSession.kt:45-55`）。候选映射保持 `index/work/settings/timestamp/tokenCount/unstable` 及原值类型，每条缓存有自己的尾索引和代际；无需让只更新 settings 的操作通知 index 缓存。
- 每条 timeline 的候选远程能力：generation/latestIndex 的初始化 Get 与持续 Flow；带 expectedGeneration 的原 IndexVersioned 全部数据读取，包括 get/getExact、floorToIndex/ceilToIndex、indexesIn/valuesIn。前端不必为查询持有完整索引；已知区间可本地命中，未知区间允许直接 RPC。公共抽象现采用普通泛型父接口与具体 RPC 显式 override，不使用 generic @Rpc；服务分组与寻址仍可继续审查。
- generation 表示该 timeline 的内容失效代际，不表示 Cache4k 的命中/淘汰：正常追加只推进 latestIndex；成功删除现存 suffix 或替换旧内容才提升 generation；TTL、容量淘汰、无实际删除的 revert 不构成内容变化。当前类尚无 generation 或可订阅 latestIndex，需后续实施，不是已有能力。
- 当前 `set` 强制 index 大于当前尾索引，`revert` 删除 suffix，因此“同代已存在的 exact 值不变”有实现基础（`CachedAgentStorage.kt:240-278`）。缓存应继续按实际存储 index 保存值；`get(index)` 的 floor 结果、空查询及尚未追加的区间不能当作永久事实，同代追加仍须补齐相关索引。
- 代际变更、timeline 读取和缓存填充需有一致的本地临界区/代际校验，覆盖读取期间的 revert；前端收到新代后同样拒绝旧代迟到结果。不能仅在 RPC 返回前比较一个 generation 数值，却让底层旧 loader 填回新代缓存。现有 get/getExact 只检查 owner 存活，valuesIn 无代际检查（`CachedAgentStorage.kt:148-177,216-237`），尚不能宣称原实现已满足这一要求。
- 各 timeline 的缓存代际独立，不替代原 history action 的 expectedGeneration；后端 owner 重建时旧前端缓存不可仅凭相同数字继续复用。具体重连仍沿用待审查的重建缓存方向，不先增加持久全局版本。
- 六条独立通知不构成 Agent 级原子快照：现有完成工具会依次写 index/work、unstable、timestamp，最后发布 Agent latestIndex/state（`Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt:376-393`）。当前弱同步允许展示短暂滞后，但不能将任意混合读取推导成强一致 runtime 状态；业务准入、execution 和写入仍由后端负责。
- 分开的 generation/latestIndex Flow 可以保留弱同步，但旧尾索引或旧代读取不能推进新缓存的已加载边界；实际初始化、回退再追加与流交错仍需验证。不为此预设新状态 DTO 或多客户端协调。
- 只读 timeline 不提供 set/revert/CAS：settings CAS、submit、revertHistory 等仍是后端业务命令。settings timeline 是历史读取，当前 settings StateFlow/CAS 是可编辑状态；后续适配应避免为同一当前值建立相互回写的两个 authority。
- 此边界覆盖持久化 timeline，不覆盖尚未落盘的 Responses 流片段、执行状态或交互命令；这些仍需对应 RPC。传输原值不引入 DTO，但也不自动提供数据脱敏或有界响应，必须保持既有前端可见范围及按需读取。
- 本轮是源码评估与文档修订，没有创建 RPC 或修改 CachedIndexVersioned；未运行测试、编译或网络原型。后续继续逐项审批，不将边界合理性视为批量实施授权。

FIXME:

### 首项契约：IndexTimelineRpc

- 用户已批准包含 get/floor/ceil 的修订版 `IndexTimelineRpc`，以及 rpc-contract 的 clean-models 依赖与编译检查。本项只处理 `KodexAgentStorage.index`，不扩展其他五条 timeline，不改后端 CachedIndexVersioned 或 frontend cache 实现。
- 目标由原 `sessionIndex` 定位 persisted root Agent 的 index timeline；generation/latestIndex 都属于这条后端缓存，不混用 Agent 全局尾索引或 HistoryIndexViewModel 代际。
- 首次落下的接口形状（省略 public 与注释；后续已改为继承 `TimelineRpc<CleanIndexEntry>` 并为以下全部方法增加 override，见继承核查一节）：

```kotlin
@Rpc
interface IndexTimelineRpc {
    suspend fun getGeneration(sessionIndex: Int): Long
    fun getGenerationFlow(sessionIndex: Int): Flow<Long>

    suspend fun getLatestIndex(sessionIndex: Int): Int
    fun getLatestIndexFlow(sessionIndex: Int): Flow<Int>

    suspend fun get(
        sessionIndex: Int,
        generation: Long,
        index: Int,
    ): CleanIndexEntry

    suspend fun floorToIndex(
        sessionIndex: Int,
        generation: Long,
        index: Int,
    ): Int?

    suspend fun ceilToIndex(
        sessionIndex: Int,
        generation: Long,
        index: Int,
    ): Int?

    suspend fun indexesIn(
        sessionIndex: Int,
        generation: Long,
        fromInclusive: Int,
        toInclusive: Int,
    ): List<Int>

    suspend fun getExact(
        sessionIndex: Int,
        generation: Long,
        index: Int,
    ): CleanIndexEntry?

    suspend fun valuesIn(
        sessionIndex: Int,
        generation: Long,
        fromInclusive: Int,
        toInclusive: Int,
    ): List<Pair<Int, CleanIndexEntry>>
}
```

- 两个状态 Get 仅初始化，两条 Flow 均含订阅时当前值；latestIndex 为本 timeline 的最大存储 index，空 timeline 为 -1。generation 只读，无 CAS。
- 用户要求 get/floorToIndex/ceilToIndex 也进入 RPC，不以“前端能从完整索引推导”为由省略。六个数据读取入口保留原 exact/floor/ceil/稀疏/inclusive range 语义；范围以两个 Int 表达，不新增范围 DTO 或 serializer。indexesIn/valuesIn 按 index 升序，只返回实际存在项；空区间返回空列表。getExact/floorToIndex/ceilToIndex 的 null 只表示对应值或边界不存在；get 返回该位置可见值，在第一个存储 index 之前读取仍失败。代际不匹配应失败，不伪装成 null/空列表。
- 契约允许前端远程执行未知区间的 get/floor/ceil，无需先补齐全索引；本地未命中不代表后端不存在。本轮不实施缓存策略。
- 已新增 `Kodex/rpc/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/IndexTimelineRpc.kt:7-58`，在该模块添加 `api(project(":agent-storage-clean-models"))`。CleanIndexEntry 及既有多态模型直接复用，未修改原模型注解、原模块配置或持久化格式。
- 编译结果：复用原 Temurin 25 Daemon，JVM、linuxX64、linuxArm64、mingwX64 四项目标实际编译通过（五目标命令 BUILD SUCCESSFUL，36 秒）。macosArm64 为 SKIPPED，不能计为通过；之前仅 settings 契约的五目标成功不代表当前依赖图仍能在 Linux 编译 macOS。
- macOS 跳过原因经单目标 `--info` 核实：`Cross compilation should be possible with project dependencies` 条件为 false。依赖链包含 `agent-storage-clean-models → tool-unified-exec-contract → utils-shell-client` 的平台 cinterop；未切换设备、移除依赖或修改无关模块来绕过。
- 首次编译因原 Daemon 忙而自动新建 Daemon，未满足复用要求，已取消本次客户端并在新 Daemon 空闲后仅结束该新进程；未干扰原 Daemon。确认原 Daemon 空闲后重试成功，Java 路径保持不变。
- 本轮只验证契约及生成代码的编译，没有单元测试、序列化往返、generation 并发/缓存或网络运行验证；未实现客户端与后端。macOS 编译仍待具备对应依赖构建条件时验证，不将本次检查表述为所有目标通过。

FIXME:

### 同构批次：其余五条 timeline

- 用户明确授权同批补齐其余五条 timeline，沿用已批准的 IndexTimelineRpc 切面；本次批量授权不扩展到其他领域 RPC、原模型序列化改造或客户端/后端实现。
- 新源码均位于 `Kodex/rpc/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/`：

| 原 timeline | 契约文件 | 原值类型 |
| --- | --- | --- |
| work | `WorkTimelineRpc.kt:16` | `StableWorkEvent` |
| settings | `SettingsTimelineRpc.kt:17` | `KodexAgentSettings` |
| timestamp | `TimestampTimelineRpc.kt:16` | `Instant` |
| tokenCount | `TokenCountTimelineRpc.kt:15` | `Long` |
| unstable | `UnstableTimelineRpc.kt:17` | `List<UnstableCleanEvent>` |

- 每条保留与 IndexTimelineRpc 相同的 10 个方法：两个仅初始化的 metadata Get、两个包含当前值的 Flow，以及六个带 Session/generation 的只读查询。范围仍为闭区间；没有 set/revert/CAS 或其他新增操作。
- SettingsTimelineRpc 只读历史 settings，不替代 AgentSettingsRpc 的当前 settings CAS；UnstableTimelineRpc 中空列表是合法的已存值，getExact 返回 null 才表示该代该 index 无记录。
- 本批只新增五个契约文件和本文记录，复用现有模块依赖，没有修改原模型、依赖声明、缓存、展示、时间戳功能或其他并行任务。六条 timeline 的契约声明现已齐全，但原 CachedIndexVersioned 仍未增加 generation/订阅实现。
- 静态签名核对：使用 uv 运行一次性脚本，以 IndexTimelineRpc 为基准逐方法比较挂起标记、参数、返回值替换；五条各 10 个签名均匹配。JVM 产物也确认每条具有 10 个方法和编译插件生成的 RPC stub。这是结构/生成结果检查，不是单元测试或网络验证。
- 编译前确认原 Gradle Daemon（Temurin 25）空闲并复用相同 JVM；运行原五目标编译命令，JVM、linuxX64、linuxArm64、mingwX64 实际通过（1 秒）。macosArm64 仍因上一节已核实的依赖/cinterop 条件 SKIPPED，未计为通过，没有切换设备或修改无关模块。
- 仅契约及生成代码编译；未运行单元测试、序列化往返、缓存代际并发或 RPC 网络验证，没有客户端/后端实现。不创建 Git commit，任务继续留在 Planning。

FIXME:

### 泛型父接口与 RPC 继承核查（0.10.3）

- 用户先要求研究六条同构 timeline 的接口继承，随后明确选择普通泛型父接口与具体 RPC 显式 override 的模式；现授权按此调整六条契约，不引入生成脚本或修改编译插件。
- 原理：编译插件扫描具体 `@Rpc` 服务，生成客户端 stub、方法描述符及服务调用入口；生成器按扫描结果逐方法生成，不是在运行时反射遍历父接口来补齐 RPC。[官方扫描入口](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/compiler-plugin/compiler-plugin-backend/src/main/kotlin/kotlinx/rpc/codegen/extension/RpcIrServiceProcessor.kt)、[stub 生成器](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/compiler-plugin/compiler-plugin-backend/src/main/kotlin/kotlinx/rpc/codegen/extension/RpcStubGenerator.kt)。
- 明确限制一：`@Rpc interface TimelineRpc<T>` 被检查器以 `TYPE_PARAMETERS_IN_RPC_INTERFACE` 拒绝；服务方法本身也不能声明类型参数。普通 Kotlin 泛型接口不受这一条限制。[官方检查器](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/compiler-plugin/compiler-plugin-k2/src/main/kotlin/kotlinx/rpc/codegen/checkers/FirRpcServiceDeclarationChecker.kt#L27-L44)。
- 抽象类不能作为替代服务契约：虽然 `@Rpc` 的 Kotlin annotation target 包含 `CLASS`，IR 服务生成入口还明确要求 `declaration.isInterface`，不会为抽象类生成服务 stub/描述符。不要把注解可放置的位置等同于支持的服务形态；普通抽象类可用于后端实现复用，但不消除 RPC 接口声明。[注解定义](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/core/src/commonMain/kotlin/kotlinx/rpc/annotations/Rpc.kt#L41-L44)、[生成入口](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/compiler-plugin/compiler-plugin-backend/src/main/kotlin/kotlinx/rpc/codegen/extension/RpcIrServiceProcessor.kt#L15-L18)。本项为源码核查，未制作抽象类编译样例。
- 明确限制二：扫描器遍历 `service.declarations`，跳过 `isFakeOverride` 方法。因此普通泛型父接口加空的具体 `@Rpc` 子接口，不能直接替代当前契约：仅继承而未重声明的方法不会进入该扫描结果。这里不将未复现的具体编译错误或运行异常写成事实。[官方扫描器](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/compiler-plugin/compiler-plugin-backend/src/main/kotlin/kotlinx/rpc/codegen/extension/RpcDeclarationScanner.kt#L22-L37)。
- 已用 `javap` 交叉核对本机实际使用的 `2.4.0-0.10.3` compiler-plugin-k2/backend 缓存产物：泛型服务诊断和跳过 fake override 的分支与该 tag 源码一致。结论针对当前 kRPC 编译插件，不推定 gRPC 预览版或未来版本相同。
- 已选择的复用方式：普通 `TimelineRpc<T>` 统一业务类型约束与公共语义，六条具体 `@Rpc` 接口仍逐项显式 `override` 并写明具体类型；它统一抽象，不消除六份传输签名。父接口不加 `@Rpc`，不新增 DTO、依赖或默认实现。
- 构建期模板生成不采用；客户端本地通用适配实现仍不在本轮范围。
- 已新增 `Kodex/rpc/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/TimelineRpc.kt:5-80`，将原 IndexTimelineRpc 的公共语义与 10 个方法移入普通父接口；六条服务保持原方法名、挂起标记、参数与返回类型，仅增加具体父类型及显式 override。Index 的重复方法注释移到父接口，其他服务引用父接口公共语义。
- 实际编译验证：复用原 Temurin 25 Daemon，JVM、linuxX64、linuxArm64、mingwX64 编译通过（五目标命令 BUILD SUCCESSFUL，1 秒）；macosArm64 仍因既有传递依赖/cinterop 条件 SKIPPED，不计为通过。
- 使用 uv 一次性脚本逐项核对六条服务与父接口的类型替换及 10 个显式 override 签名；使用 javap 检查 JVM 产物，每条都有 10 个 stub 方法、对应 invokator 及 callable 描述符初始化，普通父接口没有生成服务 stub。此前对显式重声明方式的源码推断现已得到编译和生成结构验证。
- 本轮未运行单元测试、序列化往返或 RPC 网络验证；没有客户端/后端实现，也未验证普通父接口加空子接口的失败样例。只修改契约与本文，不改依赖、原模型或其他任务，仍处于 Planning。

FIXME:

### 下一项审查：设置领域与派生状态

- 用户否定独立 OpenAiModelCatalogRpc 的服务分组，要求将模型目录作为更大设置 RPC 范围的一环，与 settings 及相关派生 StateFlow 一起审查。独立服务提案撤回，没有创建源码。
- 分组按领域职责，而不是每个内部 store/catalog 对应一个服务；同一服务可以同时提供 settings 的 Get/Flow/CAS 和相关只读状态的 Get/Flow，不因此把派生数据塞进持久化 settings 值或给只读派生值增加 CAS。
- 当前事实需与迁移归类区分：OpenAiModelCatalog 自己持有 models StateFlow，以 BuiltInModelCatalog 初始化并从 provider 刷新，不是直接 `settings.map`。Application 创建共享目录并把其 models 注入多处消费方（`Kodex/openai/model-catalog/impl/src/commonMain/kotlin/io/github/stream29/kodex/openai/modelcatalog/OpenAiModelCatalogImpl.kt:34-47,68-71`；`Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt:319,373,405,420,500-508`）。归入设置领域不等于已实现随所有设置更新自动重建目录。
- 已获批签名包含 GlobalRpc 的 `getModels(): List<ModelInfo>` 和 `getModelsFlow(): Flow<List<ModelInfo>>`；原类型已有序列化支持，rpc-contract 已依赖 openai-models。Get 只初始化，后续靠包含当前值的订阅流；目录读取本身不需要 Session 寻址。
- 保持全局设置与按 Session 版本化 KodexAgentSettings 的区别；用户确认全局服务命名为 `GlobalRpc`，settings 是其中的方法组，模型目录作为同一全局服务的关联读取，不擅自并入已有 AgentSettingsRpc。是否需要普通父接口组合仍未决定。
- 后端设置快照类型要结合已确认的前后端设置拆分审查；不得直接把包含前端字段的当前完整设置对象跨线，也不得把认证凭据作为派生状态发送给前端。
- 原 refresh/resolve/close 不因服务归组自动成为 RPC 方法，已有后端能力不移除；其他关联状态是否需要远程 Get/Flow，继续从现有调用和所有权核对。
- 本轮只记录归组方向与源码事实，没有新增契约、修改模型/依赖或编译；其他领域仍逐项审批。

FIXME:

#### GlobalRpc 分组已确认，具体切面待审查

- 用户明确将全局类型命名为 `GlobalRpc`，而非 SettingsRpc 或独立 OpenAiModelCatalogRpc；settings 是全局服务的一组方法，不以设置页面限制服务范围。已有 AgentSettingsRpc 保持 Session 寻址，GlobalRpc 的全局方法不带 sessionIndex。本次确认不等于其所有方法、其他领域的归属或原类型修改已获批。
- 全局 settings 方法组以 Get/Flow/CAS 表达；模型目录使用同一 GlobalRpc 上的 `getModels/getModelsFlow`，只读、无 CAS。首次批准的六字段 GlobalSettings 已落下，但用户最新方向仅考虑 frontend hooks，不设计 backend hooks；hooks 应退出全局 RPC 快照，源码调整尚未实施。其余五字段不变，不用原始 KodexGlobalSettings 跨线。
- 当前真源字段为 authSource、shell、newLineKey、contextSources、newSession、sessionTitle、sidebars、mcpServers、hooks（`Kodex/app/shared/settings/contract/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/KodexGlobalSettings.kt:32-47`）。字段拆分候选：
  - 前端：newLineKey，以及 sidebars 的左右内容选择；宽度按已确认方向退出设置，作为启动时初始化的临时值。除宽度外，此处归属仍待批准。
  - 后端：authSource、shell、contextSources、newSession、sessionTitle、mcpServers；后端持久化不等于完整原值均可公开读取。hooks 按最新方向只考虑前端，不再作为后端配置候选。
  - 已持久化 Session 的 KodexAgentSettings 仍由 AgentSettingsRpc 管理；NewSession 默认值属于全局设置，不为设置页的每个子页面重复建立真源。
- 派生状态分两类：
  - 已传输真源上的纯展示计算留在前端：GlobalSettingsViewModel 与 NewSessionSettingsViewModel 已 combine settings/models 来生成 modelOptions 等值（`Kodex/app/viewmodel/settings/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/GlobalSettingsViewModel.kt:118-135`、`NewSessionSettingsViewModel.kt:39-53`）。这类 StateFlow 无须逐个搬到 RPC，现有展示模型也不因此需要补序列化。
  - 依赖后端运行的认证、usage、MCP 等关联状态，可归在设置领域暴露 Get/Flow，复用现有脱敏类型；具体选哪些入口及命令仍逐项审批。不能把所有这些状态都称为 settings 的纯函数投影。
- 具体限制：MCP 原配置包含 secret header/environment 和 OAuth 凭据，既有 checklist 要求下游只接收脱敏摘要（`Kodex/mcp/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpSettings.kt:56-154`；`checklist/mcp-management.md`）。原始全局快照不能直接跨线再让前端完整 CAS；也不能靠把 secret 替换成空值来构造可回写的假快照。
- MCP 已有 `McpManagedServerState`、`McpServerDraft` 和 `McpSecretDraft.Keep/Replace` 可作为复用基础（`Kodex/mcp/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpManager.kt:32-120`）。保持 manager 命令与脱敏读取的候选不引入新 DTO；这不是本轮新增 MCP 契约或改变持久化结构的授权。
- 认证沿用 SettingsAuthenticationState 等无凭据投影，不传 OpenAiAuthState 的已认证原值。登录、usage reset、MCP 授权等有运行生命周期的操作不能仅凭 settings CAS 替代；本轮先定领域分组，不把完整 GlobalSettingsViewModel 自动照搬成 RPC。
- 归组讨论阶段仅更新本文；随后用户批准的首批模型、序列化补齐与契约实施见下节。其余原模型修改仍需逐项审批，未授权完整迁移或架构 checklist 变更。

FIXME:

#### GlobalRpc 首批：settings 与 models

- 用户首次批准创建与 contract 并列的 `Kodex/rpc/model`，并批准六字段快照及四个原类型的最小序列化补齐。以下记录该次实现与验证；后续仅 frontend hooks 方向将移除其中 hooks，尚未改源码。该模型是传输快照，不是新持久化格式，不修改现有前后端设置文件。
- 新模块 `:rpc-model` 复用 `kodex.kmp-cli` 和 serialization 插件，依赖原 `:app-shared-settings-contract`；已有 includeModuleTree 自动发现，无需再次改 settings.gradle.kts。rpc-contract 新增 api 依赖 rpc-model；模型模块不依赖契约或 kRPC 插件。
- `Kodex/rpc/model/src/commonMain/kotlin/io/github/stream29/kodex/rpc/model/GlobalSettings.kt:21-35` 包含 authSource、shell、contextSources、newSession、sessionTitle、hooks。六字段均必填，不使用客户端本机默认 shell 或配置补缺；保留原 Hook 名称非空校验。
- 不含 newLineKey/sidebars 或 mcpServers；这是已批准的本次公开快照范围，前端文件实际拆分与 MCP 管理 RPC 仍未实施。
- `Kodex/rpc/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/GlobalRpc.kt:16-48`：

```kotlin
@Rpc
interface GlobalRpc {
    suspend fun getSettings(): GlobalSettings
    fun getSettingsFlow(): Flow<GlobalSettings>
    suspend fun compareAndSetSettings(expect: GlobalSettings, update: GlobalSettings): Boolean
    suspend fun getModels(): List<ModelInfo>
    fun getModelsFlow(): Flow<List<ModelInfo>>
}
```

- Get 仅初始化，两个 Flow 均含当前值；settings CAS 失败仍等待流推进，不 Get 刷新。models 只读，不为 provider 刷新新增命令；两个状态源不承诺原子联合快照。
- CAS 只比较并替换公开快照所覆盖的六字段，在后端已有设置写入临界区内完成，保留 MCP 等未公开字段；false 仅表示值比较不匹配，不吞掉验证/持久化错误。后端仍需遵守既有设置和 Hook 管理规则；本轮仅写下契约，没有实现 CAS、mapper 或持久化。
- 原类型改动仅为 AgentContextSourceSettings、AgentContextCustomSource、KodexNewSessionSettings、SessionTitleSettings 加 @Serializable；agent-context-contract 补既有 serialization 插件与 core 依赖。未改字段、默认值、业务方法或原持久化私有 GlobalSettingsFile 及其嵌套文件模型。
- 构建环境：首次命令遇到状态检查后的 IDEA 同步占用原 Daemon，Gradle 自动新建的 Daemon 不符合复用规则；已取消自己的构建，并在该新 Daemon 空闲后结束它，未中断 IDEA/原 Daemon。确认原 Temurin 25 Daemon 空闲后复用重试成功。
- 编译：同时检查 rpc-model/rpc-contract 的五目标；两模块 JVM、linuxX64、linuxArm64、mingwX64 实际通过，macosArm64 均因既有传递依赖/cinterop 条件 SKIPPED，不计为通过。命令同时包含下面两项 jvmTest，BUILD SUCCESSFUL（1 分 10 秒）；未切换设备或改无关依赖。
- 生成检查：javap 确认 GlobalRpc 的 5 个 stub 方法、invokator 和 callable 描述符完整；GlobalSettings serializer 只包含获批六字段，四个原嵌套类型的 serializer 均已生成。这是生成结构检查，不是序列化往返运行验证。
- 回归测试：`:app-shared-settings-contract:jvmTest` 3 项、`:app-shared-settings-filesystem:jvmTest` 16 项全部通过，无失败或跳过，覆盖原设置内存、文件与权限行为；没有修改原测试或持久化文件模型。
- 未运行新增 GlobalSettings 的序列化往返、Native 可执行程序或 RPC 网络/CAS/订阅验证；未实现客户端、后端、文件迁移或 MCP 管理 RPC。代码与文档空白检查通过，无 Git commit，继续留在 Planning。

FIXME:

#### GlobalRpc 覆盖审查

- 用户要求检查还缺哪些能力；本轮只审查并更新本文，不添加方法或撤销此前获批的六字段模型。现有 5 个方法覆盖公开 settings 和 models，不是完整的全局应用边界。
- 应补齐的全局能力，具体签名及原类型序列化仍需审批：
  - 认证：无凭据认证摘要 Get/Flow、reload/logout，以及登录 attempt 的启动、状态、授权 URL 交付与取消；authSource CAS 只切换来源，不能替代上述行为。参考 `Kodex/app/shared/auth/contract/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/KodexAuthStore.kt:16-39`、`Kodex/app/contract/settings/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/OpenAiLoginViewModel.kt`。
  - 账号用量：脱敏 usage Get/Flow、refresh 和 usage reset 操作；原 CodexAccountUsageState.Redeeming 携带 reset attempt，不能直接当公开状态传输。沿用现有 SettingsAccountUsageState 投影与后端 account/idempotency 约束，不为 RPC 复制状态机。参考 `Kodex/openai/account-usage/contract/src/commonMain/kotlin/io/github/stream29/kodex/openai/accountusage/CodexAccountUsageStore.kt:7-54`、`Kodex/app/contract/settings/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/GlobalSettingsViewModel.kt:100-138`。
  - MCP：脱敏服务器状态 Get/Flow，以及 add/edit/delete/setEnabled/login/cancelLogin/logout/reconnect、Codex import preview/apply 和授权 URL 交付。mcpServers 已从公开 settings 排除，因此当前 CAS 不覆盖这些功能。参考 `Kodex/mcp/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpManager.kt:229-266`。
  - Session 集合管理：轻量列表、创建并初始化、打开/复用后端 owner、archive/unarchive、完整 fork、delete；集合入口可归入 GlobalRpc，单 Session 运行接口仍另审。不能把返回 ViewModel/带 lambda 的 registry 方法直接序列化；复用既有值和 index 寻址。参考 `Kodex/app/contract/session/src/commonMain/kotlin/io/github/stream29/kodex/app/session/contract/PersistedSessionViewModelRegistry.kt:12-51`、`Kodex/agent-session/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/contract/KodexSession.kt:112-139`。
- 不能从已有模型推断已具备的行为：当前 Session catalog 是 popup-owned refresh 模型，repository listEntries 是挂起查询；跨客户端变化如何通知、前端如何筛选尚待审查，不能直接声称已有共享 catalog StateFlow。
- 条件性入口：
  - 后端目录枚举/路径解析可按需提供，但前端目录弹窗与导航仍是本地对象；同机范围不自动等于所有文件读取均需 RPC。参考 `Kodex/app/viewmodel/path-picker/src/commonMain/kotlin/io/github/stream29/kodex/app/pathpicker/DirectoryPickerBrowser.kt:27-60`，宿主文件边界仍未定。
  - 显式后端 shutdown、版本匹配/连接初始化取决于后端部署协议；不先加 ping、通用 capability registry 或把原 Application.shutdown 直接变为前端退出行为。
- 不应搬到 GlobalRpc：tab selection、popup/SettingsPage、布局和宽度、child 引用及 close、纯 modelOptions 派生。前端 closeTab 不得沿用当前 release/shutdown 路径停止其他前端共享的任务；关闭订阅与释放后端 owner 的规则仍需后续实现设计。
- 首轮 settings 的两项 Hooks 审查点（最新仅 frontend hooks 方向不再将它们作为 GlobalRpc 的待补能力）：
  - 当前 GlobalSettings.hooks 含完整 command，并随普通 settings Get/Flow 暴露；既有 `HookManager` 只连续发布 name/type，command 仅在显式 editorDraft 时读取（`Kodex/hook/contract/src/commonMain/kotlin/io/github/stream29/kodex/hook/contract/HookManager.kt:12-47`、`checklist/hooks.md`）。之前批准包含 hooks，不代表已解决这一旧约定的迁移差异；保留六字段源码并提请确认，不擅改 checklist 或补第二套 Hook 真源。
  - HookConfiguration 是有执行顺序语义的 Map，而 Kotlin Map.equals 只比较键和值，不比较迭代顺序；GlobalSettings 的默认 data-class 相等性因此不能区分同条目不同顺序。按当前 CAS 的“相等即无变化”规则，顺序变化无法被表达为有效更新。这是既有 Map 表示在新全量 CAS 上的限制，不是已运行复现的 RPC 故障，也不等于要求新增排序功能。[官方 Map 相等性约定](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.collections/-map/)；顺序要求见 `checklist/hooks.md`，类型见 `Kodex/hook/contract/src/commonMain/kotlin/io/github/stream29/kodex/hook/contract/HookSettings.kt:12-20`。是否保留 manager 命令或调整公开快照的顺序语义，待用户选择。
- 建议后续补认证和 MCP，再处理 Session 集合及 usage；Hooks 按下面最新方向处理，不再补后端 Hook 管理 RPC。这里只给出候选顺序，不代表获得实施授权。
- 验证范围：读取当前源码与 checklist，并核实官方 Map.equals 约定；没有修改源码、编译或运行测试，不沿用上一轮 19 项设置测试声称新全局能力或上述 Hooks 边界已验证。

FIXME:

#### 最新方向：仅考虑 frontend hooks

- 用户明确区分 frontend/backend hooks，现阶段不做 backend hooks，只考虑 frontend hooks；这取代之前将 hooks 纳入后端全局快照的方向。
- 迁移切面据此应从 rpc/model.GlobalSettings 移除 hooks 及对应校验，并移除 GlobalRpc 的 Hook 管理表述；不新增远程 Hook CRUD/状态接口。当前源码仍是前次六字段版本，尚未执行本次修改。
- 需要确认 frontend 的含义：仅前端本地行为/观察通知，还是保留控制后端执行的旧 Hook 语义、只把命令执行位置放到前端。后者还需要请求/返回边界及无前端时的行为，不能默认为已获批。
- 已核实旧契约含控制型 Hook：ToolHooks.onPreToolUse 返回 PreToolUseResult，TurnHooks.onUserPromptSubmit/onStop 返回会影响流程的结果（`Kodex/hook/contract/src/commonMain/kotlin/io/github/stream29/kodex/hook/contract/tool/ToolHooks.kt:4-8`、`turn/TurnHooks.kt:4-8`；调用见 `Kodex/agent-runtime/decorator/turn-hook/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/turnhook/TurnHookRuntime.kt:49,95`）。不能宣称仅改配置所有权即可保留全部旧行为。
- 尚未确定前端 Hook 事件集合、配置模型、多个前端的触发范围、重连/无人在线时的补触发语义；不提前设计广播或回调协议。本轮只记录方向和待确认行为，不改现有 Hooks 实现、持久化文件或 checklist。

FIXME:

### 长 Session：索引内存优化单独记录

- 用户质疑前端全量持有索引对超长 Session 不友好。评估：RPC/IndexVersioned 本身不要求这样做；既然远程保留 get/floor/ceil 与区间读取，前端可以只保存 generation/latestIndex、当前展示窗口及有限值缓存。
- 用户确认先记录该问题，继续推进 RPC 切面，不将前端索引内存有界作为 RPC 分离的前置条件；本轮不推进缓存窗口化、侧栏或滚动重构。
- 区分两个目标：避免每次经 RPC 重传全索引，可通过轻量通知与区间增量读取解决；避免前端最终持有全索引，还需要调整缓存与展示。前者不以完成后者为条件。
- 初期允许前端保留完整索引列表以复用现有逻辑，值内容仍按需加载与有限缓存；RPC 保留 get/floor/ceil 等完整查询能力，不将未来实现锁死在全索引方案。
- 后续独立优化候选：exact 值按需缓存，未知位置与区间直接 RPC，窗口随滚动加载和淘汰。是否额外缓存查询区间及其覆盖信息以后按需要决定，不先引入通用区间缓存框架。
- 前后端无须复用完全相同的缓存布局：后端现有 CachedIndexVersioned 持有完整索引，可服务查询；前端未来可只复用只读 IndexVersioned 访问语义和按需缓存策略。原类的搬迁/抽取仍未授权。
- 缓存以实际存储 index 的 exact 值为基础；get 的可见值、floor/ceil、空结果和未来区间可能受同代追加影响，不能把这些查询结果当作仅 generation 改变才失效的永久缓存。最小实现可先直接委托这些查询。
- 已核实当前展示仍依赖全列表：HistoryIndexViewModelImpl 初次/重建读取至 latest，并在追加时执行 `current.indexes + appended`（`Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexViewModel.kt:115-149`）；侧栏用 `window.indexes` 供给 LazyColumn，并以完整列表位置/大小绘图（`Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt:360-377`）。LazyColumn 仅按需组合行，不意味着该索引列表有界。
- 所以“前端有界”还需要后续审查窗口加载、滚动锚点与边界绘制，不能只替换底层 RPC 后继续原样拼接全列表。主 History 已有窗口加载机制可作为复用参考，不在本项直接修改展示约定或代码。
- 区间 API 不自动保证响应有界：即便前端初期最终保留全索引，也可分段初始化、随后只拉新增部分，避免每次重复请求 `0..latestIndex`。现有遍历扩展的指数增大查询区间仍需在远程使用时审查；是否需要按条数分页及其签名待讨论，不预设新 DTO。
- 本节仅记录索引优化的评估结论，不授权修改缓存/展示或运行性能验证；IndexTimelineRpc 的落地与编译单独记录于首项契约一节。

FIXME:

### 前端缓存与 generation 核查

- 用户要求检查：前端缓存后端数据时，现有 generation 等失效机制在前后端分离后是否仍有效。
- 本轮检查当前源码与测试定义；子模块 HEAD 为 `92b572d4`，不是跨 RPC 运行验证。发现当前 history 操作已使用 `untilExclusive` 与独立 `expectedGeneration`，本节以当前接口为准。
- 结论：代际失效机制可继续复用，不需要因此新增 DTO 或多端协调协议；但不能只传一个 generation 数值，丢掉其所属 owner、请求与缓存生命周期。

#### 已核实的现有机制

- **主 History 缓存**：`itemCache` 按 storage index 复用 child；普通追加、窗口翻页保持 generation。破坏性替换提升 generation，释放条目与 group 缓存并发布新代窗口；当前还会按保留窗口裁剪缓存，不是只等 generation 变化才回收。
  `Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt:60-73,235-242,333-374,421-484,548-581`。
- **旧异步读取**：item context 捕获 generation，检查 owner 未关闭且 `activeGeneration` 仍匹配；Message 读取结束后才据此发布 Ready/Failed。分页请求另外校验 exact window 对象，不能由 generation 单独替代。
  `Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt:169-202,225-232`、
  `Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/MessageHistoryItemViewModel.kt:21-43`。
- **HistoryIndex 已有代际保护的读取形状**：`HistoryIndexWindow(generation, indexes)` 是已有值类型，但用户已否定将完整窗口持续跨线传输；现有 `load(generation, index)`、`loadDetail(generation, index)` 读取前后校验代际及条目存在性，这种保护仍可作为按需 timeline RPC 的参考。
  `Kodex/app/contract/agent/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/HistoryIndexViewModel.kt:7-59`、
  `Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexViewModel.kt:82-99,152-173`。
- **前端已按代际隔离缓存**：侧栏 row key 带 generation/index；`remember` 与加载 `LaunchedEffect` 绑定 ViewModel/generation/index。主 History 则以 child ViewModel 对象作为 row key；这些本地身份规则可保留，不需传输对象地址。
  `Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt:365-419`、
  `Kodex/app/view/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt:146-158`。
- **原始值缓存是另一层**：`CachedIndexVersioned` 自己维护 index 列表和限容量、访问过期的值缓存；revert 删除 suffix 时清空值缓存。它没有供远程消费者订阅的 generation，后端清缓存不会自动清除前端副本。
  `Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt:112-139,165-177,266-286`。
- **业务操作已有代际校验**：revert 校验 `expectedGeneration`、边界与运行状态；fork 还检查 exact rootAgent owner。迁移应让 RPC 保留这些入口，不让客户端缓存的校验代替后端校验。
  `Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt:309-315,413-428`、
  `Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt:302-331`。

#### generation 的作用域与限制

- 主 History 和 HistoryIndex 各自在 ViewModel 构造时从 0 开始计数，分别根据 latest index 回退和观察到的 ExternalWrite 完成状态失效；两者不是同一个全局版本号，也不是持久化 storage revision。
  `Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt:60-73,137-155,715-726`、
  `Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexViewModel.kt:44-78,115-136`。
- 相同 generation 只在同一 owner、同一数据源生命周期内有意义；不能跨 Session、混用主 History/HistoryIndex，或在后端 owner 重建后仅凭数字相同继续命中旧缓存。
- generation 表示破坏性替换，不是每次变化的 revision：同代仍会追加条目、改变窗口范围或完成 group 投影。仍需消费已有窗口内容及 child 状态，不能只比较 generation 后跳过其余更新。
- 主 History 当前窗口及分页还依赖对象身份；零新增 DTO 不等于直接序列化 `HistoryItemWindow` 和 child 引用，具体 RPC 映射仍待确认。

#### 跨 RPC 后的最小保留条件（推论，待确认）

- **代际以对应后端 owner 为准**：最新方向是轻量订阅后端 generation，而非传整窗；不要在每个前端根据远程 `latestIndex` 或瞬时 `ExternalWrite` 独立重新计算。反例：断线期间先回退、再追加到原 index，前端恢复时可能只看到相同尾索引，无法据此识别历史已替换。
- **代际与内容一起解释**：按需请求携带 generation，结果绑定发起时的代际；不能将旧内容当作新代内容。前端收到新代后清理旧代 child/详情/加载状态，再本地重建窗口。
- **读取结果绑定发起时的 owner 和 generation**：后端保留已有带 generation 的读取校验；前端用捕获的请求参数和本地 owner 判断是否仍应接收结果。返回值不必为了这个检查再套 DTO；现有 `remember` key、加载 scope 和 `isCurrent` 模式可复用。
- **允许短暂旧展示，不允许跨代污染**：旧代结果在新代窗口到达前暂时展示符合当前弱同步要求；新代已到达后，旧请求的迟到结果不能重新填入新代缓存。这是缓存归属正确性，不是多用户强一致性要求。
- **重连不跨未知 owner 复用缓存**：最小候选是新连接或重新打开时重建本地代理/缓存，重新取得窗口；不需要先新增持久 epoch。若以后要求跨连接保留缓存命中，再明确如何证明仍是同一后端 owner。
- **storage 缓存复用需保留代际保护**：底层 `getExact(index)` 没有 generation 入参，不能仅复制其按 index 命中的规则到前端。最新方向是远程只读 timeline 增加代际校验，本地缓存沿用其索引/按需值读取能力；原 CachedIndexVersioned 的 filesystem 耦合仍需调整审批。
- 普通完整 `StateFlow<T>` 快照继续以新值替换本地投影，不因 History 的需要给所有状态追加统一 revision。

FIXME:

#### 证据与后续验证范围

- 已有测试定义覆盖主 History revert 后 generation 加一、旧目标失效、旧窗口仍可索引，以及翻页复用 child identity：
  `Kodex/app/viewmodel/history/src/commonTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryModelsTest.kt:540-603`。
- HistoryIndex 测试定义覆盖追加、回退以及同 index 替换后的代际变化：
  `Kodex/app/viewmodel/agent/src/commonTest/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexViewModelTest.kt:102-165`。
- 已有 action 测试定义包含 stale generation 和 foreign owner 拒绝：
  `Kodex/app/viewmodel/session/src/commonTest/kotlin/io/github/stream29/kodex/cli/session/AgentHistoryActionTest.kt:33-103,134-198`。
- 缓存核查阶段未运行上述测试或搭建 RPC 原型；静态检查证明了机制与校验入口存在，后续首批 settings 契约编译也不构成缓存的跨进程验证。
- 候选验证重点：追加时缓存复用；回退后同 index 新内容；新代发布后旧读取迟到；断线期间回退再追加；owner 重建后 generation 从 0 开始；同代翻页/窗口替换不得被误判为无变化。

## 其余待定边界

- 设置：前后端文件位置与字段归属、旧配置是否迁移或忽略、侧栏默认宽度的具体初始化基准。
- 后端：按需或显式启动、发现与避免重复启动、是否按 Home 隔离、空闲退出条件。
- 生命周期：连接/订阅取消的资源释放、已接受任务的后端所有者、关闭 tab 与关闭 Session 的区别、显式后端 shutdown。
- 多客户端：状态合法性与并发处理沿用既有后端规则，不再作为新的协作策略待选；继续讨论草稿归属与具体 RPC 入口映射，不引入多用户控制权、协同编辑或通用仲裁系统。
- Flow：关注初始值、后续更新、流切换、重连与资源释放的基本正确性；回执丢失及慢客户端结合具体访问路径核对，不将跨端强同步列为迁移前提，不预设持久事件日志、任意位置重放或全局事件总线。
- 宿主：终端通知、路径选择、文件/图片访问、登录回调分别由哪侧执行；保持后端凭据不向前端泄露。
- 兼容性：前后端版本匹配方式、所选发布与现有依赖组合、四个 Native 目标的实际传输与运行能力。

## 后续验证候选范围与当前证据

- 当前判断：具备 RPC 命令与 Flow 订阅的技术基础，优先复用现有模型；全局公开 settings 已作为获批例外新增传输快照。完整迁移可行性尚未通过运行实现验证。
- 候选验证：现有类型序列化与接口生成、目标 Native artifact/引擎及连接、单 Session 打开/提交/Stop、流式输出、History 按需读取与失效。
- 再按已确认需求验证同一 Session 的多端操作、断连后任务继续、重连恢复、慢客户端和资源释放；多端写入检查 RPC 路径是否保留既有后端合法性保证，不要求恢复或猜测用户的跨端操作意图。
- 完整迁移的验证范围仍待讨论；当前仅授权契约模块依赖与编译检查，未授权运行跨进程原型或性能测量。
- 迁移指南随讨论逐项补全；未决项保留为问题，进入 Executable 或落实完整迁移及架构约定仍需用户明确授权。
