# Task Tree

- [done] 将CLI history迁移到通用LazyColumn
  - [done] 定义带稳定业务key的轻量`HistoryEntry`投影
  - [done] 保持streaming转committed时的逻辑identity
  - [done] 在可见item内部按当前终端宽度格式化和换行
  - [done] 为每个frontend和session维护`HistoryUiState`
  - [done] 实现follow-tail、unread和稳定阅读位置
  - [done] 将展开状态移到frontend-local、session-local UI状态
  - [done] 删除共享ViewModel中的history viewport状态和事件
  - [done] 接入pointer、paging、End和modal隔离语义
  - [done] 删除旧`TuiLazyColumn`及wrapped-row viewport实现
  - [done] 覆盖streaming、resize、session切换、多frontend和焦点行为
  - [done] 运行Mosaic、CLI components和CLI app跨平台测试
  - [done] 在真实PTY中验证滚轮、paging、streaming、resize、session和modal链路
  - [done] 更新设计记录、API snapshot并清理临时文件

# Details

- 状态：已完成。JVM、Linux x64和macOS ARM64测试通过，并在真实PTY中验证了长历史、滚轮、paging、streaming、resize、session切换和modal链路。
- history保持自然时间顺序；follow-tail通过显式末尾定位实现。
- UI状态属于frontend和session，不写入共享ViewModel或AgentStorage。
- 非follow-tail期间新增内容只更新unread，并由stable key维持阅读位置。
- 文本格式化属于history renderer；LazyColumn不理解transcript。
