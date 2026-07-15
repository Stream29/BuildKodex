# Task Tree

- [done] 收紧 AgentContextInjection contract
  - [done] 调查 provider 默认值、环境模型与调用面
  - [done] 将 provider 重命名为注入对象并改为直接属性
  - [done] 令环境、日期、时区和 shell 成为必需数据
  - [done] 将 DeveloperInstructions 改为 value class
  - [done] 更新 renderer、状态机、测试与决策记录
  - [done] 验证所有受影响的 host target

# Details

`AgentContextInjection` 表示一次有效的结构化注入。环境上下文始终存在；没有 AGENTS.md 或 skill 时继续使用空列表，开发者指令仍可缺省。
