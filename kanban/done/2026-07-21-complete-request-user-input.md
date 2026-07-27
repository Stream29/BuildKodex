# Task Tree

- [done] 补齐`request_user_input`
  - [done] 明确UI作为pending tool call的host处理边界
  - [done] 对齐上游响应模型和序列化格式
  - [done] 将ToolSpec注册进Agent settings的可用工具集合
  - [done] 在CLI中渲染问题并收集结构化答案
  - [done] 使用`completeToolCall`逐个回填调用结果
  - [done] 覆盖单个和同批多个pending call的测试
  - [done] 在真实PTY中验证完整交互链路

# Details

`request_user_input`不需要专用Runtime，也不需要额外的host交互状态。

- Agent settings注册其ToolSpec，但不把它加入`CodexToolRuntime`的自动handler集合。
- UI调用`resume()`后读取`CodexAgentStateValue.ToolPending`，按tool name识别并解析`RequestUserInputArgs`。
- UI将填写结果编码为`RequestUserInputResponse`，构造对应`FunctionCallOutput`并调用`completeToolCall`。
- 当前批次全部完成后，UI再次调用`resume()`。
- 多个pending tool call复用现有逐个完成语义，不增加第二套状态机。
- 权限确认和Apps consequential tool确认属于另一类host交互，不并入本任务。

当前已有输入模型和ToolSpec；缺少响应模型、ToolSpec注册、CLI交互和结果回填。
