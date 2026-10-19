# Task Tree

- **`等待层级契约并讨论真正的 subagent 工具语义`()**
- `设计自动审批与模型配置默认规则`()
- `统一 subagent 与嵌套工具的执行上下文表达`()
- `协调前端交互并明确旧工具替代方式`()
- `形成实施方案与验收条件并等待授权`()

# Details

## 目标与状态

- 用户认为当前 `suggest_subagent_task` 是半成品：所有任务都创建普通 root Session，造成 Session 膨胀；不能 auto approve，且创建交互要求人类手动选择模型与推理强度配置。
- 在 Session 内 Agent 层级上重做真正的 subagent 能力，不把现有 suggest 工具直接包装成 CLI。
- 明确支持 auto approve，以及无需每次人工配置模型和推理强度的路径；审批范围、配置来源与优先级尚待设计。
- 当前处于 discussion；本轮仅授权建卡与依赖记录，不授权恢复旧版或实现新工具。

## 依赖与下游

- 共同上游：[Codex 对齐](2026-09-30-align-kodex-with-codex-rs.md)；不要求其全部候选能力先完成。
- 依赖[Session 内 Agent 层级](2026-10-19-restore-session-agent-hierarchy.md)的领域与后端契约。
- 与[层级 UI](2026-10-19-design-session-agent-hierarchy-ui.md)协调交互；设计可共同推进，整体交付须验证实际前端路径。
- 下游：[Tool as CLI](2026-10-03-discuss-integrate-tool-cli-into-kodex.md)。先形成并交付真正的 subagent 能力与统一执行上下文，再进行对应工具 CLI 化。

## 上下文边界与候选方式

- 大部分工具不需要感知调用者 Session；调用关联的意义是维护结构化并发关系，不应扩散为通用工具业务参数。
- 统一考虑 Agent → subagent、terminal command → 嵌套工具调用的执行关系；不预设两类对象必须采用同一种业务模型。
- terminal command 的子进程环境变量可作为自动传递执行关联的候选载体；变量名、载荷、父执行标识及失效处理尚未确定。
- 不先为 Tool as CLI 冻结专用 Session 上下文协议；少数确实读写 Agent 状态的工具单独明确业务依赖。

## 待讨论

- 创建、等待、结果回流、后续交互及取消需要哪些能力；工具名称与接口不沿用旧版作为默认答案。
- auto approve 的启用、适用范围和人工确认路径；不能把支持自动审批理解为默认无条件批准。
- 模型、推理强度等配置的默认值、继承、覆盖和合法组合；保留必要的人工调整能力，但不要求逐次手工选择。
- 父执行结束、取消或失败时如何处理子执行；区分 Session 内 Agent 的存续和一次运行的结构化并发关系。
- 独立 CLI 调用如何建立执行归属，以及嵌套 CLI 如何附着已有 owner；不凭环境变量存在推导权限或取消语义。
- 旧 `suggest_subagent_task` 的替换或过渡方式，以及已有 pending/stable 历史兼容边界。

## 后续验收检查

- 真正子任务创建在原 Session 的 Agent 层级下，不继续按子任务制造顶层 Session。
- 手动审批与 auto approve 路径均可验证；默认配置路径无需人类逐次选择模型与推理强度。
- 验证父子取消、完成等待、失败归属及结果交付；跨 RPC/进程边界不退化成无归属后台工作。
- 与前端层级 UI 联合验收；统一上下文可供后续 CLI 复用，但不提前声称 CLI 已实现。

## 历史参考

- [旧版 Multi-agent V2](../done/2026-07-21-implement-multi-agent-v2.md)：已被废弃，不原样恢复。
- [当前 suggest 工具](../done/2026-09-04-suggest-subagent-task-interaction.md)：新任务将重新审定其普通 root Session、人工确认和不回流结果等限制；历史交付记录保持不变。
