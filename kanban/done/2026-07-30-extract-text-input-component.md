# Task Tree

- [done] 将 Composer 通用文本输入原语迁移至 `cli:components`
  - [done] 提取文本值、编辑操作、Unicode 光标与文本布局原语
  - [done] 提取可复用的 `TextInput` 组件与终端文本展示辅助函数
  - [done] 迁移 app Composer，并保留提交和换行快捷键策略
  - [done] 将回归测试迁移到 components 并验证 app 集成

# Details

- 用户于 2026-07-30 明确要求提取现有 Composer 中可复用的文本输入原语。
- `NewLineKey`、`SubmitKey`、消息提交和会话状态属于 app，不进入通用组件。
- 不在本任务实现输入框高度上限、垂直滚动或上下方向键编辑语义。
- `:cli-components:jvmTest`、`:cli-app:jvmTest`、`:cli-components:linuxX64Test` 与 `:cli-app:linuxX64Test` 已通过。
