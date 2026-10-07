# Task Tree

- `Move the real Composer contract, state and implementation`()
- `Bind fixed-Agent command ports and sibling presentation`()
- `Connect the actual input renderer to its child`()
- `Remove compiler-only aliases, receipt wrappers and duplicate host submission`()
- `Run component and host regression after the hard cutover`()
- `Archive accepted real Composer cutover and hand off independent validation limits`()

# Details

## Reconciled status — 2026-10-07

- **DONE — 原 Composer 唯一状态/submit 算法及真实宿主接线已交付**。
  `821b6627` 是下文硬迁移历史切片，不是最终验收基线。
- [真实宿主修复](2026-10-05-repair-composer-host.md)以实际 Enter 输入验证
  失败、取消、恢复；[前端终验](2026-10-07-final-audit-frontend-ownership.md)
  追踪原声明/生产 renderer；其后 parent-scope/错误出口等确认问题由
  [限定收尾](2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)
  补修、独立复审和集中验证。[核心 native E2E](2026-10-07-accept-sealed-cli-core-e2e.md)
  与 [发布](2026-10-07-release-kodex-0-4-10.md#publication-verified--v0410)
  交付实际 main `83749434`。
- 下文平台未跑与 History 压力风险保留为原 checkpoint；后续 History fixture
  修正/真实输入证据见收尾，不宣称旧超时唯一根因已证明或 B2/U 全绿。
  独立性能、窄屏策略、全部故障矩阵及迁移总验收不属于本卡归档结论。

## Component layout

```text
app/component/composer/
  spec/
  impl/viewmodel/
  impl/view/
```

- 唯一 `ComposerViewModel` 保留原 `app.agent.contract` 身份。
- `ComposerState.text`、cursor、revision 是同一份编辑权威；不再额外命名 `draft`
  然后提供一个 `text` legacy getter 让旧消费者编译。
- pending steer/running/request-input 是窄的后端或 sibling presentation，不复制 answer draft。
- widget buffer、focus、viewport 和 layout 只属于 renderer；cursor-only 更新也触发接线。
- 文本与 cursor 只经原 `update(text, cursorOffset)` 入口更新；删除重复的
  `updateDraft` 实现及忽略 cursor 的默认转发。

## Submission ownership and ablation

- 唯一 draft submission state machine 是 `ComposerViewModel.submit(expectedRevision)`。
- 删除 `AgentViewModel.submitComposer`、RPC presentation 的另一套 submit mutex/算法及
  第二种 submission result；消费者直接调用真实 Composer child。
- 原 `AgentViewModel.submit(content)` 保留：它接收已经准备好的外部 content，不读取/
  消耗 Composer，因此不是第二套 draft submission。
- Submit/Steer 依赖是返回 `Unit` 的 suspend 操作，正常返回分别表示真实 append 完成/
  steer 已接受。删除只有一个分支的 receipt wrapper；失败或取消不能伪造正常返回。
- UI/调用者仍接收有真实不同分支的 `ComposerSubmissionResult`（submitted/steer/empty/
  stale/unavailable/failure）；它不是 redundant 单值类型。
- Draft 的 Composer Agent-command ports 明确拒绝误调用，不返回无副作用的成功；
  New Session materialize 属于该准确 draft/registry owner。
- accepted-versus-persisted、caller cancellation、close/replacement late result、failure
  retention/reporting、准确 owner/revision 接线保持 spec 所述语义。
- 删除未被生产使用的 `replaceOwner`：它只改 identity、却继续使用旧 owner 的依赖，
  不是真正的替换。真实替换关闭旧 child，创建绑定新依赖的 child；迟到完成不污染新 child。

## Host and renderer

- persisted Agent 使用真实 `ComposerView`，宿主原有 pending steer/status rows 仍只渲染一次。
- New Session 页面直接使用原 registry child 和同一个 Composer；原输入 UI 的 submit
  callback 走准确 draft 的 materialize 接线，不包第二个 page ViewModel。
- 关闭 Composer 只释放其观察，不假装 backend rollback；registry/Agent owns lifetime。
- Source ports 与真实 RPC 命令绑定不是被删除的模型兼容层。
- Agent 和 New Session 共享同一真实 `ComposerInput` widget，实现只有一份；原输入
  交互测试也直接验证这一实现。submit 测试使用真实键盘事件，不手调 ViewModel 代替 UI。

## Validation gates

- 原文本/cursor/revision 字段只有一个声明，没有 legacy getter 或第二套 host draft command。
- fake-dependency 测试 submit/steer 顺序、stale revision、cancel、failure、close/replacement。
- Mosaic 真输入测试 newline、submit capture、wrapping/sizing 和 cursor-only 更新。
- RPC 实际 steer/append 及 Application 原 UI/recording 回归。
- Draft 误调 Agent submit 不消耗输入、不创建实体；页面 unmount 不关闭 registry child。
- 硬迁移后的 Composer 15 项测试与宿主回归通过；当前 runtime 尚未投影到 UI 时，
  submit 仍按真实 running 值选择 steer，而非错误 append。
- 整批验证结果与未闭合的 History 压力用例风险见
  [硬迁移记录](2026-10-04-extract-history-aggregate-component.md#current-checkpoint)。
- 用户授权的代码切片已提交为 `821b6627`；Native、JS、真实 CLI、IDE sync 未运行，
  整批的剩余验证仍按硬迁移记录跟踪。
