# Task Tree

- [done] 明确Mosaic与Kodex TUI的扩展边界
  - [done] 列出必须在Mosaic中实现的终端和节点能力
  - [done] 列出Mosaic开口后可由下游实现的组件
  - [done] 区分专用`LazyTranscript`与通用`LazyColumn`的实现边界
  - [done] 将结论写入TUI交互组件checklist

# Details

现有Mosaic公开键盘modifier、绘制modifier、终端尺寸和物理光标定位，但不公开终端事件消费、节点几何或鼠标模式控制。
本任务只定义边界，不实施Mosaic或TUI代码。
