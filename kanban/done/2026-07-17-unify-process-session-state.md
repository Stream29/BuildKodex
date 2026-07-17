# Task Tree

- [done] 统一 `ProcessSession` 的并发状态与事件顺序
  - [done] 收紧公开会话 API，移除 `Channel`、`Deferred` 和独立输出抽象
  - [done] 在单个会话状态机内原子管理输入、输出、生命周期与关闭
  - [done] 让各平台实现通过会话事件入口接入平台 I/O
  - [done] 将测试改为行为 API，并覆盖读取取消、输出保留与终态顺序
  - [done] 验证 JVM、Node、Linux x64 测试及其余 host target 编译

# Details

`ProcessSession` 现在直接提供 `write`、`read` 和 `close`。内部状态机在同一把读写锁下管理待写入输入、合并输出、进程终态、输入写入租约和资源关闭权；等待只观察版本 `StateFlow`，不会消费数据。

异常终态会先终止子进程，平台 I/O 资源只会在正在进行的 stdin 写入收尾后关闭。JVM、Node JS、Linux x64 已通过真实进程测试；Linux ARM64 与 MinGW 的主、测试源集已编译。macOS 实机验证未执行，因为当前 Linux 主机不能处理 macOS cinterop。
