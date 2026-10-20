# Shell Client

- 隐藏`ShellClient`构造器，只允许通过`CoroutineScope.ShellClient()`创建。
- 每个`ShellClient`必须是创建方scope中可独立取消的子节点。
- 资源所有者必须自行持有并关闭自己的`ShellClient`。
- 不同功能边界不得共享`ShellClient`；Hook与unified exec分别持有独立实例。
- 关闭所有者或父scope时，必须级联取消全部进程session与IO任务。
- Unified exec通过`StateFlow<UnifiedExecSettings>`观测全局shell设置。
- 新进程启动时读取当前全局shell；已运行进程不随设置变化。
- 工具调用显式指定的shell优先于全局设置。
- Unified exec每次调用应等待进程结束或yield期限到达，再读取已有的有界输出缓冲区；不得因第一批输出到达就提前返回。
- 普通管道进程必须复用`utils:process-client`；`ShellClient`只负责shell调用、文本IO与PTY。

## 等待参数调整

- 2026-10-20 / Session989：`exec_command` 与 `write_stdin` 的 `yield_time_ms` 均改为模型调用必填，要求 agent 按命令选择等待时间，不再依赖省略参数的默认等待；适用于空输入 poll 和非空 stdin 写入。
- 两个工具显式等待时间的最大值统一为 `600000` 毫秒（10 分钟）；这是单次调用的最长等待窗口，提前完成就提前返回，到期仍运行则返回 session 及已有输出，不将 yield 当成命令执行超时。
- Home `0.5.2` migration 一次性补齐旧 typed command 历史；新 `ExecCommandArguments`/`WriteStdinArguments` 的 yield 字段无默认值，不在普通运行解码器中保留旧格式兼容。迁移缺失值分别使用历史值 `10000`/`250` 毫秒，已有 Long 值保持不变；遵循 [Home 迁移规则](kodex-home.md)。
- 目标是让多数命令在一次工具调用内取得最终结果，减少纯续等的模型调用；复用系统锁，不新增锁系统。实施方案、历史参数兼容及验证跟踪见 [Agent 等待锁体验](../kanban/executable/2026-10-02-cross-platform-agent-synchronization.md)。
- 本次仅完成 `0.5.2` 源码改动及可用环境验证，未发布、安装或迁移真实 Home；未测平台与验收状态以任务记录为准。
