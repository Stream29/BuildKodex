# Task Tree

- [done] 封闭 stable 事件类型并恢复分派完备性
  - [done] 检查共享根、开放桥接接口与分派缺口
  - [done] 使用 Kotlin 2.4.0 验证完备性和同包继承约束
  - [done] 确认统一 package、删除开放桥接的方案
  - [done] 核对迁移文件、调用点和序列化名称
  - [done] 明确编译期约束与旧数据兼容验证
  - [done] 确定已完成建议的历史展示要求
    - [done] Index Item 独立展示且不参与 work 聚合折叠
    - [done] 确定原任务与最终选择的只读表单
    - [done] 确定 accepted Session meta 展示
    - [done] 确定失败态展示
  - [done] 审查封闭层级与历史展示的集成阻塞
  - [done] 完成可执行计划并确认实施范围
  - [done] 实施封闭层级与完整分派
    - [done] 移动模型文件并删除开放桥接接口
    - [done] 更新生产与测试代码的类型引用
    - [done] 修正历史及持久化完整分派
  - [done] 验证并更新相关类型指导
    - [done] 验证旧 JSON 读取与重编码兼容
    - [done] 验证增加事件会触发编译错误
    - [done] 验证独立条目展示、聚合边界与错误日志
    - [done] 完成 JVM 测试和 Linux CLI 构建

# Details

## 执行结果

- 已迁移 19 个模型文件及两个测试文件，删除开放桥接，并更新所有旧 package 引用。
- `HistoryItemLoadContext`、普通工具分类、标题/状态、`CleanEventView`
  均使用穷尽分派；AgentState 存储路由穷尽 index/work，不引入共享根 serializer。
- `StableTimelineGoldenTest` 固定旧 index/work、compaction point、retained
  suggestion 和模型投影；命令 action/result golden 另用 `8d4cfba7` 的旧模型源码
  在隔离目录编译生成。迁移后 JVM/native 解码和重编码断言均通过。
- filesystem 测试直接写入旧 accepted JSON，再经真实文件存储读取，确认 index
  归属、任务和 Session URI 不变。
- Kotlin 2.4.0 隔离实验编译实际 clean-models、history contract 和 ViewModel
  源码：基线成功；分别增加 index tool、work tool、非工具事件均因真实
  历史分派不完备而失败，没有修改正式联合或添加永久测试变体。
- clean-models、filesystem、AgentState、history/agent ViewModel JVM 检查通过；
  相关历史 UI 定向回归及 Linux native 测试通过，CLI JVM 编译/release 链接通过。
- 更新类型指导；没有数据迁移、用户历史改写、提交或发布。完整 UI 测试的
  非本次定向用例失败及运行验证限制见统筹任务记录。

## 已确认方案

- 用户于 2026-09-07 接受本方案，随后授权推进并执行；已完成本任务定向验收。
- 展示实施已拆至独立[任务](2026-09-07-render-completed-session-suggestions.md)；
  本文保留已确认设计和集成约束，展示进度由该任务管理。
- stable 事件的封闭继承链统一到
  `io.github.stream29.kodex.agentstorage.cleanmodels.stable`，文件仍按事件拆分。
- `StableCleanEvent` 改为 sealed；其 `CompletedTool` 保持 sealed。
- 删除开放的 `StableCleanEvent.CompletedTool.Index` 和 `.Work`。
- `StableIndexEvent.CompletedTool` 和 `StableWorkEvent.CompletedTool`
  直接继承 `StableCleanEvent.CompletedTool`，同时保留各自的时间线父类型。
- `StableIndexEvent`、`StableWorkEvent` 及相关 sealed 类型、具体事件移到同一
  package；`CleanIndexEntry`、`CleanCompactionPoint`、`CompactionRetainedItem`
  等相连层级一起调整，不能留下跨包的 sealed 继承链。
- 保留现有类型名称、index/work 时间线归属和工具输出类型约束；
  时间线区别由类型表达，不再由 package 表达。
- 不新增 wrapper 或第二套 UI 事件模型；`unstable` 层级不因此搬动。
- `CleanOpenAiEvent` 保持公共投影能力接口，不用于完整事件分派。
- 存储路由穷尽时间线分支；事件展示穷尽具体事件，或逐层穷尽 sealed 子族。
- 完整分派不允许 `else` 或宽泛分支吞掉未明确处理的事件；
  新增事件时，需要定义相应行为的位置应产生编译错误。

## 已确认的历史归类

- `StableSuggestSubagentTaskToolEvent` 已属于 `StableIndexEvent.CompletedTool`；
  Index 归属不是待决事项。
- 用户明确：作为 Index Item，它独立展示，不参与 work 聚合折叠，
  不并入 `Take n actions`，并作为 breaker 分隔前后的可聚合 work。
- 现有建议事件落入普通 `HistoryItemKind.Tool` 是展示适配遗漏。
  但实际投影已将 Index 事件作为独立 anchor，WorkGroup 仅从 work 存储取子项；
  `isFoldable` 只用于检查这些子项，不能据此断言建议事件已经被聚合。
  修复应保留现有时间线边界，不新增分组机制。
- 后续仅讨论条目内容与状态展示，不再重新询问是否作为独立 Index Item。

## 已确认的只读展示

- 沿用 `request_user_input` 已完成历史的只读表单形态，不使用普通工具的
  折叠标题和 JSON 详情。
- 标题为 `Suggested Sessions`；按原顺序显示所有任务，名称加粗独占一行。
- 创建成功时，按输入/结果数组顺序，在每项名称的下一行附对应 Session URI，
  再显示完整 prompt；不另列重复名称清单，不附加执行状态。
- 未创建 Session 时不显示 URI 行，完整 prompt 直接接在名称下；内容按宽度换行。
- 正常完成时，任务之后显示最终选择：`[● Accept]` 或 `[● Reject]`，
  不显示未选选项。
- 拒绝备注使用 Other 同款 `  > ` 文本呈现；无备注时不保留空输入框。
- 历史记录不保留可操作按钮、输入框或配置菜单。
- 工具失败时沿用 `request_user_input` 的历史失败态：保留原任务内容，
  末尾显示带具体原因的失败信息；不伪造 Accept/Reject，不显示成功 Session URI。
- 工具失败是已加载事件的结果展示，必须与条目读取/解码失败的孤立红色
  `Error` 区分。

## 实施前已核实的问题

- [共享 stable 根](../../Kodex/agent-storage/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableCleanEvent.kt#L11)
  是开放接口，`CompletedTool` 的 Index/Work 中间接口也开放。
- [历史标题和状态](../../Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/HistoryItemHeaderFactory.kt#L97)
  用运行时错误兜底，遗漏了已完成的 `suggest_subagent_task`。
- [详情渲染](../../Kodex/app/view/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/CleanEventView.kt#L126)
  虽然没有 `else`，但接收开放根，遗漏事件可静默不渲染。
- 历史条目分类及 AgentState 的 stable/index/work 存储分派也使用开放边界。
- 独立 Kotlin 2.4.0 编译实验确认：开放根的非穷尽 when 语句可编译；
  真正 sealed 根的遗漏会编译失败；开放中间接口阻断叶子完备性；
  跨 package 直接继承 sealed 接口会编译失败。实验文件已清理。

## 迁移范围

- 路径均相对 `Kodex/`。模型源目录前缀为
  `agent-storage/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/`。
- 将 `index/` 的 8 个源文件移至 `stable/`：
  `CleanCompactionPoint.kt`、`CleanIndexEntry.kt`、`CompactionRetainedItem.kt`、
  `StableIndexEvent.kt`、`StableMessageEvent.kt`、`StablePlanUpdate.kt`、
  `StableRequestUserInputToolEvent.kt`、`StableSuggestSubagentTaskToolEvent.kt`。
- 将 `work/` 的 11 个源文件移至 `stable/`：
  `InvalidToolInvocation.kt`、`StableCommandExecutionToolEvent.kt`、
  `StableFallbackToolEvent.kt`、`StableImageGenerationToolEvent.kt`、
  `StableImageViewToolEvent.kt`、`StableMcpToolEvent.kt`、`StablePatchToolEvent.kt`、
  `StableProviderEvent.kt`、`StableToolSearchEvent.kt`、`StableWebSearchToolEvent.kt`、
  `StableWorkEvent.kt`。
- 修改原位 `StableCleanEvent.kt` 的 sealed 层级；
  不为共享根额外引入新的序列化入口，保持时间线专用 serializer。
- 本轮源码扫描发现 113 个 Kotlin 文件引用旧 `stable.index` / `stable.work` package，
  涵盖 clean models、storage、state、runtime、session、tool、MCP、app contract /
  viewmodel / view 和 integration tests；此数字是审计快照，不是实施时的固定清单。
- 更新所有 import 和全限定类型引用；将两个位于旧 `stable/work` 测试 package 的
  `StableToolEventSerializationTest.kt`、`StablePatchToolEventSerializationTest.kt`
  同步移到 `stable` 测试 package，避免依赖旧同包名称解析。
- 不新增模块或改变模块依赖方向；不修改正在运行的 Session 数据。

## 序列化审计

- 本轮静态扫描发现 43 个可序列化具体类型（包含事件、action/result 分支），
  均带显式 `@SerialName`；未发现依赖默认全限定类名的具体类型 discriminator。
- 部分 sealed 根未指定 `@SerialName`，其 descriptor 名称会随 package 改变；
  不把“具体类型有显式名称”当作完整兼容性证明。
- [文件存储入口](../../Kodex/agent-storage/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemAgentStorage.kt#L25)
  分别使用 `CleanIndexEntry.serializer()` 与 `StableWorkEvent.serializer()`，
  并使用 `OpenAiJsonCodec`，不序列化共享的 `StableCleanEvent` 根。
- 保持现有字段、默认值、显式名称、discriminator、事件投影和时间线归属。
  当前没有证据要求新增数据迁移；是否兼容必须由以下测试确认，失败时先查明原因。

## 实现切分

1. **先固定兼容样本**：在移动类型前，复用现有序列化测试的事件构造，
   保存旧 serializer 产生的 JSON golden；包括 index/work 事件、compaction point、
   retained item，以及嵌套 action/result。使用合成内容，不复制用户任务正文。
2. **移动并封闭类型**：完成 19 个源文件移动、两个测试文件移动及引用更新；
   删除桥接接口，保留 `CompletedTool` 的工具输出约束。
3. **修正完整分派**：
   - `agent-state/impl/.../KodexAgentStateImpl.kt`：工具完成和普通 stable 写入按
     index/work 封闭类型路由，移除未知事件兜底。
   - `app/viewmodel/history/.../HistoryItemLoadContext.kt`：保持 Index Item 独立、
     不可聚合折叠的分类，不能让建议事件落入普通可聚合 Tool。
   - `app/viewmodel/history/.../HistoryItemHeaderFactory.kt`：穷尽状态和摘要；
     专用展示与普通工具路由不能用宽泛分支隐藏新增类型。
   - `app/viewmodel/history/.../ToolHistoryItemViewModel.kt`：核对普通工具分类，
     避免新事件未经明确决定自动进入普通工具路径。
   - `app/view/history/.../CleanEventView.kt`：穷尽 stable 事件的详情渲染。
   - 全项目编译并审查由 sealed 化暴露的其他 stable 分派，不机械删除
     字符串、数值等非事件联合上的合法 `else`。
4. **补齐展示集成**：依据用户确认的历史 UI，覆盖 accepted、rejected、failure；
   恢复现有“加载失败记录完整异常”的要求，不用空分支、TODO 或通用错误
   临时绕过新增的编译约束。
5. **验证与文档**：执行下述检查，再按 checklist 维护规则更新本任务涉及的
   类型层级与分派指导；不顺带改写无关的旧设计文档。

## 验证方案

- **兼容测试**：迁移后使用实际时间线 serializer 和 `OpenAiJsonCodec` 读取旧 golden，
  验证类型、字段、模型可见投影和重编码后的 JSON 结构不变；测试不能只做
  新 serializer 自己的 round trip。
- **完备性验证**：在隔离的临时源码副本中，分别增加一个 index tool、work tool
  和非工具 stable 事件，编译实际分派调用方，确认应定义新行为的分派点失败；
  存储路由已完整覆盖 index/work 时不要求其按叶子失败。验证后删除临时副本，
  不向正式 sealed 联合永久添加测试事件。
- **历史集成**：从旧 accepted/rejected/failure JSON，经历史读取和 item 加载，
  验证独立条目展示；在前后放置可聚合 work，确认建议事件不被收入 WorkGroup，
  且前后的 work 不跨越该 Index Item 合并。不能只测确认面板或纯 serializer。
- **失败态展示**：验证原任务和具体失败原因可见，且没有选中决定或成功 URI；
  不得把合法的 failure 事件转成历史 item 加载失败。
- **错误路径**：真实读取/解码失败仍显示红色 Error，并记录原始异常；
  合法但遗漏展示的事件应在编译期被拒绝。
- JVM 检查目标：
  `:agent-storage-clean-models:jvmTest`、`:agent-storage-filesystem:jvmTest`、
  `:agent-state-impl:jvmTest`、`:app-viewmodel-history:jvmTest`、
  `:app-view-history:jvmTest`；对其他修改的行为模块补跑对应测试。
- 编译与链接：`:app-cli:compileKotlinJvm`、
  `:app-cli:linkReleaseExecutableLinuxX64`，覆盖应用所依赖的生产模块。
- 上述为实施前制定的验证方案，实际执行结果见本文开头。

## 实施前的范围收敛

- 独立 Index Item 和不参与 work 聚合折叠已确认，不再作为推进阻塞点。
- 原任务内容、最终选择、逐项 URI 和失败态的只读展示均已确认，
  不再存在这部分 UI 决策依赖。
- 具体历史 item、renderer 与测试已拆为独立任务；
  [原任务](../done/2026-09-04-suggest-subagent-task-interaction.md)保留引入时的历史记录。
- 本任务不包含发布或 Git 提交；目前不动既有未提交改动及其他会话的任务文件。

## 本轮阻塞审查

- 审查范围是 sealed 化和已完成建议的历史展示修复；已确认的产品决策
  没有新冲突。下列是集成依赖与验收门槛，不是新增产品选择。
- **集成顺序**：封闭 `StableCleanEvent` 后，遗漏建议事件的详情 `when`
  会成为编译错误。因此先固定旧 JSON，再迁移类型并补齐已确认的展示，
  最后整体验证；不能以空分支或运行时错误让 sealed 任务先单独假完成。
- **主历史接线**：补齐 `app/contract/history` 的专用只读 item/state、
  `app/viewmodel/history` 的加载实现及 `HistoryItemKind` 映射、
  `AgentHistoryViewModel` 的构造与 storage index 映射，以及
  `AgentHistoryView` 的 renderer、content type 和条目交互映射。
  新 item 不实现 WorkGroup 子项接口；测试覆盖定位、右键恢复/分叉及懒加载。
- **Index 侧栏入口**：用户确认仅显示 `suggest subagents`，
  不展示任务详情、决定、URI 或失败原因，不新增结构化预览 payload。
  完整只读展示仅用于主历史；侧栏沿用现有简单文本路径。
- **错误诊断**：`ToolHistoryItemViewModel` 捕获 Throwable 后只设置 Failed，
  原始异常被丢弃。涉及的加载路径需保留取消传播并记录原始异常；
  合法工具 failure 不进入加载失败分支，不扩展为新的通知系统。
- **兼容门槛**：显式 SerialName 只是静态审计依据；旧时间线 JSON、
  retained items 和嵌套结果的迁移后兼容仍待测试，不能先宣称兼容或新增迁移。
- **行为门槛**：主历史覆盖 accepted/rejected/failure，
  验证名称下 URI、完整 prompt、无操作控件、work 不跨 Index 合并；
  侧栏验证仅显示 `suggest subagents`，不展开结果内容。
  补跑涉及的 agent ViewModel 与 application View 对应测试。
- 用户认可其余审查结论：集成顺序、主历史接线、异常日志与兼容验证门槛
  按上述计划执行，无新增待决事项。
- 本节记录实施前的源码审查；不覆盖本文开头的后续执行结果。
