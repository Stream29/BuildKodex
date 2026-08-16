# Task Tree

- [done] 优化 Settings 选择器与双书签侧栏
  - [done] 将 Settings 选择字段改为同行下拉触发器
  - [done] 将 MCP 和 Hooks 操作移到标题行
  - [done] 将收起侧栏拆为两个方形书签
  - [done] 分离 Agent tree 与 Terminal sessions 面板
  - [done] 恢复侧栏宽度展开和收起动画
  - [done] 更新渲染与交互测试
  - [done] 更新 TUI 设计决策
  - [done] 运行格式化和回归验证

# Details

- 状态：`done`。
- 用户已明确要求直接落地。
- Settings 中持续显示的选择控件使用与底部状态栏相同的下拉触发器，不再并排显示多个 selected 反显按钮。
- 选择字段的标题与当前值触发器放在同一行。
- MCP servers 和 Hooks 的标题与管理操作放在同一行，操作位于标题之后。
- 收起状态显示 `Agent tree →` 与 `Terminal sessions →` 两个无圆角书签。
- 两个书签分别展开对应面板；继续保留 hover 展开、点击固定及菜单打开期间保持展开的语义。
- 侧栏从零列到完整宽度双向动画，收起状态不占用 History 布局宽度。
- IDEA error inspections 未发现问题。
- `:app-view-settings:linuxX64Test :app-view-application:linuxX64Test` 通过。
- `:app-cli:linkReleaseExecutableLinuxX64` 通过。
- 已在隔离配置的 100×30 终端中验证 Settings 布局、下拉菜单、双书签及逐列展开动画。
