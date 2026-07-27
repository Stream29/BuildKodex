# Task Tree

- [done] 将Session控制移至带色顶栏
  - [done] 完成规划
    - [done] 盘点Session按钮、底部状态区、popup anchor与现有配色
    - [done] 固定顶栏、按钮和Session菜单的布局与颜色
    - [done] 明确与Session surface及PopupMenu任务的边界
  - [done] 建立Session顶栏
    - [done] 在界面首行增加全宽单行`SessionTopBar`
    - [done] 将Session标题按钮左对齐放入顶栏
    - [done] 为顶栏保留一行并相应缩减history viewport
    - [done] 对长标题按terminal cell宽度截断，不换行或越出终端
  - [done] 迁移Session交互
    - [done] 将Session按钮、popup anchor和inline rename从底部状态区移到顶栏
    - [done] 保持无Session时新建入口及有Session时现有按钮标签语义
    - [done] 保证按钮标题来自root Session，选择child Agent不改变顶栏标题
    - [done] 让rename输入框在顶栏原位替换按钮并由Enter提交
    - [done] 让Session菜单从顶栏按钮下方向下覆盖history
    - [done] 保持New、Fork、Rename、Sessions和Settings行为不变
  - [done] 收敛底部状态区
    - [done] 删除底部重复的Session按钮与标题
    - [done] 将token count保留在底部状态区
    - [done] 保持model、reasoning、tier、Stop和mode的布局与行为
  - [done] 应用Session表面配色
    - [done] 顶栏使用`Color(28, 68, 74)`
    - [done] Session按钮及rename输入面使用`Color(36, 78, 84)`
    - [done] Session菜单使用`Color(42, 42, 46)`和白色前景
    - [done] 使用通用PopupMenu背景能力覆盖菜单完整矩形
  - [done] 完成验证
    - [done] 覆盖有Session、无Session、running、child选择、长标题和inline rename布局
    - [done] 覆盖顶栏按钮点击、菜单向下展开、选择及dismiss
    - [done] 验证顶栏、按钮和菜单背景覆盖各自完整区域
    - [done] 验证窄窗口及terminal resize后的history与底部状态布局
    - [done] 验证底部不再重复显示Session标题且token count仍然可见
    - [done] 运行CLI components与CLI app相关JVM/Linux测试及Linux Native可执行文件链接
    - [done] 删除临时文件并更新任务状态，不创建Git commit

# Details

- 状态：已完成。Linux CLI app的52项测试和CLI components测试通过，Linux Native可执行文件链接成功；JVM测试仍被Mosaic既有的JDK 22 FFM生成绑定缺失阻塞。
- IntelliJ IDEA 2026.2当前正在打开本项目。
- 本任务是[Session surface重构](2026-07-22-redesign-session-surface.md)中Session专用UI的视觉与布局后续，不推进repository、懒加载、runtime或multi-agent工作。
- 本任务以现有Session菜单、rename和browser行为为基线，只迁移标题控件的位置并增加表面配色。

## 布局与交互决策

- 顶栏固定为一行并位于history之前；composer、separator及底部状态区的相对顺序不变。
- Session标题按钮左对齐；按钮以外的剩余cell全部使用顶栏背景色。
- active标签继续显示`s<index>`、running `*`和单行root Session标题；无选中Session时按照后续新建会话状态设计显示`[New session]`。
- token count继续属于底部状态信息，不进入Session按钮或顶栏。
- 窄终端优先保留可操作的Session按钮边界，并按terminal cell宽度截断标题；顶栏不得换行。
- inline rename在同一标题槽位替换按钮并使用按钮背景色；提交、取消和焦点语义保持不变。
- 顶栏位于surface首行，现有`AboveStart`位置策略会在anchor上方无空间时将菜单放到下方；本任务不新增业务offset或新的position provider。
- Session菜单作为overlay覆盖history，不占用正常布局高度。

## 颜色决策

- 新增Session语义常量，不让新代码依赖`SettingsDialog*`常量名称，也不在本任务中引入全局主题抽象。
- `SessionTopBarBackground = Color(28, 68, 74)`。
- `SessionButtonBackground = Color(36, 78, 84)`。
- `SessionMenuBackground = Color(42, 42, 46)`。
- Session顶栏、按钮和菜单使用白色前景，保证现有暗色表面上的可读性。
- 顶栏背景覆盖完整终端宽度；按钮背景只覆盖按钮或rename输入面占据的cell。
- 菜单背景必须清除并着色完整的已测量菜单矩形，包括短item尾部与空白cell，不能只给item文字增加`Modifier.background`。

## PopupMenu边界

- 通用`backgroundColor`参数、完整矩形清除和组件级验证已由[PopupMenu改进任务](../done/2026-07-22-improve-tui-popup-menu.md)完成。
- 本任务只在Session菜单调用点传入`SessionMenuBackground`，不复制一套Session专用菜单容器或填充逻辑。
- PopupMenu任务迁移Session调用点时必须保留本任务确定的顶栏anchor、向下展开结果和Session菜单颜色。

## 当前实现位置

- [`MosaicView.kt:258`](../../CodexLite/cli/app/src/mosaicMain/kotlin/io/github/stream29/codex/lite/cli/app/MosaicView.kt#L258)在History之前组合`SessionTopBar`。
- [`MosaicView.kt:308`](../../CodexLite/cli/app/src/mosaicMain/kotlin/io/github/stream29/codex/lite/cli/app/MosaicView.kt#L308)声明Session popup并应用菜单背景色。
- [`MosaicView.kt:1381`](../../CodexLite/cli/app/src/mosaicMain/kotlin/io/github/stream29/codex/lite/cli/app/MosaicView.kt#L1381)从root Session摘要生成单行顶栏布局。
- [`MosaicView.kt:1406`](../../CodexLite/cli/app/src/mosaicMain/kotlin/io/github/stream29/codex/lite/cli/app/MosaicView.kt#L1406)渲染全宽顶栏、Session按钮与inline rename。
- [`MosaicView.kt:1472`](../../CodexLite/cli/app/src/mosaicMain/kotlin/io/github/stream29/codex/lite/cli/app/MosaicView.kt#L1472)定义不再包含Session控制的底部状态布局。
- [`MosaicView.kt:2049`](../../CodexLite/cli/app/src/mosaicMain/kotlin/io/github/stream29/codex/lite/cli/app/MosaicView.kt#L2049)定义Session表面配色。
- [`TuiButton.kt:20`](../../CodexLite/cli/components/src/mosaicMain/kotlin/io/github/stream29/codex/lite/cli/components/TuiButton.kt#L20)现有modifier已足以给Session按钮应用背景色。
- [`TuiPopupMenu.kt:275`](../../CodexLite/cli/components/src/mosaicMain/kotlin/io/github/stream29/codex/lite/cli/components/TuiPopupMenu.kt#L275)提供本任务复用的菜单背景色能力。

## 验收重点

- production composition与纯文本renderer都必须为顶栏扣除一行，最终输出不得超过terminal高度。
- empty、active、running、renaming和选择child Agent时，顶栏始终占一行且Session标题来源正确。
- ANSI snapshot必须证明顶栏背景覆盖整行、按钮背景覆盖按钮cell、菜单背景覆盖完整overlay矩形，history字符不会从菜单空白cell透出。
- Session菜单的keyboard、pointer、outside dismiss和关闭后焦点恢复语义不得因anchor迁移而变化。
- terminal resize后标题截断、history viewport、composer和底部状态区都保持合法。
