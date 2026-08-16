# Hooks

实现或修改 Hooks 时遵守以下决策。

- 在顶层`hook/`下保留`hook:contract`、`hook:impl`和`hook:tool-utils`模块。
- `hook:contract`定义Kodex原生配置、`HookManager`、窄运行时端口、事件request/result和公共context；不得依赖Codex storage、进程执行或Runtime实现。
- `KodexHooks`聚合`TurnHooks`、`ToolHooks`和`CompactionHooks`；消费方仍只依赖所需窄端口。
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
- 支持`pre_tool_use`、`post_tool_use`、`user_prompt_submit`、`stop`、`pre_compact`和`post_compact`。
- 不支持`permission_request`、Session、Subagent或Codex专属事件。
- 不提供matcher；type触发时运行该type的全部Hook，命令需要筛选事件时自行读取stdin中的payload。
- `HookManager`只提供按名称添加、编辑和删除；重命名必须保留原配置位置，并拒绝名称冲突。
- Settings主层显示`Hooks`、`Add`和每个`名称 · type`条目；详情只显示名称与type，完整command只在显式编辑时读取。
- 应用通过`CoroutineScope.KodexHooksImpl`工厂传入源自`KodexGlobalSettings`的`StateFlow<HookSettings>`；所有打开的Agent共用同一实例。
- `KodexHooksImpl`只在配置变化时按type解析命令；每次调用读取一次不可变快照。
- Hook名称直接作为运行身份；不得根据列表位置生成ID。
- Hook进程使用当前Agent cwd、宿主进程环境和固定600秒超时，通过默认shell执行command。
- 每个命令的stdin是一个Kodex原生JSON对象，包含`name`、`type`、`session_id`、`turn_id`、`cwd`、`model`和事件专属`payload`。
- Tool payload使用`tool_name`、`tool_input`、`tool_use_id`；post额外包含`tool_response`。
- User prompt payload只包含`prompt`。
- Stop payload包含`stop_hook_active`和可空`last_assistant_message`。
- Compaction payload只包含`trigger`，取值为`manual`或`auto`。
- Hook协议不得发送`hook_event_name`、`transcript_path`、`permission_mode`或其他Codex兼容字段。
- `pre_tool_use`输出只接受`{"action":"continue"}`或`{"action":"block","reason":...}`；block不得修改工具输入。
- `user_prompt_submit`输出只接受`continue`或`block` action，并可携带一个`context`和可空`reason`。
- `stop`输出只接受`finish`、携带非空`prompt`的`continue`，或携带可空`reason`的`stop`。
- `post_tool_use`、`pre_compact`和`post_compact`忽略命令输出。
- 控制型输出使用严格JSON解码；空输出、未知字段、非法JSON、启动失败、超时和任意非零退出码均fail-open。
- 退出码2没有特殊语义；stderr不参与控制结果。
- Coroutine cancellation必须原样传播。
- `pre_tool_use`按声明顺序串行执行；第一个block终止Hook链并阻止工具。
- `user_prompt_submit`按声明顺序串行执行并依次累积context；第一个block终止Hook链。
- `stop`按声明顺序串行执行；`finish`继续检查下一个Hook，第一个有效`continue`或`stop`终止Hook链。
- Stop continuation的`hookRunId`使用产生continuation的Hook名称。
- `post_tool_use`、`pre_compact`和`post_compact`的同type命令并发执行。
- Tool Hook覆盖普通本地工具、MCP和`update_plan`；宿主直接处理的`request_user_input`与client tool search不进入Tool Hook路径。
- PostToolUse只观察成功完成的工具调用，不修改工具输出。
- 用户输入先持久化，再执行UserPromptSubmit；additional context作为developer-role history持久化。
- Stop Hook在自然结束候选或唯一pending调用为`request_user_input`时执行；其continuation作为实际user-role消息持久化。
- root与subagent当前都安装相同的Turn、Tool和Compaction Hook链。
- 所有手动和自动compaction统一经过`PreCompact -> compaction core -> PostCompact`；Compaction Hook不能阻止compaction。
- Hook不定义专属观测sink；运行状态统一归入Runtime观测系统。
