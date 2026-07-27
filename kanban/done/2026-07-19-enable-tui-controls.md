# Task Tree

- [done] 让TUI demo使用可交互控件
  - [done] 为Mosaic补充显式鼠标追踪、节点命中测试和指针捕获/冒泡
  - [done] 为焦点宿主补充`Tab`与`Shift+Tab`遍历
  - [done] 建立复用的`Pressable`与`Button`组件
  - [done] 将demo中的会话操作接为可聚焦、可点击按钮
  - [done] 覆盖键盘、鼠标和终端运行路径

# Details

当前demo只使用焦点宿主和文本输入目标，未实现焦点遍历、指针分发或按钮组件。本任务使鼠标和键盘调用同一语义操作，并保持Mosaic负责终端鼠标模式和节点事件路由，Codex Lite负责焦点与控件状态。

已通过Linux Native单元测试和真实PTY验证：`Tab`/`Shift+Tab`遍历、`Enter`/空格激活、主键点击、新建会话与计划模式切换均可用；终端会在启动时开启并在退出时恢复鼠标追踪模式。
