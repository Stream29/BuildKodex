# Task Tree

- [done] 将进程标准输入改为确认写入的 `SendChannel<String>`
  - [done] 用 rendezvous `SendChannel<PendingStdin>` 串行化平台写入
  - [done] 将写入完成或失败回传给公开 `send`
  - [done] 区分 stdin EOF、进程终态与会话取消
  - [done] 更新各平台的 stdin 关闭路径
  - [done] 覆盖真实进程下的写入确认与关闭行为

# Details

公开 `send` 在内部平台 `writeToProcess` 完成后才返回。内部 Channel 不缓冲输入；进程结束或会话取消时必须唤醒已挂起的发送方。`trySend` 只表示即时交接；由于 `SelectClause2` 是 sealed 类型，`onSend` 不能安全映射为 `PendingStdin`，因此明确拒绝该用法。

已通过 Linux JVM、Node JS、Linux x64 Native 与 macOS ARM64 Native 的真实进程测试；Linux ARM64 与 MinGW x64 测试源集已编译。
