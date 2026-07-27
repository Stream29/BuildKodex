# Task Tree

- [done] 修复TUI终端滚轮输入
  - [done] 用真实TTY确认鼠标报文是否到达Demo
  - [done] 确认内联渲染与终端坐标的关系
  - [done] 选择并实现可靠的全屏/滚轮输入路径
  - [done] 覆盖终端协议与Demo交互回归测试
  - [done] 在Linux真实TTY验证滚轮行为

# Details

用户确认键盘历史导航有效，但物理鼠标滚轮未触发历史滚动。此前的测试只注入了Mosaic测试终端事件，不能证明真实TTY报文、坐标和渲染基准正确衔接。

- tmux在普通屏幕中默认截获滚轮；Demo现在进入备用屏幕并启用`1003`全量鼠标追踪与`1006` SGR坐标。
- Linux真实TTY确认启动顺序为`1049h -> 1003h -> 1006h`，退出时按反序恢复。
- `:mosaic-tty-terminal:linuxX64Test`与`:tui-demo:linuxX64Test`通过；前者的配置缓存受Mosaic现有`cklib`插件限制，后者复用了配置缓存。
