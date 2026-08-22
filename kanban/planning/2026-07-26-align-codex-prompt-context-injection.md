# Task Tree

- 修复Codex提示词与上下文注入
  - [done] 对照最新上游源码确认缺口和影响范围
  - [done] 确认舍弃OpenAI返回的model-specific instructions
  - 制定完整修复Plan
  - 与用户确认Plan和实施边界
  - 按确认后的Plan实施修复
  - 验证标准Responses与Responses Lite请求投影
  - 验证skills、Planning guidance和Agent mode上下文

# Details

- 状态：`await planning`。
- 上游基线：`openai/codex`提交`61a44880a85d2fd0d8770908dea5733495e571c8`。
- 当前只完成问题调研和任务登记，尚未制定修复Plan，也未开始实现。
- Plan完成并经用户确认后，任务进入修复阶段。
- kanban记录不构成自动推进授权；后续阶段仍按用户请求推进。

## 已确认决策

- 舍弃OpenAI `/models`返回的model-specific instructions，包括`base_instructions`及`model_messages`中的指令内容。
- 这些内容不进入本地模型元数据、不持久化、不注入上下文，也不提供离线fallback。
- 新Session的`instructions`不从这些返回值填充；保持默认空值是预期行为。

## 已确认问题

- `/models`返回的`include_skills_usage_instructions`和`use_responses_lite`未进入本地模型元数据。
- 默认模型使用Responses Lite，但本地尚未按Lite协议投影工具和reasoning context。
- skills目录提示词仍是旧格式，缺少按模型条件注入的使用规则。
- Planning与`update_plan` guidance现固定注入，后续只对齐客户端自有提示词。
- Multi-agent仍缺少当前上游的root/sub-agent身份和usage hints；Single/Multi模式重置文本已覆盖。
- AGENTS.md、基础环境上下文角色和远程compaction策略目前没有发现同等级阻断问题。

## Plan约束

- 区分客户端静态模板、项目上下文和每轮动态上下文的所有权与消息角色。
- 不建模、持久化或注入OpenAI返回的model-specific instructions。
- 明确Session创建、恢复、模型切换和compaction时其余上下文的解析与持久化语义。
- 明确标准Responses与Responses Lite的独立请求投影和测试矩阵。
- 保留项目决策：不恢复上游实验开关；`request_user_input`是否进入请求工具列表由当前Agent的`RequestUserInputMode`显式控制。
- 与[模型可见工具描述对齐](../done/2026-07-25-align-tool-descriptions.md)协调，但不重复其通用工具描述工作。
