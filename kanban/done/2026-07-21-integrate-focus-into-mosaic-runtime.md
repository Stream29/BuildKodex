# Task Tree

- [done] 将焦点系统内置到Mosaic runtime
  - [done] 在Mosaic布局树中建立焦点目标、作用域和Owner
  - [done] 实现焦点路径键盘分发及默认Tab/方向导航
  - [done] 实现鼠标命中聚焦、模态约束和焦点恢复
  - [done] 将终端光标绑定到当前焦点目标
  - [done] 迁移Kodex组件并删除手写焦点注册层
  - [done] 更新焦点设计checklist与测试
  - [done] 在Linux和macOS验证Mosaic与Kodex TUI

# Details

布局树是焦点关系的唯一事实来源。普通组件只声明自身可聚焦；程序化`FocusRequester`仅作为特殊场景的显式控制入口。Dialog和菜单由组件内部声明模态焦点作用域，不要求业务调用方手动选择或恢复焦点。
