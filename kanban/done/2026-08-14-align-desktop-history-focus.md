# Task Tree

- [done] 对齐 Desktop 历史与焦点体验
  - [done] 对照 CLI 与 Desktop history renderer
  - [done] 对照 CLI 与 Desktop 焦点路径
  - [done] 还原历史消息展示和条目操作
  - [done] 还原焦点、遍历和快捷键行为
  - [done] 补充一致性测试并验证

# Details

- 用户指出 Desktop 历史消息渲染和焦点系统与 CLI 使用体验脱节。
- TUI 的历史信息结构、展开语义、条目焦点、分页焦点移动和上下文操作是本次修正基准。
- Desktop 当前把多类工具压成原始 `toString()`，流式事件也只拼接少数 delta；应逐类映射 TUI 的消息标题、语义摘要、状态、分组详情和计划展示。
- Desktop 当前只有可操作历史行才可聚焦，且没有 per-Agent 滚动状态、follow-tail、半页滚动后的焦点迁移或 composer 初始焦点。
- 在 history frontend 增加 Desktop-local UI state，以稳定 key 管理 `LazyListState`、follow-tail 和分页目标；application renderer 按 Session/Agent 保留该状态。
- 已提交历史项始终是整行焦点和 secondary-action surface；pending、streaming 与 marker 不进入条目焦点序列。
- 用 Compose 焦点、滚动和 Material 控件映射 TUI 语义，不修改共享 ViewModel contract。
- 验证包括 Desktop renderer/paging/focus UI 测试、应用测试、启动 smoke test 和 Linux package；修改方案已确定，可以执行。
- Desktop 按视觉时间顺序声明 history，使嵌套控件、已提交条目和 composer 共用 Compose 的自然焦点序列。
- 最终验证通过 90 项 JVM/Linux 测试；Desktop 进程进入事件循环并持续到 60 秒 smoke timeout；DEB 已重新生成。
