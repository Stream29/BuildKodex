# Task Tree

- [done] 对齐模型可见工具描述
  - [done] [静态工具描述](../done/2026-07-25-align-static-tool-descriptions.md)
  - [done] [`request_user_input`描述](../done/2026-07-25-align-request-user-input-description.md)
  - [done] [`unified_exec`描述](../done/2026-07-25-align-unified-exec-descriptions.md)
  - [done] [Multi-agent描述](../done/2026-07-25-align-multi-agent-descriptions.md)

# Details

- 以当前`shared-context/codex`中的Rust实现为来源。
- 不自行补充、缩写或改写模型可见描述。
- 逐个处理子任务。
- 任何涉及环境、运行时依赖、公开签名或现有抽象的修改，必须先与用户确认。
- Multi-agent静态描述为最终方案，不引入运行时动态spec。
