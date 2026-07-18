# Task Tree

- [done] 将模型目录改为自启动的内置目录
  - [done] 在构造函数注入Codex CLI storage，并由AutoCloseable拥有独立CoroutineScope
  - [done] 固化当前Rust内置模型的窗口元数据作为同步初始快照
  - [done] 后台依次导入本地缓存并刷新远端，失败时保留上一个有效快照
  - [done] 更新调用方和测试，验证加载顺序与上下文预算

# Details

目录构造不能执行阻塞I/O；它以硬编码内置模型立即发布StateFlow值，再在自身独立协程中完成缓存和远端刷新。`close()`取消该协程，但不关闭外部注入的OpenAiClient。
