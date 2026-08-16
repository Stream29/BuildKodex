# Task Tree

- [done] 统一单选字段并恢复旧侧栏
  - [done] 盘点仍使用按钮组的单选字段
  - [done] 将单选字段替换为下拉选单
  - [done] 恢复单一侧栏与旧书签
  - [done] 保留侧栏宽度展开动画
  - [done] 补充渲染与交互测试
  - [done] 更新 TUI 交互决策
  - [done] 运行格式化和回归验证

# Details

- 状态：`done`。
- 用户已明确要求直接落地。
- 表单或问答中的互斥值选择属于本任务；Settings 页面导航、Session 标签和 Agent tree 属于导航组件，不改为下拉选单。
- 当前剩余的互斥按钮组是 MCP 编辑器的 Transport、OAuth，以及 `request_user_input` 的答案选项。
- 侧栏恢复为同时包含 Agent tree 与 Terminal sessions 的单一面板，以及原有单列圆角书签。
- 继续保留已经恢复的零列至完整宽度展开/收起动画。
- MCP 编辑器复用 Settings 的同行下拉字段与 host-level 下拉菜单。
- `request_user_input` 为每个问题持有独立下拉状态；菜单保留选项说明，`Other` 继续切换到自由文本输入。
- 验证覆盖 Components、Agent、Settings 与 Application Mosaic 模块，并构建 Linux release executable。
- IDEA 格式化、定向构建与检查通过；仅保留 MCP 编辑器中的弱重复代码提示。
- `:app-view-components:linuxX64Test :app-view-agent:linuxX64Test :app-view-settings:linuxX64Test :app-view-application:linuxX64Test` 通过。
- `:app-cli:linkReleaseExecutableLinuxX64` 通过。
- 已在隔离配置的 100×30 终端中验证单一圆角书签、侧栏宽度动画、Settings 同行下拉字段，以及 MCP 编辑器的 Transport、OAuth 下拉触发器。

## 1. 互斥值选择

- 所有表单和问答中的 A/B/C 互斥选项使用单个下拉触发器，不再并列或纵列显示多个选项按钮。

### 用户审批

- 所有 A/B/C 选一控件改为下拉选单。

## 2. 侧栏样式

- 撤回双书签、双面板结构，恢复原有单一侧栏和圆角书签。
- 保留侧栏展开与收起动画。

### 用户审批

- 新侧栏样式不好看，恢复旧样式。
