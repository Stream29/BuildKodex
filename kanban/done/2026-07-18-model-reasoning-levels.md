# Task Tree

- [done] 补齐模型推理强度元数据
  - [done] 在 `ModelInfo` 中保留默认档位和支持列表
  - [done] 让推理档位兼容服务端新增值
  - [done] 在 Codex 请求投影中处理 `ultra` 语义
  - [done] 验证序列化和受影响模块

# Details

Codex `/models` 以 `default_reasoning_level` 和有序的 `supported_reasoning_levels` 描述每个模型的可选推理强度。
