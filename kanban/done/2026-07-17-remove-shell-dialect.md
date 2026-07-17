# Task Tree

- [done] 移除公开的 `ShellDialect`
  - [done] 将命令行参数与工作目录包装收回各平台 `Shell` 实现
  - [done] 删除 `Shell.dialect` 及其生产代码调用点
  - [done] 让 common 进程测试通过平台 test helper 选择 shell 命令
  - [done] 验证 JVM、Node、Linux x64 测试与其余 host target 编译

# Details

`Shell` 是进程层唯一的公开 shell 模型；命令语法分类只属于平台实现细节。

POSIX 的 shell invocation 负责工作目录包装；JVM、Node 与 MinGW 继续使用各自进程 API 的工作目录参数。JVM、Node JS、Linux x64 已通过真实进程测试；Linux ARM64 与 MinGW 测试源集已编译。macOS 实机验证未执行，因为当前 `macbook` 主机名无法解析。
