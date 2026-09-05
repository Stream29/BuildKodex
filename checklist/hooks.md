# Hooks

实现或修改 Hooks 时遵守以下决策。

- 在顶层`hook/`下保留`hook:contract`、`hook:impl`和`hook:tool-utils`模块。
- `hook:contract`定义Kodex原生配置、`HookManager`、窄运行时端口、事件request/result和公共context；不得依赖Codex storage、进程执行或Runtime实现。
- `KodexHooks`聚合`TurnHooks`、`ToolHooks`、`CompactionHooks`和`ErrorHooks`；消费方仍只依赖所需窄端口。
- `KodexGlobalSettings.hooks`是唯一持久化与运行时真源；Hooks不得读取或导入Codex配置。
- Hooks配置直接使用保持声明顺序的名称映射：

```yaml
hooks:
  guard_tools:
    type: pre_tool_use
    command: ./hooks/guard-tools.sh
```

- 映射key是非空、全局唯一且稳定的Hook名称；同一type可以配置多个不同名称。
- Hook body只能包含`type`和`command`；不得增加source、matcher、environment、timeout、enable、status或导入来源字段。
- 支持`pre_tool_use`、`post_tool_use`、`user_prompt_submit`、`stop`、`pre_compact`、`post_compact`和`unhandled_error`。
- 不支持`permission_request`、Session、Subagent或Codex专属事件。
- 不提供matcher；type触发时运行该type的全部Hook，命令需要筛选事件时自行读取stdin中的payload。
- `HookManager`只提供按名称添加、编辑和删除；重命名必须保留原配置位置，并拒绝名称冲突。
- Settings主层显示`Hooks`、`Add`和每个`名称 · type`条目；详情只显示名称与type，完整command只在显式编辑时读取。
- 应用通过`CoroutineScope.KodexHooksImpl`工厂传入源自`KodexGlobalSettings`的`StateFlow<HookSettings>`；所有打开的Agent共用同一实例。
- `KodexHooksImpl`只在配置变化时按type解析命令；每次调用读取一次不可变快照。
- Hook名称直接作为运行身份；不得根据列表位置生成ID。
- Hook进程使用当前Agent cwd、宿主进程环境和固定600秒超时，通过默认shell执行command。
- 每个命令的stdin除`unhandled_error`外是一个Kodex原生JSON对象，包含`name`、`type`、`uri`、`turn_id`、`cwd`、`model`和事件专属`payload`。
- `unhandled_error`命令的stdin为纯文本UTF-8的`exception.message`（若为null则传入空字符串`""`），并在写入后立即关闭stdin，末尾不补换行符；不包含JSON包装、堆栈、cause或多余元数据。
- Tool payload使用`tool_name`、`tool_input`、`tool_use_id`；post额外包含`tool_response`。
- User prompt payload只包含`prompt`。
- Stop payload包含`stop_hook_active`和可空`last_assistant_message`。
- Compaction payload只包含`trigger`，取值为`manual`或`auto`。
- Hook协议不得发送`hook_event_name`、`transcript_path`、`permission_mode`或其他Codex兼容字段。
- `pre_tool_use`输出只接受`{"action":"continue"}`或`{"action":"block","reason":...}`；block不得修改工具输入。
- `user_prompt_submit`输出只接受`continue`或`block` action，并可携带一个`context`和可空`reason`。
- `stop`输出只接受`finish`、携带非空`prompt`的`continue`，或携带可空`reason`的`stop`。
- `post_tool_use`、`pre_compact`、`post_compact`和`unhandled_error`忽略命令输出。
- 控制型输出使用严格JSON解码；空输出、未知字段、非法JSON、启动失败、超时和任意非零退出码均fail-open。
- `unhandled_error`检查每个命令的返回结果；非零退出码或无可用退出状态均记录Hook名称与已知状态，不重试、不递归报告；协程取消仍原样传播。
- 退出码2没有特殊语义；stderr不参与控制结果。
- Coroutine cancellation必须原样传播。
- `pre_tool_use`按声明顺序串行执行；第一个block终止Hook链并阻止工具。
- `user_prompt_submit`按声明顺序串行执行并依次累积context；第一个block终止Hook链。
- `stop`按声明顺序串行执行；`finish`继续检查下一个Hook，第一个有效`continue`或`stop`终止Hook链。
- Stop continuation的`hookRunId`使用产生continuation的Hook名称。
- `post_tool_use`、`pre_compact`、`post_compact`和`unhandled_error`的同type命令并发独立执行。
- Tool Hook覆盖普通本地工具、MCP和`update_plan`；宿主直接处理的`request_user_input`、
  `suggest_subagent_task`与client tool search不进入Tool Hook路径。
- PostToolUse只观察成功完成的工具调用，不修改工具输出。
- 用户输入先持久化，再执行UserPromptSubmit；additional context作为developer-role history持久化。
- Stop Hook在自然结束候选或唯一pending调用为`request_user_input`时执行；其continuation作为实际user-role消息持久化。
- 每个打开的root Agent都安装同一套Turn、Tool和Compaction Hook链；当前不存在本地subagent runtime或父子Hook链。
- 所有手动和自动compaction统一经过`PreCompact -> compaction core -> PostCompact`；Compaction Hook不能阻止compaction。
- Hook不定义专属观测sink；运行状态统一归入Runtime观测系统。
- `unhandled_error`仅在操作最终失败边界（如Agent/Session异常通知、设置更新队列失败）触发一次；协程取消（CancellationException）、重试中的短暂错误或已转为普通工具返回结果的错误绝不触发。
- 应用共享报告入口先记录原始Throwable，再在应用拥有的任务中投递message；未配置Hook也保留日志，接入该入口的Settings队列不重复记录同一异常。
- 异常Hook使用操作开始或排队提交时捕获的来源Agent cwd；无来源Agent时回退应用启动目录，不从当前选中会话或失败后的设置推导目录。
- 打开会话由Registry报告；Fork初始化失败由Registry报告，已取得Session后的完整Fork操作由Session报告；Catalog只补报Fork成功后的刷新失败；Revert校验与执行失败由Agent报告，UI不重复报告。
- Session关闭不取消应用拥有的异常通知；应用关闭取消通知任务及命令，不保证进程崩溃时必达。
