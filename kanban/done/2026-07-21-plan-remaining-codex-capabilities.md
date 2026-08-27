# Task Tree

- [done] 收口剩余Codex能力规划
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
  - [done] 复核Plugins、Goals和其余延期能力的优先级

# Details

- 状态：已完成。
- 本文件只维护剩余能力的规划覆盖情况，不跟踪各独立实现任务何时完成。
- Codex Apps 已在调研后正式拒绝；Multi-agent V2 和第一期单 Agent Hooks 已完成实现。
- 用户已废弃二期扩展Hooks规划；Kodex自有Hooks设置与Codex显式导入由[独立可执行任务](../executable/2026-08-15-manage-hooks-in-kodex-settings.md)跟踪。

## 调研基线

- 上游基线：`openai/codex`的`origin/main`，提交`cf821e8ec850c6d8380feea0e84859dd8ff54cd0`。
- 当前项目已经具备Responses API、remote compaction v2、AgentStorage/AgentState、基础Runtime、工具契约、tool search和可运行CLI。
- 本计划不包含多环境执行、sandbox和Azure适配。

## 独立实现任务

- Codex Apps：[已拒绝](2026-07-21-integrate-codex-apps.md)。
- Multi-agent V2：[已完成的实现任务](../done/2026-07-21-implement-multi-agent-v2.md)。
- 第一期单Agent Hooks：[已完成的实现任务](../done/2026-07-22-implement-single-agent-hooks.md)。
- Kodex自有Hooks设置与Codex显式导入：[可执行任务](../executable/2026-08-15-manage-hooks-in-kodex-settings.md)。

## 延期能力

- Plugins安装与manifest编排。
- Goals自动续跑。
- Memory。
- Code mode、dynamic tools和工具建议。
- Realtime/audio、personality和artifacts。

剩余Codex能力已经完成规划覆盖；已废弃的二期扩展Hooks不再保留独立任务。
