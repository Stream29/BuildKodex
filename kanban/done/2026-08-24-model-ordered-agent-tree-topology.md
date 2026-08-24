# Task Tree

- [done] 修正 Agent tree 的有序部分树建模
  - [done] 确认现状与根因
    - [done] 复现多位编号 sibling 的字符串乱序
    - [done] 复现 descendant 被排到其他 branch 之后
    - [done] 确认 repository 顺序在 Session projection 中丢失
    - [done] 确认 contract 与测试没有 preorder oracle
  - [done] 确定目标 topology contract
    - [done] 让 `nodes` 成为顺序的唯一 authority
    - [done] 定义 root-first depth-first preorder
    - [done] 保留 repository 的稳定 sibling 顺序
    - [done] 保持 discovery、materialization 与 expansion 分离
  - [done] 收紧 topology state invariant
    - [done] 记录有序部分树语义
    - [done] 校验 root 唯一且位于首位
    - [done] 校验 parent-before-child 与连续 subtree
    - [done] 保留未发现 descendant 的 `hasChildren` 语义
  - [done] 重构 Session topology discovery 与 projection
    - [done] 按 parent 保存有序 direct-child edges
    - [done] 原子 reconcile entry 新增、删除与 index 复用
    - [done] 以 DFS 生成 canonical preorder
    - [done] 删除 depth 与 `AgentAddress` 排序
    - [done] 保持按需 materialization 与精确 revision
  - [done] 固定 frontend 消费语义
    - [done] 让 sidebar 只过滤 canonical preorder
    - [done] 让其他 tree renderer 不再推断顺序
  - [done] 增加分支树回归测试
    - [done] 覆盖非法 contract 顺序
    - [done] 覆盖 12 个 sibling 的 repository 顺序
    - [done] 覆盖多 branch、多 depth 的 preorder
    - [done] 覆盖 catalog 变化与惰性 materialization
    - [done] 覆盖展开后 descendant 紧邻 parent
  - [done] 更新长期约束
    - [done] 更新 Session topology checklist
    - [done] 更新 ViewModel state checklist
  - [done] 完成分层验证
    - [done] 运行相关 contract、ViewModel 与 Mosaic 测试
    - [done] 运行 IntelliJ 检查与 CLI link
    - [done] 使用隔离 fixture 验证真实 terminal tree
    - [done] 检查 diff、临时文件与并行工作树
  - [done] 获得用户授权进入 executable

# Details

- 状态：`done`；实现与分层验证已完成。

## 已确认的问题

- `Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt:405-421`
  读取有序 `listEntries()` 后只保留 node/parent/depth，丢失 direct-child edge 的顺序。
- 同文件 `:524-541` 再按 `depth + AgentAddress.agentId` 排序，得到 breadth-first、字符串序，而不是树的 preorder。
- `Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionAgentSidebar.kt:312-328`
  只过滤 `nodes`，因此完整保留了错误顺序。
- 典型结果包括 `child-1, child-10, child-11, child-2`，以及 `root, A, B, A1`；展开 `A` 后 `A1`
  不会紧邻 `A`。
- repository 已承诺稳定有序 snapshot，见
  `Kodex/agent-session/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/contract/KodexSession.kt:52-69`；
  filesystem 实现也按数值 entry index 排序，见
  `Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt:215-228`。
- 现有 contract 只校验 identity、parent 与 depth，见
  `Kodex/app/contract/session/src/commonMain/kotlin/io/github/stream29/kodex/app/session/contract/PersistedSessionState.kt:42-103`；
  它允许 structurally non-preorder 的列表。

## 目标模型

- `PersistedSessionTopologyState` 继续使用轻量 flat list，不改成递归 DTO。
- `nodes` 明确定义为已发现部分树的 canonical root-first depth-first preorder。
- sibling 顺序直接继承对应 repository `listEntries()` snapshot 的顺序。
- `nodes` 的位置是唯一顺序 authority；不增加可与列表漂移的 `siblingIndex` 或 display-name 排序字段。
- `AgentAddress` 只表达 identity，不参与展示顺序。
- `parentAddress` 与 `depth` 保留，用于关系校验、缩进和 ancestor 过滤。
- `hasChildren` 继续表示底层可能存在 direct children；它不要求 descendants 已发现或 materialize。
- topology discovery、Agent ViewModel materialization、history loading 和 renderer-local expansion 保持独立。

## Contract 与 projection 方案

- 在 `PersistedSessionTopologyState` 构造时要求：
  - `nodes` 非空且首项是 `rootAddress`。
  - 只有 root 可以没有 parent，且 root depth 为零。
  - 每个后续节点的 parent 必须是当前 preorder ancestor stack 中 depth 减一的节点。
  - depth 不得跨级增加；离开 subtree 后不得再次进入该 subtree。
- Session ViewModel 内部增加按 parent 索引的有序 direct-child edge sequence；edge 同时保留 repository entry index
  与 resolved child address，避免以 address 或名称反推顺序。
- 每次 direct catalog 变化都以完整 repository snapshot 原子替换该 parent 的 edge sequence。
- 已删除或同 index 被新 identity 复用的 edge 会移除其已发现 subtree、observer 和 materialized ViewModel；若 selected
  Agent 位于被移除 subtree，则回退到最近仍存在的 ancestor，最终至少回退 root。
- `refreshProjection()` 从 root 沿有序 edge sequence 做 DFS；删除当前 depth/address comparator。
- topology 只在语义 snapshot 改变时递增 revision；运行状态、名称和 materialization 状态继续投影到相同 node。

## Frontend 方案

- `visibleNodes()` 继续保留 expansion 为 renderer-local state。
- 它只按 ancestor expansion 过滤 canonical preorder，不排序、不重建另一棵树。
- branching fixture 必须证明：
  - 展开 `A` 后顺序为 `root, A, A1, B`。
  - 折叠 `A` 后只隐藏 `A` 的连续 subtree。
  - 12 个 direct children 保持 repository 顺序，编号 `10` 不插到 `2` 前。
- 同步检查
  `Kodex/app/view/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/RootSessionRendering.kt:7-18`
  等其他消费者只消费 canonical order。

## 测试与验证

- Contract tests：
  - 接受 root-only、branching preorder 和 `hasChildren = true` 但 descendants 未发现的 snapshot。
  - 拒绝 root 非首项、额外无 parent node、depth 跨级、parent 晚于 child、subtree 不连续。
- Session ViewModel tests 使用既有 in-memory repository 与真实 Flow，不新增 mock：
  - `0..11` siblings 按 repository 顺序发布。
  - `root/A/A1/B` 与多 branch 深树按 preorder 发布。
  - direct catalog 新增、删除和 index 复用后不保留 stale node。
  - 初始仍只 materialize root；展开只 materialize direct children；深层选择仍复用 exact Agent ViewModel。
- Mosaic tests 使用 branching topology 与受限 viewport，断言展开 descendant 紧邻 parent，不只比较 node set。
- 目标 Gradle 验证覆盖 `:app-contract-session`、`:app-viewmodel-session`、`:app-view-session`、
  `:app-view-application` 的 JVM/Linux X64 相关 tests，以及 `:app-cli:linkDebugExecutableLinuxX64`。
- 使用 IntelliJ IDEA 对改动文件运行检查；再用隔离 HOME 的真实 CLI fixture 验证多位 sibling 和嵌套 branch。
- 更新 `checklist/cli-session-view-models.md` 与 `checklist/cli-view-model-state.md`，固定 ordered partial-tree contract
  和“identity 不决定 order”的约束。

## 非目标

- 不修改 repository 或 storage schema。
- 不改变 Agent address、名称、选择、展开和视觉样式。
- 不递归 materialize descendants，不读取额外 history。
- 不引入 recursive topology DTO、第二份 sibling order authority 或 mocking framework。
- 不覆盖当前工作树中的其他用户改动。

## 执行结果

- Contract 已固定 root-first depth-first preorder，并拒绝 forest、depth skip 与 subtree reentry。
- Session ViewModel 已保留 repository direct-entry 顺序，以有序 edge 做 DFS，并清理删除或 index 复用产生的 stale subtree。
- Sidebar 只过滤 canonical preorder；其他 tree renderer 直接消费相同列表顺序。
- JVM 与 Linux X64 的相关 contract、Session ViewModel 和 Mosaic 测试通过；完整 Session ViewModel 与 Application View JVM
  测试通过。
- IntelliJ 改动文件构建、`app-cli:linkDebugExecutableLinuxX64` 与 targeted diff check 通过。
- 隔离 HOME 的真实 CLI 显示 `Child 0` 至 `Child 11` 数值顺序；展开 `Child 0` 后 `Grandchild 0` 紧邻其 parent，随后才是
  `Child 1`。
- 临时 fixture、生成源码与 tmux Session 已清理；并行工作树中的其他改动未修改。
