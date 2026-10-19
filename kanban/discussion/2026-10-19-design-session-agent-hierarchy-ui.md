# Task Tree

- **`等待层级契约并讨论 Session 内 Agent 导航交互`()**
- `设计层级展示选择与 Agent 视图生命周期`()
- `对齐 subagent 创建审批与运行状态交互`()
- `形成可审阅交互方案与验收场景`()
- `等待用户确认方案及实施授权`()

# Details

## 目标与状态

- 为恢复后的 Session 内 Agent 层级提供可用前端 UI；用户指出这是本轮相较后端逻辑更难的部分，单独建卡跟踪。
- 当前处于 discussion，仅授权任务记录，不预选旧版 Agent Tree UI 或直接进入实现。

## 依赖与下游

- 所属主线：[Codex 对齐](2026-09-30-align-kodex-with-codex-rs.md)下游的 Session 层级 → subagent → Tool as CLI；具体前置如下。
- 依赖[Session 内 Agent 层级](2026-10-19-restore-session-agent-hierarchy.md)的身份、拓扑及 RPC 契约。
- 与[subagent 工具重设计](2026-10-19-redesign-subagent-tools-and-execution-context.md)协调创建、审批、配置及运行状态交互；两卡共用层级契约，不互相建立循环前置。
- 本卡是[Tool as CLI](2026-10-03-discuss-integrate-tool-cli-into-kodex.md)对应工具 CLI 化的前置之一；后端层级可用不等于 UI 已交付。

## 待讨论

- 顶层 Session 与内部 Agent 的导航关系；避免以每个子 Agent 新开独立 Session 的方式继续制造膨胀。
- 层级展开、选择、状态展示与大量后代的可达性；具体布局及加载策略需审阅。
- 切换 Agent 时历史、输入草稿、工具输出及终端资源如何准确绑定原 owner。
- 多前端、重连、层级变化及选中 Agent 失效时的行为，区分视图生命周期和后端执行生命周期。
- 与重制工具的手动审批、auto approve 及默认配置反馈衔接，不要求 UI 为每次创建重新强制选择模型和推理强度。

## 后续验收检查

- 在同一 Session 内可发现、选择并查看子 Agent，而非只能从顶层 Session 列表寻找。
- 验证深层/大量 Agent、运行/完成/失败状态及窄终端下的交互可达性。
- 切换、折叠或关闭视图不串用 Agent 状态，不因视图操作意外取消后端工作。
- 实际交互验收覆盖创建、审批、配置默认值与执行结果查看；不以静态树渲染代替端到端验收。

## 参考

- [当前 CLI Session 与 Agent ViewModel 边界](../../checklist/cli-session-view-models.md)；其中 root-only 规则是待替代的当前实现，不代表本次层级方案已完成。
- [旧版 Agent 树拓扑任务](../done/2026-08-24-model-ordered-agent-tree-topology.md)：仅供回顾。
