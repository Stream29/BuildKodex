# Task Tree

- [done] 让TUI demo实时渲染Responses流
  - [done] 为会话快照维护按流项目区分的文本与推理预览
  - [done] 处理增量与完成事件，确保预览不会重复或丢失
  - [done] 将预览实时投影到终端历史视图，并在提交后切换为稳定历史
  - [done] 覆盖流式文本、推理和完成回填的测试
  - [done] 在真实PTY与Responses API路径验证增量重绘

# Details

流预览按`item_id`、输出序号和内容序号保存。`*.delta`追加对应片段，`*.done`替换对应片段，`output_item.done`在持久化历史项可确认时移除同一项目的预览，避免稳定历史与流预览短暂重复。

Linux Native `:tui-demo:linuxX64Test`通过。真实PTY调用Responses API确认了部分文本即时重绘、完成后切换为稳定assistant历史项，并确认终端鼠标模式在退出时恢复。
