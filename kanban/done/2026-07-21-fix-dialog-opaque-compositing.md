# Task Tree

- [done] 修复dialog覆盖输出内容时的字符穿透
  - [done] 在运行中的TUI复现并定位TextSurface合成语义
  - [done] 为TuiDialog增加覆盖完整测量矩形的字符清除
  - [done] 增加底层文本穿透回归测试
  - [done] 运行组件测试并重新链接TUI

# Details

Mosaic的`Modifier.background()`只更新cell背景色，不替换已有字符。dialog与历史内容绘制到同一`TextSurface`时，底层输出字符会穿过dialog色块。

`TuiDialog`现在先以空格覆盖自身测量矩形，再绘制调用方内容。JVM与Linux Native组件测试通过，Linux TUI可执行文件已重新链接；运行中的旧进程未重启，以免丢失内存会话。
