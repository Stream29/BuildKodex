# Task Tree

- [done] 将无终态流结束视为可续跑
  - [done] 让requestResponseApi在缺少协议终态时返回`RequestFinish.Resumable`
  - [done] 删除不再使用的stream-ended异常并更新契约记录
  - [done] 更新EOF行为测试
  - [done] 运行相关Gradle验证

# Details

- 用户明确接受无终态EOF自动续跑的风险，后续出现问题再调整策略。
- `response.failed`与`response.incomplete`仍以保留诊断信息的异常传播。
- 仅修改AgentState缺省返回、OpenAI异常模型、对应契约文档与测试；compaction runtime已有`Resumable`续跑和成功日志，无需新增分支。
- 验证通过：`openai-models:jvmTest`、`agent-state-impl:jvmTest`、`agent-runtime-decorator-compact:jvmTest`。
