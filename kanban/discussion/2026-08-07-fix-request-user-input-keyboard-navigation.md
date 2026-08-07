# Task Tree

- 改善 `request_user_input` 表单的键盘方向导航

# Details

- 状态：`await refinement`。
- 等待用户后续明确细化；当前不进入规划或实现。
- 表单最多占 12 行，并使用普通 `verticalScroll` 裁剪内容。
- `TextInput` 未消费上下方向键，按键会进入 Mosaic 默认空间焦点搜索。
- 普通 `verticalScroll` 未提供 beyond-bounds focus 与 bring-into-view；视口外控件不会成为候选，焦点可能跳到表单外。
- 选择 `Other` 会增加文本框行数，使后续问题或 `Submit` 更容易离开视口。
- 后续问题未完成时 `Submit` 禁用且不可聚焦；该行为需要与跨视口导航缺陷区分。
- 已用临时 Linux X64 自动化探针复现；探针已删除，未修改实现。
