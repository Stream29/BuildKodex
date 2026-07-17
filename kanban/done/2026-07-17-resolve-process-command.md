# Task Tree

- [done] 在进程边界解析命令配置
  - [done] 将可空工作目录与 shell 转为内部非空命令模型
  - [done] 仅由统一解析入口处理默认值与输入错误
  - [done] 让各平台 launcher 消费内部命令模型

# Details

本任务依赖平台 shell 建模完成；外部工具 DTO 的可空语义保持不变。`ExecCommandArguments` 在统一入口转换为带非空 `Path` 与 `Shell` 的 `ShellProcessCommand`，各平台 launcher 不再处理默认值或无效 shell。

JVM、Node JS、Linux x64 已通过真实进程测试；Linux ARM64 与 MinGW 测试源集已编译。macOS 实机验证未执行，因为当前 `macbook` 主机名无法解析。
