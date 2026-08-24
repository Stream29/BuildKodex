# Task Tree

- [done] 逐项审查并还原 History View
  - [done] 对比重构前后的 View 与 renderer
  - [done] 盘点 stable、pending、streaming 和 virtual View
  - [done] 记录全局空格分隔规则
  - [done] 用户逐项填写修改意见
    - [done] 审查 stable View
    - [done] 审查 pending View
    - [done] 审查 streaming View
    - [done] 审查 virtual 和 fallback View
  - [done] 汇总确认后的修改边界
  - [done] 确定轻量 header 联合类型
  - [done] 确定 command session 展示路径
  - [done] 确定全局分隔和布局修改
  - [done] 实现 History header contract
  - [done] 恢复 stable 与 pending 摘要
  - [done] 修复 command session 动态展示
  - [done] 最小化 collapsed WorkGroup 资源
  - [done] 修复 Unknown 与 composer 布局
  - [done] 清理全局中点分隔符
  - [done] 修复 MCP 完成状态映射
  - [done] 补齐 snapshot 和状态回归测试
  - [done] 完成相关模块构建验证

# Details

- 状态：`done`。
- 用户已授权执行完整修改计划。
- `wait_agent` 已确定保持通用 `Wait for an agent`，不修改工具协议。
- 当前提交：`5eaa0cd0`。
- 对比基线：`154b865d` 中重构前的 History View。
- 已按逐项审查结论完成实现。
- 已修复 MCP `isError` 被按 success 语义反向解释的问题。
- JVM 回归、IDE 构建、Linux X64 release 链接和真实历史交互验证均通过。
- 保留新的 ViewModel 状态机、延迟加载和 View 纯渲染边界。
- 用户已确定：所有原本使用 `·` 的分隔位置默认改用单个空格。
- 渲染示例使用代表性数据，`当前` 和 `计划` 分别表示现有输出与拟修改输出；`[dim]`、`[red]` 和 `[green]` 表示终端样式。
- 以下示例统一使用目标空格分隔；`当前` 只对比内容语义，不复刻现有的 `·`。
- 可展开 View 同时列出折叠态和展开态；单态 View 明确标记为不存在折叠/展开切换。
- `需恢复` 表示新轻量 header 丢失了旧 renderer 已有的信息。
- `保持` 表示当前路径仍复用旧 renderer，不计划改变展示。
- `既定变更` 表示行为来自已经确认并提交的 contract，不按旧 View 回滚。

## Stable View

### UserMessage

- 判断：保持。
- 修改计划：继续复用完整 message renderer，保留 `You` 标题、正文、换行和 elapsed。
- 渲染示例：

  ```text
  单态：You +1.2s
        Please inspect the history view.
  ```

- 用户修改意见：

### AssistantMessage

- 判断：保持。
- 修改计划：继续复用完整 message renderer，保留 phase 标题、正文、换行和 elapsed。
- 渲染示例：

  ```text
  单态：Assistant commentary +850ms
        I’ll compare the old and new renderers.
  ```

- 用户修改意见：

### DeveloperMessage

- 判断：保持。
- 修改计划：继续复用完整 message renderer，保留 dim 样式、正文和 elapsed。
- 渲染示例：

  ```text
  单态：[dim] Developer +0s
        [dim] Keep the View free of storage reads.
  ```

- 用户修改意见：

### AgentMessage

- 判断：保持。
- 修改计划：继续复用完整 inter-Agent message renderer，保留 author、recipient、正文和 elapsed。
- 渲染示例：

  ```text
  单态：planner → root +3.4s
        The comparison is complete.
  ```

- 用户修改意见：

### Reasoning

- 判断：既定变更。
- 修改计划：按已提交 contract 保持单行 `Thinking`，不恢复旧的可展开 readable detail；核验 dim 样式、elapsed 和 WorkGroup 子项行为。
- 渲染示例：

  ```text
  单态：[dim] Thinking +2.1s
  ```

- 用户修改意见：

### InvalidToolCall

- 判断：需恢复轻量 header 一致性。
- 修改计划：折叠态保留失败摘要和红色状态；展开态继续复用 Invocation 与 Error renderer。
- 渲染示例：

  ```text
  折叠：[red] > Unable to call a tool +20ms
  展开：[red] v Unable to call a tool +20ms
        Tool: exec_command
        > Invocation
        > Error
  ```

- 用户修改意见：

Unable to call a tool -> Model emitted an invalid tool call

### ServerToolSearch

- 判断：基本保持。
- 修改计划：折叠态保留 `Load tools from the server` 和 provider status；展开态继续展示原始 Arguments。
- 渲染示例：

  ```text
  折叠：> Load tools from the server +430ms
  展开：v Load tools from the server +430ms
        Tool: server_tool_search
        > Arguments
        > Tools
  ```

- 用户修改意见：

ServerToolSearchCall的arguments是不是我们遗漏了明确建模？
如果有办法能提取得出来keywords（或者叫别的名字，总之就是搜索使用的关键词），那我们就改成：
Cloud tool search: <keywords>
否则只能写成Cloud tool search了

### Hosted WebSearchCall

- 判断：需恢复。
- 修改计划：折叠态从 action 恢复 query 或具体动作摘要，例如搜索、打开页面和页内查找；展开态继续复用 hosted renderer。
- 渲染示例：

  ```text
  当前折叠：> Search the web +1.1s
  计划折叠：> Search the web: Kotlin Duration +1.1s
  计划展开：v Search the web: Kotlin Duration +1.1s
            Tool: hosted_web_search
            > Arguments
  ```

- 用户修改意见：



### Hosted ImageGenerationCall

- 判断：需恢复。
- 修改计划：折叠态恢复 revised prompt 摘要；无 prompt 时保留通用标题；展开态继续复用 hosted renderer。
- 渲染示例：

  ```text
  当前折叠：> Generate an image +8.2s
  计划折叠：> Generate an image: a terminal dashboard +8.2s
  计划展开：v Generate an image: a terminal dashboard +8.2s
            Tool: hosted_image_generation
            > Result
  ```

- 用户修改意见：

### ContextCompaction

- 判断：保持。
- 修改计划：保持单行 dim 标记和 elapsed，不引入加载或展开状态。
- 渲染示例：

  ```text
  单态：[dim] Context compacted +12ms
  ```

- 用户修改意见：

### Patch

- 判断：需恢复。
- 修改计划：折叠态恢复 `Edited n files` 或失败时的 `Editing n files`，保留状态颜色和 elapsed；展开态继续复用原 patch renderer、Changes 和分页。
- 渲染示例：

  ```text
  当前折叠：> Apply patch +95ms
  计划折叠：> Edited 2 files +95ms
  计划展开：v Edited 2 files +95ms
            Tool: apply_patch
            > Changes
  ```

- 用户修改意见：

增加一个特化：在只编辑了一个文件时，写成Edited <filename without path>。
这会需要我们写一个联合类型来表示。

### ExecCommand

- 判断：需恢复。
- 修改计划：折叠态恢复 `Run command: <single-line command>`；展开态继续复用 Arguments、Process 和 Result renderer，不把完整 result 放入 collapsed state。
- 渲染示例：

  ```text
  当前折叠：> Run command +4.3s
  计划折叠：> Run command: ./gradlew :app-view-history:jvmTest +4.3s
  计划展开：v Run command: ./gradlew :app-view-history:jvmTest +4.3s
            Tool: exec_command
            > Arguments
            > Process
            > Result
  ```

- 用户修改意见：

要保证旧行为不失效：terminal session的生命周期影响展示的颜色。
Run command: -> Run:

### WriteStdin

- 判断：需恢复可由 stable event 得出的摘要。
- 修改计划：折叠态根据 session id 和 chars 显示 wait、read 或 send-input 摘要；不向 History ViewModel 注入 `AgentShellSessionRegistry`；展开态继续由 shell session 补充源 command。
- 渲染示例：

  ```text
  当前折叠：> Write to process +5s
  计划折叠：> Wait for terminal session 42 +5s
  计划展开：v Wait for terminal session 42 +5s
            Tool: write_stdin
            > Arguments
            > Process
            > Result
  ```

- 用户修改意见：

保留之前的行为，尽力解析出真实的原始命令。
解析出原始命令的时候：
Interact with <command>
Wait for <command>
还有，Write to process -> Interact with terminal session <n>

### JsonTool

- 判断：需恢复。
- 修改计划：折叠态恢复 known-tool 人类可读名称映射；未知工具保留 qualified name；展开态继续展示 Arguments 和 Result。
- 渲染示例：

  ```text
  当前折叠：> clock.curr_time +80ms
  计划折叠：> Check the current time +80ms
  计划展开：v Check the current time +80ms
            Tool: clock.curr_time
            > Arguments
            > Result
  ```

- 用户修改意见：

### TextTool

- 判断：需恢复。
- 修改计划：折叠态恢复 known-tool 人类可读名称映射；未知工具保留 qualified name；展开态继续展示 Arguments 和文本 Result。
- 渲染示例：

  ```text
  当前折叠：> get_context_remaining +30ms
  计划折叠：> Check remaining context +30ms
  计划展开：v Check remaining context +30ms
            Tool: get_context_remaining
            > Arguments
            > Result
  ```

- 用户修改意见：

### CustomTool

- 判断：需恢复。
- 修改计划：折叠态恢复 known-tool 人类可读名称映射；未知工具保留 qualified name；展开态继续展示 Input 和 Result。
- 渲染示例：

  ```text
  当前折叠：> web.run +1.6s
  计划折叠：> Search the web +1.6s
  计划展开：v Search the web +1.6s
            Tool: web.run
            > Input
            > Result
  ```

- 用户修改意见：

适度展示工具参数，比如Search the web: <...>

### ImageGenerationTool

- 判断：需恢复。
- 修改计划：折叠态恢复 prompt 摘要；展开态继续展示 Arguments、生成结果、hint 和保存路径。
- 渲染示例：

  ```text
  当前折叠：> Generate an image +9.7s
  计划折叠：> Generate an image: a blue terminal window +9.7s
  计划展开：v Generate an image: a blue terminal window +9.7s
            Tool: image_gen.imagegen
            > Arguments
            > Result
  ```

- 用户修改意见：

### ImageViewTool

- 判断：需恢复。
- 修改计划：折叠态恢复 image path 摘要；展开态继续展示 Arguments、媒体引用和 output detail。
- 渲染示例：

  ```text
  当前折叠：> View image +120ms
  计划折叠：> View image: build/report.png +120ms
  计划展开：v View image: build/report.png +120ms
            Tool: view_image
            > Arguments
            > Result
  ```

- 用户修改意见：

### McpTool

- 判断：保持。
- 修改计划：折叠态继续显示 qualified name 和状态；展开态继续展示 Arguments 和 MCP Result。
- 渲染示例：

  ```text
  折叠：> filesystem.read_file +45ms
  展开：v filesystem.read_file +45ms
        Tool: filesystem.read_file
        > Arguments
        > Result
  ```

- 用户修改意见：

### SpawnAgent

- 判断：需恢复。
- 修改计划：折叠态恢复 task name 摘要；展开态继续展示 Arguments 和 Result。
- 渲染示例：

  ```text
  当前折叠：> Start agent +12.4s
  计划折叠：> Start agent: inspect renderer regressions +12.4s
  计划展开：v Start agent: inspect renderer regressions +12.4s
            Tool: spawn_agent
            > Arguments
            > Result
  ```

- 用户修改意见：

### SendMessage

- 判断：需恢复。
- 修改计划：折叠态恢复 target agent 摘要；展开态继续展示 Arguments 和 Result。
- 渲染示例：

  ```text
  当前折叠：> Message agent +220ms
  计划折叠：> Message agent: reviewer +220ms
  计划展开：v Message agent: reviewer +220ms
            Tool: send_message
            > Arguments
            > Result
  ```

- 用户修改意见：

### FollowupTask

- 判断：需恢复。
- 修改计划：折叠态恢复 target agent 摘要；展开态继续展示 Arguments 和 Result。
- 渲染示例：

  ```text
  当前折叠：> Continue agent task +6.8s
  计划折叠：> Continue task for agent: reviewer +6.8s
  计划展开：v Continue task for agent: reviewer +6.8s
            Tool: followup_task
            > Arguments
            > Result
  ```

- 用户修改意见：

Continue -> Resume

### WaitAgent

- 判断：基本保持。
- 修改计划：保持简洁等待摘要；展开态继续展示 Arguments 和 Result。
- 渲染示例：

  ```text
  折叠：> Wait for an agent +3s
  展开：v Wait for an agent +3s
        Tool: wait_agent
        > Arguments
        > Result
  ```

- 用户修改意见：

Wait for <agent_name>

### InterruptAgent

- 判断：需恢复。
- 修改计划：折叠态恢复 target agent 摘要；展开态继续展示 Arguments 和 Result。
- 渲染示例：

  ```text
  当前折叠：> Interrupt agent +35ms
  计划折叠：> Interrupt agent: reviewer +35ms
  计划展开：v Interrupt agent: reviewer +35ms
            Tool: interrupt_agent
            > Arguments
            > Result
  ```

- 用户修改意见：

### ListAgents

- 判断：需恢复。
- 修改计划：有 path prefix 时恢复范围摘要，否则保持通用标题；展开态继续展示 Arguments 和 Result。
- 渲染示例：

  ```text
  当前折叠：> List agents +40ms
  计划折叠：> List agents under: reviewers +40ms
  计划展开：v List agents under: reviewers +40ms
            Tool: list_agents
            > Arguments
            > Result
  ```

- 用户修改意见：

### PlanUpdate

- 判断：既定变更。
- 修改计划：保持专用 checklist View 和完整展示，保留 explanation、step status 和 elapsed，不纳入 WorkGroup。
- 渲染示例：

  ```text
  单态：• Updated Plan +15ms
          └ Restore lightweight summaries
            [x] Compare old renderer
            [>] Update headers
            [ ] Add regression tests
  ```

- 用户修改意见：

### RequestUserInput

- 判断：既定变更。
- 修改计划：保持完成后完整展示，不增加 collapsed 状态；保留首个问题摘要、问题内容、回答和 elapsed。
- 渲染示例：

  ```text
  折叠：不存在；contract 规定完成后始终完整展示。
  展示：v Ask the user: Which layout should be used? +8s
        Tool: request_user_input
        > Arguments
        > Result
  ```

- 用户修改意见：

既然不折叠就不要显示折叠图标。

### ToolSearch

- 判断：需恢复。
- 修改计划：折叠态恢复 search query 摘要；展开态继续展示 Arguments、工具列表或错误。
- 渲染示例：

  ```text
  当前折叠：> Search for tools +340ms
  计划折叠：> Search available tools: browser automation +340ms
  计划展开：v Search available tools: browser automation +340ms
            Tool: tool_search
            > Arguments
            > Result
  ```

- 用户修改意见：

### WebSearchTool

- 判断：需恢复。
- 修改计划：折叠态恢复 search、image、open、click、find、screenshot、finance、weather、sports 和 time 的具体动作摘要；展开态继续展示 Arguments 和 Result。
- 渲染示例：

  ```text
  当前折叠：> Search the web +2.5s
  计划折叠：> Check the weather +2.5s
  计划展开：v Check the weather +2.5s
            Tool: web.run
            > Arguments
            > Result
  ```

- 用户修改意见：

## Pending View

### PendingFunctionTool

- 判断：保持并做一致性核验。
- 修改计划：保留 known-tool 名称、running 状态和 Arguments；与 stable 折叠摘要共享同一套名称规则。
- 渲染示例：

  ```text
  折叠：[green] > Check the current time
  展开：[green] v Check the current time
        Tool: clock.curr_time
        > Arguments
  ```

- 用户修改意见：

### PendingCustomTool

- 判断：保持并做一致性核验。
- 修改计划：保留 known-tool 名称、running 状态和 Input；与 stable 折叠摘要共享同一套名称规则。
- 渲染示例：

  ```text
  折叠：[green] > Search the web
  展开：[green] v Search the web
        Tool: web.run
        > Input
  ```

- 用户修改意见：

### PendingPatch

- 判断：保持。
- 修改计划：继续复用 pending patch renderer，保留文件摘要、diff 展开和分页。
- 渲染示例：

  ```text
  折叠：[green] > Editing 2 files
  展开：[green] v Editing 2 files
        Tool: apply_patch
        > Changes
  ```

- 用户修改意见：

类似上面说到的，要做联合类型分支

### PendingPlanUpdate

- 判断：保持。
- 修改计划：继续使用专用 `Updating Plan` checklist View。
- 渲染示例：

  ```text
  单态：[dim] • Updating Plan
              [>] Compare renderers
              [ ] Restore headers
  ```

- 用户修改意见：

### PendingExecCommand

- 判断：保持并做一致性核验。
- 修改计划：保留 command 摘要、Arguments 和 running process；确保摘要规则与 stable ExecCommand 相同。
- 渲染示例：

  ```text
  折叠：[green] > Run command: ./gradlew test
  展开：[green] v Run command: ./gradlew test
        Tool: exec_command
        > Arguments
        > Process
  ```

- 用户修改意见：

类似上面的non-pending版本修改

### PendingWriteStdin

- 判断：保持并做一致性核验。
- 修改计划：保留 shell-session command、wait/read/send-input 摘要和 process 状态；明确它比 stable collapsed header 拥有更多 live session 信息。
- 渲染示例：

  ```text
  折叠：[green] > Read output: ./gradlew test
  展开：[green] v Read output: ./gradlew test
        Tool: write_stdin
        > Arguments
        > Process
  ```

- 用户修改意见：

类似上面的non-pending版本修改

### PendingImageGeneration

- 判断：保持并做一致性核验。
- 修改计划：保留 prompt 摘要和 Arguments；与 stable header 使用相同摘要规则。
- 渲染示例：

  ```text
  折叠：[green] > Generate an image: a terminal dashboard
  展开：[green] v Generate an image: a terminal dashboard
        Tool: image_gen.imagegen
        > Arguments
  ```

- 用户修改意见：

### PendingImageView

- 判断：保持并做一致性核验。
- 修改计划：保留 path 摘要和 Arguments；与 stable header 使用相同摘要规则。
- 渲染示例：

  ```text
  折叠：[green] > View image: build/report.png
  展开：[green] v View image: build/report.png
        Tool: view_image
        > Arguments
  ```

- 用户修改意见：

### PendingMcpTool

- 判断：保持。
- 修改计划：保留 qualified name、running 状态和 Arguments。
- 渲染示例：

  ```text
  折叠：[green] > filesystem.read_file
  展开：[green] v filesystem.read_file
        Tool: filesystem.read_file
        > Arguments
  ```

- 用户修改意见：

### PendingMultiAgent

- 判断：保持并做一致性核验。
- 修改计划：保留各 operation 的参数摘要和 Arguments；与 stable header 使用相同摘要规则。
- 渲染示例：

  ```text
  折叠：[green] > Start agent: inspect renderer regressions
  展开：[green] v Start agent: inspect renderer regressions
        Tool: spawn_agent
        > Arguments
  ```

- 用户修改意见：

### PendingRequestUserInput

- 判断：待核验 stable/pending 展开语义。
- 修改计划：保留问题摘要和 Arguments；在 planning 前固定 pending 状态是否继续允许局部展开。
- 渲染示例：

  ```text
  折叠：[green] > Ask the user: Which layout should be used?
  展开：[green] v Ask the user: Which layout should be used?
        Tool: request_user_input
        > Arguments
  ```

- 用户修改意见：

### PendingToolSearch

- 判断：保持并做一致性核验。
- 修改计划：保留 query 摘要和 Arguments；与 stable header 使用相同摘要规则。
- 渲染示例：

  ```text
  折叠：[green] > Search available tools: browser automation
  展开：[green] v Search available tools: browser automation
        Tool: tool_search
        > Arguments
  ```

- 用户修改意见：

### PendingWebSearch

- 判断：保持并做一致性核验。
- 修改计划：保留具体 web action 摘要和 Arguments；与 stable header 使用相同摘要规则。
- 渲染示例：

  ```text
  折叠：[green] > Check the weather
  展开：[green] v Check the weather
        Tool: web.run
        > Arguments
  ```

- 用户修改意见：

### PendingInvalidToolCall

- 判断：保持。
- 修改计划：保留失败颜色、Invocation 和 Error。
- 渲染示例：

  ```text
  折叠：[red] > Unable to call a tool
  展开：[red] v Unable to call a tool
        Tool: exec_command
        > Invocation
        > Error
  ```

- 用户修改意见：

### PendingServerToolSearch

- 判断：保持。
- 修改计划：保留 provider status、Arguments 和 dim 样式。
- 渲染示例：

  ```text
  折叠：[green] [dim] > Load tools from the server
  展开：[green] [dim] v Load tools from the server
        Tool: server_tool_search
        > Arguments
  ```

- 用户修改意见：

## Streaming View

### Response Started

- 判断：保持。
- 修改计划：保留 `Starting response…` 单行状态。
- 渲染示例：

  ```text
  单态：[dim] Starting response…
  ```

- 用户修改意见：

### Streaming Message

- 判断：保持。
- 修改计划：保留增量正文、换行和 content-size 通知。
- 渲染示例：

  ```text
  单态：Assistant streaming
        I’m comparing the old render
  ```

- 用户修改意见：

### Streaming AgentMessage

- 判断：保持。
- 修改计划：保留 author、recipient 和增量正文。
- 渲染示例：

  ```text
  单态：planner → root streaming
        The first regression is
  ```

- 用户修改意见：

### Streaming Reasoning

- 判断：保持。
- 修改计划：保留 streaming reasoning 的单行或增量展示，不与 stable Reasoning 的 contract 混合。
- 渲染示例：

  ```text
  单态：[dim] Thinking streaming
        [dim] Comparing collapsed headers
  ```

- 用户修改意见：

### Streaming ToolCall

- 判断：保持并做一致性核验。
- 修改计划：保留实时 tool call 摘要；核验完成交接到 pending/stable 时标题不发生无意义变化。
- 渲染示例：

  ```text
  折叠：[green] > Run a command
  展开：[green] v Run a command
        Tool: exec_command
        > Input
  ```

- 用户修改意见：

### Streaming Unknown

- 判断：保持。
- 修改计划：保留兼容性 fallback，不增加业务解释。
- 渲染示例：

  ```text
  单态：[dim] Receiving response item…
  ```

- 用户修改意见：

显示明确的Unknown，可以展开查看json

### Compacting

- 判断：保持。
- 修改计划：保留 `Compacting context…` 状态行。
- 渲染示例：

  ```text
  单态：[dim] Compacting context…
  ```

- 用户修改意见：

## Virtual 和 Fallback View

### WorkGroup

- 判断：保持。
- 修改计划：保留 `Take n actions`、collapsed/expanding/expanded 状态和 nested child renderer；核验 child 展开后使用各自完整 View。
- 渲染示例：

  ```text
  折叠：> Take 3 actions +6.4s
  展开：v Take 3 actions +6.4s
        Thinking +1.1s
        > Run command: ./gradlew test +5.2s
        > Edited 1 file +100ms
  ```

- 用户修改意见：

一定要确保未展开的情况下的资源消耗最小


### TurnTimeMarker

- 判断：保持。
- 修改计划：保留 `Worked for <duration>` 独立行和毫秒舍入。
- 渲染示例：

  ```text
  单态：---Worked for 42.315s---
  ```

- 用户修改意见：

### ActiveTurnDuration

- 判断：保持。
- 修改计划：继续显示在输入框分割线左侧，不合并进 History timeline。
- 渲染示例：

  ```text
  单态：Worked for 42s--------------------------------
  ```

- 用户修改意见：

这个不是left-right的布局，而是box那种覆盖式布局，不然那个向下滚动的图标不能正确居中

### Loading Row

- 判断：保持。
- 修改计划：item payload 未加载时保持空行；结构分页时继续显示 `Loading history…`。
- 渲染示例：

  ```text
  单态：Item loading: <blank line>
        History paging: [dim] Loading history…
  ```

- 用户修改意见：

### Error Row

- 判断：保持。
- 修改计划：item 读取失败继续显示红色 `Error`；结构加载失败继续显示带原因的 History error。
- 渲染示例：

  ```text
  单态：[red] Error
        [red] History error: unable to read stable item 54
  ```

- 用户修改意见：

### Empty Row

- 判断：保持。
- 修改计划：无 stable、pending 和 streaming 内容时继续显示 empty marker。
- 渲染示例：

  ```text
  单态：[dim] No conversation history items
  ```

- 用户修改意见：

## 全局空格分隔规则

- 判断：用户已确定，默认执行，不再保留中点分隔。
- 修改计划：实现时清理所有 production View 中作为分隔符使用的 `·`，并同步更新 snapshot 和字符串断言。
- 已盘点的 History 位置：
  - Assistant phase。
  - streaming message、AgentMessage 和 Reasoning 标题。
  - header 与 elapsed。
  - media reference 与 detail。
  - Agent name 与 status。
  - patch `remaining` 提示。
- 已盘点的其他 View 位置：
  - Session catalog 标题与 last activity。
  - Hook settings 名称与类型。
  - MCP settings server、status 和 import 类型。
  - Account identity、plan、usage window 和 limit 状态。
- 渲染示例：

  ```text
  当前：Assistant · commentary · +1.2s
  计划：Assistant commentary +1.2s

  当前：Review session title · 5m ago
  计划：Review session title 5m ago

  当前：[healthy · Healthy (2 tools)]
  计划：[healthy Healthy (2 tools)]

  当前：5h 20% · Week 30%
  计划：5h 20% Week 30%
  ```

- 用户修改意见：

Assistant commentary -> Assistant，不再区分commentary/final answer
Review session title 5m ago，但是5m ago应该dim
healthy Healthy (2 tools)这很明显有问题，直接显示Healthy (2 tools)
5h 20%/Week 30%


## 汇总后的规划原则

- 只把一行展示所需的轻量字符串和状态存入 collapsed header。
- 不让 View 重新读取 storage。
- 不让 collapsed state 持有完整 event、result、diff body 或 nested child payload。
- Expanded 状态继续复用现有完整 renderer。
- stable 与 pending 使用同一套纯摘要规则，live shell session 特有信息保留在 pending/expanded renderer。
- 所有 header 元素和 elapsed 默认使用单个空格分隔，不使用 `·`。
- 每个确认修改的 View 都增加 header 或 renderer 回归测试。
