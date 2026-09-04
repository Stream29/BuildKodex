# TUI交互组件

实现Kodex TUI时遵守以下决策。

## 分层

- Mosaic fork负责通用的终端输入、指针路由、布局树焦点、键盘遍历和物理光标；不包含Kodex业务动作或业务控件。
- Kodex TUI负责控件状态、组件内键盘语义、业务快捷键映射和对话历史虚拟化，不维护第二套焦点树或动作路由树。
- 每个可触发动作必须由同一个语义操作同时服务键盘和鼠标，不能为两种输入维护分叉的业务逻辑。

## Mosaic修改边界

- Mosaic必须在`runMosaic`层提供显式鼠标追踪配置，并由TTY实现负责启用和恢复SGR坐标与按键拖拽模式；下游不能安全地自行写终端控制序列。
- Mosaic必须提供公开的`PointerEvent`、`Modifier.onPreviewPointerEvent`和`Modifier.onPointerEvent`入口，并在内部按节点边界和绘制层级完成命中测试、捕获与冒泡；下游不应直接消费`Terminal.events`，该channel只有一个正确消费者。
- Mosaic TTY终端事件channel使用无损的`UNLIMITED`队列以完整保序；不得以`DROP_OLDEST`丢弃IME整段提交、快速输入或其他已解析事件。
- Mosaic TTY必须解析`CSI number;modifier:event-type ~`功能键的修饰符和事件类型；Menu/Application键同时支持canonical `57363`与legacy `CSI 29~`，不能在终端解析层丢失。
- Mosaic runtime忽略Kitty `57441..57454`独立修饰键功能事件，并将`57358`大写锁定功能键投影为非文本键名，不能将这些私用区键码投影为文本；实际按键继续携带修饰状态，关联文本仍优先作为文本。
- Mosaic TTY必须启用并在退出时恢复 bracketed paste（DEC 2004）；完整粘贴以单个`PasteEvent`交给runtime，不能把其中的字节拆成普通按键事件。
- `PasteEvent`沿当前焦点路径按preview、目标、bubble分发，且不触发未消费按键的默认焦点遍历；文本输入以一次编辑原子插入内容并规范化换行。
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
- Mosaic的`SubcomposeLayoutState`提供可取消的precompose和premeasure handle；premeasure返回缓存后的实际尺寸，正式measure按相同key接管slot并复用预测量结果，detach或取消时释放未接管slot。
- Mosaic的同步`Remeasurement`先重测请求节点并复用同constraints的clean descendants；目标尺寸不变时按原位置重新placement，尺寸变化时回退完整root layout。普通帧布局在测量前保守失效整棵树，不能跨composition变化复用陈旧几何。

## 下游组件边界

- `Pressable`、`Button`、`Checkbox`、`TextInput`、`Modal`、`Menu`、`List`和`Tabs`通过Mosaic焦点、布局、绘制和指针modifier组合实现；当前二元设置只实现Checkbox，不预建Toggle/Switch抽象。
- `ScrollableState`、`ScrollState`、`ScrollInteractionSource`、横纵滚动modifier和滚动条属于Kodex；它们只使用Mosaic提供的输入、布局和指针能力。
- Kodex主题通过`@Immutable`语义颜色/文字值、`staticCompositionLocalOf`、主题提供函数和主题访问对象组合；具体RGB只出现在默认主题中，业务视图消费角色而非具体颜色。
- 通用`LazyColumn`属于Kodex组件层，但依赖Mosaic提供的measure-time subcomposition、viewport clipping和beyond-bounds focus协议。
- Conversation history只是`LazyColumn`的调用方；follow-latest、展开、history demand和Agent/Session切换状态不得进入通用滚动组件。
- `LazyColumn`只访问当前viewport、overscan与beyond-bounds item；History item access可以注册异步旧端需求，但composition与measure不得执行storage I/O或语义投影。
- `LazyColumn`的已测量窗口是按终端行计算的缓存，不是数据边界；滚动跨出缓存时累积pending delta并同步remeasure，只有provider真实首尾可以返回部分或零消费。
- 非边界位置的滚动消费行数不得受item拆分粒度影响；当前几何足够时走快速路径，跨缓存时由measure返回真实消费量。
- `LazyColumn`在当前输入、布局和绘制工作之后，沿最近视觉滚动方向逐项precompose并premeasure缓存外内容；累计实际高度达到两个viewport时停止。正式measure复用这些结果；方向、provider或生命周期变化时取消陈旧预取，预测量不改变滚动消费量。
- History旧端和新端的demand marker参与`LazyColumn` overscan；marker被测量后异步补充结构chunk，因此正常滚动前会形成viewport派生的有界窗口，不在滚轮处理或measure中同步读取storage。

## 焦点与键盘

- Mosaic runtime维护唯一`FocusOwner`；每次布局后从节点树重建焦点投影，但焦点目标和作用域身份跟随稳定布局节点。
- 焦点目标增删无关modifier时必须保持身份；目标离开布局后，焦点恢复到活动作用域记住的目标、自动聚焦目标或首个合法目标。
- `FocusScope`定义方向导航边界；`Tab`与`Shift+Tab`在当前活动范围内循环。`focusTrap`激活时限制焦点，嵌套或同级覆盖层关闭后恢复进入前的精确目标。
- 键盘分发顺序固定为：焦点路径preview、当前目标、焦点路径bubble、未消费的Tab或方向键默认遍历。
- Dialog和每层PopupMenu在`focusTrap`外层preview路径直接处理无修饰Escape；其他按键继续交给当前焦点路径，即使弹层没有可聚焦子项也不需要独立动作宿主。
- 业务快捷键在拥有对应状态的screen边界显式映射到与按钮相同的回调；应用级处理必须在Dialog或PopupMenu打开时显式停用，不建立通用动作注册表或快捷键scope。
- 未修饰方向键先交给当前焦点目标；未消费时由Mosaic按已放置的终端边界在当前`FocusScope`内选择同方向最近目标。`Page`键和空格仍由焦点目标自行决定。
- `FocusRequester`是程序化焦点逃生口，不用于实现普通组件注册、菜单初始焦点或对话框焦点恢复。
- Mosaic的焦点Owner是唯一的物理终端光标出口；每个可聚焦控件通过`focusCursor`提供局部光标锚点。
- `TextInput`维护动态编辑光标；按钮、列表等非文本控件使用固定锚点，使终端光标与当前焦点一致。
- 鼠标按下可将焦点移到命中的可聚焦控件，但 pointer focus 不发起 bring-into-view 或移动viewport；滚轮和非交互区域不改变焦点。
- 鼠标悬停只更新控件视觉状态，不改变焦点或物理终端光标位置。

## 基础组件

- `Modifier.focusable`：声明组件可聚焦；稳定目标身份、点击聚焦和默认遍历由Mosaic提供。
- `Pressable`：统一Enter、Space与主鼠标点击的按压、取消和激活语义，并消费Mosaic提供的hover状态；secondary callback在鼠标触发时接收控件局部释放坐标，在`Shift+F10`或Menu/Application键触发时接收`null`。
- `Button`：基于`Pressable`的命令控件；方括号是固定边界，焦点只使用物理终端光标。默认交互样式下悬停使用Bold，按下使用Invert+Bold，选中使用Invert，选中且悬停使用Invert+Bold，禁用使用Dim；持有明确container/on-color语义配对的调用方改用保色交互样式，以Bold表达hover、press和selected，不得通过Invert交换配对颜色。调用方可指定启用且空闲时的文本样式，交互状态样式仍优先。
- `Popup`：由`TuiPopupHost`提供同一终端表面的覆盖层；触发器通过`Modifier.tuiPopupAnchor`报告最终边界，位置策略根据锚点、宿主和已测量内容计算坐标，调用方不手写`onPlaced`或偏移。
- `TuiDialog`：复用`TuiPopupHost`居中覆盖终端表面；内容声明`focusTrap`，Mosaic负责进入首个控件、限制焦点和关闭后恢复原焦点，未处理的背景指针事件被拦截。组件自身在preview阶段处理无修饰Escape并调用独立回调；该回调默认关闭对话框，业务弹窗可先退出局部模式，弹窗外点击仍调用普通dismiss。
- Modal打开时通过Mosaic的TextStyle覆盖原语只为背景已有cell追加Dim，不绘制空格、不改变背景字符；弹窗内容随后正常覆盖该效果。
- `TuiDialog`在调用方绘制modifier和内容之前以单宽空格清除覆盖范围，使背后的双宽字符先拆成独立cell，再应用连续的弹窗背景；不要把该清屏顺序下沉为Mosaic全局宽字符语义。
- 对话框操作使用尾端对齐的共享操作行，dismissive action位于confirming action之前；危险确认使用error角色，Cancel作为默认焦点。Path Picker是目录浏览器例外：`Select`位于列表上方并默认聚焦，底部只保留`Cancel`。
- `TuiDialog`只提供模态行为，不隐式决定业务表面样式；settings对话框使用不透明的满宽背景，并将标题、Codex home、换行键和操作栏绘制为无内边距的连续色块。
- 设置表单中的互斥值选择只显示一个`TuiDropdownTrigger`，候选项和selected反显只出现在弹出菜单中；Settings字段与MCP Transport遵守同一规则，OAuth和其他true/false设置使用共享`TuiCheckbox`，Settings页面导航、Session标签和Session sidebar仍使用导航组件。`request_user_input`的Ask User回答选项是例外：逐项显示单选按钮和说明，`Other`继续切换到自由文本输入。
- Settings保留现有布局和极窄终端行为，不新增compact单栏变体。左侧页面固定按General、OpenAI、MCP、Hooks、Current session、New session排列；General承载Codex home、Sidebars和Input，OpenAI承载认证来源、账号状态和Codex usage，MCP与Hooks各自承载管理内容，Title generation位于New session的Model behavior之后。持续显示的选择字段将标题与下拉触发器放在同一行，并统一使用中性surface字段背景；章节标题使用`surfaceContainerHigh/onSurface`与title样式，普通条目使用surface背景，支持文案使用`onSurfaceVariant`，弹出菜单使用`surfaceContainer`，操作栏使用中性色块。Settings按钮统一使用保色交互：普通文字操作使用中性表面上的primary，主要确认使用primary/onPrimary，最终危险确认使用error/onError，内容按钮使用surface/onSurface，选中导航使用secondaryContainer/onSecondaryContainer，禁用按钮回退中性色。Codex home和Working directory将标题与Browse放在同一条目行并复用Path Picker；MCP servers与Hooks的管理按钮紧随各自标题，且管理标题行使用区别于条目内容的surface角色。OpenAI账号操作按来源纵向分项，窄终端不得把重新登录、Reload和退出压入同一尾端行；Title model和Title reasoning在自动标题关闭时保持禁用并解释依赖关系。
- `PopupMenu`：作为`TuiPopup`内容，由可聚焦按钮处理方向键和Enter，每层菜单在preview阶段处理无修饰Escape；根菜单关闭，子菜单返回父菜单。有子项时右方向键展开子菜单、左方向键返回父菜单，父项通过锚点定位子菜单；可见项数按宿主可用高度裁剪，空间允许时完整显示；菜单外主键点击由最外层弹层的关闭回调处理。
- `ContextMenu`：复用`PopupMenu`，默认按触发器局部的secondary释放坐标定位并限制在host内；`null`键盘坐标回退到触发器起点。普通`PopupMenu`、下拉菜单和子菜单继续使用各自的触发器相对位置。
- `Checkbox`：基于`Pressable`的二元状态控件，`[ ]`/`[x]`标记与label属于同一个可聚焦、可点击表面；Enter、Space和主鼠标点击切换值，checked不额外使用selected反显。当前不预建Toggle/Switch抽象。
- `TextInputState`：由组件层持有草稿、Unicode标量活动光标、可视行期望列、最多100项的undo/redo事务和输入视口偏移；这些状态不进入业务ViewModel。
- `TextInput`：不维护应用内选区；文本选择完全交给终端，用户通过`Shift+Drag`建立终端原生选区。组件统一处理可视行上下移动、硬行首尾移动、`Ctrl+W`、undo/redo、粘贴和鼠标点击放置光标；带`Shift`的移动键和指针按下不消费，无修饰`Up`/`Down`只在对应方向仍有相邻可视行时消费，位于首行或末行时交还Mosaic执行方向焦点遍历；获得焦点时负责终端物理光标，业务层只映射换行、提交等领域快捷键。
- `TextInputLayout`：默认保留单硬行横向裁剪；启用soft wrap时按字素簇安全的终端cell宽度生成带UTF-16源范围的可视行，并让光标、键盘移动和鼠标命中共用同一映射。
- 主Composer为现有会话和新会话启用soft wrap；输入高度按屏幕可用行数增长，溢出后使用frontend-local纵向视口，滚轮只滚动该视口，下一次编辑或光标移动重新保持活动光标可见。
- `ScrollableState`：沿一个逻辑轴以终端cell接收delta并返回实际消费量；边界零消费不得吞掉输入。
- `ScrollState`：维护普通eager容器的绝对cell位置、最大值和viewport大小。
- `horizontalScroll`：按终端列裁剪和放置eager内容；原生横向滚轮滚动列，并兼容纵向滚轮；纵向容器不消费原生横向滚轮，边界零消费继续冒泡；未修饰`PageUp`/`PageDown`按一整个横向viewport翻页。
- `LazyListState`：维护首个可见item index、item内行偏移、layout info和request型定位，不包含follow-tail语义。
- History的scroll-to-latest按钮在分隔线上以supporting样式静置，hover时使用统一按钮规则叠加Bold。
- `LazyColumn`：通过stable key、`contentType`、subcomposition slot reuse和可变高度测量，只组合可见、overscan及必要的beyond-bounds item。
- `LazyColumn`的item provider使用interval content与anchor附近的key-index map；无关重组不得枚举完整`itemCount`，slot reuse容量保持有界并按滚动基准调优。
- `EllipsizedText`：只渲染单个hard line，从有限布局约束读取实际可用宽度，并按Unicode终端cell安全添加省略号；容器宽度变化后必须重新测量，调用方不能传入全局终端宽度或固定列数代替布局宽度。
- `Menu`、`List`和`Tabs`：使用`focusable`、焦点作用域和`Pressable`组合，不各自实现焦点协议。

## 对话视图

- 当前Session只显示一个root Agent，不提供AgentMode、Multi-agent开关或Agent Tree切换。
- 顶栏当前Session标签使用selected的Invert状态；溢出标签通过横向滚动保持可达，未修饰`PageUp`/`PageDown`翻动一整个标签viewport，选择变化后自动将当前标签带入viewport。
- Agent与New Session状态栏将Settings固定在首行尾端；其他完整控件按顺序换到后续行，History和Composer按实际状态栏行数分配高度。
- Agent与New Session状态栏显示响应式cwd按钮：真实Session只更新当前root Agent，虚拟New Session只更新当前标签草稿；Agent运行期间仍可更新cwd并影响后续请求，窄表面退化为`cwd`标签，目录选择继续复用独立path picker。
- Agent运行期间模型配置、`ask user`/`no question`提问模式、cwd与Settings继续可用；Agent与New Session状态栏都使用独立提问模式下拉按钮，Session设置显示当前值，New session设置管理后续草稿默认值。只有不能并发执行的Compact隐藏，Stop继续作为主要运行控制。
- 运行状态栏不显示Fork；分叉入口属于已提交history条目的上下文菜单，不与状态栏动作重复。
- 模型、推理强度与service tier通过一个模型配置选择器原子变更；菜单按model、reasoning、当前模型目录允许的tier形成三级结构。
- 模型配置按钮显示`[<model> <reasoning>]`，仅在tier非`default`时追加` <tier>`；不保留独立tier按钮。
- 会话名称存储在`KodexAgentSettings.threadName`中：未显式命名的root以`Session <index>`初始化，首条文本用户消息可触发自动标题替换，图片-only输入保留默认名称，显式更新和fork保留对应设置快照。
- Session命名弹窗由输入框的普通Enter直接提交，不显示额外的Rename/Save操作按钮；Escape仍通过通用`TuiDialog`关闭。
- 长历史直接复用AgentSession的full stored-index cache与容量1,024的raw-value LRU；只物化有限tail及用户实际访问的旧端记录，不增加第二套raw page/index cache，具体遵守[CLI ViewModel状态与懒History](cli-view-model-state.md)。
- 当前单一Mosaic frontend直接使用对应`AgentHistoryViewModel`持有的`LazyListState`、follow-latest和展开状态；切换Session或Agent时复用该稳定状态。
- 主题的`background`角色必须保持`Color.Unspecified`；主History不应用surface背景、不绘制背景空格，保留终端原生复制语义。
- History frontend不观察viewport手动分页；`peek(index)`提供无副作用key/content type，item content通过`get(index)`向ViewModel注册合并后的旧端需求。
- Follow-tail是`AgentHistoryViewModel`持有的显式用户意图；只有pointer滚轮或paging向旧历史实际消费行数时关闭，零消费不关闭，用户或被动布局到达最新边缘时恢复。
- 布局位置只能恢复、不能关闭follow-tail；focus relocation和programmatic定位不改变意图，follow-tail开启时由History调用方纠正尺寸、换行和item高度变化造成的偏移。
- 用户离开尾部后由stable child key保持阅读位置；流式新增内容不移动视口。
- 用户离开尾部时，Composer分隔线中央以`[↓]`覆盖显示回到底部操作；点击后直接调用`AgentHistoryViewModel`恢复follow-latest并请求最新位置，到达最新位置后隐藏。
- History的未修饰`PageUp`/`PageDown`每次滚动半个可见窗口；滚动完成后，`PageUp`聚焦窗口顶部完全可见的已提交条目，`PageDown`聚焦窗口底部完全可见的已提交条目。
- 每个已提交history条目的完整多行区域是一个可聚焦的secondary-action surface；不显示hover或focus背景；pending、streaming、loading、failure和empty marker不提供条目菜单。
- 已提交history条目通过右键、`Shift+F10`或Menu/Application键打开host级`Revert to here`/`Fork from here`菜单；仅当所选Agent没有active turn job且处于稳定状态时允许弹出，Session、Agent、generation、target或anchor失效后立即关闭；Fork命令调用所属`PersistedSessionViewModel.fork(exactAgent, target)`。
- History list只订阅`HistoryViewModel`分别发布的committed window、pending tools与streaming item；committed count、key与item访问捕获同一个window快照，每个committed row接收稳定`HistoryItemViewModel` child，同generation的普通更新复用未变child实例。
- History row直接使用稳定`HistoryItemViewModel`实例作为Composable key并提供语义`contentType`，不增加单独的identity wrapper。
- Revert允许整窗generation失效；frontend不实现suffix级Compose失效或独立history cache。
- 每个`HistoryItemViewModel`持有自己的展开state；切换一个item只更新该child，child被移除或generation变化时清理对应state。
- Committed row异步读取raw event；完成前显示一行空白，失败时显示一行红色`Error`并记录完整异常。通用LazyColumn不得依赖conversation、session、message或transcript模型。
- 工具调用的折叠行使用实际动作的语义摘要；对 MCP 或没有可靠语义摘要的通用工具，直接显示原始工具名称。
- 工具调用标题将展开符、语义前缀和摘要作为完整单行交给`EllipsizedText`，按History容器的实际布局宽度统一裁剪；摘要生成只规范化换行，不能提前按固定列数截断。消息和详情继续按实际宽度换行。
- 工具调用的外层展开后先显示原始工具名称；参数、结果、输出等 payload 分组仍默认折叠。`apply_patch` 成功时用 `Edited n files` 汇总实际编辑文件数，未完成或失败时用 `Editing n files` 表示尝试中的编辑。
- Unified Exec history只有在当前 client 中能观察到匹配 session 且其尚未完成时才显示 `running`；查不到 session 的已提交命令显示 `finished`，不得由缺失状态推断进程仍在运行。
- `write_stdin` history只在live session可观察时用原始command改善展示，不把该command复制到stable history；session移除后按持久化的session ID回退展示。
- 收起侧栏不保留整列宽度，只在History对应外侧覆盖单行展开按钮；左右侧栏分别持有全局内容和首选宽度，默认均为28列。侧栏宽度在零列和当前首选宽度之间双向动画；悬停临时展开，点击固定展开，按钮与面板之间的动画不打断悬停语义。
- 展开侧栏的内侧一列是无字符splitter；空闲时分别延续同一行原有的侧栏标题或正文背景，不形成常驻分隔线。Hover使用`onSurface`在`surfaceContainer`上的8%状态层，按下和拖动统一使用16%状态层；普通主键按下由Mosaic捕获后持续拖动，`Shift+Drag`不消费并保留终端原生文本选择。拖动只更新内存布局并至少保留一列主内容，释放时才将最终宽度持久化。
- Terminal session行按终端cell宽度换行并保留原始换行，通过右键、`Shift+F10`或Menu/Application键打开host级ContextMenu；菜单使用与侧栏区分开的不透明背景。`Close session`调用同一`UnifiedExecProcessSession.close()`终止进程，菜单打开期间对应面板不得收起。
- 当某个 Agent 的 `ToolPending` 恰好只有一个 `request_user_input` 调用时，在 history 与 composer 之间显示该 Agent 私有的选择/自由文本表单；切换到`no question`不得移除已有表单或答案草稿，提交仍先以 `completeToolCall` 写入答案，再恢复同一 runtime。
