# Task Tree

- `Review original nine item contracts and real renderer behavior`()
- `Audit the attempted split for parallel state and lifecycle wrappers`()
- `Discard the rejected duplicate item projects and Legacy renderer bridges`()
- `Move the real item family with AgentHistoryViewModel`()
- `Validate item and host behavior after the hard cutover`()
- `Archive real item-family cutover and supersede the rejected nine-project proposal`()

# Details

## Reconciled status — 2026-10-07

- **DONE — 原九类 item 随真实 `AgentHistoryViewModel` 迁移已交付；
  九套平行组件提案是 SUPERSEDED，不计作已实现 API**。
- [前端终验 C3](2026-10-07-final-audit-frontend-ownership.md#c3--original-cross-timeline-history-portable-viewporteffect-all-nine-items)
  核对原具体类型、生产装配和 renderer；随后
  [renderer 状态解耦](2026-10-07-decouple-history-renderer-state.md)及
  [限定收尾](2026-10-07-plan-spec-impl-closure-and-consistency.md#history-controls-and-remaining-limits)
  保留真实 item/window/nonce/anchor 行为、释放回归和真实输入压力测试。
  [核心 native E2E](2026-10-07-accept-sealed-cli-core-e2e.md)和
  [发布](2026-10-07-release-kodex-0-4-10.md#publication-verified--v0410)已交付 main `83749434`。
- 下文旧 1000-item 超时、失败窗口与 `821b6627` 的未跑平台保留历史事实。
  后来确认 fixture 混用窗口并做条件消融及准确源回归，不证明两次旧失败唯一根因；
  History 同 mount admission、Work 性能和全平台 B2/U 仍不整体关闭。

## Scope and correction

- 九类交互仍在范围内：Message、Reasoning、Tool、Patch、Plan Update、Request User
  Input、Suggest Subagent Task、Context Compaction、Work Group。
- 用户要求一次性硬迁移，原类型直接接管；不可用平行模型或 adapter 让旧宿主过编译。
- 本任务原先采用的九套 `app.component.*historyitem.contract` 状态/投影并非原类型
  的物理迁移。它们和 `Legacy*` view、重复 target/header、child union wrapper 已删除。
- 当前真实 item 家族作为 History 组件的契约/实现/渲染职责共同迁移，详见
  [AgentHistory 硬迁移](2026-10-04-extract-history-aggregate-component.md)。
- 不为了维持此前项目数公开内部 loading hooks、加 factory/resolver 或做空转发项目。
  若未来细分 Gradle topic，直接拆分当前真实声明，不重新发明业务模型。

## Single source

- 所有原 item FQCN 保持 `app.history.contract.item`。
- 契约和 KDoc 位于 `app/component/history/spec`。
- 真正 payload load、header projection、expand/collapse、取消与释放位于
  `app/component/history/impl/viewmodel`。
- 完整领域 UI 位于 `app/component/history/impl/view`；直接读取真实 item state。
- Work Group 直接拥有 `WorkGroupChildHistoryItemViewModel`，保留严格 newest-first
  child order；不再建立另一个 child-handle/VM wrapper union。
- Reasoning/Compaction 保留原轻量摘要 VM，不额外制造未使用的 lazy read 状态机。

## Behavioral gates

- Loading 空占位、Failed error row、ready/expanded payload 和 collapsed summary 保持。
- Tool/Patch 的状态、展开取消、shell 完成状态及真实细节渲染保持。
- Request User Input 仍为历史答案展示，不接管 pending answer draft。
- Suggest Subagent Task 的任务结果、反馈和真实 meta 展示保持。
- Work Group 的 count/range/order、展开、折叠与 child release 保持。
- entry 菜单捕获真实 item/generation/storage index；不存在空 action callback 桥。
- 运行原 contract、item state/header、paging/invalidation 和 Mosaic host/UI 测试。

## Historical checkpoint — 821b6627

- 上述代码硬迁移和旧生产项目清理已执行；原 History 86 项测试的最终矩阵通过。
- 1000-item UI 压力用例曾偶发超时，后续重跑成功不能视为根因闭合；详细结果见
  [硬迁移记录](2026-10-04-extract-history-aggregate-component.md#open-validation-risk)。
- 先前仅证明平行模型测试通过的结果不代表本轮通过。
- Native、JS、真实 CLI、IDE sync 尚未运行。
- 用户授权的代码切片已提交为 `821b6627`；本任务与完整 History 任务共同保留
  未闭合验证风险，不归档 done。
