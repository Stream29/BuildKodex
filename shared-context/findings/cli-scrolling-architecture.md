# CLI滚动体系审查

## 当前数据流

- `SessionSnapshot`先投影为语义历史，再按终端宽度展开为单行`TerminalHistoryRow`。
- `CodexCliViewModel`只保存一个`historyScrollOffset`，语义是距列表末尾省略的行数。
- `renderTerminalFrame`结合行数、视口高度和offset临时构造`TuiLazyColumnState`。
- 鼠标滚轮由`TuiLazyColumn`转换为新offset；PageUp、PageDown、Home、End由屏幕根节点转换。
- 新offset通过`CodexCliEvent.SetHistoryScrollOffset`写回ViewModel。
- `TuiLazyColumn`只组合当前可见的固定单行item。`itemKey`目前只用于Compose重组。

## 已正确的部分

- 终端实际滚动单位就是字符行。先将可变高度语义块展开为固定单行item，契约清楚且便于裁剪。
- 末尾锚定且offset为0时，新流式内容会保持底部可见。
- Mosaic不会对滚轮建立pointer capture，也只在左键按下时改变焦点；滚轮分发路径正确。
- 历史viewport只组合可见行，长对话不会把所有行都放进布局树。
- 当前Codex Rust TUI主要借用终端原生scrollback保存已完成历史；本项目使用alternate screen，必须维护自己的viewport，不能直接照搬。

## 正确性缺口

### 非末尾视口会随追加内容跳动

`End`锚定下首行位置为`maximumScrollOffset - scrollOffset`。用户离开末尾后，若末尾新增`d`行而offset不变，首行也会增加`d`，原本可见的内容被推走。

- offset为0：保持追随末尾，行为正确。
- offset大于0：应保持原首行稳定，当前实现却向新内容方向跳动。
- 展开工具详情、折叠推理、改变终端宽度也会改变总行数，触发同类问题。

### 稳定key没有参与位置恢复

`TerminalHistoryRow.key`已经存在，但只传给Compose的`key`。它不会在item增删后恢复首个可见行。Compose的`LazyListScope`明确使用稳定key在数据集变化时保持首个可见item；我们的滚动状态尚未实现这一层语义。

### 状态归属过高

滚动位置取决于终端宽高、换行结果和当前前端，因此不是共享agent状态。

- 一个ViewModel中的单个offset会被所有前端共享。
- 多个session也共用同一个offset。
- 切换session、New和Fork路径没有一致地重置或恢复各自位置。

滚动状态应由每个Mosaic前端按session持有，而不是放在公共`CodexCliViewModel`中。

### offset没有被持续归一化

内容缩短时，临时`TuiLazyColumnState`会读取归一化值，但ViewModel仍保留旧的大offset。内容随后增长时，旧值可能重新生效，导致视口无用户操作地跳回较早位置。

### 输入消费边界不完整

- 滚轮到达列表边界后仍返回已消费，未来无法支持嵌套滚动或外层接管。
- PageUp和PageDown位于屏幕全局未处理事件中；弹层未消费它们时，可能滚动弹层后的历史。
- LazyColumn只组合可见焦点目标，方向键无法自然遍历到未组合条目，也没有`bringIntoView`联动。

## Compose可借鉴的语义

- [`LazyListState`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/LazyListState)是独立、可提升的UI状态持有者，公开首个可见item及其偏移和主动滚动操作。
- [`LazyListScope`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/LazyListScope)要求key稳定且唯一，并用key在数据变化时维持首个可见item。
- [状态提升指南](https://developer.android.com/develop/ui/compose/state-hoisting)建议将UI状态提升到需要它的最低共同祖先，而不是放入无关的共享状态层。
- [滚动原语](https://developer.android.com/develop/ui/compose/touch-input/scroll/scroll-modifiers)要求返回实际消费的delta，为边界传播和嵌套滚动保留语义。

终端按整行滚动，不需要像Compose一样引入像素offset、惯性动画或手势互斥；稳定key、状态归属和实际消费量仍应对齐。

## 已被取代的初步方案

以下方案只保留为调查过程记录，不再作为实施目标。它仍以预展开后的row和transcript专用状态为核心，无法提供任意可组合、可变高度item所需的测量、裁剪和焦点语义。

- 保留`TuiLazyColumn`在`cli:components`，暂不把高层列表组件下沉到Mosaic runtime。
- 将`TuiLazyColumnState`改为可remember的UI状态持有者，由每个Mosaic前端按session保存。
- 用两种位置表达：`FollowEnd`表示追随末尾；`Anchored(rowKey, fallbackIndex)`表示保持当前首行。
- 用户从末尾向上滚动时切换为key锚定；回到底部时恢复`FollowEnd`。
- 数据追加、折叠和重排后优先按key恢复首行；key消失时选择相邻fallback位置。
- 将行key建模为语义item、区段和换行序号的组合，避免依赖展示文本或storage index。
- 从ViewModel移除`historyScrollOffset`及其事件；ViewModel继续只提供语义历史和展开状态。
- 边界滚轮返回未消费；PageUp和PageDown只路由给当前有效的滚动容器，modal打开时不得穿透。
- 焦点移动到视口外条目时，由列表状态执行`bringIntoView`后再完成焦点迁移。

## 已确认的目标架构

### 通用滚动基础

- `ScrollableState`只处理整数行delta、边界和实际消费量。
- `ScrollState`表示普通eager容器的绝对行位置、最大值和viewport大小。
- `ScrollInteractionSource`独立描述pointer、keyboard、focus relocation和programmatic来源。
- `Modifier.scrollable`只解释输入，不负责内容移动、测量或裁剪。

### Mosaic底层能力

- keyed measure-time subcomposition在当前父composition和`MosaicNode`树中创建、复用、移动和释放slot。
- viewport clip统一作用于draw、pointer、普通focus投影、focus scope和physical cursor。
- focus系统提供通用beyond-bounds搜索与relocation握手；它不理解列表或history。

### 通用LazyColumn

- `LazyListScope`接受任意Mosaic composable，并以唯一stable key维持composition identity和滚动锚点。
- `LazyListState`公开首个可见item index、item内行偏移、layout info和request型定位操作。
- layout只组合visible、overscan、显式beyond-bounds和focused或pinned item。
- 数据与约束变化优先按stable key恢复位置；key消失后按旧index和尾部填充规则归一化。
- `LazyColumn`不理解文本、conversation、session、streaming或follow-tail。

### History调用方

- history投影为带稳定业务key的普通`HistoryEntry`；格式化和换行只在可见item内容中执行。
- 每个Mosaic frontend为每个session维护独立`HistoryUiState`，其中包含`LazyListState`、follow-tail、unread和展开状态。
- follow-tail是业务策略：用户有效地向历史方向滚动后退出，回到底部或执行End后恢复。
- 非follow-tail期间新增或流式增长不移动阅读位置，只更新unread。
- 共享ViewModel不保存终端viewport位置或history展开状态。

### 首版边界

- 以终端整数行滚动，不实现像素滚动、fling、惯性和动画定位。
- 不实现`reverseLayout`、lazy intrinsic height或部分delta的nested-scroll remainder。
- 不用固定overscan代替beyond-bounds focus，也不建立transcript专用lazy组件。

## 验证矩阵

- 位于末尾时连续流式追加，始终显示最新内容。
- 离开末尾后连续追加，首个可见row key保持不变。
- 展开或折叠视口上方、内部和下方条目，锚点分别保持合理。
- 改变终端宽高后，尽量保持同一语义条目可见。
- 切换session和多个前端同时观察时，滚动位置彼此独立。
- modal打开时键盘与滚轮不穿透；边界滚轮可交给外层。
- 焦点通过Tab和方向键跨越视口边界时，目标先滚入可见区。
- 10,000个可变高度item只组合和测量viewport、overscan及必要的临时item。
- stable key在prepend、insert、remove、reorder和宽度变化后维持合法锚点。
