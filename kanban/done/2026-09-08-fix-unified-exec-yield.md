# Task Tree

- [done] 修复短命令被提前拆成两次工具调用
  - [done] 对照 Rust 确认等待语义偏差
  - [done] 明确工具层修复与回归测试方案
  - [done] 实现等待退出或期限到达后读取输出
  - [done] 覆盖短命令、分批输出、续写与超时轮询
  - [done] 验证 JVM 与 Linux native 真实进程测试

# Details

- 用户已要求修复；仅调整 unified-exec 等待策略，不改变底层输出缓冲区即时读取契约。
- Rust 在等待期限内持续收集输出，不因第一批输出到达而返回。
- 复用 ShellClient 已有的有界输出缓冲区：等待 `exitCode` 或超时，再 `drain()`，避免新增累积缓冲区。
- 已检查普通管道与 POSIX PTY：退出状态在输出读取、缓冲刷新后完成；无需复制 Rust 独立输出关闭信号的等待机制。
- 修复前 JVM 测试 21 项中 4 项回归测试失败，确认可以检出本问题。
- 修复后 JVM、Linux x64 各 21 项测试通过，并强制复跑通过；包含真实管道、PTY、短命令单次完成、分批输出、续写、期限返回、取消与会话清理。
- 验证使用现有 JDK 25：`:tool-unified-exec-impl:jvmTest --rerun :tool-unified-exec-impl:linuxX64Test --rerun`。
- 验证命令需设置 `TMPDIR=/tmp`：当前 kotlinx-io 在 Linux 未设置 `TMPDIR`/`TMP` 时返回空临时目录，工作目录测试会因此失败；未修改生产代码绕过此环境问题。
- 将真实工作目录测试改用真实时间；交互测试在累计输出中检查 `ready`，不再假设 shell 在 250 毫秒内输出。
- 等待语义已记录于 [Shell Client](../../checklist/shell-client.md)；`git diff --check` 通过。IDE 对生产文件未报问题；最终测试文件检查超时，编译与执行检查已通过。
- 未进行 macOS、Windows、JS 实测；未替换运行中的 CLI、发布版本或创建提交。
