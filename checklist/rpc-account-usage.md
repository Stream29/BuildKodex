# RPC 账号用量

- 按 [RPC 迁移适用范围](rpc-architecture.md#适用范围)使用本文件。
- GlobalRpc 提供 getAccountUsage/getAccountUsageFlow/refreshAccountUsage，不另建账号用量服务。
- 复用 SettingsAccountUsageState 的 Unavailable、Loading、Available、Failed、Redeeming 与原 CodexAccountUsageSnapshot，不复制 DTO。
- Get/GetFlow 遵循 [RPC 状态](rpc-state.md#get-与-getflow)：Get 初始化，Flow 提供当前值及后续变化。
- refreshAccountUsage 要求后端重新请求供应商用量，不是前端缓存刷新，也不消费 reset credit。
- 刷新结果由状态流体现；Unit 返回不保证 Available 或前端缓存已同步，失败可由 Failed 表达，认证不可用时保持 Unavailable。
- previous 仅保留同账号的 fallback；账号改变时清除旧快照，不将旧账号用量用于新账号。
- 用量状态是后端共享的非持久化投影，不混入 BackendSettings、认证凭据或前端弹窗状态。
- 认证与用量流不承诺联合原子快照；前端不得用混合快照绕过后端账号校验。
- 只序列化公开状态与其实际使用的嵌套值；Redeeming 不传私有 reset attempt 或幂等数据。
- 保留原可空字段语义，区分未知计数与零、不可用详情与已知空列表。
- 单个前端取消观察不关闭共享 usage store。

## 重置券选择

- GlobalRpc 使用单一 `consumeUsageReset(creditId: String): CodexRateLimitResetOutcome`；复用原结果枚举，不增加 prepare、attempt 或结果重放 RPC。
- 重置必须由前端明确选择具体券并二次确认，提交非空 creditId；不提供 null、空串或后端自动选券的回退。
- 从已有 usage 快照读取券 id 与日期，选择列表和确认界面须让用户辨认所选券的到期日期；未知日期明确表示未知，不伪装为永久有效。
- 只有可用次数但没有具体明细时，不允许发起不指定券的重置；提示明细不可用并允许刷新。
- 后端只消费指定券，保留账号校验；券失效或不可用时报告失败/原业务结果，不静默换用另一张。
- 按调用准入时的当前账号提交指定 creditId，依赖供应商校验账号与券的有效性；不新增前端确认期的账号参数、身份标记或绑定保证，不恢复 prepare/attempt RPC。
- 不把 attempt 暴露为前端 RPC 协议；供应商幂等键由后端内部处理。
- 调用失败后前端刷新用量并由用户决定是否再次操作，不自动重新消费；刷新失败不伪装为已取得最新状态。
- 保留 Reset/NothingToReset/NoCredit/AlreadyRedeemed 的确定结果；已取得消费结果后，用量刷新失败不抹掉该结果。
