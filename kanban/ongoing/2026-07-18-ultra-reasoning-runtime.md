# Task Tree

- 对齐`Ultra` reasoning的runtime语义
  - 在multi-agent runtime中将`Ultra`作为proactive multi-agent mode的触发条件
  - 保持Responses API请求投影为`max`
  - 增加同时验证请求投影与proactive行为的对齐测试

# Details

`Ultra`是仅由部分模型声明的独立 reasoning 选项。当前Kotlin已保留该模型能力并在请求层投影为`max`，但尚未实现Rust Codex对应的proactive multi-agent runtime语义。该任务等待multi-agent runtime开始实现时恢复。
