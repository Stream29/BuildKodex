# Task Tree

- 优化 History Index 侧栏内容与 checkout 交互
  - [done] 固定单行、hover 与 checkout 语义
  - [done] 扩展 History Index frontend contract
    - 增加懒加载单行摘要
    - 增加 hover 完整内容读取
    - 增加 generation-scoped target 校验
  - [done] 实现各 index entry 的内容投影
    - 映射消息、工具事件与 compaction
    - 统一空内容、图片、加密内容与失败 fallback
    - 隐藏旧 storage 的 initial Compaction point
    - 处理 append、shrink 与同 index refresh
  - [done] 实现主 History 的 storage-index 导航
    - 解析 stable event 与 compaction target
    - 直接加载目标附近的有限窗口
    - 应用 checkout 的 follow-latest 语义
  - [done] 实现 History Index 侧栏交互
    - 渲染内容摘要与统一前景色
    - 实现可进入、换行和滚动的 hover preview
    - 实现 index 信息与 Check out 右键菜单
  - [done] 验证 ViewModel 与 Mosaic 行为
    - 覆盖所有 entry 内容投影
    - 覆盖 refresh 与 checkout 导航
    - 覆盖 hover、菜单和 follow-latest
  - [done] 运行格式化、静态检查与相关测试
  - [done] 修复 hover popup 回归
    - [done] 确保 popup 覆盖区域清除背景字符
    - [done] 按最后鼠标位置放置 popup
    - [done] 覆盖背景与位置更新测试
    - [done] 运行格式化、静态检查与相关测试
  - [done] 统一 Request user input hover 与 History UI
    - [done] 抽取共用只读表单行投影
    - [done] 使用共用渲染器显示 hover 内容
    - [done] 覆盖已选项、描述与自由输入
    - [done] 运行格式化、静态检查与相关测试

# Details

- 当前状态：Request user input 的 hover popup 已与主 History UI 统一。
- 用户已授权进入 executable。

## 执行结果

- IntelliJ IDEA 格式化和 file problems 检查已完成。
- `git diff --check` 已通过。
- 以下 Gradle 任务已通过：
  - `:app-viewmodel-agent:jvmTest`
  - `:app-viewmodel-history:jvmTest`
  - `:app-view-application:jvmTest`
  - `:app-contract-agent:check`
  - `:app-contract-history:check`
  - `:app-viewmodel-agent:check`
  - `:app-viewmodel-history:check`
  - `:app-view-application:check`
  - `:app-cli:check`
- Gradle 输出的 configuration-cache problems 来自现有 Mosaic/插件配置；构建结果为
  `BUILD SUCCESSFUL`。
- Hover popup 现在会先清除覆盖区域的底层字符，再绘制不透明容器背景。
- Hover popup 按当前行内最后一次鼠标位置放置，并在终端边缘自动翻转或收敛。
- 新增的背景覆盖、位置更新和鼠标位置捕获测试已通过。
- 主 History 与 History Index hover 现在共用同一套已完成只读表单行投影和渲染器。
- Hover 只显示实际选中项、选中项描述和自由输入；未选项不再展开。
- Secret answer 继续通过共用投影显示为 `[hidden]`。
- 新增的 Request user input hover 一致性测试已通过。
- `:app-contract-agent:check`、`:app-viewmodel-agent:jvmTest` 和
  `:app-view-application:compileKotlinLinuxX64` 已通过。
- 完整 `:app-view-application:jvmTest` 当前有三个与本改动无关的 splitter/tab
  滚动测试失败；本次新增测试在同一次执行中通过。
- `:app-view-history:jvmTest` 当前被现有测试 fake 缺少
  `requestScrollToStorageIndex` 实现阻塞；production JVM 和 Linux 编译已通过。
- 最终 IDEA file problems 检查因 IDE MCP 断开不可用；改动文件此前已由 IDEA
  格式化，Gradle 编译与 `git diff --check` 已通过。

## 已确定需求

- 每个可见 index entry 继续对应一行。
- 不显示旧 storage 中 index 0 的 initial Compaction point。
- 默认行只显示 entry 内容，不显示类型名称或角色前缀。
- 暂不使用颜色区分 entry 类型。
- hover 时显示悬浮窗，包含 entry 类型和未截断的完整内容。
- storage index 默认不出现在行内。
- 右键菜单显示该行的 storage index。
- 每行支持通过右键菜单执行 `Check out`。
- `Check out` 只将主对话历史滚动到对应位置，不修改或截断历史。
- 保留旧到新排列、单分支连线和 Lazy List。
- 保留初次定位最新端及 follow-latest 行为。
- `TurnMarker` 已取消，不属于 entry 类型。
- Assistant Final message 是 turn 结束标记。
- `HistoryIndexViewModel` 继续由 `AgentViewModel` 直接持有，每个 Agent 一份。
- 滚动状态继续由各侧栏前端独立 `remember`。

## 当前实现边界

- `HistoryIndexEntry` 当前只包含 `index` 和 `kind`，见
  `Kodex/app/contract/agent/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/HistoryIndexViewModel.kt:24`。
- 可见行已经通过 exact index 懒加载 entry，内容摘要可以沿用该读取边界。
- 现有 `AgentHistoryTarget` 和 revert action 是破坏性历史替换协议，不适用于
  History Index 的 `Check out`。
- `Check out` 需要新增纯导航路径，将 History Index storage index 映射到主对话
  History 的可见位置。
- History Index generation 只用于拒绝已失效的导航请求，不触发历史修改。

## 通用单行规则

- 保持一行一条 entry，不展开正文。
- 将换行和连续空白规范化为空格。
- 内容超出侧栏宽度时使用现有 ellipsis。
- 单分支图形保留在行首。
- 不使用 `User:`、`Final:` 等类型或角色前缀。
- 单分支图形和内容使用统一前景色。
- entry 类型只在 hover popup 中显示。
- 所有可显示 part 都不存在或只包含空白时，默认行统一显示 `[empty]`。
- 纯图片内容显示 `[image]`，不视为空内容。
- encrypted content 显示 `[encrypted content]`，不视为空内容。
- index entry 读取或解码失败时，默认行显示 `[error]`。
- loading 状态继续使用短占位符，不能改变行高。

## Hover 完整预览

- hover popup 锚定当前行。
- popup 首部显示 entry 类型。
- popup 正文显示未截断的完整内容。
- popup 不显示 storage index；index 只保留在右键菜单中。
- 完整内容保留原始段落结构，不使用单行空白规范化。
- 图片、encrypted content 和结构化工具结果需要使用可读占位或摘要。
- `isSecret` 的 Request user input 答案固定显示 `[hidden]`，与主 History 一致。
- 鼠标可以从当前行移入 popup，移入后 popup 保持打开。
- 超出 popup 可用高度的内容支持内部滚动。
- popup 从所在侧栏向主内容区展开：左侧栏向右，右侧栏向左。
- popup 最大可使用侧栏之间的主内容区域，不覆盖 History Index 本身或 tab bar。
- 内容少于可用空间时不强制填满 popup。
- 超出可用宽度的长行自动换行，不提供横向滚动。
- 稳定 hover 约 300ms 后打开，快速扫过行时不显示。
- 鼠标离开行后保留短暂过渡时间，允许进入 popup。
- 右键菜单打开时关闭并抑制 hover popup。
- 需要确保 popup 不改变列表焦点、滚动状态或 follow-latest 状态。
- `[error]` 行仍支持 hover。
- Error popup 标题显示 `Error`，正文显示读取或解码失败信息。

## 各类型投影

### Compaction point

- 只显示真正发生的 context compaction。
- 没有消息正文，默认行固定显示 `Context compacted`。

### User message

- 主内容来自 `ContentItem.InputText` 和 `ContentItem.OutputText`。
- 多个文本 part 按原顺序连接并规范化空白。
- 图片 part 使用短占位符，例如 `[image]`。
- 默认行不显示 `User:` 前缀。

### Assistant message

- 用于 `phase == null` 的兼容数据。
- 主内容来自所有文本 part。
- 图片 part 使用与 User message 相同的占位规则。
- 默认行不显示 `Assistant:` 或兼容状态标记。
- hover popup 显示完整类型名称。

### Assistant commentary

- 主内容来自所有文本 part。
- 默认行不与 Assistant Final 做额外视觉区分。
- 默认行不显示 `Commentary:` 前缀。

### Assistant final

- 主内容来自所有文本 part。
- 该 entry 同时承担 turn 结束标记。
- 默认行不与 Assistant commentary 做额外视觉区分。
- 默认行不显示 `Final:` 前缀。

### Developer message

- 主内容来自所有文本 part。
- 图片 part 使用统一占位规则。
- Developer 身份只在 hover popup 中显示。
- 默认行不显示 `Developer:` 前缀。

### Agent message

- 默认行只显示消息正文。
- 文本 part 按原顺序连接。
- encrypted content 使用短占位符，不在侧栏解密或展开。
- 默认行不显示 author、recipient 或消息流向前缀。
- 默认行不增加 `Agent:` 类型前缀。
- hover popup 显示完整 author、recipient 和消息正文。

### Request user input

- entry 同时包含 questions 和完成结果。
- 默认行只显示每个 question 的 `question` 文本。
- 没有 question 或所有 question 文本都为空时显示 `[empty]`。
- 默认行不显示 header、options、answers、完成结果或 failure。
- 多个 questions 按原顺序形成单行摘要并使用 ellipsis。
- hover popup 显示完整 questions、options、answers 和完成结果。
- `isSecret` question 的答案在 hover popup 中固定显示 `[hidden]`。
- Failure 的完整信息只在 hover popup 中显示。

### Plan update

- entry 包含可选 explanation 和完整 plan snapshot。
- 默认行只显示 plan 中最后一个 `status != StepStatus.Pending` 的条目。
- 所有条目都是 Pending 时，默认行显示第一个 Pending 条目。
- plan 为空或选中条目的 step 为空时显示 `[empty]`。
- 默认行不显示 explanation、其他 plan 条目或状态汇总。
- hover popup 显示 explanation 和完整 plan snapshot。

## 右键菜单与 Check out

- 菜单至少包含：
  - 非操作信息项 `Index: <storage index>`。
  - 操作项 `Check out`。
- 所有仍存在的 committed History Index rows 都可作为导航目标。
- `Check out` 不调用 revert，不截断 storage，不改变 Agent 状态。
- `Check out` 不显示破坏性确认对话框。
- `Check out` 不复用破坏性操作的 `canReplaceHistory` 能力约束。
- Agent 正在运行或 streaming 时，`Check out` 仍然可用。
- 定位旧记录后新内容继续追加，但主 History 保持在目标位置。
- `Check out` 不改变 History Index generation、列表位置或 follow-latest 状态。
- 主对话 History 需要按 storage index 加载并滚动到对应的可见条目。
- Check out 到非最新记录后，主 History 退出 follow-latest。
- 用户回到主 History 最新端后，恢复 follow-latest。
- Stable index event 定位到同 storage index 的主 History item。
- Compaction point 定位到 `index + 1` 的 Context compaction item。

## 具体修改计划

### Frontend contracts

- 保留 `HistoryIndexWindow` 的 oldest-first sparse indexes 和 generation。
- 为 `HistoryIndexEntry` 增加 renderer-neutral 的单行 `summary`。
- 增加单独的 hover detail DTO 和 exact detail load API。
- Preview 只在可见行组成时加载；完整 detail 只在稳定 hover 后加载。
- Detail API 接收 generation 和 index，拒绝已经失效的行请求。
- 增加 generation-scoped `contains`，供延迟 hover 和右键菜单执行前校验。
- 为 `AgentHistoryViewModel` 增加纯导航
  `requestScrollToStorageIndex(storageIndex)`；不复用 revert target。

### History Index projection

- `HistoryIndexViewModelImpl` 继续 exact-read `index` timeline，不读取 work timeline。
- 扫描窗口时过滤 `index == 0` 的 legacy initial `CleanCompactionPoint`。
- Preview 将换行和连续空白收敛为单个空格，再交给现有 ellipsis。
- Detail 保留段落和换行，只把图片及不可读取内容替换为已确定占位符。
- Message projections 复用 `ContentItem` 顺序；Agent message 的 preview 只投影正文。
- Request user input preview 只连接 question 文本；detail 投影完整问题、选项、结果，
  并隐藏 secret answers。
- Plan update preview 选择最后一个 non-Pending step；不存在时选择第一个 Pending step。
- Preview 或 detail 的可见内容为空时返回 `[empty]`。
- 行 preview 读取失败时保留失败信息，renderer 显示 `[error]` 并用于 Error hover。
- Error detail 使用去除 storage index 和内部路径的稳定失败信息；原始异常不直接渲染。
- 普通 latest-index 增长继续增量追加 indexes；latest-index 缩短时重扫并增加 generation。
- 观察 AgentState external-write 边界；结束 index 不大于开始 index 时强制重扫并增加
  generation，覆盖相同 latest index 的重写。
- ViewModel 不缓存 decoded event 或完整 detail。

### Main History navigation

- 在 History command actor 中增加 storage-index seek command，串行化窗口替换和滚动。
- Stable index event 的 display target 是原 storage index。
- Compaction point 的 display target 是 `index + 1` 的 Context compaction work item。
- 使用 `readHistoryChunk()` 从 display target 直接读取，不从当前 viewport 逐页遍历。
- 保持 History 的 bounded window；导航只替换 materialized chunks，不增加 destructive
  generation。
- 根据当前 snapshot 判断 target 两侧是否仍有可见结构，正确发布 `hasOlder` 和
  `hasNewer`。
- 原子发布目标窗口后，按 streaming、pending 和 newer-demand prefix 计算实际
  `LazyListState` item index。
- Target 不是最新可见 History item 时关闭主 History follow-latest；后续 append 只更新
  `hasNewer`。
- Target 是最新可见 History item 时复用现有 scroll-to-latest 行为。
- 定位失败通过现有 `AgentHistoryLoadState.Failed` 发布，不修改 storage 或 Agent 状态。

### Mosaic frontend

- History Index row 使用 `TuiPressable` 提供 secondary-click 和键盘 context-menu
  入口；primary action 保持为空。
- 行内只渲染单分支图和 summary，不渲染 index、类型或类型颜色。
- Root popup host 持有一个 exact hover request 和一个 exact context-menu request。
- Hover request 捕获 Agent、HistoryIndexViewModel、side、generation、index 和 row
  anchor；row disposal 或 generation 变化只关闭匹配的 request。
- Selected Agent、sidebar content 或 application popup 改变时关闭旧 hover 和菜单 request。
- Hover 约 300ms 后读取 detail；鼠标位于 row 或 popup 时保持，离开两者后使用短暂
  grace period 关闭。
- Popup 使用自定义 position provider，从左侧栏向右、从右侧栏向左，占用主内容区的
  可用上限。
- Popup 内容自动换行；超出高度时使用独立 `LazyListState` 纵向滚动，不请求焦点。
- 打开右键菜单时关闭并抑制 hover；关闭菜单后由新的稳定 hover 重新触发。
- 菜单使用 disabled 信息项显示 `Index: <storage index>`，并提供 `Check out` 命令。
- 菜单执行前校验 selected Agent、generation 和 index；有效时调用主 History 导航 API。

## 验收与验证

- ViewModel tests：
  - 覆盖九种 `HistoryIndexEntryKind` 的 summary 和 detail。
  - 覆盖空内容、图片、encrypted content、secret answer 和失败信息。
  - 覆盖 initial marker 过滤、增量 append、shrink 和 same-index refresh。
  - 覆盖 stable event、compaction output、older/latest target 和 streaming append。
  - 断言 checkout 不修改 storage、Agent state 或 History destructive generation。
- Mosaic tests：
  - 断言默认行只有 graph 和 summary。
  - 断言 index 与类型只出现在各自的 menu 或 hover surface。
  - 断言普通和 Error hover 都不泄露 storage index。
  - 使用虚拟时间覆盖 hover delay、row-to-popup grace 和 disposal。
  - 覆盖长内容换行、popup 内滚动和左右侧栏展开方向。
  - 覆盖 context menu 抑制 hover、stale target 拒绝及 running Agent checkout。
  - 保留 oldest-first、follow-latest 和新增 entry 自动出现的现有覆盖。
- 使用 IntelliJ IDEA 格式化改动文件并检查 file problems。
- 使用当前 Gradle Daemon JVM 运行：
  - `:app-viewmodel-agent:jvmTest`
  - `:app-viewmodel-history:jvmTest`
  - `:app-view-application:jvmTest`
  - 受影响模块的 `check`
  - `:app-cli:check`

## Planning 阻塞审查

- 没有未解决的产品语义或架构级阻塞。
- `readHistoryChunk()` 已提供直接 target seek 所需的结构读取。
- `TuiPopupHost`、`TuiPopup`、`TuiPressable` 和 `LazyColumn` 已提供 hover、右键、
  overlay 与内部滚动基础设施。
- 当前两个 sidebar 都在 root popup host 内；root 已掌握左右 sidebar 与主内容宽度，
  可以计算约定的 inward popup bounds。
- Initial Compaction point 清理不阻塞本任务；本任务先过滤 legacy marker，新 storage
  的写入清理由独立 discussion task 处理。
- Same-index refresh 不要求新增全局 storage repository；复用 AgentState
  external-write boundary 即可。
- Checkout 是用户明确触发的导航，不属于布局恢复型 programmatic scroll；关闭
  follow-latest 与现有 viewport 修正规则不冲突。

## 关联任务

- 停止新 storage 写入 initial Compaction point，见
  `kanban/discussion/2026-09-01-remove-initial-compaction-point.md`。
