# 上下文注入的类型、生命周期与持久化边界

## 目的

本文件分析 Codex Rust 中所有主要的模型可见上下文来源，并据此区分：

- 哪些内容是对话历史的一部分，必须持久化。
- 哪些内容是当前请求的运行时视图，应动态生成。
- 哪些内容是配置状态，应持久化设置值而不是提示文本。
- 哪些内容只服务于审计或复现，应脱离模型 history 保存。

这里的“持久化”特指写入可供后续模型请求和 compaction 使用的 `AgentStorage` history。把内容写入日志、timeline 或审计快照不等于把它放进模型 history。

## 先分清四条数据通道

Rust Codex 并不存在一个统一的“context injection”通道。一次模型请求至少有四种不同性质的数据：

```text
Responses request
  = request configuration
  + transient request context
  + durable model history
  + protocol-level transport state
```

### Request configuration

这是 Responses API 的顶层请求配置，不是 `ResponseItem` history：

- base instructions
- model、reasoning、service tier
- tool specs、dynamic tools、output schema
- provider/auth/transport options

Rust 的 `Prompt` 同时区分 `input`、`tools` 与 `base_instructions`；正常 Responses 路径把后两者放到 wire request 的顶层字段。`SessionMeta` 还会保存 base instructions 和 dynamic tools，以便恢复会话配置。

例子：同一段历史在 `gpt-5-codex` 与另一个模型上继续，模型基础指令和工具规格可能不同。这不是“模型曾经听到一条用户消息”，而是本次请求采用的协议配置。

结论：配置需要版本化保存，供 resume、fork 和审计恢复；不要把它伪装成普通历史消息。

### Transient request context

这是调用时从当前宿主环境得出的内容。它可能以 developer 或 contextual-user message 的形式进入本次模型输入，但本质不是对话事实。

例子：

- 当前有效的 `AGENTS.md`。
- cwd、shell、文件系统权限、网络状态、日期、时区。
- 当前可用的 Apps、plugins、skills 目录。
- 当前 token budget、当前时间提醒、上下文窗口提示。

结论：默认动态计算，并在一次实际 LLM 请求开始时固定快照。不要在 SSE 流期间重新读取并改变同一个请求的前缀。

### Durable model history

这是模型已经见过、且后续继续同一逻辑工作时仍必须保留的事实。它包括用户、assistant、tool 的正常对话内容，也包括少数“由宿主生成但已成为事实”的上下文。

例子：

- 用户显式选择后展开的完整 `SKILL.md`。
- hook 返回的额外指令文本。
- 已批准的命令前缀或网络规则。
- 已中断 turn 的标记。
- subagent 的状态通知。

结论：必须保存精确内容或等价的不可变快照。不能在恢复时重新运行 hook、重新读取已经变更的 skill 文件，或从当前权限配置反推过去的批准结果。

### Audit and replay metadata

有些内容不应成为模型 history，但仍可能需要记录，以支持调试、审计或精确复现。

例子：

- 某次请求使用的 `AGENTS.md` 来源、路径、hash、正文快照。
- 当时的环境快照与权限 profile。
- 当前时间提醒实际发送的时间。
- 运行时 provider 的版本和扩展输出来源。

结论：这是单独的 snapshot 或 timeline 数据，不应通过伪造 `ResponseItem` 进入 compaction checkpoint。

## Rust Codex 的实际分层

Rust 的实现将不同类型混合在一个 `ResponseItem` 历史里，以支持增量 context diff 和 resume；但它同时维护了 `SessionMeta`、`TurnContextItem` 和 `WorldStateItem` 等辅助状态。

- `SessionMeta` 保存 session 级 base instructions 与 dynamic tools。
- `TurnContextItem` 保存每个真实 user turn 的有效模型、权限、环境、mode 等 baseline。
- `WorldStateItem` 保存上下文 diff 比较所需的 snapshot，不是直接发送给模型的文本。
- 实际模型可见文本大多仍会记录为 `ResponseItem`。

`build_settings_update_items` 的源码明确承认：它尚未覆盖 `build_initial_context` 产生的全部模型可见输入，resume/fork 的完全确定性仍依赖补充持久化或 replay event。这说明不能把 Rust 当前“全部写 history”的结果误认为唯一正确的抽象。

相关位置：

- `shared-context/codex/codex-rs/core/src/session/mod.rs:3187`
- `shared-context/codex/codex-rs/core/src/context_manager/updates.rs:240`
- `shared-context/codex/codex-rs/protocol/src/protocol.rs:3017`
- `shared-context/codex/codex-rs/protocol/src/protocol.rs:3139`

## 类型一：基础模型协议与请求配置

### 内容

- model 的 base instructions。
- tool specs、namespace、dynamic tools、output schema。
- 模型选择、reasoning effort、service tier、provider 选择。
- compact prompt 和 API transport 选项。

### Rust 行为

session 启动时，base instructions 的优先级是 config override、历史 `SessionMeta`、当前模型默认 instructions。dynamic tools 也在 thread 启动时确定，并保存于 session metadata。

正常 Responses 请求中，`base_instructions` 与 `tools` 是 request 字段；它们不与对话 `input` 混为同一个 `ResponseItem` 列表。只有兼容路径才可能把这类信息折叠成 developer message。

### 例子

用户在已经有十轮对话的线程中启用一个新的 MCP tool：

```text
历史：用户要求修复 Kotlin 编译错误。
当前配置：新增 request_user_input 工具。
```

模型接下来应能调用新工具，但不需要把“新增工具”当成一条用户历史消息。恢复线程时，如果目标是复刻当时可用工具，应恢复工具配置快照；如果目标是按当前 runtime 继续，应使用当前工具集。

### 持久化判断

- 保存配置值和版本：需要。
- 作为 `HistoryItem` 追加：不需要。
- 作为请求顶层字段发送：需要。

## 类型二：当前宿主环境

这一类信息回答的是“模型现在运行在哪里、能做什么”。它不是用户对话本身。

### AGENTS.md

Rust 先合并 home instructions 与从 project root 到 cwd 的项目 instructions，再渲染成带 `# AGENTS.md instructions` 和 `<INSTRUCTIONS>` 标记的 contextual user fragment。

Rust 的 `WorldState` 对 AGENTS 保存 `{ directory, text }` snapshot：

- 首次出现时发送完整 instructions。
- 内容变更时发送“新 instructions 替换旧 instructions”的文本和完整新正文。
- 消失时发送移除通知。

这是为“保留旧 prompt history，只发送差量更新”服务的设计。

例子：

```text
请求 1：项目 AGENTS.md 要求使用 Gradle Kotlin DSL。
用户修改 AGENTS.md：额外要求所有测试使用 testBalloon。
请求 2：模型必须看到新规则，而不是继续依赖旧规则。
```

若 Kotlin 侧每次请求都构造完整 prompt，AGENTS 应属于 transient request context：请求 1 固定读取到的版本，下一次模型请求再读取当前版本。它不应写入历史，也不应被 compaction 保留下来。

若需要可审计地回答“这次请求看到了什么 AGENTS”，单独记录来源路径、有效文本 hash 和可选正文快照。不要为了审计把 AGENTS 文本变成 history。

相关位置：

- `shared-context/codex/codex-rs/core/src/agents_md.rs:48`
- `shared-context/codex/codex-rs/core/src/context/user_instructions.rs:9`
- `shared-context/codex/codex-rs/core/src/context/world_state/agents_md.rs:19`

### Environment context

Rust 的 `EnvironmentsState` 可渲染：

- 环境 id、cwd、状态与 shell。
- 当前日期和时区。
- network 与 filesystem permission context。
- 已运行的 subagent 信息。

例子：

```text
请求 1：cwd 是 /repo，网络被禁止，workspace 只允许 /repo。
用户切换到 /repo/subproject，并允许访问 api.example.com。
请求 2：模型应基于新 cwd 和新权限行动。
```

这里保存旧环境文本会造成误导：旧 cwd 和旧权限不是未来的事实。应把当前环境作为每次请求的快照；若要复现旧请求，则在审计数据中保存环境 snapshot。

### 能力目录和可用性

这类内容包括：

- Apps 是否可用及其通用使用说明。
- plugins 是否可用及其通用使用说明。
- available skills catalog、skill root 和使用协议。

例子：用户安装一个 plugin 后，下一次请求应看到它现在可用；用户卸载它后，下一次请求不应继续被旧 catalog 误导。

它们与完整 skill 正文不同。catalog 是当前 capability 的索引，应动态生成；完整 skill 正文属于下一节的回合绑定材料。

### 持久化判断

- 进入 `AgentStorage` history：默认不需要。
- 每次实际模型请求前快照：需要。
- 审计 snapshot：按复现需求决定。
- compaction checkpoint：不应进入。

相关位置：

- `shared-context/codex/codex-rs/core/src/session/world_state.rs:31`
- `shared-context/codex/codex-rs/core/src/context/world_state/environment.rs:17`
- `shared-context/codex/codex-rs/core/src/context/available_skills_instructions.rs:9`

## 类型三：长期运行模式与策略状态

这类内容不是外部环境的瞬时事实，也不是一条历史事件，而是 agent 当前采用的工作策略。

### 内容

- approval policy、permission profile、sandbox/network policy。
- caller developer instructions。
- collaboration mode 与其 developer instructions。
- personality。
- realtime 是否启用。
- multi-agent mode。
- model switch 造成的 model-specific instructions。

### Rust 行为

Rust 把这类状态放进 `TurnContextItem`，并对上一个 turn 的 baseline 生成 developer instruction diff。例如 permission profile 改变时，发送新的权限说明；model 切换时，发送 `<model_switch>` 指令。

### 例子

```text
请求 1：approval policy 为 on-request。
用户在 UI 中切换为 never。
请求 2：模型应知道当前不会再等待审批；工具 runtime 也必须实际按新 policy 执行。
```

这里“当前 policy”必须以设置值为真源。历史中的旧 permission message 不能决定行为，模型提示更不能代替 runtime enforcement。

### 持久化判断

- 设置值与版本：需要。
- 旧提示文本作为永久 history：通常不需要。
- 每次请求投影当前有效设置：需要。

若产品要求 fork 精确复现历史行为，fork 点应继承该时刻的 settings version，而不是扫描旧 developer message 推断设置。

相关位置：

- `shared-context/codex/codex-rs/protocol/src/protocol.rs:3213`
- `shared-context/codex/codex-rs/core/src/context_manager/updates.rs:22`
- `shared-context/codex/codex-rs/core/src/session/mod.rs:3214`

## 类型四：由本轮用户输入选择的展开材料

这类内容通常来自当前环境，但它的“被选中”是本轮 user input 的语义一部分。因此不能只在将来重新扫描当前环境。

### 显式 SKILL.md

Rust 在用户显式提到 skill 后读取完整 `SKILL.md`，以 user-role contextual item 注入，并在本轮开始的 history 中记录该 item。available skills catalog 仅包含元数据；完整正文不会提前进入 initial context。

例子：

```text
用户：使用 $gradle 修复这个构建。
系统：展开 gradle/SKILL.md 的完整步骤，模型随后请求执行工具。
执行期间：用户或插件更新了 gradle/SKILL.md。
```

该工具调用后的继续请求必须仍看到最初展开的版本。重新读取文件会让同一轮工作在没有显式切换的情况下改变规则。

### 显式 plugin 指令

用户显式提到 plugin 时，Rust 会结合本轮加载的 plugin、MCP tools 与 Apps inventory，生成说明其当前可用能力的 developer instruction。

例子：

```text
用户：使用 plugin://github 检查 PR。
系统：把此刻该 plugin 可见的 MCP server、Apps 与 skill prefix 注入本轮。
```

若插件在工具调用期间被禁用，后续继续不能悄悄将已选择的能力说明改写成另一个版本。

### Extension turn input

extension 的 turn-input contributor 可根据当前 user input、环境和 turn store 产生注入内容。这个结果同样不能假定可重算，因为 contributor 可能依赖外部服务、随机性或可变 extension state。

### 持久化判断

- 精确正文或不可变内容快照：需要。
- 与触发 user turn 建立关联：需要。
- 重新读取当前文件或重新运行 provider：不应作为恢复策略。

相关位置：

- `shared-context/codex/codex-rs/core/src/session/turn.rs:512`
- `shared-context/codex/codex-rs/core/src/session/turn.rs:622`
- `shared-context/codex/codex-rs/core/src/plugins/injection.rs:14`
- `shared-context/codex/codex-rs/core/src/session/turn.rs:684`

## 类型五：运行时产生的模型可见事实

这类内容不是用户直接输入，但代表已经发生的外部行为或 runtime 判断。它们一旦发送给模型，就不能在恢复时重新生成。

### Hook additional context 与 HookPrompt

user prompt submit hook、tool hook 或 stop hook 可以产生额外文本。Rust 把 ordinary additional context 渲染为 developer message 并记录；stop hook 的 continuation fragments 会变成 user-role message 后继续 sampling。

例子：

```text
用户请求删除目录。
pre-tool hook 查询公司的变更窗口服务，返回：当前处于冻结期，不得删除生产文件。
```

下一步模型必须看到这条结果。恢复时重跑 hook 可能得到不同窗口，也可能再次触发副作用，因此正确做法是保存实际返回的文本与来源信息。

### 权限和网络规则的保存结果

用户批准“以后允许此命令前缀”或“允许访问该 host”后，Rust 会把这一已保存的结果通知模型。

例子：

```text
用户批准 npm install 的命令前缀。
后续模型可基于“该规则已保存”决定是否继续请求同类操作。
```

真实执行权限由 runtime policy 强制，但“本轮发生了批准”仍是模型已经看见的事实，应该持久化。

### 中断、subagent 和宿主命令结果

- 中断 marker 告知模型上个工具可能只执行了一半。
- subagent notification 告知模型另一个 agent 的状态。
- user shell command 包含命令、退出码、耗时和输出。

例子：用户中断一个正在写文件的命令后再次要求继续。若丢掉中断 marker，模型会错误假定旧工具没有任何副作用。

### 持久化判断

- 作为 history/event：必须。
- 尝试从当前 runtime 反推：不应。
- 审计中应保留来源、时间和关联 turn/tool call：建议。

相关位置：

- `shared-context/codex/codex-rs/core/src/hook_runtime.rs:539`
- `shared-context/codex/codex-rs/core/src/hook_runtime.rs:595`
- `shared-context/codex/codex-rs/core/src/session/mod.rs:2060`
- `shared-context/codex/codex-rs/core/src/tasks/mod.rs:865`

## 类型六：临时提醒和资源管理提示

这一类内容服务于当前 inference 的质量或预算控制，不代表需要长期保留的对话事实。

### 当前时间提醒

Rust 可按时间间隔、上下文窗口和 user/tool-output 边界发送 `It is ...` 的 developer message，并把它记录在 history。

例子：

```text
10:00 的请求：模型收到当前时间。
14:00 的下一次请求：模型应收到新的当前时间。
```

Rust 的持久化是其统一 history 机制的结果，不意味着 Kotlin 必须保留 10:00 的提醒。若目标是“模型知道当前时间”，动态发送当前值更直接；若目标是精确回放每一次模型可见输入，则将已发送值记入 audit timeline。

### Token budget 与 compaction 提示

Rust initial context 可以包含 token-budget context、context-window guidance 和当前 window identity。它们用于引导模型缩短输出或执行预算相关策略。

例子：

```text
当前上下文接近窗口上限。
模型收到“保持简洁”的预算提示，并产生一个较短的答案。
```

这是请求时的运行状态。下一次请求和 compaction 后的预算会不同，因此不应当作为长期用户/assistant 历史保存。

### 持久化判断

- 请求级 prompt：需要。
- `AgentStorage` history：默认不需要。
- token count、时间点、window id 的 timeline：可单独持久化。

相关位置：

- `shared-context/codex/codex-rs/core/src/session/time_reminder.rs:71`
- `shared-context/codex/codex-rs/core/src/session/mod.rs:3382`

## 类型七：扩展提供的上下文

extension 是不能按文本内容猜测生命周期的来源。Rust 至少提供三种不同入口：

- `contribute_world_state`：当前世界状态的一部分。
- `contribute_thread_context`：thread 级初始上下文。
- `contribute_turn_context` 和 turn-input contributor：与具体 turn/user input 相关的内容。

此外，extension 可以构造带 source label 的 hidden internal model context fragment。

例子：

```text
world-state extension：提供当前部署环境名。
thread-context extension：提供本产品固定的代码审查规范。
turn-input extension：根据当前 PR URL 获取该 PR 的风险摘要。
```

三者不能采用同一持久化策略：

- 当前部署环境名应动态读取。
- 固定审查规范应作为 runtime/config 的当前指令投影。
- PR 风险摘要应保存本轮实际获得的结果，避免恢复时读取到已变化的 PR。

因此，extension API 不应只有无语义的 `provideContext(): String`。provider 应按内容来源提供专门方法，调用方再根据方法的生命周期决定投递时机和持久化位置。

相关位置：

- `shared-context/codex/codex-rs/core/src/session/world_state.rs:64`
- `shared-context/codex/codex-rs/core/src/session/mod.rs:3345`
- `shared-context/codex/codex-rs/core/src/session/turn.rs:684`
- `shared-context/codex/codex-rs/core/src/context/internal_model_context.rs:63`

## 对 AgentStorage 的具体边界

`AgentStorage` 不应开一个“把任意 ContextInjection 写进去”的宽泛入口。应只保存以下两类模型输入：

- generated history：user、assistant、tool call、tool result、compaction。
- durable contextual history：回合绑定展开材料和已发生的 runtime 事实。

settings、环境和审计数据应位于各自的版本化数据线中，而不是依赖扫描 `ResponseItem` 文本恢复。

对 Kotlin 的请求投影可抽象为：

```text
request configuration
  + current ambient context
  + durable model history
```

其中：

- `AgentContextProvider` 按内容来源提供原始数据，例如 AGENTS、环境、skills catalog。
- Context-injecting state/runtime wrapper 决定在何个请求边界读取它们、如何转换成模型输入、以及是否写入 durable history。
- durable 内容只能通过窄的原子写入操作进入 storage。

这保留了 AgentState 的原子性，同时避免让 `AgentStorage` 承担 host orchestration。

## Compaction 与 server continuation 的约束

### Compaction

compaction 的可替换历史只能来自 durable model history。

反例：如果把动态 AGENTS preamble 与 history 拼成一个 `List<ResponseItem>` 后直接交给 compaction，compaction 可能把 AGENTS 当作普通 user message 保留到 checkpoint。之后即使文件已变更，旧 instructions 仍会永久存在。

正确做法是逻辑上分开：

```text
transient prefix      -> 仅本次请求，必要时也作为 compaction 请求的临时前缀
durable history       -> compaction 的实际输入与 replacement history 来源
```

### previous_response_id 或服务端会话续接

动态 context 只有在客户端控制完整输入时才可以直接替换。若未来使用 `previous_response_id` 或其他服务端 retained history：

- 服务器可能仍保留旧 AGENTS、旧环境或旧 catalog。
- 不能只在客户端认为“当前 context 已改变”。
- 必须显式发送替换 diff，或放弃该链并从完整重建的输入开始新请求。

这是 Rust 使用 `WorldStateItem` 和 replacement notices 的实际动机。

## 决策规则

对任意新注入源，按以下顺序判断：

1. 它是否描述当前世界，而不是过去发生的事件？是则动态投影。
2. 它是否由某个 user turn 选择或产生，且后续工具 continuation 必须看到同一版本？是则保存精确内容。
3. 它是否是长期策略或协议设置？是则保存设置值和版本，不保存历史提示文本。
4. 它是否来自不可安全重放的外部过程或副作用？是则保存实际结果与来源。
5. 它是否只影响本次 inference 的质量、时间或预算？是则临时投递，必要时独立记审计 timeline。

该规则把“是否持久化”从一个全局开关变成可验证的局部语义判断。

## 建议的验证场景

- 修改 `AGENTS.md` 后，下一次 LLM 请求看到新内容，旧内容不进入 compaction checkpoint。
- 显式 skill 注入后修改 `SKILL.md`，同一工具 continuation 仍使用最初展开的正文。
- hook 返回动态冻结窗口信息后 resume，不重新执行 hook，而是保留已记录结果。
- 权限 policy 改变后，runtime enforcement 与模型可见说明均采用新 settings version。
- 当前时间和 token budget 改变后，下一请求使用新值，历史中不产生无界提醒噪声。
- 使用 server-side response continuation 时，动态 context 改变会触发明确的重建或 replacement 路径。
