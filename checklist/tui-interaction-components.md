# TUI交互组件

实现Codex Lite TUI时遵守以下决策。

## 分层

- Mosaic fork负责通用的终端输入、指针路由、布局树焦点、键盘遍历和物理光标；不包含Codex Lite业务动作或业务控件。
- Codex Lite TUI负责语义动作、快捷键作用域、控件状态和对话历史虚拟化，不维护第二套焦点树。
- 每个可触发动作必须由同一个语义操作同时服务键盘和鼠标，不能为两种输入维护分叉的业务逻辑。

## Mosaic修改边界

- Mosaic必须在`runMosaic`层提供显式鼠标追踪配置，并由TTY实现负责启用和恢复SGR坐标与按键拖拽模式；下游不能安全地自行写终端控制序列。
- Mosaic必须提供公开的`PointerEvent`、`Modifier.onPreviewPointerEvent`和`Modifier.onPointerEvent`入口，并在内部按节点边界和绘制层级完成命中测试、捕获与冒泡；下游不应直接消费`Terminal.events`，该channel只有一个正确消费者。
- 按下事件被处理后，Mosaic负责将后续拖拽、释放或取消送回同一捕获目标；这使下游的`Pressable`和可拖拽滚动条不必自行维护全局路由。
- 悬停使用显式的`MouseTracking.AnyEvents`和Mosaic的`onPointerHover`；Mosaic按当前命中路径发出enter/exit，下游不自行推断离开事件。
- TUI采用鼠标优先策略：默认保持`MouseTracking.AnyEvents`，原生文本复制依赖终端提供的`Shift+拖拽`鼠标报告绕过；不要在Mosaic或下游控件中另行模拟文本选择。
- Mosaic提供`LayoutCoordinates`和`Modifier.onPlaced`，用于报告节点最终的终端表面坐标；它们是通用布局原语，不承载焦点策略。
- Mosaic的文本测量、`TextSurface`占位、节点布局和渲染必须统一使用Unicode终端cell宽度；鼠标命中与`TerminalCursor`只能消费这套坐标，不能用code point数量近似。
- Mosaic从已放置的布局树收集焦点目标、焦点作用域和目标键盘路径；只有当前焦点路径接收按键，未消费的Tab和方向键由runtime完成默认遍历。
- `Modifier.focusable`、`focusGroup`和`focusTrap`只声明焦点语义；目标和作用域身份由稳定的`MosaicNode`持有，不能依赖`Modifier.composed`中的slot位置。
- `Modifier.focusRequester`只用于必要的程序化聚焦；普通点击、Tab、方向键、模态进入和退出均由Mosaic自动处理。
- `Modifier.focusCursor`将控件内的终端cell坐标绑定到焦点目标；Mosaic只显示当前焦点目标提供的物理光标位置。
- 只有要提供任意可组合内容和可变高度项的通用`LazyColumn`时，Mosaic才需要新增子组合、项目测量缓存和虚拟布局能力；这不是下游扩展可以正确补出的接口。

## 下游组件边界

- `cli:action`只实现语义动作与快捷键作用域；焦点Owner、目标注册、遍历和物理光标不在该模块重复实现。
- `Pressable`、`Button`、`Toggle`、`TextInput`、`Modal`、`Menu`、`List`和`Tabs`通过Mosaic焦点、布局、绘制和指针modifier组合实现。
- `ScrollableState`、`ScrollState`、`ScrollInteractionSource`、滚动modifier和滚动条属于Codex Lite；它们只使用Mosaic提供的输入、布局和指针能力。
- 通用`LazyColumn`属于Codex Lite组件层，但依赖Mosaic提供的measure-time subcomposition、viewport clipping和beyond-bounds focus协议。
- conversation history只是`LazyColumn`的调用方；follow-tail、unread、展开、history prefetch和Agent/Session切换状态不得进入通用滚动组件。
- `LazyListState.layoutInfo`是调用方的viewport demand signal；history调用方在composition之外异步加载，`LazyColumn`的composition与measure不得执行storage I/O或语义投影。

## 焦点与键盘

- Mosaic runtime维护唯一`FocusOwner`；每次布局后从节点树重建焦点投影，但焦点目标和作用域身份跟随稳定布局节点。
- 焦点目标增删无关modifier时必须保持身份；目标离开布局后，焦点恢复到活动作用域记住的目标、自动聚焦目标或首个合法目标。
- `FocusScope`定义方向导航边界；`Tab`与`Shift+Tab`在当前活动范围内循环。`focusTrap`激活时限制焦点，嵌套或同级覆盖层关闭后恢复进入前的精确目标。
- 键盘分发顺序固定为：焦点路径preview、当前目标、焦点路径bubble、未消费的Tab或方向键默认遍历。
- 快捷键属于语义动作而不是视觉按钮；`TuiActionScope`在父级preview阶段分发动作，按钮、菜单等控件只复用同一个动作入口。
- 动作是否可用由动作自身的`enabled`状态决定，不由响应式布局是否暂时隐藏某个按钮决定。
- 同一动作scope内的重复快捷键必须被拒绝；内层scope可以有意覆盖外层动作。
- 模态`TuiActionScope`使用`blockParentActions`阻断父scope快捷键查找，但未匹配的按键仍要交给当前焦点目标，不能吞掉文本输入。
- 未修饰方向键先交给当前焦点目标；未消费时由Mosaic按已放置的终端边界在当前`FocusScope`内选择同方向最近目标。`Page`键和空格仍由焦点目标自行决定。
- `FocusRequester`是程序化焦点逃生口，不用于实现普通组件注册、菜单初始焦点或对话框焦点恢复。
- Mosaic的焦点Owner是唯一的物理终端光标出口；每个可聚焦控件通过`focusCursor`提供局部光标锚点。
- `TextInput`维护动态编辑光标；按钮、列表等非文本控件使用固定锚点，使终端光标与当前焦点一致。
- 鼠标按下可将焦点移到命中的可聚焦控件；滚轮和非交互区域不改变焦点。
- 鼠标悬停只更新控件视觉状态，不改变焦点或物理终端光标位置。

## 基础组件

- `Modifier.focusable`：声明组件可聚焦；稳定目标身份、点击聚焦和默认遍历由Mosaic提供。
- `Pressable`：统一Enter、Space与主鼠标点击的按压、取消和激活语义，并消费Mosaic提供的hover状态。
- `Button`：基于`Pressable`的命令控件；方括号是固定边界，焦点不改变文本，悬停使用Bold，按下使用终端反色视频，禁用使用Dim。
- `Popup`：由`TuiPopupHost`提供同一终端表面的覆盖层；触发器通过`Modifier.tuiPopupAnchor`报告最终边界，位置策略根据锚点、宿主和已测量内容计算坐标，调用方不手写`onPlaced`或偏移。
- `TuiDialog`：复用`TuiPopupHost`居中覆盖终端表面；内容声明`focusTrap`，Mosaic负责进入首个控件、限制焦点和关闭后恢复原焦点，未处理的背景指针事件被拦截，未被预览处理的Escape关闭对话框。
- `TuiDialog`只提供模态行为，不隐式决定业务表面样式；settings对话框使用不透明的满宽背景，并将标题、Codex home、换行键和操作栏绘制为无内边距的连续色块。
- `PopupMenu`：作为`TuiPopup`内容，由可聚焦按钮处理方向键、Enter和Esc；有子项时右方向键展开子菜单、左方向键返回父菜单，父项通过锚点定位子菜单；可见项数按宿主可用高度裁剪，空间允许时完整显示；菜单外主键点击由最外层弹层的关闭回调处理。
- `Toggle`：基于`Pressable`的二元状态控件；`Switch`、`Checkbox`只是在视觉上不同的`Toggle`。
- `TextInput`：维护文本、Unicode标量光标、选择和粘贴；获得焦点时负责终端物理光标。
- `ScrollableState`：以终端行接收delta并返回实际消费量；边界零消费不得吞掉输入。
- `ScrollState`：维护普通eager容器的绝对行位置、最大值和viewport大小。
- `LazyListState`：维护首个可见item index、item内行偏移、layout info和request型定位，不包含follow-tail语义。
- `LazyColumn`：通过stable key、`contentType`、subcomposition slot reuse和可变高度测量，只组合可见、overscan及必要的beyond-bounds item。
- `LazyColumn`的item provider与key index按稳定structure identity记忆；无关重组不得重新枚举整个数据window，slot reuse容量保持有界并按滚动基准调优。
- `Menu`、`List`和`Tabs`：使用`focusable`、焦点作用域和`Pressable`组合，不各自实现焦点协议。

## 对话视图

- `ModeKind`通过当前模式按钮和显式上拉菜单变更；`Ctrl+P`打开同一菜单，不隐式切换模式。
- 模型与推理强度通过一个模型配置选择器变更，显示`[<model> <reasoning>]`并按模型级联可用推理强度；不要求用户输入`/model`或`/reasoning`命令。
- service tier通过独立的状态栏选择器变更；候选项为`default`和当前模型目录明确声明的已知 tier，切换模型后不支持的已选 tier 回退为`default`。
- 会话名称存储在`CodexAgentSettings.threadName`中：首条文本用户消息初始化它，图片-only输入保持空名称，显式更新和fork保留对应设置快照。
- 长历史直接复用AgentSession的stored-index cache与raw-value LRU，按稳定upper bound循环`prevIndex/get`填充有限semantic window；不得增加第二套raw page/index cache，也不得先全量投影history再只懒组合可见item，具体遵守[CLI ViewModel状态与懒History](cli-view-model-state.md)。
- 每个Mosaic frontend按Agent storage id维护独立`HistoryUiState`；滚动位置、follow-tail、unread和展开状态不进入共享Agent ViewModel。
- History调用方用`snapshotFlow`观察viewport接近已加载前边界的事件并去重预取；只用`derivedStateOf`收敛near-boundary等阈值状态，不让每个scroll offset触发业务重组。
- 用户离开尾部后由stable key保持阅读位置；流式新增内容不移动视口，只更新未读状态。
- follow-tail时由业务层请求滚到末尾；宽度变化后由LazyColumn按同一key和item内行偏移重新归一化。
- History list只订阅有限`HistoryWindowSnapshot`；每个row接收immutable entry，同generation的普通更新复用未变entry实例。
- History row按`Agent address + window generation + semantic primary stored index/provider id`建立Composable identity并提供语义`contentType`。
- Revert允许整窗generation失效；frontend只请求当前viewport、overscan与最小semantic boundary所需的记录，不实现suffix级Compose失效协议。
- `HistoryUiState`按generation-scoped entry key持有独立展开state；切换一个item只更新该state，generation变化时清理旧state。
- 对话renderer只格式化可见与overscan item，并按entry、宽度和该item展开状态`remember`纯展示结果；通用LazyColumn不得依赖conversation、session、message或transcript模型。
