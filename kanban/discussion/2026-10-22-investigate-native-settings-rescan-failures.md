# Task Tree

- `Record the two exact prebatch Native settings failures`()
- **`Discuss whether to investigate settings rescan failure handling`()**
- `Plan an isolated investigation if authorized`()

# Details

- 集中 Native 验收发现 agent-session/filesystem 的两个测试失败：
  `failed settings rescan after published io rejects stale runtime CAS until reopen`
  与 `failed settings rescan after published cancellation rejects stale runtime CAS until reopen`。
  预期分别是 IOException/CancellationException，实际为 null，均为 AssertionError。
- 在1525份源码哈希完全匹配的本批 Native/Home/MCP 迁移前工作区快照上，
  以同名/同类型/同消息复现；两次测试源 SHA-256 均为
  `86a907e03797331c8b2b667c19dbbad15218a4a1e86b56273d7b0635abd50719`。
  这是含此前迁移的工作区快照，不是干净 HEAD，也不足以归因到更早历史。
- 新 SafeRw 消费者7项和本 suite 其余107项通过。本批未改存储算法或原断言，
  不为交付 MCP/Native 迁移而掩盖失败。本记录不构成修复授权。
- 临时基线已清理，XML和精确比较回执保留：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/native4-baseline-comparison.json`。
