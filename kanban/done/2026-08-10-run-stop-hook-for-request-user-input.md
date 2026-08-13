# Task Tree

- [done] 让 `request_user_input` 挂起边界执行 Stop Hook
  - [done] 复现并定位当前 Stop Hook 跳过原因
  - [done] 确认宿主输入完成与恢复链路
  - [done] 确定窄化的 Runtime 修改路线
  - [done] 扩展 Stop 候选分类
    - [done] 保留自然 `AssistantMessage` 候选
    - [done] 识别单个类型化 `request_user_input` 挂起
    - [done] 继续跳过其他 `ToolPending` 状态
  - [done] 实现挂起输入的 Stop 控制结果
    - [done] `Finish` 保留原挂起调用与 UI 表单
    - [done] 非空 `Continue` 先失败完成调用再注入续跑片段
    - [done] `Stop` 先失败完成调用再结束本次运行
  - [done] 更新 Stop contract 注释与 Hook 决策文档
  - [done] 补齐 request-user-input Stop Runtime 回归测试
  - [done] 运行目标模块测试与代码检查
  - [done] 获得用户授权并进入 executable

# Details

- 状态：实现与验证完成。
- IntelliJ IDEA 正在打开 `BuildKodex` 项目。
- `agent-runtime-decorator-turn-hook` 的 JVM、Linux X64测试与完整`check`均通过。
- IntelliJ 对三个修改的Kotlin文件未报告问题，增量构建成功；仅报告既有的
  Native cache弃用与跨平台cinterop警告。

## 当前行为

- `TurnHookRuntime` 在内层 `resume()` 返回后只接受
  `KodexAgentStateValue.AssistantMessage`，其他状态直接返回，因此
  `request_user_input` 产生的 `ToolPending` 不会执行 Stop Hook：
  `Kodex/agent-runtime/decorator/turn-hook/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/turnhook/TurnHookRuntime.kt:68-88`。
- 现有测试明确固定了全部 pending tool 均跳过 Stop Hook 的旧语义：
  `Kodex/agent-runtime/decorator/turn-hook/src/commonTest/kotlin/io/github/stream29/kodex/agentruntime/decorator/turnhook/TurnHookRuntimeTest.kt:189-232`。
- UI 只承接恰好一个 `PendingRequestUserInputToolEvent`，提交答案后先调用
  `completeToolCall`，再恢复同一个 Runtime：
  `Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt:273-289`、
  `:400-403`。

## 目标语义

- Stop 候选包括自然 `AssistantMessage`，以及恰好包含一个
  `PendingRequestUserInputToolEvent` 的 `ToolPending`。
- 普通宿主工具、混合 pending batch、取消和失败仍不触发 Stop Hook。
- request-user-input 候选沿用当前 `StopRequest` 与 `StopResult`，不新增 Hook
  事件或 Runtime contract。
- `lastAssistantMessage` 读取本轮内层运行新产生的最后一条 assistant 文本；
  只有工具调用而无文本时传 `null`。
- `StopResult.Finish` 接受输入等待边界，保持 pending 状态，使现有 UI 正常显示
  表单并提交答案。
- 带非空 fragments 的 `StopResult.Continue` 使用固定失败信息
  `request_user_input cancelled by Stop hook` 完成当前调用，再持久化 Hook
  continuation 并在同一 turn 内继续模型请求；后续 Stop 的
  `stopHookActive` 为 `true`。
- 空 fragments 的 `StopResult.Continue` 继续等价于 `Finish`，不取消表单。
- `StopResult.Stop` 使用同一固定失败信息完成当前调用，然后结束本次运行，
  使状态落到 `ToolCompleted`，不遗留无法继续的新用户输入阻塞。
- `StopResult.Stop.reason` 继续沿用现有行为，不额外写入模型历史或 UI。

## 预计修改面

- `Kodex/agent-runtime/decorator/turn-hook/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/turnhook/TurnHookRuntime.kt`
  - 引入类型化 Stop 候选分类。
  - 为 request-user-input 候选解释三类 `StopResult`。
  - 复用 `PendingToolEvent.toFailedToolEvent` 与 `completeToolCall`，不新增状态 API。
- `Kodex/agent-runtime/decorator/turn-hook/src/commonTest/kotlin/io/github/stream29/kodex/agentruntime/decorator/turnhook/TurnHookRuntimeTest.kt`
  - 保留普通 pending tool 跳过 Stop 的覆盖。
  - 新增 `Finish`、非空/空 `Continue`、`Stop`、assistant 文本和混合 batch
    边界覆盖。
- `Kodex/hook/contract/src/commonMain/kotlin/io/github/stream29/kodex/hook/contract/turn/Models.kt`
  - 将只描述“自然结束候选”的 KDoc 扩展为 Stop 候选。
- `checklist/hooks.md`
  - 将单个 `request_user_input` 等待记录为 Stop 的特殊候选。
  - 保持其他外部输入等待不属于 Stop 候选。

## 不修改

- 不修改 `request_user_input` schema、pending/stable clean model 或 UI 提交流程。
- 不修改 `KodexAgentState` contract、storage schema 或 Runtime `resume()` contract。
- 不把全部 `ToolPending` 泛化为 Stop 候选。
- Master Agent 与 subagent 继续共用同一套 Turn/Stop Hook 逻辑；本任务不引入分流。
- 不同时处理 SubagentStop、自然结束判定或其他 Stop Hook 重构。
- `agent-storage-contract` 已公开 clean models，turn-hook 模块无需增加 Gradle 依赖。

## 验证

- 运行 `./gradlew :agent-runtime-decorator-turn-hook:linuxX64Test`。
- 运行 `./gradlew :agent-runtime-decorator-turn-hook:jvmTest`。
- 运行受影响模块的 `check`，并检查 `git diff --check`。
- 使用 IntelliJ 检查修改文件。
