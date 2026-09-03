# Task Tree

- [done] 实现 index/work history timelines
  - [done] 确定 index/work 持久化分区
  - [done] 重构 clean event 与 agent-storage 六条 timelines
  - [done] [审批下游读写需求与 API](../done/2026-08-30-review-downstream-storage-access.md)
  - [done] 保持 turnId 位于 settings，并移除 turn marker 设计
    - [done] settings snapshot 保存 active turnId
    - [done] initialization 在 index `0` 写入初始 compaction point
    - [done] mark-new-turn 与 message writes 通过 settings snapshot 更新 turnId
    - [done] hooks 与 request metadata 从 settings snapshot 读取 turnId
  - [done] 更新 IndexVersioned、in-memory 与 filesystem implementations
    - [done] exact 与 range primitives
    - [done] cached、in-memory 与 direct filesystem range/read contract
    - [done] revert 使用 range 与 exact values 保存 suffix
  - [done] 更新 AgentState
    - [done] 路由 index/work writes
    - [done] 通过 active message window 构造请求输入
    - [done] 独立扫描 index/work 的最新 state candidate
    - [done] 拆分 compaction lock，允许期间更新 settings
  - [done] 迁移非 History consumers
    - [done] 更新 turn、compaction、tool hooks 与 request metadata
    - [done] 保持 steer 与 completed-tool contracts
    - [done] 保持 history target validation
  - [done] 重构 Session storage ownership
    - [done] 更新 cached/session timeline wrappers
    - [done] 收窄 repository full-fork API
    - [done] 实现 filesystem full raw fork
    - [done] 实现 in-memory private full clone
    - [done] 用 target revert 实现 history fork
  - [done] 重构 History projection
    - [done] 以 index entries 建立顶级 anchors
    - [done] 以 work ranges 建立 lazy groups
    - [done] 不把 turn marker 作为 History item
    - [done] 以已知边界读取 duration
    - [done] 保持 paging、invalidation 与 targets
  - [done] 编写 settings metadata 本地迁移
    - [done] 从旧 index metadata 恢复 settings lineage
    - [done] 清理 index 中的 turn markers
    - [done] 重建 index/settings latest pointers
    - [done] 增加 dry-run、校验、staging、原子切换与失败回滚
  - 验证实现
    - [done] 覆盖 IndexVersioned、storage、AgentState、Session 与 History 的主要测试
    - [done] 覆盖 compaction/settings 并发行为
    - [done] 覆盖 History collapsed work payload 不读取与展开 exact-read
    - [done] 提供 `uv` migration script
    - [done] 更新 compaction runtime test 的过期断言并重新运行受影响 checks

# Details

## Current design

- 本文早期版本曾错误地要求将 `turnId` 从 settings 移入 index marker。
- 当前设计以 settings snapshot 作为 active turn identity 的唯一持久化来源：
  - `KodexAgentSettings.turnId` 保留。
  - index timeline 只保存稳定 index events 与 compaction points。
  - 存储中不存在 `CleanTurnMarker`。
  - History 不显示或扫描 turn marker。
- `IndexVersioned.get(index)` 保持 floor-visible value 语义。
- `getExact(index)` 与 `indexesIn(range)` 为 History、AgentState 和 fork 提供精确读取与范围定位。
- `activeMessageWindowAt(index)` 位于 agent-storage contract-ext，由 AgentState 使用；
  History 保持自己的 anchor、work-group、paging 和 generation 逻辑。
- Compaction 在远程请求期间不持有阻塞 settings update 的锁；commit 使用提交时的最新 settings。
- Session repository 提供完整 fork，History fork 在 target 上执行 revert。

## Migration

- `Kodex/scripts/migrate-index-metadata-to-settings.py` 是一次性本地 `uv` 脚本。
- 它将旧 index 中的 compaction/window metadata 恢复到 settings，移除 index turn markers，
  并通过 sibling staging、校验和原子切换保护本地数据。
- 应在新版本代码和 CLI 构建完成后由用户手动运行；应用不会自动调用。

## Verification status

- 已修正 compaction runtime test，使其从 settings snapshot 验证
  `windowNumber`，不再从仅作为 marker 的 `CleanCompactionPoint` 读取窗口信息。
- 已同步修正 in-memory 与 filesystem session tests 中基于旧 turn-marker
  初始化布局的索引断言。
- 已通过以下验证：
  - `:agent-runtime-decorator-compact:jvmTest`
  - `:agent-session-in-memory:jvmTest`
  - `:agent-session-filesystem:compileTestKotlinJvm`
- 另尝试运行 `:agent-session-filesystem:linuxX64Test`；native test runner
  在本机未能在合理时间内结束，因此以 JVM 测试源码编译结果作为该改动的编译验证。
