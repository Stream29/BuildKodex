# Task Tree

- [done] 将模型目录状态改为Kotlin backing field
  - [done] 移除额外的私有`MutableStateFlow`属性
  - [done] 验证模型目录与上下文预算调用链

# Details

`OpenAiModelCatalog.models`对外保持只读`StateFlow`，内部直接使用其`MutableStateFlow` backing field。
