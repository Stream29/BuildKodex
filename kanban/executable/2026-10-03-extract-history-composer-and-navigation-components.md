# Task Tree

- `Inventory nine History item kinds, Composer, New Session and Session Tab Bar`()
- `Implement and compare original UI behavior`()
- `Audit all new concepts against the user's hard-migration rule`()
- `Delete parallel History and New Session models rather than bridge them`()
- `Integrate the real component owners and run downstream regression`()
- **`Track remaining validation after the authorized hard cutover`()**

# Details

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

## Checkpoint

- 硬迁移与真实宿主接线已实施；最终 308 项定向 JVM 测试与 Integration 测试源码
  编译通过。History 压力用例的偶发超时未闭合，详见
  [结果与风险](2026-10-04-extract-history-aggregate-component.md#current-checkpoint)。
- 当前代码切片已提交为 `821b6627`；保留未闭合的压力用例风险及未验证平台，
  不将提交授权解释为这些风险已解决，任务暂不归档 done。
