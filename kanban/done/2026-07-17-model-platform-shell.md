# Task Tree

- [done] 以平台 `expect` enum 建模 shell
  - [done] 定义 shell 的公共选择模型与解析边界
  - [done] 为 JVM、Node、POSIX、MinGW 提供严格的平台分发
  - [done] 覆盖默认 shell、显式 shell 与不支持 shell 的行为

# Details

LLM 输入仍可携带可空的 shell 字符串；进程实现只接收完成解析后的平台 shell。模型提供的字符串只用于选择已知枚举值，不会作为可执行路径传入 launcher。

JVM、Node JS、Linux x64 已通过真实进程测试；Linux ARM64 与 MinGW 测试源集已编译。macOS 实机验证未执行，因为当前 `macbook` 主机名无法解析。
