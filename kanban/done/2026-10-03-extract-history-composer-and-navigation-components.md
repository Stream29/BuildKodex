# Task Tree

- `Inventory nine History item kinds, Composer, New Session and Session Tab Bar`()
- `Implement and compare original UI behavior`()
- `Audit all new concepts against the user's hard-migration rule`()
- `Delete parallel History and New Session models rather than bridge them`()
- `Integrate the real component owners and run downstream regression`()
- `Archive accepted real component cutovers and superseded parallel-model scope`()

# Details

## Reconciled status — 2026-10-07

- **DONE — 交付原 History、Composer、New Session 和 Tab Bar 的真实硬迁移**。
  九套伪 item API 与 Aggregate 提案被否决并删除，不是“九套新 API 已实现”。
- 原 `AgentHistoryViewModel`/item 家族的
  [renderer 状态解耦](2026-10-07-decouple-history-renderer-state.md)、
  [前端终验](2026-10-07-final-audit-frontend-ownership.md)与
  [限定收尾](2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)
  提供后来实际消费者、宿主测试、故障修复和独立复审；
  [Application exact-target 验收](2026-10-06-cutover-application-and-session-roots.md#coordinator-final-handoff-and-verification)
  接管旧 index 竞态。[核心 native E2E](2026-10-07-accept-sealed-cli-core-e2e.md)
  与 [发布](2026-10-07-release-kodex-0-4-10.md#publication-verified--v0410)
  已交付 main `83749434`。
- 下文“暂不归档”限定于 `821b6627` 当时 checkpoint，现由上述有界验收替代。
  原超时/未跑平台记录不抹除；Work 性能、窄屏双侧栏策略、History admission
  同 mount one-shot、kRPC/OS/平台 B2/U 与 [总迁移门槛](../executable/2026-09-28-plan-domain-spec-impl-boundaries.md)
  仍开放，不把完成迁移当作整仓无缺陷证明。

## Scope

- 用户选定九类 History Item、Composer、New Session、Session Tab Bar，共 12 类交互。
- 后续授权完整 AgentHistory 迁移，并确认一次性硬迁移；不以“新组件测试通过且旧组件
  仍生产运行”作为完成条件。
- Application Shell、Session Sidebar、agent-state 根模块和 wire/persistence 格式不重做。
- 用户已授权提交推送当前硬迁移切片；保留用户其他任务与共享 IDE。

## Current component boundaries

| Responsibility | Actual source of truth | Task |
| --- | --- | --- |
| Agent History and nine real item kinds | `app/component/history/{spec,impl/viewmodel,impl/view}` | [History hard cutover](2026-10-04-extract-history-aggregate-component.md), [item review](2026-10-03-extract-history-item-components.md) |
| Composer | `app/component/composer/{spec,impl/viewmodel,impl/view}` | [Composer](2026-10-03-extract-composer-component.md) |
| Original New Session owner and page | `app/component/new-session/{spec,impl/viewmodel,impl/view}` | [navigation](2026-10-03-extract-new-session-and-session-tab-bar-components.md) |
| Session Tab Bar presentation | `app/component/session-tab-bar/{spec,impl/view}` | [navigation](2026-10-03-extract-new-session-and-session-tab-bar-components.md) |

- 九类真实 item 类型与行为保留；撤回的是另造的九套平行类型与 Legacy renderer 桥。
- 不强行维持之前项目数量。若 further topic split 无法直接移动真实声明，则先保留其
  内聚组件边界，不公开内部 loading hooks 或堆 wrapper/resolver。
- 原 `AgentHistoryViewModel` 与原 `NewSessionViewModel` 是唯一身份；Aggregate/Timeline
  与 copied creation-state/effect/receipt controller 全部移除。
- storage/RPC 的真实依赖绑定、registry 的 stable child ownership 和 view 的纯 visual
  slot 保留；它们不是平行业务模型。

## Validation

- 硬迁移后旧生产声明与旧项目消失，不用 alias/forwarding build 修复编译。
- 检查 conventional plugin、spec 与 implementation 依赖方向。
- 原 History item state、稀疏分页、generation/nonce、关闭/取消及准确菜单目标回归。
- New Session 准确物化/替换、已知 allocation 保留、tab 生命周期和输入 cursor 回归。
- Composer submit/steer/pending input 与 Session Tab Bar 完整宿主/UI 回归。
- Application、Agent、Session、RPC、Integration 下游编译/测试。
- 对照真实原 UI，不用 placeholder、手动调用 callback 或忽略 renderer 来充当验收。
- diff/旧坐标/唯一声明/本地链接检查；未验证平台如实记录。

## Historical checkpoint — 821b6627

- 硬迁移与真实宿主接线已实施；最终 308 项定向 JVM 测试与 Integration 测试源码
  编译通过。History 压力用例的偶发超时未闭合，详见
  [结果与风险](2026-10-04-extract-history-aggregate-component.md#current-checkpoint)。
- 当前代码切片已提交为 `821b6627`；保留未闭合的压力用例风险及未验证平台，
  不将提交授权解释为这些风险已解决，任务暂不归档 done。
