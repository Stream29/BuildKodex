# Task Tree

- `实施严格等待参数与源码版本激活`()
- `交接源码与隔离验证结果`()

# Details

- 父任务：[Agent 等待锁体验](../executable/2026-10-02-cross-platform-agent-synchronization.md)，P2 及 runtime、RPC、展示验证。
- 用户已授权源码实施；应用版本及业务协议 checkpoint `0.5.2`，管理协议不变。
- 不写 `Kodex/app/migration/`、新增等待回归文件或 unified-exec build 文件；其他构造点和 P2/P4 所列代码由本子任务负责。
- 不操作真实 Home，不提交或发布，不运行 Gradle；主代理统一验证。
- 已移除 yield 默认值和默认值隐藏逻辑，更新必填 schema、两个十分钟上限、构造点与历史显示；应用源码/业务协议为 `0.5.2`，控制协议仍为 `1`。
- 主代理核对标准 Long decoder 的接受范围并修正错误测试假设；参数、投影、runtime、UI、协议及保活相关 JVM 检查通过。完整 RPC models 的独立设置断言失败已在原始 HEAD 复现，未顺带修改；计数与未测项见父任务。
