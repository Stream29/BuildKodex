# Task Tree

- [done] 统一 History 动作时态
  - [done] 逐项审查并定稿所有 History 文案
  - [done] 建立 projection-specific 文案入口
  - [done] 改写 stable 一级 item 文案
    - [done] 改写普通工具成功与失败文案
    - [done] 改写 command、patch 与虚拟行
    - [done] 对齐 collapsed 与 expanded 标题
  - [done] 改写 pending 进行时文案
  - [done] 改写 streaming 进行时文案
  - [done] 对齐 RequestUserInput 最终 renderer
  - [done] 更新 History 行为 checklist
  - [done] 补齐状态与渲染回归测试
  - [done] 完成相关模块验证

# Details

## Scope

- 调研阶段只完成 planning；现已获授权按本文件实施。
- 状态：已完成并通过相关 JVM module tests。
- 范围是 History timeline 的动作标题。
- Message 的 actor、section 名称、按钮命令、状态栏和 raw tool metadata 不强行改成动词。
- 保留现有颜色、elapsed、展开状态、加载状态和 timeline projection。

## Findings

- History 已明确分成 committed stable items、pending tools 和 streaming item 三个独立 projection，见
  `Kodex/app/contract/history/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/AgentHistoryViewModel.kt:74`。
- Stable collapsed header 在
  `Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/HistoryItemHeaderFactory.kt:107`
  生成，但大量文案仍是 `Run`、`Search`、`Generate` 等原形。
- Stable expanded 与 pending 又在
  `Kodex/app/view/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/CleanEventView.kt:179`
  和 `CleanEventView.kt:1463` 共享同一套原形摘要；stable 保留原形是目标行为，但 pending 必须拥有独立的进行时入口。
- Stable WorkGroup、Reasoning、command 和 patch 另有独立文案入口，见
  `Kodex/app/view/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt:280`、
  `AgentHistoryView.kt:369` 和 `AgentHistoryView.kt:579`。
- Streaming message 使用 `Assistant streaming`，Reasoning 使用 `Thinking streaming`，tool call 又复用了原形摘要，见
  `Kodex/app/view/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/StreamingRequestResponseView.kt:24`
  和 `StreamingRequestResponseView.kt:326`。
- Patch 已区分 `Editing` 与 `Edited`，但 stable success 与 failure 都需改为新的 stable 原形/failure 规则，见
  `Kodex/app/view/patch/src/commonMain/kotlin/io/github/stream29/kodex/cli/patch/PatchPresentation.kt:76`。
- PlanUpdate 当前使用 `Updating Plan` 与 `Updated Plan`；stable 标题需要改为原形，见
  `Kodex/app/view/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/PlanUpdateView.kt:15`。
- Context compaction 的 `Compacting context…` / `Context compacted` 和 turn footer 的 `Worked for` 是 timeline marker，
  不按工具动作标题强制转换。
- Codex 的本地参考实现也显式区分 active 与 completed 文案，而不是做通用英文词形变化，见
  `shared-context/codex/codex-rs/tui/src/history_cell/search.rs:5` 和
  `shared-context/codex/codex-rs/tui/src/exec_cell/render.rs:397`。

## Rules

- Pending 与 streaming projection 使用进行时。
- Committed stable success 使用动词原形。
- Projection 决定语法阶段；provider `status` 只决定颜色和结果展示。
- Streaming 收到 provider `completed` status 后，在 stable handoff 前仍保持进行时，避免标题先变原形再被替换。
- Stable failure 使用 `Failed to …`，不能用进行时，也不能声称未发生的动作已经成功。
- Stable command 已实际启动但以非零状态结束时仍显示 `Run`；红色状态表达失败。
- Stable command tool call 已提交但后台进程仍运行时仍显示 `Run`、`Interact` 或 `Wait`；live process 颜色与详情独立表达进程状态。
- `PendingInvalidToolCall` 已是 terminal failure，保留 `Model emitted an invalid tool call`，不因它位于 pending 集合而改成进行时。
- Known action 使用显式文案，不实现自动追加 `-ing` 或 `-ed` 的词形算法。
- Unknown 与 MCP 使用 qualified name：
  - ongoing：`Running <qualified-name>`
  - stable success：`<qualified-name>`
  - stable failure：`Failed to run <qualified-name>`
- Noun、actor identity、timeline marker 与 terminal fallback 不需要伪造时态，例如 `Assistant`、
  `Agent <author> → <recipient>`、`Context compacted`、`Unknown` 和 `Error`。

## 逐项审查

- 每项都列出当前文案、计划文案和可直接填写的修改意见。
- `保持` 也保留评论空间，避免遗漏不需要时态变化的标题。

### Stable View

#### UserMessage

- 判断：改动。
- 当前：`You`。
- 计划：`User`。
- 说明：这是 actor header，不是动作标题。
- 用户修改意见：

You -> User

#### AssistantMessage

- 判断：改动。
- 当前：`Assistant` 或 `Assistant commentary` / `Assistant final answer`。
- 计划：`Assistant`。
- 说明：这是 actor 与 message phase，不改成 `Responded`。
- 用户修改意见：

Assistant，不要区分commentary/final answer

#### DeveloperMessage

- 判断：保持。
- 当前与计划：`Developer`。
- 说明：这是 actor header。
- 用户修改意见：

#### AgentMessage

- 判断：改动。
- 当前：`<author> → <recipient>`。
- 计划：`Agent <author> → <recipient>`。
- 说明：stable message 直接显示发送双方；`Agent` 是类型提示，不额外增加动作动词。
- 用户修改意见：

Agent <author> → <recipient>
在前面加上Agent一词作为提示

#### Reasoning

- 判断：改动。
- 当前：`Thinking`。
- 计划：`Think`。
- 示例：`[dim] Think +2.1s`。
- 用户修改意见：

#### InvalidToolCall

- 判断：保持。
- 当前与计划：`Model emitted an invalid tool call`。
- 说明：虽然它可暂存于 pending 集合，但语义已经是 terminal failure，保持该错误描述。
- 用户修改意见：

#### ServerToolSearch

- 判断：改动。
- 当前：`Cloud tool search: <paths>` 或 `Cloud tool search`。
- 计划成功：`Search cloud tools: <paths>`。
- 计划失败：`Failed to search cloud tools: <paths>`。
- 用户修改意见：

#### Hosted WebSearchCall

- 判断：改动。
- 当前：按 action 显示 `Search the web`、`Open a web page` 或 `Find text on a web page`。
- 计划：复用下方每个 Web action 的 stable 原形/failure 文案。
- 用户修改意见：

#### Hosted ImageGenerationCall

- 判断：改动。
- 当前：`Generate an image: <revised-prompt>`。
- 计划成功：`Generate an image: <revised-prompt>`。
- 计划失败：`Failed to generate an image: <revised-prompt>`。
- 用户修改意见：

#### ContextCompaction

- 判断：保持。
- 当前与计划：`Context compacted`。
- 用户修改意见：

#### Patch

- 判断：改动。
- 当前成功：`Edited <target>`。
- 当前失败：`Editing <target>`。
- 计划 pending：`Editing <target>`。
- 计划 stable success：`Edit <target>`。
- 计划 stable failure：`Failed to edit <target>`。
- 示例：`> Failed to edit Main.kt +95ms`。
- 用户修改意见：

#### ExecCommand

- 判断：改动。
- 当前 stable：`Run: <command>`。
- 当前 pending：`Run: <command>`。
- 当前 streaming local shell：`Run a command`。
- 计划 pending：`Running: <command>`。
- 计划 streaming：`Running a command`。
- 计划 stable success 或已实际执行的非零 exit：`Run: <command>`。
- 计划 stable pre-execution failure：`Failed to run: <command>`。
- 说明：stable tool call 已提交后，即使关联后台 process 仍绿色运行，标题仍保持 `Run`。
- 用户修改意见：

#### WriteStdin with input

- 判断：改动。
- 当前 stable 与 pending：`Interact with <command/session>`。
- 计划 pending：`Interacting with <command/session>`。
- 计划 stable success：`Interact with <command/session>`。
- 计划 stable failure：`Failed to interact with <command/session>`。
- 用户修改意见：

#### WriteStdin poll

- 判断：改动。
- 当前 stable 与 pending：`Wait for <command/session>`。
- 计划 pending：`Waiting for <command/session>`。
- 计划 stable success：`Wait for <command/session>`。
- 计划 stable failure：`Failed to wait for <command/session>`。
- 用户修改意见：

#### JsonTool

- 判断：仅 failure 改动。
- 当前：known tool 使用原形摘要，unknown 使用 qualified name。
- 计划：stable success 保持当前摘要；stable failure 仅使用对应 action 的 `Failed to …` 文案。
- 说明：不为 JsonTool 额外附加 `Ran` 等冗余动作。
- 用户修改意见：

不需要画蛇添足

#### TextTool

- 判断：仅 failure 改动。
- 当前：known tool 使用原形摘要，unknown 使用 qualified name。
- 计划：stable success 保持当前摘要；stable failure 仅使用对应 action 的 `Failed to …` 文案。
- 说明：不为 TextTool 额外附加 `Ran` 等冗余动作。
- 用户修改意见：

不需要画蛇添足

#### CustomTool

- 判断：改动。
- 当前：known tool 使用原形摘要；`web.run` 解析后使用 Web action 原形摘要。
- 计划：known tool 采用下方对应 action 的 stable success/failure 文案；`web.run` 采用下方 Web action 文案。
- 用户修改意见：

#### ImageGenerationTool

- 判断：改动。
- 当前：`Generate an image: <prompt>`。
- 计划 success：`Generate an image: <prompt>`。
- 计划 failure：`Failed to generate an image: <prompt>`。
- 用户修改意见：

#### ImageViewTool

- 判断：改动。
- 当前：`View image: <path>`。
- 计划 success：`View image: <path>`。
- 计划 failure：`Failed to view image: <path>`。
- 用户修改意见：

#### McpTool

- 判断：仅 failure 改动。
- 当前：`<qualified-name>`。
- 计划 success：`<qualified-name>`。
- 计划 failure：`Failed to run <qualified-name>`。
- 用户修改意见：

#### MultiAgentTool

- 判断：改动。
- 当前：所有 operation 使用原形摘要。
- 计划：stable success 保持下方每个 multi-agent operation 的原形摘要；failure 使用对应 failure 文案。
- 用户修改意见：

#### PlanUpdate

- 判断：改动。
- 当前：`Updated Plan`。
- 计划：`Update Plan`。
- 说明：失败的 generic `update_plan` 不走此 stable variant，见下方 `update_plan` function fallback。
- 用户修改意见：

#### RequestUserInput

- 判断：局部改动。
- 当前有 questions：直接显示只读问题与用户答案。
- 当前空 questions fallback：`Ask the user`。
- 当前失败行：`Unable to submit: <message>`。
- 计划有 questions：保持无额外动作标题的只读 form。
- 计划空 questions fallback：`Ask the user`。
- 计划失败行：`Failed to submit: <message>`。
- 说明：布局由 `kanban/done/2026-08-24-fix-request-user-input-history-view.md` 定稿，本任务只对齐文案。
- 用户修改意见：

#### ToolSearch

- 判断：改动。
- 当前：`Search available tools: <query>`。
- 计划 success：`Search available tools: <query>`。
- 计划 invalid arguments：`Failed to search available tools: <query>`。
- 用户修改意见：

#### WebSearchTool

- 判断：改动。
- 当前：按 `SearchCommands` 使用 Web action 原形摘要。
- 计划：复用下方每个 Web action 的 stable success/failure 文案。
- 用户修改意见：

### Virtual and fallback View

#### WorkGroup

- 判断：保持。
- 当前与计划：`Take <n> actions`。
- 示例：`> Take 3 actions +4.2s`。
- 用户修改意见：

#### TurnTimeMarker

- 判断：保持。
- 当前与计划：`---Worked for <duration>---`。
- 用户修改意见：

#### Initial item loading

- 判断：保持。
- 当前与计划：一行空白。
- 说明：这不是动作标题。
- 用户修改意见：

#### Item load failure

- 判断：保持。
- 当前与计划：红色 `Error`。
- 说明：这是 renderer loading fallback，不引入动作时态。
- 用户修改意见：

#### History structural failure

- 判断：保持。
- 当前与计划：`History error: <message>`。
- 用户修改意见：

### Known function-tool actions

#### `get_context_remaining`

- 适用来源：JsonTool、TextTool、CustomTool、PendingFunctionTool、Streaming FunctionCall。
- 当前：`Check remaining context`。
- 计划 pending/streaming：`Checking remaining context`。
- 计划 stable success：`Check remaining context`。
- 计划 stable failure：`Failed to check remaining context`。
- 用户修改意见：

#### `clock.curr_time`

- 适用来源：JsonTool、TextTool、CustomTool、PendingFunctionTool、Streaming FunctionCall。
- 当前：`Check the current time`。
- 计划 pending/streaming：`Checking the current time`。
- 计划 stable success：`Check the current time`。
- 计划 stable failure：`Failed to check the current time`。
- 用户修改意见：

#### generic `exec_command` and `shell.run`

- 适用来源：fallback tool event；专用 command item 仍以 `ExecCommand` 项为准。
- 当前：`Run a command`。
- 计划 pending/streaming：`Running a command`。
- 计划 stable success：`Run a command`。
- 计划 stable failure：`Failed to run a command`。
- 用户修改意见：

#### generic `write_stdin`

- 适用来源：fallback tool event；专用 command item 仍以两个 `WriteStdin` 项为准。
- 当前：`Interact with a terminal session`。
- 计划 pending/streaming：`Interacting with a terminal session`。
- 计划 stable success：`Interact with a terminal session`。
- 计划 stable failure：`Failed to interact with a terminal session`。
- 用户修改意见：

#### generic `view_image`

- 适用来源：fallback tool event；专用 ImageViewTool 仍优先展示 path。
- 当前：`View an image`。
- 计划 pending/streaming：`Viewing an image`。
- 计划 stable success：`View an image`。
- 计划 stable failure：`Failed to view an image`。
- 用户修改意见：

#### generic `image_gen.imagegen` and `hosted_image_generation`

- 适用来源：fallback function/custom event 与 streaming FunctionCall。
- 当前：`Generate an image`。
- 计划 pending/streaming：`Generating an image`。
- 计划 stable success：`Generate an image`。
- 计划 stable failure：`Failed to generate an image`。
- 用户修改意见：

#### generic `request_user_input`

- 适用来源：fallback function/custom event。
- 当前：`Ask the user for input`。
- 计划 pending：`Waiting for user input`。
- 计划 stable success：`Ask the user for input`。
- 计划 stable failure：`Failed to collect user input`。
- 用户修改意见：

#### `tool_search` and `server_tool_search`

- 适用来源：fallback function/custom event。
- 当前：`Search available tools`。
- 计划 pending/streaming：`Searching available tools`。
- 计划 stable success：`Search available tools`。
- 计划 stable failure：`Failed to search available tools`。
- 用户修改意见：

#### generic `web.run` and `hosted_web_search`

- 适用来源：未解析 action 的 JsonTool、TextTool、fallback function/custom event 与 streaming FunctionCall/CustomToolCall。
- 当前：`Search the web`。
- 计划 pending/streaming：`Searching the web`。
- 计划 stable success：`Search the web`。
- 计划 stable failure：`Failed to search the web`。
- 用户修改意见：

#### generic `update_plan`

- 适用来源：failed PendingPlanUpdate 持久化成的 StableTextToolEvent。
- 当前：`Update the plan`。
- 计划 pending：`Updating the plan`。
- 计划 stable success：`Update the plan`。
- 计划 stable failure：`Failed to update the plan`。
- 用户修改意见：

#### Unknown function or custom tool

- 适用来源：JsonTool、TextTool、CustomTool、PendingFunctionTool、PendingCustomTool、Streaming FunctionCall、
  Streaming CustomToolCall。
- 当前：`<qualified-name>`。
- 计划 pending/streaming：`Running <qualified-name>`。
- 计划 stable success：`<qualified-name>`。
- 计划 stable failure：`Failed to run <qualified-name>`。
- 用户修改意见：

### Web actions

- 以下 detail 文案适用于已解码 `SearchCommands` 的 stable 和 pending WebSearchTool，以及 stable/pending
  CustomTool `web.run`。
- Streaming FunctionCall 与 CustomToolCall 不解析 partial raw input，统一使用上方 generic `web.run` 文案。

#### `search_query`

- 适用来源：WebSearchTool、PendingWebSearchTool、CustomTool `web.run`、HostedWebSearchCall。
- 当前：`Search the web: <query>`。
- 计划 pending：`Searching the web: <query>`。
- 计划 stable success：`Search the web: <query>`。
- 计划 stable failure：`Failed to search the web: <query>`。
- 用户修改意见：

#### `image_query`

- 适用来源：WebSearchTool、PendingWebSearchTool、CustomTool `web.run`。
- 当前：`Search images: <query>`。
- 计划 pending：`Searching images: <query>`。
- 计划 stable success：`Search images: <query>`。
- 计划 stable failure：`Failed to search images: <query>`。
- 用户修改意见：

#### `open`

- 适用来源：WebSearchTool、PendingWebSearchTool、CustomTool `web.run`、HostedWebSearchCall。
- 当前：`Open a web page`。
- 计划 pending：`Opening a web page`。
- 计划 stable success：`Open a web page`。
- 计划 stable failure：`Failed to open a web page`。
- 用户修改意见：

#### `click`

- 适用来源：WebSearchTool、PendingWebSearchTool、CustomTool `web.run`。
- 当前：`Follow a web link`。
- 计划 pending：`Following a web link`。
- 计划 stable success：`Follow a web link`。
- 计划 stable failure：`Failed to follow a web link`。
- 用户修改意见：

#### `find`

- 适用来源：WebSearchTool、PendingWebSearchTool、CustomTool `web.run`、HostedWebSearchCall。
- 当前：`Find text on a web page`。
- 计划 pending：`Searching a web page for text`。
- 计划 stable success：`Search a web page for text`。
- 计划 stable failure：`Failed to search a web page for text`。
- 说明：不用 `Found`，避免错误声称命中。
- 用户修改意见：

#### `screenshot`

- 适用来源：WebSearchTool、PendingWebSearchTool、CustomTool `web.run`。
- 当前：`Capture a web page`。
- 计划 pending：`Capturing a web page`。
- 计划 stable success：`Capture a web page`。
- 计划 stable failure：`Failed to capture a web page`。
- 用户修改意见：

#### `finance`

- 适用来源：WebSearchTool、PendingWebSearchTool、CustomTool `web.run`。
- 当前：`Look up market data`。
- 计划 pending：`Looking up market data`。
- 计划 stable success：`Look up market data`。
- 计划 stable failure：`Failed to look up market data`。
- 用户修改意见：

#### `weather`

- 适用来源：WebSearchTool、PendingWebSearchTool、CustomTool `web.run`。
- 当前：`Check the weather`。
- 计划 pending：`Checking the weather`。
- 计划 stable success：`Check the weather`。
- 计划 stable failure：`Failed to check the weather`。
- 用户修改意见：

#### `sports`

- 适用来源：WebSearchTool、PendingWebSearchTool、CustomTool `web.run`。
- 当前：`Check sports information`。
- 计划 pending：`Checking sports information`。
- 计划 stable success：`Check sports information`。
- 计划 stable failure：`Failed to check sports information`。
- 用户修改意见：

#### `time`

- 适用来源：WebSearchTool、PendingWebSearchTool、CustomTool `web.run`。
- 当前：`Check the time`。
- 计划 pending：`Checking the time`。
- 计划 stable success：`Check the time`。
- 计划 stable failure：`Failed to check the time`。
- 用户修改意见：

#### Web fallback

- 适用来源：没有可识别 command 的 WebSearchTool、PendingWebSearchTool 与 CustomTool `web.run`。
- 当前：`Use web search`。
- 计划 pending：`Using web search`。
- 计划 stable success：`Use web search`。
- 计划 stable failure：`Failed to use web search`。
- 用户修改意见：

### Multi-agent actions

#### `spawn_agent`

- 当前：`Start agent: <task>`。
- 计划 pending：`Starting agent: <task>`。
- 计划 stable success：`Start agent: <task>`。
- 计划 stable failure：`Failed to start agent: <task>`。
- 计划 streaming FunctionCall：`Starting an agent`。
- 用户修改意见：

#### `send_message`

- 当前：`Message agent: <target>`。
- 计划 pending：`Sending message to agent: <target>`。
- 计划 stable success：`Message agent: <target>`。
- 计划 stable failure：`Failed to send message to agent: <target>`。
- 计划 streaming FunctionCall：`Sending a message to an agent`。
- 用户修改意见：

#### `followup_task`

- 当前：`Resume task for agent: <target>`。
- 计划 pending：`Resuming task for agent: <target>`。
- 计划 stable success：`Resume task for agent: <target>`。
- 计划 stable failure：`Failed to resume task for agent: <target>`。
- 计划 streaming FunctionCall：`Resuming an agent task`。
- 用户修改意见：

#### `wait_agent`

- 当前：`Wait for an agent`。
- 计划 pending/streaming：`Waiting for an agent`。
- 计划 stable success：`Wait for an agent`。
- 计划 stable failure：`Failed to wait for an agent`。
- 用户修改意见：

#### `interrupt_agent`

- 当前：`Interrupt agent: <target>`。
- 计划 pending：`Interrupting agent: <target>`。
- 计划 stable success：`Interrupt agent: <target>`。
- 计划 stable failure：`Failed to interrupt agent: <target>`。
- 计划 streaming FunctionCall：`Interrupting an agent`。
- 用户修改意见：

#### `list_agents`

- 当前：`List agents` 或 `List agents under: <prefix>`。
- 计划 pending：`Listing agents` 或 `Listing agents under: <prefix>`。
- 计划 stable success：`List agents` 或 `List agents under: <prefix>`。
- 计划 stable failure：`Failed to list agents` 或 `Failed to list agents under: <prefix>`。
- 计划 streaming FunctionCall：`Listing agents`。
- 用户修改意见：

### Pending View

#### PendingFunctionTool and PendingCustomTool

- 判断：改动。
- 当前：调用 shared base-form helper，例如 `Run a command`、`Search the web` 或 raw qualified name。
- 计划：调用上方对应 action 的 pending 文案。
- 说明：不读取额外数据，不增加 pending item state。
- 用户修改意见：

#### PendingPatchTool

- 判断：保持。
- 当前与计划：`Editing <target>`。
- 用户修改意见：

#### PendingPlanUpdate

- 判断：保持。
- 当前与计划：`Updating Plan`。
- 用户修改意见：

#### PendingCommandExecutionTool

- 判断：改动。
- 当前：`Run`、`Interact with` 或 `Wait for`。
- 计划：复用上方 command 的 `Running`、`Interacting` 或 `Waiting` 文案。
- 用户修改意见：

#### PendingMultiAgentTool

- 判断：改动。
- 当前：多 agent action 的原形摘要。
- 计划：复用上方每个 multi-agent action 的 pending 文案。
- 用户修改意见：

#### PendingImageGenerationTool

- 判断：改动。
- 当前：`Generate an image: <prompt>`。
- 计划：`Generating an image: <prompt>`。
- 用户修改意见：

#### PendingImageViewTool

- 判断：改动。
- 当前：`View image: <path>`。
- 计划：`Viewing image: <path>`。
- 用户修改意见：

#### PendingMcpTool

- 判断：改动。
- 当前：`<qualified-name>`。
- 计划：`Running <qualified-name>`。
- 用户修改意见：

#### PendingRequestUserInputTool

- 判断：改动。
- 当前：`Ask the user: <question>`。
- 计划：`Waiting for user input: <question>`。
- 用户修改意见：

#### PendingToolSearch

- 判断：改动。
- 当前：`Search available tools: <query>`。
- 计划：`Searching available tools: <query>`。
- 用户修改意见：

#### PendingWebSearchTool

- 判断：改动。
- 当前：Web action 原形摘要。
- 计划：复用上方每个 Web action 的 pending 文案。
- 用户修改意见：

#### PendingInvalidToolCall

- 判断：保持。
- 当前与计划：`Model emitted an invalid tool call`。
- 说明：它已经失败，不显示伪进行时。
- 用户修改意见：

#### PendingServerToolSearch

- 判断：改动。
- 当前：`Cloud tool search: <paths>`。
- 计划：`Searching cloud tools: <paths>`。
- 说明：即使 provider status 已为 `completed`，只要仍在 pending projection 就保持进行时。
- 用户修改意见：

#### RequestUserInputPanel

- 判断：保持。
- 当前与计划：直接显示问题、选项和 `Submit`；提交中显示 `Submitting…`。
- 说明：`Input requested` 已被移除，不重新添加。
- 用户修改意见：

### Streaming View

#### `HistoryStreamingItem.Started`

- 判断：保持。
- 当前与计划：`Starting response…`。
- 用户修改意见：

#### Streaming message

- 判断：改动。
- 当前：`Assistant streaming`。
- 计划：`Assistant responding`。
- 用户修改意见：

#### Streaming AgentMessage

- 判断：改动。
- 当前：`<author> → <recipient> streaming`，缺少双方时为 `Agent message streaming`。
- 计划：`Sending message: <author> → <recipient>`，缺少双方时为 `Sending an agent message`。
- 用户修改意见：

#### Streaming Reasoning

- 判断：改动。
- 当前：`Thinking streaming`。
- 计划：`Thinking`。
- 用户修改意见：

#### Streaming FunctionCall

- 判断：改动。
- 当前：shared base-form helper，例如 `Run a command` 或 `Search the web`。
- 计划：复用上方 known function 的 generic streaming 文案，不解析 partial arguments。
- 说明：`web.run` 使用 `Searching the web`；multi-agent action 使用不含 task/target 的 ongoing 文案。
- 用户修改意见：

#### Streaming CustomToolCall

- 判断：改动。
- 当前：shared base-form helper，例如 `Run a command` 或 raw qualified name。
- 计划：复用上方 known function 的 generic streaming 文案，不解析 partial input。
- 说明：`web.run` 使用 `Searching the web`。
- 用户修改意见：

#### Streaming ClientToolSearchCall

- 判断：改动。
- 当前：`Search available tools`。
- 计划：`Searching available tools`。
- 用户修改意见：

#### Streaming ServerToolSearchCall

- 判断：改动。
- 当前：`Cloud tool search: <paths>`。
- 计划：`Searching cloud tools: <paths>`。
- 用户修改意见：

#### Streaming LocalShellCall

- 判断：改动。
- 当前：`Run a command`。
- 计划：`Running a command`。
- 用户修改意见：

#### Streaming WebSearchCall

- 判断：改动。
- 当前：`Search the web`。
- 计划：`Searching the web`。
- 说明：WebSearchCall 的 provider status 改为 `completed` 时，在 stable handoff 前仍显示该进行时标题。
- 用户修改意见：

#### Streaming ImageGenerationCall

- 判断：改动。
- 当前：`Generate an image`。
- 计划：`Generating an image`。
- 用户修改意见：

#### Streaming FunctionCallOutput

- 判断：改动。
- 当前：`Receive a tool result`。
- 计划：`Receiving a tool result`。
- 用户修改意见：

#### Streaming McpToolCallOutput

- 判断：改动。
- 当前：`Receive an MCP tool result`。
- 计划：`Receiving an MCP tool result`。
- 用户修改意见：

#### Streaming CustomToolCallOutput

- 判断：改动。
- 当前：`Receive a tool result`。
- 计划：`Receiving a tool result`。
- 用户修改意见：

#### Streaming ClientToolSearchOutput

- 判断：改动。
- 当前：`Receive available tools`。
- 计划：`Receiving available tools`。
- 用户修改意见：

#### Streaming ServerToolSearchOutput

- 判断：改动。
- 当前：`Receive available tools`。
- 计划：`Receiving available tools`。
- 用户修改意见：

#### Streaming AdditionalTools

- 判断：改动。
- 当前：`Update the available tool catalog`。
- 计划：`Updating the available tool catalog`。
- 用户修改意见：

#### Streaming generic ToolCall fallback

- 判断：改动。
- 当前：`Run a tool`。
- 计划：`Running a tool`。
- 用户修改意见：

#### Streaming Unknown

- 判断：保持。
- 当前与计划：`Unknown`。
- 说明：无可靠语义时保留 noun fallback。
- 用户修改意见：

#### `HistoryStreamingItem.Compacting`

- 判断：保持。
- 当前与计划：`Compacting context…`。
- 用户修改意见：

## Implementation route

- 不修改 `AgentHistoryViewModel` 的三段 projection，不新增 item VM、StateFlow、loading job 或 storage read。
- 不把英文文案或通用 action union 加入 public History contract。
- Stable collapsed header 继续由 `HistoryItemHeaderFactory` 生成，使用显式的 stable 原形/failure 文案。
- Stable expanded tool 标题直接复用对应 item state 已持有的 lightweight header；不再独立推导另一份标题。
- Command 与 patch 的现有 typed header 已足以选择 stable 原形/failure 文案，不增加字段。
- Pending 与 streaming 在 `app-view-history` 内共享 ongoing formatter；不能再调用无 phase 的 base-form helper。
- 将 wording helper 从 `CleanEventView.kt` 和 `HistoryItemHeaderFactory.kt` 的大段 renderer 逻辑中拆到按 command、web、
  multi-agent 和 generic tool 分类的小文件；不建立新的 Gradle module。
- Status normalization 与 tense formatting 分离；不能通过 `status == completed` 推导 stable 文案。
- Completed RequestUserInput 的布局继续由
  `kanban/done/2026-08-24-fix-request-user-input-history-view.md` 负责；本任务只在其最终 renderer 上对齐 fallback 与失败文案。
- 实现完成后，更新 `checklist/cli-view-model-state.md`，记录 stable 使用原形、pending/streaming 使用进行时、
  failure 使用 `Failed to …` 的规则。

## Validation

- `HistoryItemHeaderFactoryTest` 覆盖 known、unknown、MCP、web、multi-agent 的 stable success/failure 文案。
- `CleanEventViewTest` 覆盖 pending 进行时以及 stable collapsed/expanded 标题一致性。
- `StreamingRequestResponseViewTest` 覆盖 message、AgentMessage、Reasoning、tool input 与 streaming-only output 文案。
- 增加 provider status 已为 `completed`、但 streaming tail 尚未 handoff 时仍显示进行时的回归用例。
- `AgentHistoryEntryInteractionTest` 覆盖 `Think`、`Take n actions`、command 展开以及 live shell 颜色不改变原形。
- `PatchPresentationTest` 与 `PatchToolEventViewTest` 覆盖 `Editing`、`Edit` 和 `Failed to edit`。
- RequestUserInput renderer 完成后，覆盖无冗余动作标题、`Ask the user` fallback 和 `Failed to submit`。
- 使用具体 clean event fixture，不新增 mock。
- 运行 `app-viewmodel-history`、`app-view-history`、`app-view-patch` 的测试与格式检查。
- 构建 CLI release executable，并用一段包含 streaming → pending → stable、失败工具、WorkGroup 和后台 command 的 session
  做端到端快照核对。
