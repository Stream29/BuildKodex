# Task Tree

- [done] 实现 `exec_command` 与 `write_stdin`
  - [done] 建立 host-only 异步进程与会话基建
  - [done] 定义并实现 `exec_command` JSON tool
  - [done] 定义并实现 `write_stdin` JSON tool
  - [done] 覆盖进程输出、会话续写、超时与取消行为

# Details

不实现 sandbox、权限审批或远程环境选择。两个工具共享本地进程会话管理器，并只支持 host convention 覆盖的 target。

`tty=true` 明确返回工具失败，直到存在跨平台 PTY 后端。JVM、Node JS、Linux x64 使用真实子进程通过输出、续写、超时和取消测试；Linux ARM64 与 MinGW 测试源集已编译。macOS 实机验证未执行，因为当前 `macbook` 主机名无法解析。
