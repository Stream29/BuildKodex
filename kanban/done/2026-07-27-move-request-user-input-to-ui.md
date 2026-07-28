# Task Tree

- [done] 将`request_user_input`收敛到CLI UI
  - [done] 核对AgentState、Runtime与CLI现有调用链
  - [done] 删除专用RequestUserInputRuntime及模块依赖
  - [done] 从单个ToolPending调用投影UI表单
  - [done] 由CLI直接构造输出并调用completeToolCall
  - [done] 迁移测试并修正相关清单
  - [done] 完成跨平台验证

# Details

`request_user_input`是宿主UI交互，不建立专用Runtime状态。CLI只在当前
`ToolPending`恰好包含一个`request_user_input`调用时展示表单，并直接通过
AgentState完成该调用。
