# Instructions、Skills、Plugins

## AGENTS.md：全局指令

全局用户指令通过 `UserInstructionsProvider` 加载。

默认的 home-directory 实现是 `CodexHomeUserInstructionsProvider`：

- 先检查 `AGENTS.override.md`。
- 再检查 `AGENTS.md`。
- 只接受普通文件。
- 遇到 invalid UTF-8 时会替换非法字节，并记录 warning。
- 空内容会被忽略。
- 返回 `LoadedUserInstructions`，里面包含可空 instructions 和 warnings。

实现位置是 `shared-context/codex/codex-rs/codex-home/src/instructions/mod.rs:24`。

这是一层宿主 provider，不是某个工具的内部逻辑。它可以被 extension 或 embedding host 替换。

Kotlin 侧应该保留类似接口：

- provider 返回一份合法的 loaded-instructions snapshot。
- warnings 和 instruction text 分开。
- session spawn 决定是调用 provider，还是从 parent session 继承。

## AGENTS.md：项目发现

项目级 instructions 由 `load_project_instructions` 加载，入口在 `shared-context/codex/codex-rs/core/src/agents_md.rs:48`。

源码注释说明了发现规则：

- 项目文档主要叫 `AGENTS.md`。
- 可以通过 config 增加 fallback filenames。
- Codex 会从 cwd 向上走到 project root。
- project root 由 configured root markers 决定，默认是 `.git`。
- 找到的文件按 project root 到 cwd 的顺序拼接。
- 不会越过 project root 继续向上找。

默认项目文件名是 `AGENTS.md`。本地 override 文件名是 `AGENTS.override.md`，见 `shared-context/codex/codex-rs/core/src/agents_md.rs:37`。

读取过程不是简单的 `readText`：

- 遵守 `project_doc_max_bytes`。
- 先检查 metadata。
- 非文件会跳过。
- 文件不存在不会报错。
- invalid UTF-8 会产生 warning。
- 超过剩余 byte budget 时会截断。
- 记录 provenance，包括 source path、environment id 和 cwd。

这些逻辑在 `read_agents_md`，位置是 `shared-context/codex/codex-rs/core/src/agents_md.rs:91`。

路径发现也不是直接使用本地文件系统。它接收 `ExecutorFileSystem`，并通过这个 filesystem 检查 marker 是否存在，见 `shared-context/codex/codex-rs/core/src/agents_md.rs:167`。

Kotlin 侧的项目 scanner 不应该假设本地 POSIX 文件系统。它应该依赖宿主 environment 提供的 filesystem abstraction。这样本地、远程和未来环境托管的运行方式才会一致。

## AGENTS.md：组装与渲染

Rust 会把全局 user instructions 和项目 `AGENTS.md` entries 合并到 `LoadedAgentsMd`。

`LoadedAgentsMd` 保留 provenance 和顺序。legacy text 路径里，如果同时存在全局指令和项目文档，会用下面的分隔符隔开：

```text
--- project-doc ---
```

常量位置是 `shared-context/codex/codex-rs/core/src/agents_md.rs:42`。

渲染后的 prompt fragment 是 contextual user fragment：

- role 是 `user`。
- opening marker 是 `# AGENTS.md instructions`。
- body 里包含可选 directory suffix。
- instruction text 被包在 `<INSTRUCTIONS>` 里。

renderer 在 `shared-context/codex/codex-rs/core/src/context/user_instructions.rs:9`。

initial-context builder 会把渲染后的 user instructions 追加到 contextual user sections，位置是 `shared-context/codex/codex-rs/core/src/session/mod.rs:3033`。

重要区别是：`AGENTS.md` 不是普通用户消息。它是宿主 context builder 注入的 contextual user fragment。

## AGENTS.md：生命周期

`AGENTS.md` 内容不会每个 turn 都无脑重发。

Rust 会维护 reference context item：

- 如果没有 baseline，就注入完整 initial context。
- 如果已有 baseline，steady-state turn 只发 settings diffs。
- 仍然会持久化最新的 `TurnContextItem`，这样 resume 和 replay 能恢复 baseline。

实现位置是 `record_context_updates_and_set_reference_context_item`，见 `shared-context/codex/codex-rs/core/src/session/mod.rs:3175`。

compaction 也可能重新建立 context baseline。它会替换 history，并在必要时重新注入完整 initial context。普通 new-window 路径在 `shared-context/codex/codex-rs/core/src/session/mod.rs:3137`。

这支持我们当前 Kotlin storage 的方向：

- raw history 保持 append-friendly。
- context baseline 显式建模。
- compaction 更新 checkpoint/prefix，而不是任意修改旧用户记录。

## Skills：Manager 与 Roots

skills 由 `SkillsManager` 管，不属于模型 loop。

loader 的基础模型是 `SkillRoot`：

- root path
- scope
- filesystem
- plugin id
- plugin root

类型定义在 `shared-context/codex/codex-rs/core-skills/src/loader.rs:155`。

skill root 可能来自多处：

- project config skill roots
- user skill roots
- `$HOME/.agents/skills`
- bundled system skills 的缓存目录 `$CODEX_HOME/skills/.system`
- admin/system roots，例如 `/etc/codex/skills`
- plugin-provided roots
- extra roots
- project root 到 cwd 之间的 repo-local `.agents/skills`

root resolution 代码在 `shared-context/codex/codex-rs/core-skills/src/loader.rs:235`。

repo-local `.agents/skills` 的发现逻辑在 `shared-context/codex/codex-rs/core-skills/src/loader.rs:364`。

对 Kotlin 来说，skills 应该是一个宿主子系统。它们不是 tools，而是 prompt assets 加可选依赖元数据。

## Skills：发现与排序

`load_skills_from_roots` 会扫描 roots、发现 `SKILL.md`、按 canonical `SKILL.md` path 去重、记录 filesystem mapping，然后排序最终列表。

scope 排序规则是：

- repo
- user
- system
- admin

之后再按 skill name 和 path 排序。源码位置是 `shared-context/codex/codex-rs/core-skills/src/loader.rs:215`。

这说明 Rust Codex 把 skill identity 看成 path-backed metadata。name 本身不够，因为同名 skill 可以存在。

Kotlin 的干净模型至少应该保留：

- skill name
- description
- source path
- scope
- filesystem/provider identity
- plugin provenance

单纯用字符串 name 不足以复现 Rust 行为。

## Skills：可用 Skills Prompt

可用 skills 会被渲染成 developer instructions，而不是 user instructions。

`AvailableSkillsInstructions` 实现了 `ContextualUserFragment`，但它的 role 是 `developer`，见 `shared-context/codex/codex-rs/core/src/context/available_skills_instructions.rs:34`。

它的 markers 是协议里的 skill-instruction tags：

- `<skills_instructions>`
- `</skills_instructions>`

body renderer 会构造这些部分：

- `## Skills`
- skill roots
- available skill metadata
- how to use skills

renderer 在 `shared-context/codex/codex-rs/core-skills/src/render.rs:62`。

catalog 有预算限制。`default_skill_metadata_budget` 会在有 model context window 时使用窗口比例，否则使用字符数 fallback，见 `shared-context/codex/codex-rs/core-skills/src/render.rs:143`。

initial context builder 在启用 `include_skill_instructions` 时加入这段 catalog，位置是 `shared-context/codex/codex-rs/core/src/session/mod.rs:2978`。

这和 `AGENTS.md` 的差异很大：

- `AGENTS.md` 是要遵守的指令文本。
- available skills 是能力元数据和使用协议。
- 完整 `SKILL.md` 正文不会进入 initial context。

## Skills：显式注入

当用户显式调用 skill 时，Rust 才会把完整 `SKILL.md` 正文加载进当前 turn。

per-turn 路径是 `build_skills_and_plugins`，入口在 `shared-context/codex/codex-rs/core/src/session/turn.rs:459`。

这个函数会：

- 从当前 turn 抽取 user input。
- 必要时加载 plugin 和 MCP/app inventory。
- 收集显式提到的 skills。
- 为提到的 skills 触发 MCP dependency prompt/install 流程。
- 读取完整 skill body。
- 把 skill injections 转成 contextual user `ResponseItem`。
- 同时构造 plugin 和 extension injection items。

读取完整 skill body 的函数是 `build_skill_injections`，位置是 `shared-context/codex/codex-rs/core-skills/src/injection.rs:58`。

它会通过该 skill 对应的 filesystem 读取文件，并分别记录 analytics 和 warnings。

skill selection 支持结构化 `UserInput::Skill` 和文本 mention。collector 在 `shared-context/codex/codex-rs/core-skills/src/injection.rs:133`。

完整 skill injection 和 available-skill catalog 不是同一个东西：

- catalog：developer role，只包含元数据，在 session/context setup 阶段出现。
- explicit injection：user role，包含完整 `SKILL.md` 内容，只作用于当前 turn。

Kotlin 侧应该分成两个模型：

- skill catalog entry，用于 metadata prompt。
- skill injection，用于完整正文注入。

把两者合并会让预算、provenance 和生命周期都变得混乱。

## Skills：隐式调用

Rust 还有 implicit skill invocation telemetry。

shell 和 unified exec 路径可以根据 command 或 working directory 匹配 skill path indexes。这个流程会发 telemetry 和 analytics，但不等价于把完整 skill body 注入上下文。

入口是 `maybe_emit_implicit_skill_invocation`，位置是 `shared-context/codex/codex-rs/core/src/skills.rs:48`。

Kotlin 侧可以暂缓这一块。它更偏观测和归因，不是 skill instructions 的最小机制。

## Plugins、MCP 与 Apps

plugins、MCP 和 apps 都是会影响上下文和工具可见性的宿主运行时功能。

`ThreadManager` 构造时持有 `PluginsManager` 和 `McpManager`，见 `shared-context/codex/codex-rs/core/src/thread_manager.rs:276`。

Session 初始化阶段会计算：

- auth
- MCP runtime config
- effective MCP servers
- tool plugin provenance
- auth statuses

代码位置是 `shared-context/codex/codex-rs/core/src/session/session.rs:603`。

如果 apps enabled，initial context 会加入 app connector instructions。路径在 `shared-context/codex/codex-rs/core/src/session/mod.rs:2964`。

initial context 也会加入 plugin capability summaries，位置是 `shared-context/codex/codex-rs/core/src/session/mod.rs:3001`。

per-turn 的显式 plugin 或 app mention 在 `build_skills_and_plugins` 里处理：

- 解析 plugin mentions。
- 需要 app context 时向 MCP connection manager 请求 all tools。
- 合并 plugin connectors 和 accessible connectors。
- 记录显式 app/plugin invocation analytics。

相关逻辑在 `shared-context/codex/codex-rs/core/src/session/turn.rs:479`。

对 Codex Lite 来说，这些不应该进入基础 tool contract。它们是 runtime capability discovery 层，会产出 instructions 和 tool specs。
