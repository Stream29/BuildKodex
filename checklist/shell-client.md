# Shell Client

- 隐藏`ShellClient`构造器，只允许通过`CoroutineScope.ShellClient()`创建。
- 每个`ShellClient`必须是创建方scope中可独立取消的子节点。
- 资源所有者必须自行持有并关闭自己的`ShellClient`。
- 不同功能边界不得共享`ShellClient`；Hook与unified exec分别持有独立实例。
- 关闭所有者或父scope时，必须级联取消全部进程session与IO任务。
- Unified exec通过`StateFlow<UnifiedExecSettings>`观测全局shell设置。
- 新进程启动时读取当前全局shell；已运行进程不随设置变化。
- 工具调用显式指定的shell优先于全局设置。
- 普通管道进程必须复用`utils:process-client`；`ShellClient`只负责shell调用、文本IO与PTY。
