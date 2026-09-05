# Task Tree

- [done] 接通应用异常报告与纯文本 UnhandledError Hook
  - [done] 修复验收发现的报告缺口
    - [done] 补齐打开会话、Fork 创建和 Revert 校验的报告边界
    - [done] 统一记录原始异常并消除 Settings 重复日志
    - [done] 记录每个失败 Hook 的名称与退出状态
    - [done] 在 Agent 操作开始时捕获 cwd
    - [done] 补充真实组件与脚本回归测试
    - [done] 核实 Session 测试失败并据实更新验收记录
    - [done] 完成最终回归与宿主 Native 检查
  - [done] 梳理全项目的错误处理现状与断层点
    - [done] 梳理 OpenAI 响应层（风控拦截、流失败与盲目重试）
    - [done] 梳理 UI 通知层（AgentNotification 未接入界面渲染）
    - [done] 梳理 工具执行层（异常逃逸导致 ToolPending 死锁）
    - [done] 梳理 交互与生命周期操作层（Fork/Revert/Settings 写静默吞异常）
    - [done] 梳理 上下文与元数据发现层（AGENTS.md/Skills 告警丢弃）
  - [done] 确定轻量异常报告与纯文本 Hook 路线
    - [done] 内部直接传递原始 Throwable，不建立结构化错误体系
    - [done] 外部 Hook stdin 只接收 exception.message，不使用 JSON
    - [done] 核对现有 Hook、应用装配及失败操作边界
  - [done] 完成实施前决策
    - [done] 将原任务收敛为异常报告与纯文本 Hook
    - [done] 确认仅日志和 Hook，不增加应用内展示
    - [done] 确认异常 Hook 命令工作目录
      - [done] 确认沿用其他 Hook 的执行目录机制，不为异常 Hook 特设固定目录
      - [done] 无来源 Agent 时仍触发，回退应用启动目录
    - [done] 确认超时与并发资源策略
      - [done] 沿用所有现有 Hook 的固定 600 秒超时
      - [done] 复用现有并发，不增加队列、限流、合并或重试
  - [done] 扩展现有 Hook 能力
    - [done] 增加 UnhandledError 类型、ErrorHooks 窄接口及 NoOp
    - [done] 聚合 ErrorHooks 并复用现有配置管理与类型索引
    - [done] 通过 stdin 发送可空 message 的纯文本表示
    - [done] 保持其他 Hook 的 JSON 协议和控制语义不变
    - [done] 补充 Settings 类型标签
  - [done] 接通应用异常报告链路
    - [done] 在应用装配层创建接收 Throwable 的共享报告函数
    - [done] 在批准范围内明确每个失败操作的唯一报告边界
    - [done] 接入 Agent 操作及 Settings 写入队列
    - [done] 接入 Fork、Revert 和打开会话的最终失败边界
    - [done] 隔离 Hook 失败并保持协程取消语义
  - [done] 验证与同步约定
    - [done] 验证纯文本字节、空消息、多行、Unicode、引号及 EOF
    - [done] 验证取消排除、单次报告和多 Hook 失败隔离
    - [done] 验证配置快照、会话关闭与应用关闭的生命周期
    - [done] 验证 Settings 失败后仍可处理后续写入
    - [done] 回归原有 Hook 协议与设置增删改
    - [done] 按最终决策更新 Hooks SOP 并运行相关模块检查

# Details

## 调查背景

当前系统在多个层次存在错误处理不完善、异常静默吞掉、错误诊断丢失以及 UI 界面完全无感知的现象。本次调查覆盖了 `openai`、`agent-runtime`、`agent-state`、`tool`、`agent-context` 以及 `app`（TUI/ViewModel）等模块。

## 现状调查结果

### 1. UI 层：通知机制处于“黑洞”状态
- **现状**：在 `AgentRuntimeViewModel.kt` 中，`submit`、`resume`、`clearPending`、`forceCompact`、`executeHistoryRevert` 等操作发生异常时，均会调用 `publishFailure()` 生成 `AgentNotification(level = Error, ...)` 并存入 `mutableNotification`，通过 `AgentViewModel.notification` 暴露。
- **问题**：在当前整个 UI 视图层（`AgentRuntimeScreen.kt`、`RuntimeStatusBar.kt`、`AgentHistoryView.kt`）中，**没有任何组件订阅或渲染 `viewModel.notification`**。
- **后果**：任何后台异步错误发生时，UI 上仅表现为运行动画突然停止，没有任何提示、报错弹窗或状态栏文本，用户无从得知发生了什么。

### 2. 模型请求与风控层：错误详情被丢弃并转化为盲目重试
- **现状**：
  - `KodexAgentStateImpl.kt` 在处理 SSE 流事件 `ResponsesStreamEvent.Failed` 时，直接返回 `terminalReason = RequestFinish.Retryable`，事件对象中的 `FailedResponse.error`（含 `code`、`message`、`type`）被完全丢弃。
  - `KodexAgentCompactionRuntime.kt` 捕获到 `RequestFinish.Retryable` 后盲目重试，重试 3 次耗尽后抛出 `AgentResponseRetryLimitExceededException(3)`。
  - `OpenAiClient.kt` 在 SSE 流处理的 `.catch` 中若遇到可重试传输异常，直接结束流而不重新连流，导致下游收到截断响应。
- **问题与后果**：
  - 当模型因安全风控拒绝（如 `code = "content_filter"`）、参数非法或配额问题报错时，这些非可重试错误依然被重试 3 次，白白消耗资源。
  - 最终抛出的异常是“重试次数耗尽”，OpenAI 返回的真实拒绝原因完全被掩盖。

### 3. 工具执行层：异常逃逸导致 `ToolPending` 会话死锁
- **现状**：
  - 在 `KodexToolRuntime.kt` 的 `handleToolCall` 中，执行 `val completed = tool.handle(pending)` 外部缺乏顶层 `try / catch` 异常保护。
  - 具体工具自身的异常捕获极窄：
    - `ApplyPatchTools.kt` 仅捕获 `IllegalArgumentException`，底层文件读写 `IOException` 等会直接击穿。
    - `UnifiedExecTools.kt` 仅捕获 `UnifiedExecToolException`。
    - `WebRunTools.kt` 对 `client.run` 无任何捕获，网络/超时直接抛出。
    - `JsonToolBuilder.kt` 仅捕获参数反序列化异常，未保护工具体执行。
- **问题与后果**：
  - 工具抛出未捕获异常时，整个回合崩溃，但该工具事件**未被标记为完成或失败持久化入库**。
  - 状态停留在 `KodexAgentStateValue.ToolPending`，会话陷入死锁：用户无法提交新消息，点击 Resume 依然会执行该故障工具再次崩溃，只能被迫使用“Clear Pending”强制清除工具调用。

### 4. 交互操作层：异常被 `runCatching` 或空 `catch` 静默吞掉
- **分叉会话（Fork）**：`SessionTreeCliScreen.kt` 的 `onFork` 中捕获了 `catch (_: Throwable)`，注释声称“由 Session 发布失败”，但 `session.fork()` 并无发布逻辑，出错时静默失败。
- **回退历史（Revert）**：`SessionTreeCliScreen.kt` 触发回退时使用 `runCatching`，失败时不向用户报出。
- **设置更新队列**：`SettingsUpdateQueue.kt` 执行后台保存时发生异常仅记录日志，界面没有任何保存失败反馈。
- **会话目录打开**：从会话列表（Catalog）打开会话时未加异常保护，读取或加锁失败可能直接导致协程未捕获异常崩溃。
- **OAuth 登录回调**：`LocalKodexLogin.kt` 收到浏览器回调错误时，丢弃了服务端的错误说明，统一硬编码为 `"The browser sign-in was not completed."`。

### 5. 上下文与元数据发现层：告警信息未透传
- **现状**：`FileSystemAgentsMd.kt` 和 `FileSystemSkillsResolver.kt` 探测过程中生成的结构化告警（如 UTF-8 非法、文件过大截断、文件读取失败、Skill YAML 解析失败等）在 `AgentContextPrefixResolver.kt` 组装前缀时被完全丢弃。
- **问题与后果**：关键的指令文件或技能被静默截断/忽略，开发者和模型无法感知上下文缺失。

### 6. 流式历史展示层：异常中断时无任何状态留存
- 在 `StreamingRequestResponseView.kt` 中，仅定义了正常的流式阶段展示。
- 若流式响应在传输中途断开或失败，流式视图直接消失，历史时间线末尾没有任何失败卡片或异常占位符，用户体感为模型回复突然蒸发。

## 已确定路线

- 用户于 2026-09-05 授权细化计划并迁移至 planning，后续授权进入 executable 并执行完毕。
- 用户已裁决将原任务收敛为异常报告与纯文本 Hook，而非分阶段保留全部治理项。
- 用户已裁决仅日志和 Hook；不增加悬浮提示、通知历史或其他应用内错误展示，不扩展现有 AgentNotification UI。
- 上面的现状调查仅作历史背景；请求重试分类、工具状态恢复、上下文告警和流式历史持久化不属于本任务交付或后续待办，不据此创建新任务。
- 内部报告入口直接接收原始 `Throwable`，不新增错误 ID、分类、request/context 或事件总线。
- Hook 只获取 `exception.message` 原文；不发送 JSON、会话信息、堆栈或 cause。
- 用户要求命令执行目录与其他 Hook 一样，不为异常 Hook 特设固定目录；工作目录属于内部执行参数，不进入 stdin。
- 用户确认无来源 Agent 的异常也触发 Hook，执行目录回退应用启动目录；有来源 Agent 时使用其 cwd，不取当前选中会话。
- 窄接口确定为 `suspend fun onUnhandledError(message: String?, cwd: Path)`，加入现有 `KodexHooks`；cwd 仅供进程执行，不能只靠 message 推导，不增加 context 数据类。
- 纯文本传输按 UTF-8 编码，不追加换行，发送后关闭 stdin；null 映射为空输入，不替换成异常类名。
- 原始 message 可能包含敏感信息；不承诺脱敏，不拼接入 shell command。
- 配置保持 `hooks.<name>.type/command`，新类型序列化名称为 `unhandled_error`。
- 原有 Hook 的 JSON 输入保持不变；异常 Hook 直接复用底层进程执行，不经过 JSON 投影。
- 用户确认沿用固定 600 秒超时，不为异常 Hook 增加单独超时参数或配置。
- 用户确认复用现有并发方式：每次最终失败独立触发，同类型命令并发执行；不新增队列、限流、合并或重试。
- 正常取消、仍在重试中的失败、已经转成正常工具失败结果的异常不触发。
- 在最终结束操作的边界报告一次；下层抛出的异常不重复报告。
- Hook 输出不控制业务；Hook 失败仅记日志，不递归报告，不替换原异常。

## 修改与实现

- `hook/contract/.../HookSettings.kt`: 增加 `UnhandledError` 枚举；新增 `ErrorHooks` 和 `NoOpErrorHooks`。
- `hook/contract/.../KodexHooks.kt`: 聚合 `ErrorHooks`。
- `hook/impl/.../KodexHooksImpl.kt`: 实现纯文本通知路径，按调用读取配置快照，传递 UTF-8 message，null 转换为空串，隔离异常。
- `hook/impl/.../HookExecution.kt` 与 `ShellClientHook.kt`: 将通用执行参数从 `inputJson` 改为 `input: String`，复用 stdin、600 秒超时与进程清理机制。
- `app/view/settings/.../HookSettingsContent.kt`: 补齐 `HookType.UnhandledError -> "Unhandled error"` 分支。
- `app/viewmodel/settings/.../SettingsUpdateQueue.kt`: 支持 `reportError` 回调并在写入失败时调用。
- `app/viewmodel/settings/.../SessionSettingsViewModel.kt`: 捕获 `configuration.workingDirectory` 快照并透传给 `SettingsUpdateQueue`。
- `app/viewmodel/settings/.../GlobalSettingsViewModel.kt` 与 `NewSessionSettingsViewModel.kt`: 注入 `reportUnhandledError` 并连接到 `SettingsUpdateQueue`。
- `app/viewmodel/settings/.../SettingsViewModel.kt`: 统一装配 `reportUnhandledError` 与 `startupWorkingDirectory`。
- `app/viewmodel/agent/.../AgentRuntimeViewModel.kt`: 操作开始时捕获cwd，`publishFailure`转发原异常及目录；补齐Revert校验边界。
- `app/viewmodel/session/.../SessionViewModels.kt`: Registry、Session及Catalog分别负责自己的失败阶段，不重复报告。
- `app/viewmodel/application/.../Application.kt`: 装配全局报告函数，过滤取消、记录原始异常，异步投递message，并注入Agent、Session、Catalog和Settings。
- `checklist/hooks.md`: 完整补齐 `unhandled_error` Hook 的协议规范、工作目录规则与生命周期约定。

## 原验收记录

- 2026-09-05 验收未通过，用户授权修复，原任务重新打开至 executable；以下原实现记录不能作为最终验收结论。
- 验收发现报告边界漏接、原始异常未统一落日志、Hook 返回失败结果未记录以及 Agent cwd 未捕获。
- 验收重跑原五组 JVM 任务成功，48 个用例通过；新增 Session 测试出现 3 个失败并挂起，13 分钟后终止 worker，归因尚未确认。

- `:hook-contract:jvmTest`、`:hook-impl:jvmTest`、`:app-viewmodel-settings:jvmTest`、`:app-viewmodel-agent:jvmTest`、`:app-viewmodel-application:jvmTest` 全部编译通过并测试通过。
- 在 `ConfiguredHooksTest` 中新增 `UnhandledError` 真实脚本测试（包含纯文本输入、空消息、取消语义等），测试通过。
- 在 `SettingsViewModelTest` 中新增 `SettingsUpdateQueue` 错误报告测试，测试通过。

## 验收修复

- Registry覆盖缺失会话打开与未打开会话Fork初始化失败；Session覆盖校验、创建及后续Fork步骤，清理失败作为suppressed保留原异常。
- Agent覆盖Revert校验失败，并对提交、后台操作和Revert捕获cwd；UI只处理已报告异常，取消原样传播。
- 应用报告入口记录原始异常；Settings队列仅在未配置报告入口时自行记录日志。
- Hook逐命令检查结果，记录名称与退出码；未取得退出状态时只记录unavailable，不猜测原因。
- 新增真实应用、文件系统与脚本回归：一次报告、启动目录回退、Fork目录冲突、运行中切换cwd、会话关闭与应用关闭；没有新增mock。
- 补充Hook并发失败隔离、配置快照、调用方取消、启动失败及特殊字符原文传递测试。
- 使用HEAD中的原始AgentRuntimeViewModel和SessionViewModels作隔离源码对照，4个定向Session用例仍有相同3个失败；临时源码与Gradle init脚本已删除，工作区源码未被替换。
- 既有失败为两处历史窗口校验，以及Fork标题期望Source而实际为[fork] Source；不属于此次异常报告修复，未修改相关业务或测试期望。
- 全量Session测试的原挂起未重新运行；使用定向过滤避免已知挂起，不宣称全量Session测试通过。

## 最终验证

- 2026-09-05完成本次验收修复并移回done；没有提交Git。
- 强制重跑`:hook-impl:jvmTest`、`:app-viewmodel-settings:jvmTest`、`:app-viewmodel-agent:jvmTest`、`:app-viewmodel-application:jvmTest`，52个用例全部通过；`:hook-contract:jvmTest`无用例。
- `:hook-impl:linuxX64Test`的14个用例全部通过，覆盖真实脚本输入、并发隔离、配置快照、取消及启动失败。
- 应用ViewModel与TUI的Linux X64编译及TUI JVM编译通过；没有执行macOS、Windows或完整CLI发布构建。
- 真实应用回归的6次业务失败各产生一次原始异常日志和一次对应脚本失败日志；核对XML输出计数均为6，没有取消误报。
- Catalog回归覆盖Fork成功后的刷新失败；新增测试使用真实应用、文件系统和脚本，不新增mock。
- Native脚本测试发现临时目录可能为空或相对路径，已改为创建目录后resolve，避免切换cwd后重复拼接路径。
- Native检查使用`--no-configuration-cache`绕过已有Mosaic任务的缓存序列化不兼容；未修改全局Gradle配置。
- 混合JVM/Native构建曾出现历史索引测试5秒超时；分离后强制重跑通过，未放宽断言或超时。
- IDEA格式化及`git diff --check`通过；临时基线源码、init脚本、运行日志和新增测试目录已清理。
- 既有3个Session失败已由HEAD源码对照确认，本次未改动相关行为；不将其记为本次修复通过的测试。
