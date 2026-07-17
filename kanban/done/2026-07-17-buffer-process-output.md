# Task Tree

- [done] 将进程输出从 Channel 改为取消安全的合并缓冲区
  - [done] 定义公开的输出读取契约与终态语义
  - [done] 用读写锁和版本通知实现原子追加、消费、超时等待
  - [done] 将各平台进程会话接入缓冲区，并保留输入 Channel
  - [done] 覆盖输出合并、超时、取消和进程终止行为
  - [done] 验证 JVM、Node、Linux x64 测试与其余 host target 编译

# Details

输出读取不再直接消费 `ReceiveChannel`。`ProcessOutput` 合并 stdout/stderr 并序列化读取；内容只在无挂起的写锁临界区内取走。等待由版本 `StateFlow` 驱动，因此超时或取消等待不会移除内容。

JVM、Node JS、Linux x64 已通过真实进程测试；Linux ARM64 与 MinGW 测试源集已编译。macOS 实机验证未执行，因为当前 `macbook` 主机名无法解析。
