# Task Tree

- 修复Codex提示词与上下文注入
  - [done] 对照最新上游源码确认缺口和影响范围
  - 制定完整修复Plan
  - 与用户确认Plan和实施边界
  - 按确认后的Plan实施修复
  - 验证标准Responses与Responses Lite请求投影
  - 验证skills、collaboration mode和Multi-agent上下文

# Details

- 状态：`await planning`。
- 上游基线：`openai/codex`提交`61a44880a85d2fd0d8770908dea5733495e571c8`。
- 当前只完成问题调研和任务登记，尚未制定修复Plan，也未开始实现。
- Plan完成并经用户确认后，任务进入修复阶段。
- kanban记录不构成自动推进授权；后续阶段仍按用户请求推进。

## 已确认问题

- `/models`返回的`base_instructions`、`model_messages`、`include_skills_usage_instructions`和`use_responses_lite`未进入本地模型元数据。
- 新Session的`instructions`保持空值，Codex客户端自有的模型基础指令没有进入请求。
- 默认模型使用Responses Lite，但本地尚未按Lite协议投影基础指令、工具和reasoning context。
- skills目录提示词仍是旧格式，缺少按模型条件注入的使用规则。
- Plan模式模板已正确对齐；Default模式仍错误复用了基础提示词中的Planning片段。
- `request_user_input`在Default模式下被无条件暴露，与上游默认关闭的实验开关不一致。
- Multi-agent缺少当前上游的模式重置文本、root/sub-agent身份和usage hints。
- AGENTS.md、基础环境上下文角色和远程compaction策略目前没有发现同等级阻断问题。

## Plan约束

- 区分模型基础指令、客户端静态模板、项目上下文和每轮动态上下文的所有权与消息角色。
- 不手工维护上游完整模型基础提示词；优先使用模型目录数据，并设计离线fallback。
- 明确Session创建、恢复、模型切换和compaction时的指令解析与持久化语义。
- 明确标准Responses与Responses Lite的独立请求投影和测试矩阵。
- 与[模型可见工具描述对齐](../executable/2026-07-25-align-tool-descriptions.md)协调，但不重复其通用工具描述工作。
