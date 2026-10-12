# Ultrafast 返回档位与告警判定

## 覆盖与证据

- 2026-10-12 11:03 UTC 的只读调查；未访问凭据、发送模型请求或控制共享 GUI。
- 本机 Session 730–790 的 response diagnostics 中，已记录的请求/返回组合为：
  Default → default 5011 条，Priority → default 282 条，
  Ultrafast → default 1928 条。仅统计有 requested_service_tier 的已保存记录；
  不是请求总数、速度测量或失败率，活动 Session 后续可能继续增加记录。
- 其中 Session 772–790 有 439 条 Ultrafast 记录，两模型分别为 Sol 392、
  Astra 47，返回值全部为 default。
- [Session 790 的实例](file:///home/stream/.kodex/sessions/790/token-count/5.json#L1)
  保存 requested_service_tier=ultrafast、service_tier=default、正常输出 usage。
  未确认它就是用户看到提示的那次请求。
- [记录投影](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L595-L607)
  直接保存 response.serviceTier，没有把 null 补成 default。
- [修复前的告警判定](https://github.com/Stream29/Kodex/blob/bbbc9525d5aec1bab6244ad44760dbeea9939e80/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/UltrafastTierWarnings.kt#L28-L42)
  仅比较请求和返回字符串；不检查请求失败、服务端拒绝、实际路由或速度。
  缺失/空白字段不告警，未知非空字符串也会告警。
- [上次错误出口修复](../../kanban/done/2026-10-12-restore-agent-error-hook.md#L10-L18)
  保留了 Warning，当前 Ultrafast 是唯一生产 AgentNotification 的路径。

## 官方资料与源码核对

- 检索时间同上；以下为选择性摘要，不是官方文档副本。
- [Ultrafast API guide](https://developers.openai.com/api/docs/guides/ultrafast-mode)
  规定 API 请求 service_tier=ultrafast，HTTP 也受支持。
  API-key 路径不能自动视为 ChatGPT Codex 订阅后端的回报契约。
- [Codex Speed](https://learn.chatgpt.com/docs/agent-configuration/speed)
  区分订阅访问/用量与 API-key 计费；未在所查页面给出
  Codex 订阅 response.service_tier 必须回显 ultrafast 的保证。
- [Fast mode FAQ](https://help.openai.com/en/articles/11647665-fast-mode-faq)
  的 API alpha 要求额外 OpenAI-Service-Tier header，仍与 API guide
  示例不同；不能据此向现有订阅请求增加 header。
- [上游 rust-v0.162.0 SSE 解析](https://raw.githubusercontent.com/openai/codex/rust-v0.162.0/codex-rs/codex-api/src/sse/responses.rs)
  的 ResponseCompleted 提取 id、usage、usage_metadata、end_turn，
  没有将 service_tier 解析成完成事件中的路由证明。
  本地参考 commit `604061ce51d194a3aa6aad3b3170240d096e1725` 具有同样结构。
- Kodex 当前 Ultrafast 请求仍发送 body 档位与
  x-codex-routing-hint=model=…;tier=ultrafast，不因告警改写选择。

## 结论边界

- 已确定客户端告警的完整原因：把非空返回值与请求值不相等提升为 Warning。
- 已有样本表明 default 不只出现在 Default 请求；单独使用这个字段不足以
  判断用户所述正常快速响应异常，不能作为已证实降级或无权限的证据。
- 尚不能确定服务端为何填写 default，或证明每次请求的实际硬件/路由/速度。
  “字段可能属于外层通用回报，而非 Ultrafast 路由证明”仅为待验证解释；
  没有服务端实现、明确回报契约或路由遥测，不将其写成事实。
- 用户已明确真正的错误和警告都走通知 Hook；是否构成警告与通知出口
  必须分开处理，不能把这条未证实异常改送错误 Hook 当作修复完成。
- 最终出口决定为 Warning 复用现有 Error Hook；不新增 Hook 类型/JSON，
  成功响应的 tier 差异仅保留诊断。实施与验证见
  [本次任务](../../kanban/done/2026-10-12-ultrafast-warning-and-hook-routing.md)。
