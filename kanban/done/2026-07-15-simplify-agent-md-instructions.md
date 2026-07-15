# Task Tree

- [done] 简化 AGENTS.md 指令 contract
  - [done] 调查 `AgentsMdInstructions` 和 `provideAgentMd` 的使用面
  - [done] 迁移模型、provider 与渲染器 API
  - [done] 更新调用方、测试和相关决策记录
  - [done] 验证所有受影响的 host target

# Details

将 AGENTS.md 原始来源直接表示为 `List<AgentsMdInstruction>`；`provideAgentMd` 始终返回该列表，空列表表示没有可注入的 AGENTS.md 指令。
