# Task Tree

- `Compare active task cards against later accepted code and release evidence`()
- `Archive only superseded or genuinely completed scope with current inbound links`()
- `Keep unfinished performance, platform and design work explicit`()
- `Report every moved card and retained gate`()
- `Deliver independent reconciliation handoff for coordinator review`()

# Details

- 用户明确要求纠正忘记更新的看板状态；这是文档对账，不授权重复实施旧 API。
- Owner scope: outer `kanban/` completed old migration child cards, completed
  subscription-usage card, and exact inbound links. Do not change user Draft,
  source code, Gradle active implementation records or unrelated research files.
- Exception authorized for moved Gradle cards: retarget only exact inbound links
  to the four planning→executable cards and repair their now-invalid relative
  links to files that remain in planning. No Gradle task-body/status edits.
- Compare with current v0.4.10 code and later accepted tasks, not phase-folder names.
  AgentState compaction, New Session exact target and local Hook restoration were
  completed later; do not leave old descriptions as current confirmed defects.
- Do not mark the domain parent or full E2E matrix done: automated dependency
  enforcement and B2/U remain. History performance and narrow-sidebars policy remain
  unfinished; original nine fake History item modules and Aggregate proposal were
  rejected, not future implementation work.
- Superseded plans may be archived as superseded, never as if rejected algorithms
  were implemented. Keep old commit-bound evidence and failure logs intelligible.
- No Git commits/pushes by this work line; coordinator stages only reviewed records.

## Reconciliation handoff

- **DONE — 独立文档对账已由协调者核对并接受**。归档 12 张过期卡片；
  未执行 Git 读取/写入命令、构建、测试、IDE、设备或网络操作，不暂存/提交。
- 上句描述工作线操作范围。协调者另核对实际 diff、链接及归档范围，
  仅提交本批状态和入链更新；Gradle、完整 E2E 和 B2/U 仍未完成。
- 对账以已交付 main v0.4.10 / `83749434` 为后来来源，不以文件夹、旧 TODO
  或孤立绿色测试判完成。12 张旧卡的 Task Tree 无进行中加粗语句，
  文首明确归档范围、后来证据和历史 checkpoint 的适用时点。

### Moved cards — exact inventory

原路径前缀除另注均为 `kanban/executable/`，目标均为 `kanban/done/`，
文件名不变；每张文首对账保留后来验收、实际 API/消费者或发布证据。

| 卡片 / 当前对账位置 | 归档的真实范围 |
| --- | --- |
| [2026-09-29-complete-openai-spec-impl-split.md](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-09-29-complete-openai-spec-impl-split.md#L14) | 实际 OpenAI 契约、client/storage/mock 迁移；不宣布 live/account/platform 全绿 |
| [2026-09-30-migrate-all-utils-to-spec-impl.md](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-09-30-migrate-all-utils-to-spec-impl.md#L15) | 原 Utils/Tool/MCP 真实契约接管与修复；伪接口退役，不计作实现 |
| [2026-10-01-migrate-agent-context-rpc-hook-roots.md](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-01-migrate-agent-context-rpc-hook-roots.md#L16) | Context/RPC 交付；旧控制 Hook 范围后来退役，不恢复旧 API |
| [2026-10-02-migrate-agent-session-root.md](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-02-migrate-agent-session-root.md#L14) | 原 Session 契约/仓库、后续缓存和清理验收 |
| [2026-10-03-extract-composer-component.md](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-03-extract-composer-component.md#L12) | 唯一原 Composer 状态/submit 及实际输入宿主 |
| [2026-10-03-extract-history-composer-and-navigation-components.md](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-03-extract-history-composer-and-navigation-components.md#L12) | 真实四组组件硬迁移；被否决平行方案为 superseded |
| [2026-10-03-extract-history-item-components.md](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-03-extract-history-item-components.md#L12) | 原九类 item 家族；九套伪 item API 未作为已实现归档 |
| [2026-10-03-extract-new-session-and-session-tab-bar-components.md](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-03-extract-new-session-and-session-tab-bar-components.md#L12) | 原 New Session/Tab Bar 与后来 exact-target 命令 |
| [2026-10-04-extract-history-aggregate-component.md](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-04-extract-history-aggregate-component.md#L13) | 原 `AgentHistoryViewModel` 硬迁移；Aggregate 提案被否决 |
| [2026-10-05-reaccept-spec-impl-refactor.md](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-05-reaccept-spec-impl-refactor.md#L25) | 完成首轮审查/接受的有界修复；旧 B1 已有后续闭环，B2/U 仍独立 |
| [2026-09-20-subscription-usage-cache-and-tool-output.md](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-09-20-subscription-usage-cache-and-tool-output.md#L24) | 原 `planning/` → done；持久 turn-state、用量 snapshot、冻结迁移已交付，不证明耗额/命中因果 |
| [2026-10-03-investigate-new-session-tab-crash.md](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-03-investigate-new-session-tab-crash.md#L12) | 原 `discussion/` → done；精确实例 API/实际调用/gated 测试证明旧 index 竞态已修，无重复源码 patch |

### Kept open or untouched

- [Domain master](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-09-28-plan-domain-spec-impl-boundaries.md#L350)：
  自动 main-source spec 依赖/cycle 门禁及整体验收未完；仅改其七个已移动子卡链接。
- [Full E2E parent](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-07-seal-repair-baseline-and-build-e2e-cli.md#L1)：
  已接受的核心 E2E 不覆盖整套浏览器/平台/故障矩阵；未改状态或正文。
- [Runtime settings discussion](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/discussion/2026-10-01-fix-runtime-settings-edit-crash.md#L32)：
  local Hook 出口恢复不能单独证明每个字段同字段冲突验收；全卡保持原状，不发明冲突政策。
- [History/Work performance discussion](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/discussion/2026-09-28-plan-fix-history-work-scroll-after-rpc.md#L1)：
  有界迁移、fixture 修正和压力回归不等于实际滚动性能已解决；未改。
- [Scoped closure limits](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-plan-spec-impl-closure-and-consistency.md#L647)：
  History 同 mount admission、kRPC shutdown/error、OS publication 后取消、Curl
  地址复用/EOF、MCP 异常图和全平台/PTY 故障仍按 B2/U；窄屏双侧栏策略、
  MCP mixed/unsupported/skip、工具未支持能力也未一并标绿。
- Gradle master/CI/模型/项目优化仍由协调者实施；原 Isolated Projects/Mosaic
  研究卡未迁移阶段、未擦除 blocked/失败研究记录。docs/manual/demos、
  Draft 与用户其他笔记未改业务内容。

### Exact inbound-only changes — coordinator review allowlist

以下 **33 个文件只改精确入链或移动引起的相对路径**，不改业务状态。
迁移卡之间原 same-phase 相对链接现在仍有效，未机械重写。所有旧源码提交绑定
URI 保留；仅移动卡的行号片段随文首插入平移，历史报告文字/结论不重写。

| 文件 / 位置 | 精确目标与变更 |
| --- | --- |
| [token-count downstream](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-09-21-adapt-downstream-token-count-snapshot.md#L24) | subscription planning→done；消费端说明原 L152→当前 L174 |
| [fork experiment](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-02-complete-fork-publication-experiments.md#L24) | rescue planning→executable |
| [IDE measurement](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-02-measure-real-idea-gradle-sync.md#L23) | rescue planning→executable |
| [configuration research](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-02-research-gradle-configuration-architecture.md#L24) | rescue planning→executable |
| [local publication experiment](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-04-validate-settings-triggered-local-fork-publication.md#L3) | Packages/rescue planning→executable 两入链 |
| [audit Agent/navigation](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-05-audit-agent-navigation-components.md#L9) | reaccept parent、history-task reference→done |
| [audit backend](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-05-audit-backend-roots.md#L9) | reaccept parent；H1/H7 Context 和 H3 Session URI→done，行号平移 |
| [audit other utils](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-05-audit-other-utils.md#L10) | reaccept parent、utils migration URI→done，L88→L110 |
| [audit platform](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-05-audit-platform-process-utils.md#L11) | reaccept parent、migration-map URI→done，行号平移 |
| [audit Runtime/OpenAI](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-05-audit-runtime-openai.md#L9) | reaccept parent、OpenAI migration URI→done |
| [audit Settings](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-05-audit-settings-account-components.md#L9) | reaccept parent→done |
| [audit Tool/MCP](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-05-audit-tool-mcp.md#L9) | reaccept parent→done |
| [repair Composer](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-05-repair-composer-host.md#L10) | reaccept parent→done |
| [repair platform](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-05-repair-platform-contracts.md#L10) | reaccept parent→done |
| [repair Settings](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-05-repair-settings-lifetimes.md#L11) | reaccept parent→done |
| [repair storage](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-05-repair-storage-cleanup.md#L11) | reaccept parent→done |
| [repair verification](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-05-repair-verification-and-ablation.md#L12) | reaccept parent→done |
| [Application root cutover](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-06-cutover-application-and-session-roots.md#L79) | New Session discussion→done |
| [frontend root migration](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-06-hard-migrate-frontend-ownership-roots.md#L130) | New Session discussion→done |
| [behavior ownership](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-06-reassign-shared-application-behavior.md#L36) | first reaccept→done，保留剩余风险锚点 |
| [closure record](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-plan-spec-impl-closure-and-consistency.md#L117) | first reaccept→done |
| [domain master](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-09-28-plan-domain-spec-impl-boundaries.md#L88) | 七个移动子卡入链→done，正文/状态不变 |
| [Gradle rescue](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-02-rescue-gradle-development-experience.md#L353) | 两个仍在 planning 的 fallback 文件改 `../planning/` |
| [Gradle model/resource](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-02-plan-gradle-model-resource-optimization.md#L31) | 仍在 planning 的 binary onboarding 改 `../planning/` |
| [native Gradle mechanism](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/planning/2026-10-02-verify-native-gradle-sync-mechanisms.md#L22) | 当时仅修入链；后续 full-source/IP 剩余研究已延期 |
| [Mosaic IP research](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-03-migrate-mosaic-for-isolated-projects.md#L6) | 后续按已完成原型/回滚归档，不表示生产 IP 完成 |
| [Xiaoxin validation](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-03-validate-gradle-plan-on-main-xiaoxin.md#L33) | 后续按已完成历史诊断归档；当前验收转父任务 |
| [binary onboarding](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/planning/2026-10-02-plan-binary-fork-onboarding.md#L19) | rescue/model planning→executable |
| [archive publication fallback](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/planning/2026-10-02-plan-github-fork-artifact-publication.md#L3) | Packages/rescue planning→executable |
| [Gradle research finding](file:///home/stream/ACodeSpace/push/BuildKodex/shared-context/findings/2026-10-02-gradle-development-experience-research.md#L83) | rescue/project optimization planning→executable |
| [Packages route finding](file:///home/stream/ACodeSpace/push/BuildKodex/shared-context/findings/2026-10-04-github-packages-ci-route.md#L53) | Packages/rescue planning→executable |
| [local route finding](file:///home/stream/ACodeSpace/push/BuildKodex/shared-context/findings/2026-10-04-local-fork-publication-route.md#L28) | rescue planning→executable |
| [Tool handler checklist](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/tool-handler-decisions.md#L29) | utils migration 的精确 backtick 路径 executable→done |

- 移动卡内另有 New Session investigation→navigation 子卡的一条链接随之改到 done。
  四个 Gradle planning→executable 移动由协调者负责；本线没有重复移动它们，
  Packages/project optimization 的正文未写入。

### Static document review and handoff constraints

- 已按整文结构、术语、完成/替代/历史 checkpoint、Task Tree 和当前证据范围复核；
  不新增重复 SOP。对 moved cards 的正文增量为当前对账/范围限定与去掉过期执行位置，
  原历史失败、测试计数、commit 与未验证平台保留。
- 初次完整扫描：12 张卡及四个 moved Gradle plans 的 190 个本地链接，
  当前对账段路径/锚点/行号零错误；kanban/checklist/findings 内 73 个涉及这些卡
  的 inline/reference 入链全部指向现存文件。12 张卡无 trailing whitespace。
- 另识别 20 个历史/外部证据路径问题：first reaccept 的 14 个退役源码 URI，
  New Session 旧基线的 4 个退役源码路径和旧 Main 行号，及项目优化引用的
  不在本机的研究报告。保留它们的基线/外部证据语义，不伪造当前源码链接，
  不把它们混入本轮当前对账段失效链接；协调者可读后来验收获取当前真源。
- 本 handoff 的 50 个本地 URI 已追加检查，路径/锚点/行号范围零错误，
  清单恰为 12 张移动卡和 33 个 inbound-only 文件；旧路径精确扫描无残留。
  最后复核：12 张卡无跨阶段同名残留；当前对账段加 handoff 共 128 个本地
  链接、全范围 87 个目标卡 inline/reference 入链，零错误。
  不运行 Gradle 或 `git diff --check`，
  协调者自行检查并仅暂存以上 12 次移动、本卡及 33 个精确链接文件的已审 diff。
  禁止顺带暂存用户其他 dirty/untracked 文档或源码。
- 本线未创建临时文件、进程、Session 或保留资源；无需停止共享 Daemon/IDE。
