# Task Tree

- [done] 在 Stop Hook 缺少 assistant 文本时使用 pending user input
  - [done] 确认 pending 问题的稳定文本投影
  - [done] 实现 assistant 优先的回退逻辑
  - [done] 补齐 Runtime 回归测试
  - [done] 更新 Hooks 决策
  - [done] 运行定向检查与测试

# Details

- 用户已明确要求：优先使用 assistant text；缺失时查看 pending
  `request_user_input`；两者都没有时保留现有兜底。
- 修改范围限定在 Stop Hook 请求内容及其测试和决策文档。
- 保持现有 Stop wire schema，不新增字段。
- Runtime 优先传递本轮最近的非空 assistant text。
- assistant text 缺失时，按顺序连接 pending 请求中的非空
  `question` 文本，分隔符为换行。
- pending 问题文本不包含 header、options 或其他工具参数。
- assistant 和 pending 问题文本均缺失时继续传递 `null`。
- 验证包含 JVM 与 Linux X64 定向测试、模块 `check`、格式与 diff 检查。
- 状态：实现与验证完成。
- `jvmTest`、`linuxX64Test` 与模块 `check` 均通过。
- 根仓库与 `Kodex/` 的 `git diff --check` 均通过。
- Gradle 仅报告既有的弃用、跨平台 cinterop 和 npm 依赖选择警告。
