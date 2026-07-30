# Task Tree

- [done] 审查CLI滚动体系
  - [done] 梳理当前历史视图、滚动状态、键盘和鼠标事件的数据流
  - [done] 对照Mosaic与Compose可复用的滚动原语
  - [done] 分析现有实现的正确性和扩展边界
  - [done] 写出结论和初步调整方案供用户复核
  - [done] 根据用户意见重拟组件边界
    - [done] 将滚动机制拆为`ScrollableState`、`ScrollState`和`Modifier.scrollable`
    - [done] 将列表位置与布局建模为通用`LazyListState`和`LazyColumn`
    - [done] 将follow-tail、unread和session UI生命周期留在业务层
    - [done] 确认不再建设transcript专用滚动状态或lazy组件
  - [done] 用户确认修订后的架构方向
  - [done] 将确认后的详细计划记录到本任务
  - [done] 按确认方案重构CLI滚动体系
    - [done] 固化架构决策和文档
      - [done] 加载修改`checklist/`与`shared-context/`所需的维护规则
      - [done] 将实施工作拆成scroll foundation、Mosaic lazy primitives、LazyColumn/focus和history migration四个referenced subtask
        - [done] [实现通用滚动基础](../done/2026-07-22-implement-scroll-foundation.md)
        - [done] [Mosaic懒布局底层原语](../done/2026-07-22-add-mosaic-lazy-primitives.md)
        - [done] [通用LazyColumn与焦点协作](../done/2026-07-22-implement-lazy-column-and-focus.md)
        - [done] [CLI history迁移](../done/2026-07-22-migrate-cli-history-to-lazy-column.md)
      - [done] 修订`shared-context/findings/cli-scrolling-architecture.md`
      - [done] 修订`checklist/tui-interaction-components.md`中被取代的`LazyTranscript`决策
      - [done] 保留当前实现及其缺口作为历史背景，明确初步row-anchor方案已被取代
      - [done] 记录公开API、状态归属、滚动单位、输入消费和首版非目标
    - [done] 实现通用滚动基础
      - [done] 在`cli:components`定义`ScrollableState`
        - [done] 定义整数行delta的正负方向
        - [done] 返回实际消费的行数
        - [done] 暴露`canScrollBackward`和`canScrollForward`
        - [done] 暴露`isScrollInProgress`，不在state协议中加入业务需要的输入来源分类
      - [done] 在`cli:components`定义普通容器使用的`ScrollState`
        - [done] 暴露`value`、`maxValue`和`viewportSize`
        - [done] 由layout更新范围，范围缩小时立即归一化`value`
      - [done] 提供`rememberScrollState`和按需构造`ScrollableState`的入口
      - [done] 定义独立于state的通用`ScrollInteractionSource`
        - [done] 区分pointer、keyboard、focus relocation和programmatic来源
        - [done] 只在实际位置或pending定位意图变化时发出interaction
        - [done] 标记interaction是否由用户发起，不包含follow-tail语义
      - [done] 实现`Modifier.scrollable(state, orientation, enabled, reverseDirection, interactionSource)`
        - [done] 将命中区域内的滚轮转换为行delta
        - [done] 只在state实际消费非零delta时消费事件
        - [done] 保持滚轮不改变焦点且不建立pointer capture
        - [done] 不包含item、history、session、message或transcript知识
      - [done] 明确`Modifier.scrollable`只解释输入，不移动、裁剪或测量内容
      - [done] 临时让现有fixed-row viewport复用通用滚动modifier，不再自行解释滚轮
      - [done] 为方向、边界、禁用、反向和interaction来源编写测试
    - [done] 补齐Mosaic懒布局底层原语
      - [done] 实现keyed measure-time subcomposition
        - [done] 在当前父composition和`MosaicNode`树内按slot key组合内容
        - [done] 在同一测量周期返回`Measurable`
        - [done] 支持slot复用、移动、失效和dispose
        - [done] 保证active、retained或pinned slot在key移动后保持remember状态
        - [done] 保证离开保留窗口的effect被正确释放
        - [done] 保证retained slot的snapshot变化触发recomposition和layout invalidation
        - [done] 父布局dispose时释放全部slot
        - [done] 同一measure pass重复使用slot key时明确失败
        - [done] 使用相同`contentType`复用节点时不泄漏前一个key的remember状态或effect
        - [done] 防止measure-time apply changes造成布局重入或永久`needLayout`循环
      - [done] 实现统一viewport clipping
        - [done] 裁剪绘制结果
        - [done] 裁剪pointer hit path
        - [done] 裁剪普通focus bounds与候选集
        - [done] 裁剪focus scope投影和physical cursor输出
        - [done] 嵌套viewport使用所有祖先clip的交集
        - [done] 在`TextSurface`越界检查前完成绘制裁剪
        - [done] 支持首项以负行偏移放置
        - [done] 支持末项部分可见
        - [done] 保证overscan项不会越界绘制或响应pointer
      - [done] 为subcomposition生命周期、约束变化、裁剪和slot identity编写runtime测试
    - [done] 完成普通`ScrollState`的布局消费者
      - [done] 在`cli:components`实现类似`Modifier.verticalScroll(state)`的eager scrolling-layout modifier
      - [done] 由layout测量内容、更新`maxValue`和`viewportSize`并按`value`放置内容
      - [done] 内部复用同一state的`Modifier.scrollable`
      - [done] 使用Mosaic viewport clipping限制绘制、命中和focus
      - [done] 为内容变化、viewport变化、部分可见和边界归一化编写测试
    - [done] 在`cli:components`实现通用`LazyColumn`
      - [done] 定义`LazyListScope`
        - [done] 支持`item(key?, contentType?, content)`
        - [done] 支持`items(count, key?, contentType?, itemContent)`
        - [done] 未提供key时使用index identity，不保证insert或reorder后的身份保持
        - [done] 显式stable key必须唯一并在重复时明确失败
      - [done] 定义实现`ScrollableState`的长期存活`LazyListState`
        - [done] 保存`firstVisibleItemIndex`
        - [done] 保存`firstVisibleItemScrollOffset`
        - [done] 提供`layoutInfo`和visible item信息
        - [done] 提供`scrollBy`、`scrollToItem`和request型定位操作
        - [done] 提供`requestScrollToStart`和`requestScrollToEnd`等下一次measure解析的通用操作
        - [done] 不把`itemCount`或`visibleItemCount`作为调用方每帧重建的状态快照
      - [done] 实现可变高度lazy measure和placement
        - [done] 要求滚动轴具有有限`maxHeight`，无界约束明确失败
        - [done] 明确不支持lazy内容的intrinsic height查询，避免无界组合全部item
        - [done] 从当前锚点向前后组合并测量到填满viewport与overscan
        - [done] 支持任意Mosaic composable作为item内容
        - [done] 支持一个item高于整个viewport
        - [done] 只组合和测量visible、overscan、显式beyond-bounds及focused/pinned项目
        - [done] 当前含焦点item离开窗口时先保留为pinned，或在释放前完成焦点relocation
      - [done] 实现通用stable-key位置维护
        - [done] 数据prepend、insert、remove和reorder后优先恢复同一key
        - [done] 保留同一item内的数值型行偏移
        - [done] 对当前item provider建立完整、廉价且不触发composition或换行的key-to-index查找
        - [done] key消失时将旧index clamp到新范围，再由measure完成位置归一化
        - [done] item缩短时允许offset跨入后续item，尾部不足时向前回填
        - [done] 显式request定位优先于自动key恢复
      - [done] 实现约束与缓存失效
        - [done] 宽度或viewport高度变化后重新测量
        - [done] 高度缓存仅作为提示，不覆盖streaming、展开或内部状态引起的重新测量
        - [done] 避免在每帧完整组合、测量或换行全部数据
      - [done] 让`LazyColumn(state = state)`内部挂载同一state的滚动modifier
      - [done] 为可变高度、部分可见、key迁移、fallback、resize、cache和composition数量编写测试
    - [done] 打通LazyColumn输入与焦点
      - [done] 提供通用scrolling-container键盘输入适配器
        - [done] 从layout读取viewport行数并处理PageUp和PageDown
        - [done] 通过独立`ScrollInteractionSource`报告keyboard来源
        - [done] 不把paging逻辑实现为LazyColumn或history专用分支
      - [done] Home和End只由获得焦点的控件或显式页面语义动作处理
      - [done] composer获得焦点时保留Home和End的文本光标语义
      - [done] composer未消费PageUp和PageDown时，通过session surface语义动作翻阅history
      - [done] 只让当前焦点路径或显式页面语义动作命中的scrolling container处理键盘滚动
      - [done] 删除屏幕根节点中history专用的PageUp、PageDown、Home和End分支
      - [done] 在Mosaic提供通用focus-search extension与relocation handshake
        - [done] 顺序遍历在当前focus scope wrap或退出前先查询extension
        - [done] pending traversal期间消费对应输入，防止重复分发
        - [done] focus、scope、数据或overlay变化时取消过期pending请求
        - [done] 重试必须有进展或终止，不能因不可聚焦长区段无限扩展
      - [done] 由LazyColumn注册beyond-bounds provider
        - [done] provider负责按遍历方向临时扩展相邻item组合窗口
        - [done] 新布局完成后只重试一次有进展的pending focus traversal
        - [done] 完全位于clip外的目标通过独立beyond-bounds通道发现，不进入普通focus projection
        - [done] 找到目标后先执行最小bring-into-view，目标可见后再提交focus
        - [done] 到达列表边界时把遍历交回focus scope继续wrap或退出
        - [done] 搜索成功、失败或取消后回收不再需要的composition
      - [done] 覆盖Tab、Shift+Tab、ArrowUp和ArrowDown的跨viewport遍历
      - [done] 保证modal focus trap阻止背景列表的键盘与滚轮输入
      - [done] 为正反向遍历、焦点identity、bring-into-view和modal隔离编写测试
    - [done] 将conversation history迁移为LazyColumn普通调用方
      - [done] 定义廉价的语义`HistoryEntry`投影
        - [done] committed history item
        - [done] streaming item
        - [done] activity item
        - [done] empty-state item
      - [done] 为每类entry定义稳定且唯一的业务key
      - [done] 让同一逻辑输出从streaming转为committed时保持同一业务identity
      - [done] 保持history自然时间顺序，通过显式末尾定位实现follow-tail
      - [done] 删除生产路径中的完整`TerminalHistoryRow`预展开和预换行
      - [done] 只在可见LazyColumn item内容中按当前终端宽度换行
      - [done] 让expansion toggle继续作为普通可组合item内容
      - [done] 将文本格式化与换行保持为renderer职责，不让LazyColumn理解transcript
      - [done] 为每个Mosaic frontend保存`Map<sessionId, HistoryUiState>`
        - [done] `HistoryUiState`持有`LazyListState`
        - [done] `HistoryUiState`持有`followTail`
        - [done] `HistoryUiState`持有`hasUnreadTail`
        - [done] `HistoryUiState`持有frontend-local、session-local展开状态
      - [done] 实现history UI状态生命周期
        - [done] 新session和fork初始化为follow-tail
        - [done] 切换session时恢复各自状态
        - [done] 删除session时清理对应frontend状态
        - [done] 多个frontend观察同一session时互不共享viewport
      - [done] 实现follow-tail策略
        - [done] 用户通过pointer、keyboard或focus relocation向历史方向产生有效滚动后退出follow-tail
        - [done] programmatic定位不关闭follow-tail，边界零消费不伪造用户滚动interaction
        - [done] 非follow-tail时新增或流式增长只更新unread
        - [done] 非follow-tail时依靠通用stable-key逻辑维持阅读位置
        - [done] follow-tail时在尾部内容版本变化后立即提交pending `requestScrollToEnd`
        - [done] 由下一次measure同时解析新数据和末尾位置，不先按旧锚点布局一帧
        - [done] 用户滚回末尾或执行End语义动作时恢复follow-tail并清除unread
        - [done] 区分用户滚动与数据增长，避免仅由瞬时`isAtEnd`推导意图
      - [done] 从共享ViewModel删除`historyScrollOffset`
      - [done] 删除`SetHistoryScrollOffset`及提交消息时重置共享offset的逻辑
      - [done] 从共享ViewModel删除`expandedHistoryItems`
      - [done] 删除`ToggleHistoryItemExpansion`及旧事件接线
      - [done] 将需要从composer翻阅history的行为建模为session surface语义动作
      - [done] 保证modal action scope阻止该页面语义动作穿透
      - [done] 保留非交互纯文本渲染需要的格式化能力，但不与viewport state共享模型
    - [done] 完成集成验收
      - [done] 验证通用Scrollable契约
        - [done] 正向、反向和跨边界delta返回实际消费量
        - [done] 边界返回零并允许外层接管
        - [done] disabled modifier不改变状态
        - [done] wheel只影响命中列表
        - [done] pointer、keyboard、focus relocation和programmatic interaction来源准确且零消费不误报
      - [done] 验证通用LazyColumn
        - [done] empty到populated和populated到empty正确归一化
        - [done] 内容短于viewport时正确放置且不可滚动
        - [done] 可变高度item按终端行滚动
        - [done] partial item正确绘制、命中和聚焦
        - [done] duplicate key明确失败
        - [done] prepend、insert、remove和reorder保持首个可见key
        - [done] anchor删除时将旧index clamp到新范围并完成尾部回填
        - [done] 同key内容高度变化时通过measure跨item归一化位置
        - [done] viewport高度增长或缩小时正确重新填充
        - [done] 宽度变化后尽量保持同一key，并在新高度或尾部限制下归一化
        - [done] active或reuse window内的remembered state随key移动
        - [done] 10,000个item时组合和测量数量受viewport与overscan限制
      - [done] 验证history行为
        - [done] follow-tail下连续streaming始终显示最新内容
        - [done] 离开尾部后连续streaming不移动阅读位置
        - [done] streaming转为committed时不因identity变化丢失锚点
        - [done] history大幅缩短时按通用fallback恢复合法位置
        - [done] 展开viewport上方、内部和下方item时位置符合通用锚定契约
        - [done] session A和B往返后分别恢复位置
        - [done] session删除后清理frontend state map
        - [done] 两个frontend状态互不影响
        - [done] 回到底部后清除unread
        - [done] streaming更新不重新换行完整历史
      - [done] 验证输入与焦点
        - [done] dialog打开时wheel和paging keys不影响背景
        - [done] 焦点可正反向跨越未组合item
        - [done] active或reused item reorder后焦点identity随stable key保持
        - [done] 长段不可聚焦item不会导致无限pending traversal或永久扩大组合窗口
      - [done] 运行Mosaic runtime、CLI components和CLI app相关测试
      - [done] 运行相关格式化、API检查、类型检查和跨平台编译
      - [done] 手工验证滚轮、paging、streaming、resize、session切换和modal
    - [done] 清理旧实现并收尾
      - [done] 删除`TuiLazyColumn`、`TuiLazyColumnState`和`TuiLazyColumnAnchor`
      - [done] 删除或迁移`TuiLazyColumnTest.kt`
      - [done] 删除wrapped-row持久位置模型和旧viewport projection
      - [done] 删除或改写只验证绝对offset的renderer测试
      - [done] 检查Mosaic和通用组件均不依赖conversation、session、message或transcript类型
      - [done] 检查共享ViewModel不再包含终端viewport状态
      - [done] 检查生产history路径不再完整预换行全部历史
      - [done] 更新公开API snapshot及相关设计记录
      - [done] 删除验证过程中产生的临时文件
      - [done] 更新本任务状态，不创建Git commit

# Details

## 当前状态

- 2026-07-22，用户确认本文件记录的修订方案。
- 2026-07-22，滚动基础、Mosaic lazy原语、通用`LazyColumn`和history迁移均已完成。
- JVM、Linux x64和macOS arm64测试通过，并已在真实PTY中验证滚轮、分页、流式输出、resize、session切换和modal隔离。
- 原调查记录位于[`shared-context/findings/cli-scrolling-architecture.md`](../../shared-context/findings/cli-scrolling-architecture.md)。其中现状和缺口保留为历史证据，初步`LazyTranscript`、row-key anchor方案已被本决策取代。

## 结论

当前问题的根因不是transcript缺少一个专用锚点，而是滚动输入、位置状态、lazy布局和业务策略没有拆成独立层次。

- 现有`TuiLazyColumnState`是包含`itemCount`、`visibleItemCount`和offset的immutable viewport快照，不是长期存活的滚动状态持有者。
- 现有`TuiLazyColumn`同时计算位置、处理滚轮和组合列表，见[`TuiLazyColumn.kt:23`](../../Kodex/cli/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TuiLazyColumn.kt#L23)和[`TuiLazyColumn.kt:87`](../../Kodex/cli/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TuiLazyColumn.kt#L87)。
- 当前应用先完整遍历并换行历史，再只组合可见的固定单行item，见[`MosaicView.kt:851`](../../Kodex/cli/app/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/MosaicView.kt#L851)。这只实现了组合窗口化，不是任意可变高度内容的完整lazy measurement。
- 共享ViewModel中的单一`historyScrollOffset`同时跨frontend和session，状态归属过高。
- 非末尾追加跳动、稳定key未参与位置恢复、旧offset未持续归一化、边界事件被吞、Page键穿透和焦点不能跨未组合item等诊断仍然成立。

修订后的目标是复制Compose的责任边界，而不是复制Android的像素滚动物理：

```text
ScrollableState
├── ScrollState ─────────── Modifier.verticalScroll
│                              └── 内部复用Modifier.scrollable
└── LazyListState ───────── LazyColumn
                               └── 内部复用Modifier.scrollable

ConversationHistory
└── stable-key lazy items + follow-tail/unread/session UI策略
```

业务调用方使用`LazyColumn(state = state) { ... }`。`Modifier.scrollable`仍是公开通用能力，但`LazyColumn`内部负责正确挂载它，避免调用方漏挂或重复挂载。

## 分层边界

### Mosaic runtime

Mosaic只提供下游无法正确补出的通用UI底层原语：

- keyed measure-time subcomposition及slot生命周期。
- 同时覆盖draw、pointer和focus的viewport clipping。
- 通用focus-search extension、pending traversal及bring-into-view/relocation协议。

Mosaic不包含conversation、transcript、session、follow-tail或unread语义，也不负责`LazyListState`的item key映射和viewport填充算法。

现有公开`Layout`只能在内容完整组合后把已有`List<Measurable>`交给`MeasurePolicy`，见[`Mosaic Layout.kt:64`](../../Kodex/Mosaic/mosaic-runtime/src/commonMain/kotlin/com/jakewharton/mosaic/ui/Layout.kt#L64)。当前还缺少祖先viewport clip，且focus只搜索已经组合的目标，因此通用可变高度`LazyColumn`不能仅用现有下游API正确实现。

- 节点applier与子节点增删接口目前是internal，见[`mosaic.kt:431`](../../Kodex/Mosaic/mosaic-runtime/src/commonMain/kotlin/com/jakewharton/mosaic/mosaic.kt#L431)。
- 当前绘制没有祖先viewport clip，越界坐标会触发surface检查，见[`Node.kt:290`](../../Kodex/Mosaic/mosaic-runtime/src/commonMain/kotlin/com/jakewharton/mosaic/layout/Node.kt#L290)和[`surface.kt:39`](../../Kodex/Mosaic/mosaic-runtime/src/commonMain/kotlin/com/jakewharton/mosaic/surface.kt#L39)。
- 当前顺序focus遍历直接在已组合目标中wrap，见[`FocusOwner.kt:163`](../../Kodex/Mosaic/mosaic-runtime/src/commonMain/kotlin/com/jakewharton/mosaic/focus/FocusOwner.kt#L163)，因此必须在wrap或退出前插入通用extension handshake。

### `cli:components`

该模块承担类似Compose Foundation的通用组件能力：

- `ScrollableState`、`ScrollState`、独立`ScrollInteractionSource`、`Modifier.scrollable`和`Modifier.verticalScroll`。
- `LazyListScope`、`LazyListState`、`LazyListLayoutInfo`和`LazyColumn`。
- stable key迁移、item测量、viewport填充、overscan和尺寸缓存。
- 通用列表输入、边界消费和与Mosaic focus/beyond-bounds协议的集成。

这些API不得依赖任何业务类型。

### `cli:app`

应用层只负责：

- 将conversation history、streaming和activity投影成带稳定key的普通lazy items。
- 渲染可见item并在item内部根据当前终端宽度换行。
- 维护每个frontend、每个session的`HistoryUiState`生命周期。
- 实现follow-tail、unread、展开状态和页面级语义动作。

滚动算法、key到index迁移和viewport测量不属于transcript。

## 核心契约

### 滚动单位与消费

- 终端使用整数cell行作为delta和item内偏移单位。
- state返回实际消费量；只有实际消费非零时输入事件才算已消费。
- `ScrollableState`只负责滚动mutation、边界、实际消费量和进行中状态；输入来源由独立`ScrollInteractionSource`描述。
- `Modifier.scrollable`只解释输入，不移动、测量或裁剪内容。普通`ScrollState`由`Modifier.verticalScroll`消费，`LazyListState`由`LazyColumn`消费。
- `ScrollState`和`LazyListState`是不同的具体状态，不能用绝对总行数强行表示lazy list。
- 首版不实现像素滚动、fling、惯性、动画定位或Android式手势互斥。
- 当前pointer handler只能返回Boolean。首版规定state消费量非零时整个离散滚轮事件都算已消费；若请求三行而边界只消费一行，剩余两行不能传播给外层。部分delta的通用nested-scroll remainder协议留待独立扩展。

### Lazy位置与stable key

- `LazyListState`的公开位置由首个可见item index和item内行偏移组成。
- `LazyListScope`允许省略key并以index作为默认identity；这种用法不保证insert或reorder后的身份保持。
- 需要位置与composition identity保持的调用方必须提供稳定且唯一的显式key。
- layout内部记录首个可见key，在数据变化后把key解析到新index，再沿用行内偏移。
- 当前item provider提供完整且廉价的key-to-index查找；查找不得触发item composition、measurement或文本换行。
- 显式request定位优先于自动key恢复。
- key消失时将旧index clamp到新的合法范围，再由measure归一化offset并在尾部不足时向前回填；空列表归零。
- item缩短时offset可以跨入后续item，而不是强行clamp在原item末行。
- 宽度变化先尝试保持同一key和数值型行偏移，再按新高度与尾部填充要求归一化；不保证最终首项不变，也不保证仍指向同一源文本字符。
- 如果未来产品明确要求源字符级锚定，应作为业务层可选策略另行设计，不能污染通用LazyColumn。

### Follow-tail

- Follow-tail不是`ScrollableState`、`LazyListState`、`LazyColumn`或`reverseLayout`的内建行为。
- history保持自然时间顺序。follow-tail开启时，业务层在尾部内容版本变化后立即提交pending `requestScrollToEnd`，由下一次measure同时解析新数据和末尾位置。
- 用户有效地向历史方向滚动后关闭follow-tail；稳定key只负责保持阅读位置。
- 用户发起的pointer、keyboard或focus relocation只要实际向历史方向移动就属于上述滚动；programmatic定位和边界零消费不关闭follow-tail。
- 非follow-tail期间的新内容只设置unread。
- 用户回到底部或执行End语义动作后重新开启follow-tail并清除unread。
- 业务层必须保存用户意图并区分数据增长与用户滚动，不能只根据布局更新期间的瞬时`isAtEnd`推断状态。

### 换行与item粒度

- LazyColumn负责测量组合结果，不理解文本或transcript换行。
- history renderer负责可见item的Unicode cell换行，可按item key、内容版本和宽度缓存结果。
- item高度缓存不是事实来源；streaming、展开、宽度和内部状态变化都必须使相关item重新测量。
- 单个超大语义item仍会整体组合和换行。应通过合理的业务entry粒度控制，而不是让LazyColumn识别消息类型。

### Subcomposition、clipping与focus

- 普通focus projection只包含与祖先clip交集后仍可见的目标、scope和physical cursor。
- clip外的临时目标通过独立beyond-bounds搜索通道发现；LazyColumn先滚动使目标可见，再向普通focus系统提交目标。
- Mosaic只提供通用focus-search extension、pending traversal和relocation handshake；由LazyColumn provider决定扩展哪个item。
- 顺序焦点遍历必须在scope wrap或退出前查询provider；方向遍历必须在回退到外层scope前查询provider。
- pending traversal被接受后立即消费当前输入，并在focus、scope、数据、provider generation或overlay变化时取消。
- provider需要跨过不可聚焦item时只能有界扩展；成功、数据边界、无进展或取消都会终止并回收临时composition。
- stable key只保证active、retained或pinned slot在移动时保持普通`remember`与节点identity。slot被dispose后，跨重新组合的状态必须提升或使用另行定义的saveable机制。
- `contentType`只允许复用节点结构，不允许前一个key的`remember`状态或effect泄漏到新key。

## 依赖关系

```text
ScrollableState + Modifier.scrollable ───────┐
                                             ├─> LazyColumn ─> focus integration ─> history migration
Mosaic keyed subcomposition + clipping ──────┘                      ↑
Mosaic beyond-bounds + relocation ──────────────────────────────────┘
```

- 通用scroll foundation可以先实现，并用于拆开现有fixed-row组件的输入职责。
- 普通eager scrolling layout依赖Mosaic clipping；`Modifier.scrollable`本身不会产生视觉位移。
- 真正的任意可组合、可变高度LazyColumn依赖Mosaic subcomposition和clipping。
- 跨未组合item的焦点遍历依赖LazyColumn与Mosaic beyond-bounds协议。
- history迁移依赖上述能力全部可用；不能用transcript专用状态绕过缺失的底层原语。

## 首版非目标与后续边界

- 不实现pixel scroll、fling、惯性或`animateScrollToItem`。
- 首版不实现`reverseLayout`；如果以后加入，需单独覆盖方向、paging、key恢复和focus语义，且不能替代follow-tail。
- 不在首版扩展支持部分delta的nested scroll pointer协议。
- 不把滚动条作为history迁移的阻塞项；以后只能基于通用`layoutInfo`或scroll indicator state实现，不能读取transcript状态。
- 不用固定overscan替代beyond-bounds focus。
- 不把现有fixed-row viewport包装成看似通用的可变高度`LazyColumn`。
- 不创建`LazyTranscript`、`LazyTranscriptState`、`FollowEnd | Anchored(rowKey, ...)`等业务专用滚动基础组件。

## 主要风险

- Mosaic缺少measure-time subcomposition和完整clipping，这是最大前置工作。
- subcomposition必须接入现有applier、父composition context、布局失效和effect生命周期，不能复用独立canvas式临时composition。
- 部分可见item需要负向placement；draw、pointer和focus任一层未裁剪都会产生越界或交互错误。
- LazyColumn必须要求有限滚动轴约束，并明确禁止会导致无界组合的intrinsic height查询。
- focus系统当前只知道已组合节点；没有beyond-bounds时，Tab和方向键无法跨越viewport。
- focused item离开普通保留窗口时必须暂时pin，或在dispose前完成focus relocation。
- streaming、展开和约束变化会使同key item高度变化，错误复用高度缓存会破坏锚点。
- follow-tail存在数据更新与layout更新的时序风险，必须以用户意图和interaction来源为准。
- 只使用`remember(sessionId)`可能在切换后丢失旧session位置，需要frontend持有明确的session-state map并在删除时清理。
- stable-key索引可以扫描廉价key集合，但不能触发完整历史组合、测量或换行。
- 单个极大item会降低lazy收益，需要业务投影保持合理的语义粒度。

## 参考

- Compose [`ScrollableState`](https://github.com/androidx/androidx/blob/androidx-main/compose/foundation/foundation/src/commonMain/kotlin/androidx/compose/foundation/gestures/ScrollableState.kt#L77)
- Compose [`Modifier.scrollable`](https://github.com/androidx/androidx/blob/androidx-main/compose/foundation/foundation/src/commonMain/kotlin/androidx/compose/foundation/gestures/Scrollable.kt#L286)
- Compose [`LazyListState`](https://github.com/androidx/androidx/blob/androidx-main/compose/foundation/foundation/src/commonMain/kotlin/androidx/compose/foundation/lazy/LazyListState.kt#L141)
- Compose [`LazyColumn`内部滚动接线](https://github.com/androidx/androidx/blob/androidx-main/compose/foundation/foundation/src/commonMain/kotlin/androidx/compose/foundation/lazy/LazyList.kt#L136)
- Compose [`LazyList`稳定key位置维护](https://github.com/androidx/androidx/blob/androidx-main/compose/foundation/foundation/src/commonMain/kotlin/androidx/compose/foundation/lazy/LazyListScrollPosition.kt#L28)
