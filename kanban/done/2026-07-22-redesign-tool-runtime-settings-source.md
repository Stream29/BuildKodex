# Task Tree

- [done] 重新设计Tool Runtime与AgentSettings边界
  - [done] 追溯新增`KodexAgentRuntime.resume(tools: List<ToolSpec>)`及相关修改的动机
  - [done] 盘点Tool Runtime重构对设置来源与运行时投影的影响
  - [done] 设计保持`AgentSettings`单一事实来源的方案
  - [done] 收敛实现与测试
    - [done] 删除AgentState和AgentRuntime的request-local tools重载
    - [done] 让Tool Runtime按resume固定完整tool generation
    - [done] 在请求前将generation原子同步到`KodexAgentSettings.tools`
    - [done] 让handler、tool search索引和请求投影使用同一generation
    - [done] 更新CLI组装和回归测试

# Details

- 用户已授权按任务顺序实施。
- `KodexAgentSettings.tools`是Responses请求工具定义的唯一事实来源，不允许request-local参数覆盖。
- Tool Runtime在一次`resume()`开始时固定完整tool generation；仅当即将发起请求且tools发生变化时，先通过`updateSettings`持久化该generation。
- 同一次`resume()`中的工具执行、tool search与后续请求复用同一generation。
- Agent Runtime的JVM、JS/Node和Linux Native测试通过；CLI common metadata编译通过。
- CLI平台入口仍受既有Mosaic API未解析问题阻塞，不属于本任务修改范围。
