# Task Tree

- [done] 对齐`Ultra` reasoning的Multi-agent策略入口
  - [done] 确认`Ultra`复用Multi-agent V2工具与coordinator，不建立独立runtime
  - [done] 根据reasoning effort投影`ExplicitRequestOnly`或`Proactive`策略
  - [done] 将策略渲染为请求级developer instruction
  - [done] 保持Responses API请求投影为`max`
  - [done] 增加同时验证请求投影、默认限制与proactive行为的对齐测试
  - [done] 运行相关KMP测试与CLI装配检查

# Details

`Ultra`是仅由部分模型声明的组合入口：请求层投影为`max`，同时把现有Multi-agent V2的提示词策略切换为`Proactive`。工具执行、Agent调度和生命周期继续复用既有coordinator。

已完成JVM、Node、Linux Native和macOS Native测试，并完成Linux/macOS CLI装配编译。
