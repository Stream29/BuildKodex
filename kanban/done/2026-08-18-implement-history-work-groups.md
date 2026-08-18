# Task Tree

- [done] 实现 History WorkGroup 自动折叠
  - [done] 确认折叠事件与 breaker
  - [done] 确认单项、最新区段与跨 batch 语义
  - [done] 确认 summary、展开与 transient 交接
  - [done] 设计增量投影与状态保留
  - [done] 设计渲染、交互与 context action
  - [done] 设计测试、性能与 release 验证
  - [done] 增加 WorkGroup 与 breaker variant
  - [done] 实现有界 batch 与增量投影
  - [done] 接入 group renderer 与 child 复用
  - [done] 覆盖 append、batch、revert 与交互
  - [done] 运行性能验证与 release 冒烟

# Details

- 状态：`executable`。用户已明确授权开始实施；本文件不修改仍由用户推进的既有讨论文件。
- maximal sealed run 只包含 `Reasoning`、普通 `Tool` 与 `Patch`；至少两个 item 才投影为 `WorkGroup`。
- `Message`、`PlanUpdate`、`ContextCompaction` 与 completed `request-user-input` 是 breaker，并继续正常展示。
- 最新未闭合 stable run 保持逐项展示，breaker 到达后再一次性折叠。
- 旧端加载不发布可增长的 open group；batch 在有界范围内延伸到 breaker，达到上限时允许强制切分。
- 折叠行固定显示 `Take n actions`，其中 `n` 是 group child item 数量；summary 不读取 payload。
- 展开后复用现有 child renderer、child expansion state 与精确 child context action；collapsed group 不提供有歧义的 context action。
- pending、streaming 与 committed 交接允许短暂闪烁，不增加双缓冲或动画身份。
- 当前只支持单一 Mosaic frontend；滚动、group expansion 与 child expansion 均由 `AgentHistoryViewModel` 生命周期持有。
- `WorkGroup` 对外持有 sparse storage index range、child count 与 expansion state；内部保留无 decoded event 的稳定 child ViewModel，以便展开后复用状态并让旧 window 继续可索引。
- completed `request-user-input` 使用独立 non-foldable variant，但继续绑定现有工具展开 renderer。
- History loop 分别维护最新 open raw prefix 与 immutable sealed rope。普通 append 只重新投影 open prefix；旧端 batch 独立投影后追加，不扫描已发布历史。
- 旧端 batch 先读取 64 个 stable item；cutoff 落在 foldable run 时最多再读取 64 个。遇到 breaker 时包含 breaker并停止；达到上限时把 cutoff 作为强制分组边界。
- `HistorySequence` 按每个顶层 item 的 newest/oldest span 排序；storage lookup 对 `WorkGroup` 递归命中 exact child，保证 child read、context action 与 generation 校验不变。
- collapsed group 只提供自己的展开交互；expanded group 中每个 child 继续使用 `StoredHistoryEntry`，因此 child context action 保持 exact storage index。
- History committed raw read 使用 ViewModel-local bounded semaphore，避免 expanded group 同时触发无界 native file-I/O worker。
- 自动化验证覆盖：单项不分组、三类 foldable、四类 breaker、最新 open prefix 封闭、跨 64-item cutoff、强制切分、child 状态复用、revert 全量失效、summary/键鼠展开及 child context action。
- 性能验证使用大 synthetic history 检查投影只处理新增 segment，并用 release CLI 打开真实 session 验证滚动、交接和日志。
- Linux 与 macOS ARM64 的 history ViewModel/view tests 全部通过；macOS ARM64 release 与 Linux x64 cross-release 均在
  MacBook 的 Java 25 环境构建通过。
- Release CLI 打开包含 13,388 个 stable event 的真实 session，在一秒轮询窗口内完成首屏；折叠展开、收起、向旧端滚动和
  plan breaker 均正常，日志无 exception 或 history error。
- 四个同时打开的 session 完成真实 history 冒烟后 RSS 约 188 MiB，进程峰值约 225 MiB；未观察到滚动停顿或崩溃。
- 本地试用包位于 `Kodex/out/kodex-0.2.6-local-linux-x64.tar.gz`，同目录包含 SHA-256 文件。
