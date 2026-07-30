# Kodex 上下文组成成分的归属与生命周期

## 范围

本文只分析一次正常 Responses 请求中，哪些数据会影响模型可见输入，以及它们应该落在哪一层。它不规定新的公开 API，也不替代 `checklist/` 中已经确认的设计决策。

这里的“上下文”不能被当成单一容器。Rust Codex 的 initial-context builder 会把配置、宿主快照、能力目录、历史事件和运行时策略一起组装成请求；Kotlin 若照着把它们都塞进一个临时前缀，后续的 resume、compaction、fork 和工具执行都会失去清晰边界。

## 五条数据通道

一次 Kodex 请求应按以下五条通道理解：

```text
Responses request
  = 顶层请求配置
  + 临时请求前缀
  + 持久化模型 history
  + runtime 执行状态
  + 审计与复现元数据
```

### 1. 顶层请求配置

这是 `ResponsesApiRequest` 的字段，不是 `ResponseItem`。

- model、reasoning、service tier。
- API `instructions`。
- tool spec、tool choice、parallel tool calls、output/text controls。
- previous response id、prompt cache key、client metadata。

当前 Kotlin 已通过 `KodexAgentSettings` 承载这些值，并由 `CodexResponsesRequest` 投影到 wire request。它们需要随着 storage 的 settings timeline 版本化，才能使 fork 或恢复获得正确的下一个请求配置；但不应伪装成一条 developer 或 user message。

### 2. 临时请求前缀

这是一组宿主在请求开始时提供、但不是对话事实的模型可见数据。它会被渲染为 developer 或 contextual-user message，并加在普通请求的 history 前面。

当前 Kotlin 将结构化provider绑定到AgentState，并由`AgentStateImpl.requestResponseApi()`按以下顺序组装：

```text
ModeKind.render()
  + ReasoningEffort.renderMultiAgentMode()
  + contextPrefixProvider.resolve().render()
  + storage.modelInputAt(snapshotIndex)
```

该前缀不写入storage，也不会进入remote compaction的输入。`AgentTurnContext`在用户消息写入前解析prefix与skills；写入成功后，同一轮的续跑和工具continuation复用该不可变快照。下一条用户消息才重新解析文件系统变化。

`agent-context`定义结构化contract、loader和通用prompt DSL。AgentState解析并渲染绑定的provider，负责消息角色和请求拼接；ResumableAgent负责回合边界、快照冻结与持久化交付。不存在调用方传入任意`HistoryItem`的临时输入管道。

### 3. 持久化模型 history

这是模型已经看见、且之后继续同一工作仍必须保留的内容。它由用户、assistant、tool 以及少量已经发生的宿主事件构成。

宿主不能直接写底层 history；应通过 `KodexAgentState` 的语义原子操作。当前通用入口是 `injectHistory(items)`，而用户消息、工具结果和 plan 更新分别有更具体的原子操作。

一旦内容进入这条通道，它会参与后续正常请求、compaction 和 fork。因而它必须是不可变的事实快照，不能依赖“未来重新读取当前文件或重新运行 callback”来重建。

### 4. Runtime 执行状态

这是真实环境中能够执行什么、应该如何执行的状态。例如权限策略、sandbox、文件系统、网络、skill/plugin/MCP 发现、hook callback 和工具 handler。

它本身不等于模型提示词。`ResumableAgentLayer` 负责持有和执行它，并在需要时把结果投影到前三条通道：

- 将当前 tool spec 投影到请求字段。
- 生成临时 capability 或环境描述。
- 将已发生的 hook、skill selection 或运行时通知写入 history。

模型看到的权限说明不能替代权限 enforcement；模型看到的 tool catalog 也不能替代实际的 tool routing。

### 5. 审计与复现元数据

有些信息需要保存，但不应交给模型或 compaction：例如某次请求采用的 AGENTS.md 来源路径和 hash、环境快照、runtime/provider 版本、hook run id、技能文件的来源和读取时间。

这类数据应进入专门的 metadata、timeline 或审计存储，不应通过伪造 `ResponseItem` 来保存。

## 当前 Kotlin 实现的精确边界

### `KodexAgentSettings` 是请求配置真源

`KodexAgentSettings` 已包含 `instructions`、模型、推理强度、工具、tool choice 和服务等级等请求级字段。它的 `instructions` 对应 OpenAI Responses 的顶层 `instructions` 字段；它不是 developer-role message 的别名。

这意味着以下内容不应进入 `AgentContextPrefixProvider`：

- base model instructions。
- model、reasoning、service tier。
- 普通 tool 和 namespace spec。
- tool-search 的可加载 tool 列表及其协议选项。
- output schema、text format、cache key 等 wire 行为。

这些都是“本次请求怎么调用模型”的配置，而不是“模型曾经看见的一段对话内容”。

### Prefix与skill discovery相互独立

`AgentContextPrefixProvider.resolve()`返回完整的`AgentContextPrefix`，包含环境、AGENTS.md和available skills catalog。它不读取AgentState、storage或history，也不构造OpenAI item；AgentState构造时必须绑定该provider。

`SkillsResolver.resolve(cwd)`独立返回该cwd可见的`ResolvedSkills`：

- `skills`只包含可用于提示词目录的轻量metadata。
- `loadSkill`与`readResource`绑定这次解析的文件系统权限。
- 文件系统实现每轮重新枚举目录，并按canonical `SKILL.md`路径、mtime、size与可用的file key缓存metadata解析。

`AgentTurnContext`组合基础prefix和`ResolvedSkills`，作为回合感知的`AgentContextPrefixProvider`绑定到AgentState，并冻结当前逻辑user turn。metadata catalog、环境和AGENTS.md只作为临时前缀；显式选择的skill正文通过durable history写入。

### `injectHistory` 是宿主事件的窄入口

`injectHistory` 接受 `ResponseItem.HistoryItem`，并以一个原子 storage transition 按顺序追加。它适合“宿主刚刚做了某件会改变模型后续判断的事”，但不适合每轮重复发送当前日期、当前 cwd 或能力目录。

这样可以避免两种常见错误：

- 将普通动态环境快照永久留在 history，导致旧 cwd 或旧权限持续误导模型。
- 让已经选择的 skill 或 hook 结果只存在于临时前缀，导致工具调用后的后续请求丢失它。

## Rust Codex 的真实做法

Rust 的 `Session::build_initial_context_with_world_state_and_mcp` 会聚合大量来源：权限、显式 developer override、collaboration mode、realtime、personality、skills、plugins、扩展贡献、token budget 以及 world state。最终它会把 developer sections、contextual user sections 和独立 developer sections 分别组成 `ResponseItem`。

在 steady-state turn 中，Rust 不会简单重复完整 initial context，而是比较 `TurnContextItem` 和 `WorldStateSnapshot`，只向 history 追加上下文 diff。`WorldState` 的作用是保存“上一次模型可见世界状态”的比较基线，并不是所有上下文的统一真源。

这带来两个不能直接照抄的结论：

- Rust 的 `WorldStateItem` 是为增量 history 和 rollout 恢复服务的 diff metadata，不是一个通用的业务上下文 DTO。
- Rust 把大量提示词写入历史，是其 current resume/replay 实现的取舍；Kotlin 的目标是由runtime按内容生命周期决定临时投影或持久化交付，因此不需要为了对齐而建立相同的 world-state history。

## 组成成分逐项归属

### Base instructions 与模型请求参数

例子：切换模型、推理强度、输出格式，或为特定模型选择一份基础 instructions。

- 真源：`KodexAgentSettings`。
- 模型交付：Responses 顶层字段。
- 持久化：settings timeline，而非 history。
- 运行时所有者：AgentState 通过 `updateSettings` 原子更新；runtime 决定何时切换。

模型基础 instructions 和 developer-role message 的语义不同。前者是 API 配置，后者是输入内容；不要为了“都是指令”而合并它们。

### AGENTS.md

AGENTS.md 是宿主加载的指令来源，带有路径、环境和顺序等 provenance。它不是普通用户发言。

- 真源：AGENTS.md loader / host filesystem。
- 模型交付：contextual user 临时前缀。
- 持久化：当前 Kotlin 不进入 history 或 compaction。
- 审计：可单独记录 source path、有效内容 hash 与可选正文快照。
- Kotlin contract：`AgentContextPrefix.agentMd`。

Rust 会用 world-state diff 发送“替换旧 AGENTS.md”或“旧 instructions 不再适用”的通知；Kotlin不采用该路径，而是在每条新用户消息开始时重新解析。变化不会改写已经持久化的模型history，也不会改变正在续跑的turn。

### 环境、日期、时区与 shell

这类信息回答“模型当前运行在哪里”。它不是用户对话事实。

- 真源：host runtime 的环境选择与时钟。
- 模型交付：contextual user 临时前缀。
- 持久化：不进入模型 history；审计按需要单独保存。
- Kotlin contract：`AgentContextPrefix.environmentContext`。

Rust 的环境内容还可能包含 filesystem/network policy、subagent 状态和环境可用性。当前 Kotlin 故意只建模所有环境均可用的基础信息；在真正实现多环境或受限执行之前，不应提前引入假的 status 或 permission DTO。

### Available skills catalog

catalog 只告诉模型“有哪些 skill、如何找到它们”，不等于把整个 `SKILL.md` 交给模型。

- 真源：SkillRuntime 的 discovery 和 capability roots。
- 模型交付：developer-role 临时前缀。
- 持久化：不进入 history；它反映当前能力目录。
- Kotlin contract：`AgentContextPrefix.availableSkills`；`AgentTurnContext`用`SkillsResolver.resolve(cwd)`得到的`ResolvedSkills.skills`填充。

当前 `AvailableSkill` 已保留 name、description 和 path。未来若需要对齐 Rust 的复杂来源，scope、filesystem/provider identity 与 plugin provenance 属于 skill discovery 数据，不应由 tool contract 承载。

### 用户显式选择的完整 SKILL.md

例子：用户输入 `$gradle 修复构建`，runtime 在本轮读取并展开 `gradle/SKILL.md`。

- 真源：该次 selection 时读到的 skill 文件内容。
- 模型交付：contextual user history item。
- 持久化：必须进入 history，并参与后续 tool continuation、compaction 与 fork。
- 运行时所有者：`SkillSelectionRuntime`与`AgentTurnContext`。

原因是 selection 已经是本轮输入的一部分。若在 tool call 后重新读取当前文件，用户没有再次选择 skill 的情况下模型会收到不同规则；这不是合法的继续执行。

### Apps、plugins、MCP 与 capability discovery

这几者首先是 runtime capability，而不是基础 tool contract 的一部分。

- 实际可调用能力：生成本次request的tool spec。
- 通用可用性说明：可作为临时 catalog 前缀，但在对应功能存在前不建模。
- 用户显式选择某个 plugin/app 后的详细说明：作为持久化 history event。
- 连接、鉴权、安装、可用性检查和工具执行：ResumableAgent 负责。

因此不能把 plugin/MCP server 描述直接塞进 `AgentContextPrefixProvider`，更不能让一段 prompt 文字承担“这个工具是否真的可执行”的职责。

### 权限、sandbox、filesystem 与 network

权限有两个不可合并的侧面：

- runtime enforcement：决定 handler 能否实际读文件、联网或执行命令。
- model-visible explanation：帮助模型正确选择工具或避免无意义请求。

前者必须归 runtime，且以 runtime policy 为唯一真源。后者是从同一 policy 派生的提示内容，不能反过来作为 enforcement 依据。

当前 Kotlin 尚未实现相应的 tool runtime，因此不应预先将一个脱离执行语义的 `PermissionContext` 加进 `AgentContextPrefixProvider`。实现权限 runtime 时，应由该层同时维护策略、执行门禁与模型说明；策略变化所产生的模型可见更新必须走显式交付路径。

### Collaboration mode 与 personality

它们是长期运行模式，而不是通用 host context。

- collaboration mode 会影响 model、reasoning 以及一段 mode-specific developer instruction。
- personality 会影响 model-specific instructions，有时模型本身已经内置这部分内容。

Rust 中 collaboration context 的真源是session configuration中的`CollaborationMode.settings.developer_instructions`。选择内置Default或Plan模式时，models manager的preset会提供该字符串；app server或配置更新也可以替换它。创建turn时，该mode会复制到`TurnContext`。

首次建立context baseline时，Rust按developer section顺序加入模型切换、权限、显式`developer_instructions`、`<collaboration_mode>`包装的mode instructions、realtime、personality与skill catalog，随后合并为一个developer-role `ResponseItem`。因此collaboration block位于显式developer override之后、realtime/personality/skill之前。

steady-state turn不会每次重复该block。Rust比较前一个`TurnContextItem`与当前`CollaborationMode`；mode变更时才把新的`<collaboration_mode>` developer message追加到history。`include_collaboration_mode_instructions`关闭时不注入。空mode instructions不会生成清除消息，因此Rust当前实现保留既有developer历史的行为不应被误读为通用清除协议。

当前 Kotlin 将`ModeKind`作为可版本化的settings状态：`Default`和`Plan`在每次普通请求中分别由`agent-context:prefix:collaboration:render`投影固定developer block。此投影不写入history，也不参与compaction；`UpdatePlanArgs`和`ThreadGoal`仅用于settings/UI状态，不参与模型提示词。自定义developer instructions需要日后完整建模 collaboration mode，不能复用prefix字段。

### Hook 产生的上下文与 continuation

hook 不是工具。它是围绕 user input、tool invocation 或 turn stop 执行的宿主 callback。

- hook additional context：发生后作为 developer-role history event。
- stop hook continuation：发生后作为 user-role continuation history event。
- hook run id、执行时间、原始 callback outcome：审计 metadata。
- hook 是否允许继续或阻止工具：runtime 控制流。

这类内容必须持久化，因为它已经在具体 turn 中影响过模型；恢复时重新运行 hook 会产生不确定行为。

### 用户附件、`@` 引用与自动读取

用户直接输入的文本、图片或附件属于用户消息，应走 `appendUserMessage` 及其后续的结构化 content model。

若 runtime 因用户操作自动读取额外文件并把结果告知模型，这次读取是已经发生的宿主事件。其模型可见结果应作为 history item 固化，而不是在将来根据当前文件内容重新生成。文件路径、hash 和读取策略则属于审计 metadata。

### Tool output、plan 与 compaction

- tool output：走 `completeToolCall` 或专用的 `appendPlanUpdate`，不是 context injection。
- plan：plan timeline 是独立状态；`update_plan` 的模型可见 tool output 与 plan snapshot 同一原子操作提交。
- compaction：由 runtime 决定时机，由 AgentState 发起远程请求并更新 checkpoint；不是一条普通 host context message。
- token count：timeline 观测数据，用于下一次 compaction 判断；不是给模型重复注入的提示词。

## 一览表

| 组成成分 | 真源 | 模型交付 | 模型 history | Kotlin 归属 |
|---|---|---|---|---|
| Base instructions | settings | request `instructions` | 否 | `KodexAgentSettings` |
| model/reasoning/service tier | settings | request 字段 | 否 | `KodexAgentSettings` |
| tool spec/namespace | runtime capability + mode | request `tools` | 否 | request-level runtime projection |
| Plan mode | settings | 固定 developer 临时前缀 | 否 | `ModeKind` + collaboration renderer + AgentState |
| AGENTS.md | host loader | contextual user 临时前缀 | 否 | `AgentContextPrefixProvider` + `AgentTurnContext` |
| 环境、cwd、日期、时区 | host environment | contextual user 临时前缀 | 否 | `AgentContextPrefixProvider` + `AgentTurnContext` |
| skill metadata catalog | skill discovery | developer 临时前缀 | 否 | `SkillsResolver` + `AgentTurnContext` |
| 完整选中 SKILL.md | 当前 turn selection | contextual user item | 是 | `SkillSelectionRuntime` + `injectHistory` |
| hook additional context | hook outcome | developer item | 是 | hook runtime + `injectHistory` |
| hook continuation | stop-hook outcome | user item | 是 | hook runtime + `injectHistory` |
| approval / network-rule 结果 | runtime outcome | 必要时 history item | 是 | permission runtime |
| plugin/app 显式说明 | 当前 turn selection | history item | 是 | capability runtime |
| plugin/MCP 连接和鉴权 | runtime service | 间接产生 tools/context | 否 | runtime service |
| personality | session setting | request 配置和必要 developer 内容 | 否 | 未来 settings + runtime |
| tool result | tool handler outcome | tool output | 是 | `completeToolCall` |
| compaction checkpoint | AgentState | request metadata/prefix | 不作为普通 history | compaction timeline |
| 路径、hash、hook run id | audit source | 不直接交付 | 否 | metadata/timeline |

## 不应跨越的边界

- 不要把 `KodexAgentSettings` 的请求配置编码成 `ResponseItem`。
- 不要把 tool spec 放入 context prompt 以代替 request `tools`。
- 不要把 skill catalog 当成完整 skill body。
- 不要把 permissions prompt 当成 sandbox 或审批逻辑。
- 不要把每次都变化的环境信息写入持久化 history。
- 不要把已经发生的 skill/hook/自动读取结果只留在临时前缀。
- 不要把审计 metadata 伪装为模型消息。
- 不要为了模拟 Rust 的 diff 机制而引入一个泛化的 `WorldState` DTO；先有明确的动态更新需求，再定义窄的状态与交付协议。

## 源码锚点

- `Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/CompactionModels.kt:29`：`KodexAgentSettings` 与 normal request 输入。
- `Kodex/agent-context/prefix/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/contract/AgentContextPrefixProvider.kt:20`：AgentState绑定的结构化prefix resolver contract。
- `Kodex/agent-context/prefix/render/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/render/AgentContextPrefixRenderer.kt:21`：临时前缀渲染。
- `Kodex/agent-runtime/skill/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/skill/AgentTurnContext.kt:15`：按逻辑user turn解析并冻结prefix与skills。
- `Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt:101`：普通请求解析provider并拼接临时prefix。
- `Kodex/agent-state/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentState.kt:117`：受控的 history 注入入口。
- `shared-context/codex/codex-rs/core/src/session/mod.rs:3187`：Rust initial context builder。
- `shared-context/codex/codex-rs/core/src/session/mod.rs:3590`：Rust context baseline 与 world-state diff。
- `shared-context/codex/codex-rs/core/src/session/world_state.rs:12`：Rust world-state section 的来源。
- `shared-context/codex/codex-rs/core/src/context/world_state/mod.rs:88`：world-state snapshot 与 diff 语义。
- `shared-context/codex/codex-rs/core-skills/src/injection.rs:58`：完整 SKILL.md 的按 turn 注入。
