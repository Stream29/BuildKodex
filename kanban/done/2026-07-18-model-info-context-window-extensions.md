# Task Tree

- [done] 将 `ModelInfo` 上下文预算扩展归属到模型模块
  - [done] 将状态类型和 `ModelInfo` 扩展移入 `openai:models`
  - [done] 更新调用方导入并保持行为一致
  - [done] 验证模型目录和上下文预算模块

# Details

这些计算只依赖 `ModelInfo` 字段，不属于模型目录的加载职责。
