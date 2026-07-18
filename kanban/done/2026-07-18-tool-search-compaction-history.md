# Task Tree

- [done] 修正 remote compaction 的 tool search history 语义
  - [done] 删除无条件清空 `ToolSearchOutput.tools` 的请求投影
  - [done] 使状态层回归测试要求完整 schema 进入 compaction 请求
  - [done] 更新 tool search checklist 决策
  - [done] 运行状态层跨平台回归测试

# Details

Remote compaction 接受完整 `tool_search_output`。Rust 仅在 context-window 超限时，作为条件性 history 裁剪的一部分清空某些工具搜索结果；当前 Kotlin 尚未实现这条预算裁剪路径。
