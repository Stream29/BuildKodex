# Task Tree

- [done] 修复Unicode文本导致的TUI坐标偏差
  - [done] 在运行中的TUI确认偏差等于中文字符的额外cell宽度
  - [done] 为Mosaic接入跨平台Unicode终端宽度计算
  - [done] 让TextLayout与TextSurface共享终端cell语义
  - [done] 增加宽文本后的布局、焦点与鼠标命中测试
  - [done] 运行Mosaic与CodexLite跨平台测试并重新链接TUI

# Details

当前Mosaic以Unicode code point数量测量文本并逐code point占用一个`TextSurface` cell。终端实际将中文字符显示为两个cell，导致后续控件的布局、鼠标命中区域和物理光标位置全部比画面向左偏移。

修复后Mosaic通过`unicode-width-kotlin`计算终端cell宽度，`TextLayout`与`TextSurface`共享同一套cluster占位语义。JVM、Linux x64和macOS ARM64的布局、鼠标命中、焦点光标及TUI链接验证均通过。
