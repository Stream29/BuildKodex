# Task Tree

- 统一 Fork materialization 批次
  - [done] 盘点 active 看板中的直接 Fork 任务
  - [done] 确认三项任务的共享边界
  - [done] 设计统一 Storage range 语义
  - [done] 设计 repository target materialization
  - [done] 设计 filesystem raw fast path 与 fallback
  - [done] 排定产品接入顺序与验证矩阵
  - [done] 实现统一 Storage range 与 repository materialization
  - [done] 接入 History、Subagent 与 Catalog Fork
  - [done] 完成 JVM、Linux X64 与 IDEA 验证

# Details

- 状态：`done`。
- Active 看板中直接属于 Fork 的任务只有三项：
  - [filesystem Session Fork 优化](2026-08-10-optimize-filesystem-session-fork.md)。
  - [Sessions catalog 整 Session Fork](2026-08-26-add-session-catalog-fork.md)。
  - [Subagent 固定继承 Compaction 活动窗口](2026-08-27-hardcode-compaction-based-subagent-fork.md)。
- 其他 active 文档中的 `fork` 只是依赖仓库或普通派生语义，不纳入本批次。
- 本文件是统一实施入口；三个子任务保留各自产品语义、测试和完成状态。

## 耦合结论

- 三项任务共同依赖同一条底层链路：
  - 从 source 捕获 immutable exclusive boundary。
  - 选择六条 timeline 的 source range。
  - 将 range materialize 为一个尚未打开的新 AgentSession 节点。
  - 打开目标后执行调用方专属 settings 或消息写入。
  - 任一步失败时由拥有该阶段的层级补偿。
- 不能分别实现：
  - Catalog Fork 若先落地，会继续走低效的逐记录 JSON round-trip。
  - filesystem 优化若只识别完整 prefix，无法覆盖 Subagent 的 checkpoint range 与 index rebase。
  - Subagent 若直接在 Tool handler 自行复制，会重复 target 创建、缓存和失败恢复逻辑。

## 统一 Storage range

- 在 `agent-storage-contract` 增加一个明确的 Fork range value：
  - `fromInclusive >= 0`。
  - `untilExclusive > fromInclusive`。
  - source stored index `i` 只在 `fromInclusive <= i < untilExclusive` 时复制。
  - target index 固定为 `i - fromInclusive`。
- 六条 timeline 使用同一 range；不把 settings、stable 或 unstable 单独重建成近似状态。
- 每个复制到目标的 `CleanCompactionCheckpoint.historyBaseIndex` 同步减去 `fromInclusive`。
- range 必须从 source 中真实存在的 compaction checkpoint 开始，并保证目标 index `0` 具有合法的 compaction/settings 初始化快照。
- 完整 Session/History Fork 使用 `[0, boundary)`：
  - index 不发生变化。
  - checkpoint 内容不发生变化。
  - 现有 `KodexAgentStorage.forkTo(until, target)` 保持公开语义，并委托给 `fromInclusive = 0` 的统一实现。
- Subagent Fork 使用活动窗口：
  - 先沿用当前 pending-call 规则捕获 `boundary`。
  - `fromInclusive = compaction.floorToIndex(boundary - 1)`。
  - 复制 `[fromInclusive, boundary)` 并从零重定位。
- 对象级实现继续负责跨后端 fallback，并保留目标清空与 compound compensation。

## Repository materialization

- 在 `agent-session-contract` 为 target repository 增加 `createFork(sourceStorage, range)`：
  - repository 分配新 entry、创建 raw target、执行 range copy，并在完整成功后返回 target index。
  - 目标在复制前不打开 runtime，也不建立 timeline cache。
  - 新节点不包含 descendants；root target 默认没有 archive marker。
  - 失败时不留下 entry、raw storage 或 descendants。
  - caller 仍负责 source 的运行状态、target-only settings 和业务通知。
- in-memory repository 直接向未发布的 `InMemoryKodexAgentStorage.empty()` 执行对象级 range copy。
- filesystem repository：
  - 对可识别的同格式 filesystem source 使用原始数字文件 fast path。
  - 其他 source 直接向未打开的 `FileSystemAgentStorage` 执行对象级 fallback。
  - target 完成后才创建 cache，因此不存在空 cache 失真。
- Repository 只负责“创建并 materialize 新节点”，不决定 `[fork]` title、child path、`NEW_TASK` 或 catalog navigation。

## 产品层接入

- 精确 History Fork：
  - 保留 `PersistedSessionViewModel.fork(source, target)` 的 ownership、stale target 和 committed stable boundary 校验。
  - 改用 repository `createFork`。
  - 打开目标后追加 `[fork] <boundary title>`，失败则删除目标。
- 整 Session Fork：
  - 增加无参 `PersistedSessionViewModel.fork()`。
  - 只复制 root 当前 `[0, latestIndex + 1)`，包括 persisted unstable，不复制 Subagent tree。
  - 在 target 创建前拒绝未初始化或正在运行的 root。
  - 与精确 History Fork 共用 target title、打开、关闭和失败删除 helper。
- Subagent Fork：
  - `spawn_agent` 固定使用最新 Compaction 活动窗口。
  - 删除 `fork_turns` 与 `service_tier` 输入。
  - 保留 `model`、`reasoning_effort` 的缺省继承与显式 override。
  - `service_tier` 由 child 固定继承父 Agent，不由 Agent 选择。
  - 目标 materialize 后处理 comp hash 过渡、追加 child settings，再投递 `NEW_TASK`
    并恢复既有调度。
- Session Catalog：
  - Catalog 只通过 Application callback 请求 `sessionIndex` Fork。
  - Application 复用已打开 source；未打开时临时 open，并在所有退出路径 release。
  - Fork 不改变 tabs 或 selection。
  - 成功后 Catalog 关闭旧 repository，按当前 `showArchived` 从新 repository reload；reload 失败不删除已成功的 target。

## 失败与发布边界

- source 校验和 range 计算必须在 target 分配前完成。
- repository 拥有“raw target materialization 失败”的完整清理。
- caller 获得 target index 后，拥有“target-only 后置写入失败”的删除补偿。
- Catalog reload 不属于 Fork transaction；target 已成功持久化后不因 UI reload 失败回滚。
- filesystem 每个数字文件使用临时文件加 atomic move；`latest.json` 从目标实际文件重建，不复制 source pointer。
- source append-only writer 仍由 AgentState、Persisted Session 或 Tool command 边界串行化；本批次不增加 storage 内部锁。
- Subagent model override 若跨越两个已知且不同的 `comp_hash`，在 `NEW_TASK` 前先用
  checkpoint 来源模型重压缩 materialized child；失败按 target-only 后置写入失败处理。
- checkpoint 无 provider payload、hash 相同或任一 hash 缺失时不增加过渡压缩。

## 实施顺序

- 第一阶段：Storage 语义。
  - 增加 range value、range copy 和 checkpoint rebase。
  - 让既有完整 `forkTo` 委托新实现。
  - 用 in-memory storage 覆盖六条 timeline 与 compensation。
- 第二阶段：Repository materialization。
  - 扩展 Session repository contract。
  - 实现 in-memory 与 filesystem raw target 路径。
  - 增加 filesystem fast path、pointer 重建、fallback 和失败清理。
- 第三阶段：迁移现有精确 History Fork。
  - 先让现有行为消费统一入口，确认 title、navigation 与 rollback 无回归。
- 第四阶段：接入固定 Subagent 活动窗口。
  - 收敛 tool contract、兼容读取、History 展示和 Spawn 流程。
  - 接收 model catalog `comp_hash`，覆盖跨 hash 的 Spawn 前过渡压缩。
- 第五阶段：接入整 Session 与 Catalog Fork。
  - 增加无参 Persisted Session command、Application callback、Catalog reload 和 TUI menu。
- 第六阶段：统一回归与维护文档。
  - 完成跨层、跨后端、JVM、Native、Mosaic 与 IDEA 验证。
  - 更新 Fork、Session lifecycle 与 Multi-Agent 的持久决策。

## 验证矩阵

- Storage：
  - 完整 prefix 与 range/rebase 的六 timeline 等价性。
  - checkpoint payload、prefix、window lineage 与 `historyBaseIndex`。
  - 空目标要求、边界校验和失败 compensation。
- Repository：
  - root/subagent target 创建、inventory 原子发布、无 descendants/archive。
  - in-memory、filesystem 与双向跨后端 fallback。
  - cache 首次扫描、raw bytes、pointer、临时文件和失败清理。
- Multi-Agent：
  - 初始 checkpoint、真实 compaction、pending boundary、嵌套 Spawn。
  - model/reasoning 缺省继承与独立 override。
  - service tier 固定继承、`NEW_TASK` 和失败删除。
  - comp hash 相同、不同、缺失与过渡压缩失败。
  - 旧 `fork_turns`、`service_tier` JSON 未知字段兼容。
- Persisted Session/Catalog：
  - exact history 与完整 root snapshot。
  - opened、unopened、archived、running、uninitialized source。
  - 临时 source release、navigation 不变、reload/filter/sort 和 target rollback。
- Frontend：
  - Context menu 顺序为 Fork、Archive/Unarchive、Delete。
  - pointer、`Shift+F10` 与 Menu 键路由。
  - Fork 后只关闭 ContextMenu，Sessions Dialog 保持打开。
- 验证命令在 executable 阶段按实际 Gradle module task 确认：
  - 受影响模块 JVM tests。
  - Linux X64 tests 或至少 Native test compilation。
  - Mosaic tests。
  - 检测 Gradle Daemon JVM，并显式使用同一 JDK。
  - IDEA build/错误级 inspection。
  - 根仓库与 `Kodex/` 的 `git diff --check`。

## 排除项

- 不修改 storage 文件格式或增加迁移。
- 不复制或迁移既有 Subagent 数据。
- 不实现 hardlink、reflink、共享前缀、内容寻址或 GC。
- 不改变 compaction 触发和 checkpoint 内容。
- 不让 Fork 自动打开 target tab 或改变当前 selection。
- 未提交代码，由用户统一提交。
