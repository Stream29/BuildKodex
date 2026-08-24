# Task Tree

- [done] 移除文本输入内部选区
  - [done] 删除选区状态与替换语义
  - [done] 移除键盘和拖拽选取
  - [done] 保留点击光标与终端原生选择
  - [done] 更新组件与应用测试
  - [done] 更新TUI交互决策
  - [done] 验证JVM与Linux X64目标

# Details

- 用户明确决定不再维护输入框内部选区。
- 文本选择完全交给终端，用户通过`Shift+Drag`建立终端原生选区。
- 输入框继续维护编辑光标、点击放置光标、软换行、视口和撤销重做。
- `TextInputValue`只保留文本和活动光标。
- 带`Shift`的移动键不再由输入框消费；普通主键点击仍放置光标，后续拖拽不改变光标。
- 删除选区反显、选区替换、选区历史快照和内部拖拽锚点。
- 修改范围限于输入组件状态、渲染、组件与应用集成测试和对应交互决策；调用方接口只随选区参数收窄。
- `:app-view-components:jvmTest`、`:app-view-application:jvmTest`、`:app-view-components:linuxX64Test`和`:app-view-application:linuxX64Test`通过。
- IntelliJ目标文件错误检查和`git diff --check`通过。
