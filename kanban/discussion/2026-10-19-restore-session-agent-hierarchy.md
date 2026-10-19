# Task Tree

- **`等待讨论 Session 内 Agent 层级与所有权边界`()**
- `明确层级身份持久化与后端 RPC 契约`()
- `明确运行期父子关系与结构化并发语义`()
- `协调前端层级 UI 与 subagent 工具设计`()
- `形成实施方案与验收条件并等待授权`()

# Details

## 目标与状态

- 用户要求先恢复 Session 下的 Agent 层级结构，解决当前所有 subagent 都成为 root Agent、创建独立 Session 所造成的严重 Session 膨胀。
- 本卡负责领域与后端层级边界；前端 UI 单独跟踪，不以仅完成后端作为整项交付。
- 当前处于 discussion；本轮授权创建任务卡和标明依赖，不授权实现、数据迁移、提交或发布。
- 旧版已被用户否定并废弃；恢复层级能力不等于恢复旧版设计、代码或六个调度工具。

## 依赖与下游

- 共同上游：[Codex 对齐](2026-09-30-align-kodex-with-codex-rs.md)；这是方向与范围关联，不要求先完成全部 Codex 对齐。
- 后端 Daemon 前置已由[一期任务](../done/2026-10-13-design-backend-daemon.md)交付。
- 下游：[Session 内 Agent 层级 UI](2026-10-19-design-session-agent-hierarchy-ui.md)、[subagent 工具与统一执行上下文重设计](2026-10-19-redesign-subagent-tools-and-execution-context.md)。
- 两项下游可基于确认后的层级契约协调设计；实现顺序另行规划，不要求讨论阶段串行等待全部后端交付。
- [Tool as CLI](2026-10-03-discuss-integrate-tool-cli-into-kodex.md) 的对应工具 CLI 化须等待层级后端、前端 UI 和重制工具能力完成，不能只以本卡后端完成解除依赖。

## 待讨论

- Session、root Agent、后代 Agent 的身份、拓扑和持久化表达；区分 Agent 所属层级与一次运行的父子执行关系。
- Agent 创建、打开、恢复、关闭与 Session 所有权；明确父子取消、等待结束和失败传播，而非仅记录 parent ID。
- 多前端观察与操作同一 Agent 层级的 RPC 边界，供 UI 与工具共同消费。
- 现有独立 root Session、历史遗留 `subagents/` 数据如何处理；尚未决定迁移、重新挂接或清理，不自动改写旧数据。

## 后续验收检查

- 新建子 Agent 不再为每个子任务增加一个独立顶层 Session；普通独立 Session 的创建语义仍保留。
- 层级身份、拓扑与持久化恢复一致；运行期状态与持久化关系不互相冒充。
- 按确认后的所有权规则验证取消、失败、资源释放与多前端竞态。
- 端到端验收包含前端层级 UI 与 subagent 工具，不能仅凭后端测试完成整体交付。

## 历史参考

- [旧版 Multi-agent V2](../done/2026-07-21-implement-multi-agent-v2.md)：只作历史参考，不作为新方案。
- [当前任务建议方案](../done/2026-09-04-suggest-subagent-task-interaction.md)：记录普通 root Session 替代方案的历史决定。
- [当前 AgentState 与 Runtime 边界](../../checklist/agent-state-and-runtime.md)、[已结项协程生命周期研究](../done/2026-10-14-discuss-daemon-coroutine-lifecycle.md)；后者供参考，不是未完成前置。
