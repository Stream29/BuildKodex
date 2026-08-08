# Task Tree

- [done] 实现 Codex 账号 Usage 与 Reset
  - [done] 对齐账号 Usage、Rate Limit 与 Reset 协议
  - [done] 明确 Client、状态 Store 与认证边界
  - [done] 扩展 OpenAI 账号协议与 Client
    - [done] 增加 Usage、Reset Credit 与 Token Usage DTO
    - [done] 增加读取、兑换接口与 HTTP 状态解码
    - [done] 覆盖 DTO 与成功响应解码
  - [done] 实现独立账号 Usage Store
    - [done] 观察认证切换并发布账号隔离快照
    - [done] 聚合 Rate Limit、Reset Credit 与 Token Usage
    - [done] 使用可重试幂等请求兑换并刷新快照
    - [done] 覆盖纯映射与状态行为
  - [done] 集成 Global Settings
    - [done] 显示 Usage、重置时间与可用 Reset
    - [done] 增加刷新、选择、确认、兑换和重试交互
    - [done] 覆盖 Usage 设置内容渲染
  - [done] 完成应用接线与验证
    - [done] 接入应用生命周期与 CLI 组合
    - [done] 更新相关 Checklist 与任务记录
    - [done] 运行格式化、测试和编译检查
    - [done] 审查最终 Diff 与残留文件

# Details

- 用户要求在当前 Global Settings 中查看 Codex 账号 Usage，并使用可用的 Usage Limit Reset。
- 远端读取与兑换操作属于 OpenAI Client 边界；认证状态只提供请求凭据和账号身份。
- 可观察的账号 Usage 快照由独立应用级 Store 管理，不进入 `KodexAuthState` 或持久化设置。
- Client 使用 ChatGPT backend-api 的 `wham/usage`、`wham/rate-limit-reset-credits`、`wham/rate-limit-reset-credits/consume` 与 `wham/profiles/me`。
- Store 以 Rate Limit 响应为必要数据；Reset Credit 详情和 Token Usage 允许部分不可用，并在认证账号切换时立即清除旧账号快照。
- 每次逻辑兑换生成一个 UUID 幂等键；传输失败后的 UI 重试复用同一兑换尝试。
- 兑换请求携带发起时的认证快照并在发送前校验，避免认证切换时把Reset提交到其他账号；缺少账号ID时以Access Token作为快照隔离键。
- Global Settings 内直接显示主 Rate Limit、Token 汇总和 Reset 数量；兑换前必须选择可用 Credit（详情缺失时由后端选择）并再次确认。
- 兑换成功或已兑换后重新读取 Usage；其他业务结果显示明确说明，网络失败保留同一幂等尝试供重试。
- 验证不调用真实兑换接口；使用序列化、纯映射、状态与 Mosaic 渲染测试，并执行相关 Gradle 检查。
