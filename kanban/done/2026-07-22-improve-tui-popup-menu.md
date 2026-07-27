# Task Tree

- [done] 改进TUI PopupMenu
  - [done] 完成规划
    - [done] 审计Popup host、anchor、position provider、菜单状态和焦点实现
    - [done] 对照Compose的Popup、DropdownMenu和DropdownMenuItem责任边界
    - [done] 盘点Session、Mode、Tier、Model、Reasoning和Import六个生产调用点
    - [done] 明确保留TuiPopupHost、TuiPopup和TuiPopupPositionProvider
    - [done] 确定PopupMenu采用Compose风格的content scope API
    - [done] 确定支持可配置背景色并填充完整菜单矩形
  - [done] 重构PopupMenu公开契约
    - [done] 用`expanded`、`onDismissRequest`、`anchor`、`state`和`positionProvider`表达菜单生命周期与位置
    - [done] 用`TuiPopupMenuScope`和独立item composable替代`List<String>`、index callback及`itemModifier`
    - [done] 提供普通item、divider和submenu item
    - [done] 支持stable key、enabled、selected、leading content和trailing content
    - [done] 提供`backgroundColor: Color = Color.Unspecified`
    - [done] 保持低层TuiPopup可以承载任意内容，不向其中加入菜单业务语义
  - [done] 收敛菜单状态与布局
    - [done] 由长期存活的`TuiPopupMenuState`管理focused key、viewport和滚动位置
    - [done] 从业务ViewModel移除仅供菜单导航使用的active index镜像
    - [done] 按anchor周围实际可用cell空间限制菜单高度，不再由调用方传入屏幕行数
    - [done] 保持AboveStart优先向上、空间不足时向下及边界回退语义
    - [done] 支持可变高度、滚轮、溢出提示、终端resize、动态item和长标签
    - [done] 按terminal cell width对齐item内容和trailing content
  - [done] 统一输入、焦点和dismiss语义
    - [done] 打开菜单组时阻断父级shortcut和背景pointer输入
    - [done] 让每个菜单层级trap focus，hover不改变焦点
    - [done] 让Enter、Space和primary click汇合到同一item action
    - [done] 统一Escape、outside click和选择完成后的dismiss路径
    - [done] 让Right或Enter打开child，Left或Escape返回parent，outside click关闭整个菜单组
    - [done] 关闭菜单组后恢复到准确的trigger焦点
  - [done] 实现菜单背景
    - [done] 让`backgroundColor`作用于菜单容器而不是单个文本节点
    - [done] 在内容下方绘制带背景色的空白cell，覆盖测量后的完整菜单矩形
    - [done] 覆盖短item尾部、padding、divider、滚动或溢出提示以及submenu空白区域
    - [done] 防止popup下方原有字符从空白cell中透出
    - [done] 保证嵌套submenu分别使用各自配置的背景色
  - [done] 迁移现有调用点
    - [done] 迁移Session菜单
    - [done] 迁移Mode菜单
    - [done] 迁移Tier菜单
    - [done] 迁移Model菜单
    - [done] 迁移Reasoning submenu
    - [done] 迁移Session Import菜单
    - [done] 删除六处重复的index、visible-count和popup包装接线
  - [done] 完成验证
    - [done] 覆盖上下定位、边界回退、实际高度约束、resize和动态item
    - [done] 覆盖keyboard、mouse、wheel、disabled item、selected item和nested submenu
    - [done] 覆盖父级shortcut隔离、outside dismiss和trigger focus恢复
    - [done] 用终端surface或ANSI snapshot验证指定背景色覆盖完整矩形且没有字符透出
    - [done] 验证`Color.Unspecified`默认值及不同父背景下的行为
    - [done] 运行CLI components和CLI app相关测试、格式化、API检查与跨平台编译
    - [done] 删除临时文件并更新任务状态，不创建Git commit

# Details

## 当前状态

- 2026-07-22，实施和验证已完成。
- 这是独立的纯UI任务，不推进AgentStorage、AgentSession或multi-agent重构。
- IntelliJ IDEA 2026.2当前正在打开本项目。
- Popup组件与相关CLI app测试通过；完整CLI app测试首次运行时真实Codex会话用例瞬时失败，单独重跑该测试组通过。
- Linux X64测试、Linux ARM64编译和MinGW X64编译通过。
- IntelliJ检查未发现Popup组件问题，`git diff --check`通过。
- JVM编译仍被Mosaic既有的JDK 22 FFM生成绑定错误阻塞；Gradle configuration cache也因Mosaic任务不可序列化而被丢弃。

## 结论

- 当前`TuiPopupPositionProvider`以anchor bounds、surface size和content size计算cell offset，责任形状与Compose `PopupPositionProvider`一致，应当保留。
- 当前`TuiPopupMenu`把数据、选择、focus viewport和渲染压缩为`List<String>`与index callback，不适合disabled item、divider、selected marker、trailing content和submenu等组合式内容。
- 目标是对齐Compose的状态与content slot边界，同时保留terminal cell、Mosaic focus tree和上拉菜单的TUI特性。
- 业务层只持有“哪个菜单打开”和实际domain selection；composition持有focused key、viewport、滚动和submenu导航状态。

当前实现位置：

- [`TuiPopup.kt:81`](../../CodexLite/cli/components/src/mosaicMain/kotlin/io/github/stream29/codex/lite/cli/components/TuiPopup.kt#L81)定义position provider，低层popup从第175行开始。
- [`TuiPopupMenu.kt:58`](../../CodexLite/cli/components/src/mosaicMain/kotlin/io/github/stream29/codex/lite/cli/components/TuiPopupMenu.kt#L58)定义菜单状态，Compose风格菜单API从第275行开始。
- [`MosaicView.kt:299`](../../CodexLite/cli/app/src/mosaicMain/kotlin/io/github/stream29/codex/lite/cli/app/MosaicView.kt#L299)起包含Session、Mode、Tier、Model和Reasoning菜单。
- [`MosaicView.kt:1017`](../../CodexLite/cli/app/src/mosaicMain/kotlin/io/github/stream29/codex/lite/cli/app/MosaicView.kt#L1017)包含Session Import菜单。

## 目标契约

公开调用形状以此为准：

```kotlin
TuiPopupMenu(
    expanded = expanded,
    onDismissRequest = onDismissRequest,
    anchor = anchor,
    state = rememberTuiPopupMenuState(),
    positionProvider = TuiPopupPositionProvider.AboveStart,
    backgroundColor = Color.Unspecified,
) {
    TuiPopupMenuItem(
        key = item.id,
        enabled = item.enabled,
        selected = item.selected,
        trailingContent = { Text(item.shortcut) },
        onClick = item.onClick,
    ) {
        Text(item.label)
    }
    TuiPopupMenuDivider()
    TuiPopupSubmenuItem(/* child menu content */)
}
```

- `expanded`决定菜单是否进入overlay与focus tree；关闭统一经过`onDismissRequest`。
- `TuiPopupMenuState`跨重组保持focused stable key和viewport，不保存domain selection。
- item的stable key承担插入、删除和重排后的focus identity；key消失时选择相邻enabled item并归一化viewport。
- submenu属于同一个popup group，共享outside-dismiss边界，但每层有独立anchor、position和focus scope。
- 首版不复制Android像素动画、window inset或Material theme；只复用Compose的抽象边界。

## 背景色语义

- `backgroundColor`是菜单surface的显式参数，默认`Color.Unspecified`以保留调用方未指定颜色的用法。
- 仅使用`Modifier.background`不足以定义overlay的字符覆盖语义；菜单必须先以空格清除完整测量矩形，再在这些cell上应用背景色并绘制item内容。
- 指定颜色后，同一个菜单矩形内的文本cell与空白cell都必须呈现该颜色，item自行指定的背景可以覆盖对应cell。
- divider、padding、短标签补齐区域、viewport空白、overflow indicator和submenu边缘不得暴露下层字符。
- 背景填充不得扩大菜单的测量尺寸、命中区域或anchor positioning结果。

## 冻结交互约束

- Popup只参与overlay绘制与命中，不改变正常布局或history尺寸。
- Popup位置只依赖最终anchor bounds与host约束，业务调用方不手写offset。
- 菜单使用Mosaic focus tree，不另建第二套逻辑焦点；pointer hover不请求focus。
- 菜单打开时父页面快捷键不得穿透；滚轮只影响pointer命中的菜单层级。
- 可见item数量由实际可用cell高度决定，不保留固定七项上限。
- 关闭整个菜单组后恢复trigger焦点；关闭child只恢复parent item焦点。

## 验收重点

- anchor上方足够时向上展开；不足时选择下方；两侧都不足时在可用侧建立可滚动viewport。
- 动态插入、删除、禁用或重排item后，focused key和viewport保持合法且不越界。
- disabled item不能通过keyboard或pointer触发，导航会跳过它。
- child menu左右翻转时不越出host；Left、Escape和outside click符合分层dismiss语义。
- 指定背景色的ANSI输出对完整菜单矩形连续着色，原surface中的非空字符不会透过空白区域。
- 六个生产调用点不再计算`maximumVisibleItems`或镜像菜单active index。
