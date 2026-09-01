# Task Tree

- [done] 优化 Terminal Sessions 侧栏条目
  - [done] 确认单行摘要与详情语义
  - [done] 实现单点条目与 hover 详情
  - [done] 保留条目右键关闭操作
  - [done] 覆盖长命令与多条目渲染
  - [done] 运行相关检查

# Details

- 用户要求参考 History Index 的条目交互，但每项统一使用 `●`，不使用首中尾连线。
- 每个 Terminal Session 在列表中只占一行。
- 单行摘要不显示 session ID。
- hover 显示完整信息，右键继续提供操作。
- 单行使用 `● <command>`，并按实际侧栏宽度安全省略。
- hover popup 显示 session ID 和未经省略的完整 command，并允许内部滚动。
- 复用 History Index 的延迟 hover、popup 定位和不透明表面样式。
- 修改范围限于 Session sidebar 视图、其测试及本任务记录。
- 使用现有 terminal-cell 文本工具处理摘要和详情换行，不新增状态所有权。
