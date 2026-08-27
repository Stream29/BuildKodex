# Task Tree

- [done] 在compaction保留前缀中加入PlanUpdate
  - [done] 扩展现有保留策略
    - [done] 保持UserMessage保留行为
    - [done] 将PlanUpdate完整交互加入保留项
    - [done] 对两类保留项沿用现有窗口预算和超限淘汰
  - [done] 添加针对性测试
    - [done] 验证UserMessage和PlanUpdate按原顺序保留
    - [done] 验证窗口超限时淘汰旧保留项
    - [done] 验证其他history item不被额外保留
  - [done] 运行相关Agent State测试

# Details

- 实现路径已经确认，已获授权进入实现。
- 改动仅限Remote Compaction V2完成后的本地保留策略。
- 不调整远端压缩方法、触发条件、checkpoint结构或token计数语义。
- 服务端返回的encrypted compaction item保持不变。
- `RemoteCompactionV2.kt`现在将完整、成功的`update_plan` function call/output恢复为`StablePlanUpdate`，与UserMessage共同进入现有64,000-token保留窗口。
- 验证通过：`:agent-state-impl:jvmTest`与`:agent-state-impl:allTests`。
- IntelliJ IDEA正在运行该项目，但MCP连接已丢失，无法取得IDE inspection结果。
