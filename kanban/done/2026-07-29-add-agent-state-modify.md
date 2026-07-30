# Task Tree

- [done] 为AgentState增加通用storage修改原语
  - [done] 将`modify`加入AgentState contract并实现统一状态刷新
  - [done] 移除AgentState重复的initialize和revert API
  - [done] 将调用方迁移到storage原语
  - [done] 更新测试与设计规则
  - [done] 运行受影响构建和测试

# Details

- `modify`独占一次外部写入，向block临时暴露`MutableKodexAgentStorage`。
- block结束后从storage重新发布`latestIndex`与AgentState值。
- 初始化、fork和revert继续由AgentStorage原语定义。
