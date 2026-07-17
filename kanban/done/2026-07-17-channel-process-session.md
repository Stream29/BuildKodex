# Task Tree

- [done] 将 `ProcessSession` 改为 Channel 驱动的公开 API
  - [done] 公开标准输入和合并输出 Channel，以及进程完成结果
  - [done] 将 `write` 与带 yield 的 `read` 设计为扩展函数
  - [done] 将 JVM、Node、POSIX、MinGW 实现迁移为统一的 Channel 语义
  - [done] 覆盖输出、续写、超时、取消与关闭行为

# Details

保持 unified-exec 的对外协议不变；该工具通过新的扩展函数消费进程会话。`input` 是可关闭的 UTF-8 标准输入 Channel，`output` 是单消费者的合并输出 Channel，`exitCode` 表示最终完成结果；`read(yieldTime)` 先消耗已就绪输出，再至多等待一个新 chunk 或关闭。

JVM、Node JS、Linux x64 已通过真实进程测试；Linux ARM64 与 MinGW 测试源集已编译。macOS 实机验证未执行，因为当前 `macbook` 主机名无法解析。
