# Task Tree

- 收口剩余Codex能力规划
  - [done] 对照最新Codex源码盘点尚未实现的能力
  - [done] 排除多环境、sandbox和Azure适配
  - [done] [Agent持久化方案](../done/2026-07-21-plan-agent-persistence.md)
  - [done] [Codex配置与Session兼容性](../done/2026-07-22-improve-codex-config-and-session-compatibility.md)
  - [done] [`request_user_input`](../done/2026-07-21-complete-request-user-input.md)
  - [done] [AGENTS.md与skills加载](../done/2026-07-21-complete-agent-context-loading.md)
  - [done] [通用MCP基础设施](../done/2026-07-21-build-mcp-foundation.md)
  - [done] 完成Codex Apps调研并拆分独立实现任务
  - [done] 完成Multi-agent V2规划并拆分独立实现任务
  - [done] 完成第一期单Agent Hooks规划并拆分独立实现任务
  - [扩展Hooks规划](../ongoing/2026-07-22-plan-extended-hooks.md)
  - [done] 复核Plugins、Goals和其余延期能力的优先级

# Details

- 状态：只剩扩展Hooks（二期Hooks）尚未规划。
- 本文件只维护剩余能力的规划覆盖情况，不跟踪各独立实现任务何时完成。
- Codex Apps、Multi-agent V2和第一期单Agent Hooks已经完成调研或规划，后续实现分别由独立任务跟踪。

扩展Hooks完成规划并经用户审核后，本任务即可完成。

## 调研基线

- 上游基线：`openai/codex`的`origin/main`，提交`cf821e8ec850c6d8380feea0e84859dd8ff54cd0`。
- 当前项目已经具备Responses API、remote compaction v2、AgentStorage/AgentState、基础Runtime、工具契约、tool search和可运行CLI。
- 本计划不包含多环境执行、sandbox和Azure适配。

## 独立实现任务

- Codex Apps：[实现任务](../ongoing/2026-07-21-integrate-codex-apps.md)。
- Multi-agent V2：[已完成的实现任务](../done/2026-07-21-implement-multi-agent-v2.md)。
- 第一期单Agent Hooks：[已完成的实现任务](../done/2026-07-22-implement-single-agent-hooks.md)。
- 二期Hooks：[规划任务](../ongoing/2026-07-22-plan-extended-hooks.md)。

## 延期能力

- Plugins安装与manifest编排。
- Goals自动续跑。
- Memory。
- Code mode、dynamic tools和工具建议。
- Realtime/audio、personality和artifacts。

第一期Hooks规划已完成并进入独立待实现任务；二期Hooks仍为`await planning`。
