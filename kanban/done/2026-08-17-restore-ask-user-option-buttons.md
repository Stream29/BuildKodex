# Task Tree

- [done] 恢复 Ask User 回答选项
  - [done] 定位错误套用下拉规则的范围
  - [done] 恢复逐项单选按钮
  - [done] 移除 Ask User 专用下拉与弹层
  - [done] 保留自由文本 Other 流程
  - [done] 补充按钮渲染与选择测试
  - [done] 修正 TUI 交互决策
  - [done] 运行格式化和回归验证

# Details

- 状态：`done`。
- 用户已明确要求直接撤回。
- 之前错误地把“表单中的 A/B/C 互斥值使用下拉”扩展到 `request_user_input` 的 Ask User 回答面板。
- 本任务只恢复 Ask User 回答选项；Settings 的 Questions 模式等设置字段继续使用下拉选单。
- Ask User 已恢复逐项 `○`/`●` 按钮、候选说明及 `Other` 自由文本流程，并移除专用下拉状态与 PopupHost。
- 原下拉测试已替换为按钮渲染、键盘选择及 `Other` 自由文本回归测试。
- IDEA 格式化、定向构建和检查通过。
- `:app-view-agent:linuxX64Test` 与 `:app-view-application:linuxX64Test` 通过。

## 1. Ask User 回答选项

- 有候选答案时逐项显示原有单选按钮及说明。
- `Other` 继续切换到自由文本输入。
- 不再为 Ask User 创建下拉状态、触发器或弹出菜单。

### 用户审批

- Ask User 的回答选项不应该改为下拉选单。
