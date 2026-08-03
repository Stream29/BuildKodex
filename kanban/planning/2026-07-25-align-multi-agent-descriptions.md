# Task Tree

- 对齐Multi-agent工具描述
  - 对齐不依赖运行时信息的描述
  - 对比`spawn_agent`的模型目录与agent配置投影
  - 与用户确认动态spec的接口形状
  - 按确认后的设计实现
  - 增加精确描述测试
  - 运行相关模块测试

# Details

- Rust的`spawn_agent`描述依赖可用模型、agent类型、隐藏选项和usage hint。
- Kotlin当前使用静态`MultiAgentTools.specs`。
- 改变spec构造方式或引入运行时依赖前必须先与用户确认。
