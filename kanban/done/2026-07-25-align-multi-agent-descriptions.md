# Task Tree

- [done] 对齐Multi-agent工具描述
  - [done] 对齐不依赖运行时信息的描述
  - [done] 对比`spawn_agent`的模型目录与agent配置投影
  - [done] 与用户确认动态spec的接口形状
  - [done] 按确认后的静态设计实现
  - [done] 确认静态描述为最终方案
  - [done] 增加精确描述测试
  - [done] 运行相关模块测试

# Details

- Rust的`spawn_agent`描述依赖可用模型、agent类型、隐藏选项和usage hint。
- Kotlin使用静态`MultiAgentTools.specs`。
- 用户确认静态描述为最终方案，不引入model catalog依赖或运行时动态spec。
- 静态描述及精确测试已通过JVM、Linux Native和Windows Native验证。
