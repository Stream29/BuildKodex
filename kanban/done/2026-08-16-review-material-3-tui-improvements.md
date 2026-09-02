# Task Tree

- [done] 落地基于 Material 3 审查形成的 TUI 改进
  - [done] 记录审查范围、依据和现状
  - [done] 汇总用户审批意见和实施边界
  - [done] 保持现有 Settings compact 行为
  - [done] 让状态栏保留首行 Settings 并换行其余控件
    - [done] 实现保持控件完整边界的尾端固定换行布局
    - [done] 按实际状态栏行数分配 History 和 Composer
    - [done] 覆盖常规和窄终端渲染测试
  - [done] 统一 hover、press 和 selected 视觉状态
    - [done] 让 Button 和 PopupMenu 组合基础状态样式
    - [done] 让自定义 Pressable 遵守相同状态语义
    - [done] 将已有选择控件迁移到 selected 反显
  - [done] 统一对话框操作排列和危险操作默认焦点
    - [done] 新增尾端对齐的共享操作行
    - [done] 重排 dismissive 和 confirming 操作
    - [done] 为危险操作设置 Cancel 默认焦点和 error 颜色
  - [done] 为模态窗口增加背景弱化语义
    - [done] 使用只修改 TextStyle 的全屏 Dim 覆盖层
    - [done] 验证背景字符和弹窗内容保持不变
  - [done] 为会话标签增加横向滚动和翻页语义
    - [done] 新增通用横向滚动布局
    - [done] 让 PageUp 和 PageDown 按横向视口翻页
    - [done] 让当前标签在选择变化后自动进入视口
  - [done] 建立保留透明 History 的语义主题系统
    - [done] 按 Compose CompositionLocal 模式提供颜色和文字角色
    - [done] 将现有硬编码视图颜色迁移到语义角色
    - [done] 保持主 History background 为 Unspecified
  - [done] 将收起侧栏改为顶部书签形态
    - [done] 收起时不再保留整列布局宽度
    - [done] 展开时继续占用完整侧栏列
  - [done] 为非跟随 History 增加回到底部按钮
    - [done] 在 Composer 分隔线中央覆盖小型向下按钮
    - [done] 点击后恢复 follow-latest 并定位最新内容
  - [done] 更新对应 TUI 设计决策
  - [done] 运行格式化、测试和终端尺寸验证

# Details

- 状态：`done`。
- 用户审批的改进已全部落地。
- 审查包含当前 release 在 `100×30`、`60×20` 终端中的实测，以及相关 TUI 源码核对。
- Material 3 仅作为层级、状态、自适应、模态和语义样式的参考，不机械照搬 dp、圆角、阴影、触摸尺寸或动画。
- 参考准则：[Interaction states](https://m3.material.io/foundations/interaction/states/overview)、[Buttons](https://m3.material.io/components/buttons/guidelines)、[Dialogs](https://m3.material.io/components/dialogs/guidelines)、[Adaptive layouts](https://m3.material.io/foundations/layout/canonical-examples/overview)、[Tabs](https://m3.material.io/components/tabs/guidelines)、[Color roles](https://m3.material.io/styles/color/roles)、[Typography](https://m3.material.io/styles/typography/applying-type)。
- 主题实现参考 Compose 的 `@Immutable` 系统值、`staticCompositionLocalOf`、主题提供函数和主题访问对象；不引入 Android Material 依赖。
- 已确定的实施规则：
  - Focus 只使用终端物理光标。
  - Hover 使用 Bold。
  - Press 使用 Invert + Bold。
  - Selected 使用 Invert；与 Hover 组合为 Invert + Bold。
  - Settings 固定在状态栏首行尾端，其余完整控件按需换到后续行。
  - Modal scrim 只修改已有 cell 的 TextStyle，不用空格覆盖背景。
  - 主 History 不设置背景色，避免破坏终端原生复制语义。
  - 收起侧栏作为 History 左上方的小型覆盖书签；只有展开时占用整列。
  - 非 follow-latest 时在 Composer 分隔线中央显示 `[↓]`。
- 验证结果：
  - `app-view-components`、`history`、`patch`、`settings`、`path-picker`、`application` 的 `linuxX64Test` 全部通过。
  - Mosaic `DrawTextStyleOverlayTest` 通过。
  - `app-cli:linkReleaseExecutableLinuxX64` 通过；release 在 `100×30` 和 `60×20` 终端中完成实测。
  - BuildKodex、Kodex 和 Mosaic 的 `git diff --check` 全部通过。

## 1. 窄终端状态栏会裁出半个控件

- 审查建议优先级：`P0`。
- `Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/RuntimeStatusBar.kt:72-109` 顺序放置全部运行控件。
- `Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/RuntimeStatusBar.kt:133-190` 只缩短 cwd，没有为其他控件分配优先级或 overflow。
- 在 `60×20` 终端中，最右侧 Settings 实际被裁成 `[S`。
- 候选方案：
  - 按优先级预算每个控件的宽度。
  - 始终完整保留运行控制和 Settings。
  - 中窄宽度将 question mode、agent mode、cwd、Compact 等收进 `[More]`。
  - 增加 `40`、`60`、`80`、`120` 列快照测试，禁止输出不完整按钮。

### 用户审批

[settings]强制留在第一行，其他的按钮允许溢出到下一行，不必削减按钮

## 2. 键盘焦点过度依赖终端物理光标

- 审查建议优先级：`P1`。
- `Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TuiPressable.kt:65-70` 已跟踪焦点并放置终端光标。
- `Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TuiButton.kt:43-54` 没有消费焦点状态，只区分 hover、pressed、disabled 和 idle。
- 物理光标可能受终端样式影响；Bold 同时承担 hover 和部分 selected 语义。
- 候选方案：
  - Focus 使用终端光标加 Underline 或高对比背景。
  - Hover 保留 Bold，Pressed 保留 Invert。
  - Selected 使用独立活动指示器。
  - 不增加会改变控件宽度的前后缀。
- 该方案会改变 `checklist/tui-interaction-components.md` 中“焦点不改变按钮文字样式”的现有决定；获批后需要同步更新该决策。

### 用户审批

基本规则：
Focus=光标
hover=bold
press=invert+bold
selected=invert
这种情况下自然产生：
hover+selected=invert+bold
hover+press=invert+bold
这个重复可以接受

## 3. 对话框操作顺序不统一

- 审查建议优先级：`P1`。
- 删除对话框按 `[Delete] [Cancel]` 排列，且没有显式安全默认焦点：`Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt:516-534`。
- 回退对话框按 `[Revert] [Cancel]` 排列，但显式聚焦 Cancel：`Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt:649-678`。
- Settings 的单个 Close 位于操作栏左侧：`Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt:206-212`。
- 候选方案：
  - 抽取尾端对齐的共享 `DialogActionRow`。
  - 从左到右排列 dismissive action 和 confirming action。
  - 使用 `[Cancel] [Delete]`、`[Cancel] [Save]` 等一致顺序。
  - 危险操作默认聚焦 Cancel，并为确认动作使用 error 语义。

### 用户审批

合理，照做。

## 4. Settings 缺少 compact 单栏布局

- 审查建议优先级：`P1`。
- `Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt:140-180` 将导航固定为最多 `18` 列，剩余宽度全部交给内容区。
- 极窄终端中内容区可能只剩一列。
- 现有标题固定、操作栏固定和内容独立滚动应保留。
- 候选方案：
  - Compact 使用全屏单栏和顶部页面选择器。
  - Medium 仅在满足内容最小宽度时显示侧栏。
  - Expanded 保留当前最大 `84` 列的居中窗口。
- 该问题与 [现有 Settings 布局任务](2026-08-10-adjust-settings-layout.md) 重叠；本任务按用户意见保持现有 compact 行为，不修改该任务。

### 用户审批

这个行为不必更改，因为过小终端不存在合理的显示方案。

## 5. 模态窗口没有弱化背景

- 审查建议优先级：`P2`。
- `Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TuiPopup.kt:257-276` 已提供全屏指针屏障、focus trap 和内容遮盖，但没有视觉 scrim。
- 实测中，对话框外的标签栏和状态栏保持原亮度，与模态内容竞争注意力。
- 候选方案：
  - Modal 打开时将宿主内容设为 Dim。
  - 或使用 neutral surface 重绘对话框外区域。
  - 单色终端同时依靠 Dim、边界和标题表达层级。

### 用户审批

接受，这将会带来更统一的popup语义。

## 6. 会话标签溢出时会静默消失

- 审查建议优先级：`P2`。
- `Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTabBar.kt:129-145` 只返回能够放入当前宽度的标签。
- 标签栏不显示隐藏数量，也不提供直接切换隐藏标签的入口。
- 当前标签仅使用背景和 Bold 表达 active 状态：`Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTabBar.kt:108-126`。
- 候选方案：
  - 始终保留当前标签及相邻标签。
  - 增加 `[⋯ +N]` 或左右滚动入口。
  - 为当前标签增加独立 Underline 或底部活动指示器。

### 用户审批

允许横向滚动，且增加一整套横向滚动语义：PgUp/PgDn提供类似的翻页功能。

## 7. 颜色和文字样式缺少语义 token

- 审查建议优先级：`P2`。
- 应用与 Settings 重复定义具体 RGB：`Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeUiPrimitives.kt:91-103`、`Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt:1079-1089`。
- Settings 使用多条相邻背景色区分字段，层级较依赖颜色条带。
- 候选方案：
  - 建立 `surface`、`surfaceContainer`、`onSurface`、`primary`、`error`、`outline` 等 TUI 语义颜色。
  - 建立 title、section、body、label、supporting 等有限文字角色。
  - 为 True Color、ANSI 256、ANSI 16 和高对比环境提供降级方案。
  - 状态同时使用文字或符号表达，不只依赖颜色。
  - 使用标题、间距和分隔线减少逐字段背景条带。

### 用户审批

这个需要我们建立一整套主题配色系统，要好好对照compose的实现。
不过这里有个核心限制：主history展示区必须没有背景色，保持透明。不然会逼迫mosaic渲染一大堆空格填充颜色，导致控制台的复制语义坏掉。
为了保护这个复制语义。我计划让左侧栏变成类似书签的格式，只占据上面几行，而不是占据一整列。只有展开才占据一整列。

我还要加一个：现有的主输入框分隔线的中间，我想加一个scroll to bottom的按钮，就覆盖显示一个很小的下箭头就可以了，只在非跟随最新的情况下显示。
