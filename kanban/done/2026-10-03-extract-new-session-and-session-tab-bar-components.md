# Task Tree

- `Move Session Tab Bar rendering and bind exact registry identities`()
- `Audit and remove the parallel New Session model and adapter`()
- `Move the original NewSessionViewModel, implementation and page content`()
- `Connect the exact registry child to the component renderer`()
- `Validate New Session creation, tab lifetime and original UI`()
- `Archive accepted original New Session and Tab Bar cutover`()

# Details

## Reconciled status — 2026-10-07

- **DONE — 原 `NewSessionViewModel` 和 Tab Bar renderer 硬迁移已交付**，
  copied creation-state/effect/receipt/controller 已删除，不作为完成 API。
- 后续 [Application/Session 根验收](2026-10-06-cutover-application-and-session-roots.md#coordinator-final-handoff-and-verification)
  接管 exact-instance 命令：排队重复只物化一次，已关闭/已替换返回 stale/null，
  移位仍命中同一实例；[前端终验](2026-10-07-final-audit-frontend-ownership.md)
  核对 spec→原实现→真实宿主→gated tests。
  [限定收尾](2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)
  补修 parent-scope/fixture 等确认问题并独立复审。
- [核心 native E2E](2026-10-07-accept-sealed-cli-core-e2e.md)实际新建/切换草稿、
  保留 sibling 输入、持久化重开；[发布](2026-10-07-release-kodex-0-4-10.md#publication-verified--v0410)
  交付 main `83749434`。下文 `821b6627` 的未跑平台是历史 checkpoint，
  不等于当前缺少所有 CLI 验证；全部平台/故障矩阵与总迁移验收仍未关闭。

## Scope

- New Session Screen 与 Session Tab Bar；Application registry、root popup、选择、
  archive/delete/close 与准确 draft replacement 仍属于宿主。
- 用户确认一次性硬迁移；不保留兼容 facade、平行 controller 或创建 receipt/effect 镜像。

## New Session

```text
app/component/new-session/
  spec/
  impl/viewmodel/
  impl/view/
```

- 唯一公开类型仍是原 `app.session.contract.NewSessionViewModel`。
- `NewSessionViewModelArguments`/factory 和 `DefaultNewSessionViewModelFactory`、
  `RpcNewSessionViewModel` 直接迁移，不作 typealias 或转发。
- 原 `SessionViewModel` 作为跨组件 presentation interface，不再要求 sealed 子接口
  在同一个 Gradle module 声明；不另加第二套 Session 模型。
- 删除 `app.newsession.contract.*`、`NewSessionComponentAdapter` 及 copied
  settings/page/effect/creation-receipt authority。
- Composer 与 Runtime Configuration 是 registry child 的稳定真实子组件。
- 创建仍由原 `materialize()` 返回真实 persisted child，Application 替换准确 draft；
  已分配后失败不声称 rollback、不新增自动重建。
- 输入 renderer 保留空历史空间、separator、bounded wrapping、cursor 和 submit target。
- 页面接受共享 status-bar composable slot；它仅是布局输入，没有业务状态副本。
- 切换 tab/卸载 renderer 不关闭 draft；registry remove/replace 才关闭。
- Draft 的 Agent submit/steer port 不伪造成功，实际创建入口是 draft owner。

## Session Tab Bar

- spec：`app/component/session-tab-bar/spec`。
- renderer：`app/component/session-tab-bar/impl/view`。
- 原生产 `SessionTabBar` 已删除；宿主提供 immutable presentation 和 captured callbacks。
- registry ordering、selected index、关闭/归档、popup lifecycle 不迁入 renderer。
- identity 不是 label/index；陈旧 callback 必须仍能验证目标是否在 registry。
- 宿主测试 adapter 只是 fixture entrypoint，不是生产新旧 ViewModel 桥接。

## Validation gates and status

- 唯一 New Session 声明，旧 impl/view 根和旧 Gradle 坐标消失。
- 原 local edit、materialize serialization、known allocation identity、exact replacement 回归。
- tab 切换不销毁未提交 draft，关闭/重复关闭、原 cursor/revision UI 回归。
- Session Tab Bar bounds/ordering/compact/selection/context-menu 回归。
- Application、Session、RPC、组件 JVM 编译/测试；原完整 UI 输出对照。
- 当前已执行代码清理和宿主切换；原 New Session 5 项 controller 测试、Tab Bar
  5 项测试及 Application 91 项 UI 回归通过。未沿用被删除平行模型的测试作为证据。
- 本批完整结果和未闭合风险见
  [硬迁移记录](2026-10-04-extract-history-aggregate-component.md#current-checkpoint)。
- 用户授权的代码切片已提交为 `821b6627`；Native、JS、CLI smoke、IDE sync 未运行，
  整批剩余验证仍按硬迁移记录跟踪。
