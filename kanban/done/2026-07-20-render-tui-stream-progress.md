# Task Tree

- [done] 修复TUI将流式文本集中到结束时才渲染的问题
  - [done] 用用户提示词确认真实Responses SSE连续发送文本delta
  - [done] 复现并记录真实终端中的帧更新节奏
  - [done] 定位并移除本地事件流或Compose重绘的合并点
  - [done] 覆盖长文本delta的逐步可见性回归
  - [done] 在真实Responses API与终端中验证

# Details

用户提示词`你讲个故事`在原始Responses SSE中从约2.38秒到19.04秒持续产生约710个文本delta，但TUI仅在结束时显示。根因是`HttpClient.post`走Ktor的完整响应保存路径。流式请求改为`preparePost(...).body`后，真实客户端回归通过；TUI在第5秒已显示生成中的故事，随后正常完成。
