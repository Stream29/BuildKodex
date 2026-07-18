# Task Tree

- [done] 实现 `openai:model-catalog` 与 `get_context_remaining`
  - [done] 扩展 `/models` 的真实 `ModelInfo` DTO，并验证远端响应
  - [done] 建立 host 侧模型目录、缓存、刷新和模型解析逻辑
  - [done] 根据模型窗口、压缩阈值和 token timeline 计算剩余上下文预算
  - [done] 实现 `get_context_remaining` ToolSpec、普通 Tool 和 Runtime 组合入口
  - [done] 运行目录与工具的跨平台及真实 API 验证
  - [done] 记录模型目录与 token-budget 的持久决策

# Details

`openai:model-catalog` 负责模型元数据的来源与解析；`openai:client` 只执行一次 `/models` 请求。`get_context_remaining` 只接收 `suspend () -> Long?` 的剩余预算查询，不直接读取 AgentStorage 或模型目录。

远端 macOS 上已通过 JVM、JS Node、macOS ARM64 的受影响模块测试；`openai:model-catalog`实际请求了Codex`/models`，集成测试实际执行了对话续写和hosted web search。Linux与Mingw目标完成交叉编译，因当前验证机不是对应运行平台而跳过执行。
