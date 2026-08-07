# Task Tree

- [done] 完善 root Session 运行状态展示
  - [done] 固定产品语义与任务边界
    - [done] Session running 只读取对应 root Agent 的 `running`
    - [done] `n sessions` 只统计已打开的真实 Session 标签
    - [done] running 时才插入 spinner，不为空闲标签预留宽度
    - [done] 顶部 Session 标签与 Agent 侧栏都使用 spinner
    - [done] spinner 直接连接名称并使用标准 Compose Animation API
  - [done] 建立已打开 root Session 的窄展示摘要
    - [done] 从每个真实标签的稳定 root Agent ViewModel 投影标题与 `running`
    - [done] 只在标题或 root `running` 变化时更新对应展示摘要
    - [done] 让溢出而未显示的真实标签仍参与 terminal title 计数
    - [done] 保持 New Session 草稿和已关闭历史不参与计数
  - [done] 统一运行指示器动画
    - [done] 使用经典十帧 Braille spinner 与 100 ms 间隔
    - [done] 使用 Mosaic 提供的 Compose InfiniteTransition animation
    - [done] 仅在存在可动画的 root 或当前树 Agent 时组合无限动画
    - [done] 让标签和侧栏共享同一帧，避免同屏动画相位不同
    - [done] 将 spinner 直接插入运行中名称的开头
    - [done] 保持空闲名称原样并删除两处后缀 `*`
  - [done] 管理 terminal title
    - [done] 生成固定格式 `n sessions (m running)`
    - [done] 仅在两个计数变化时写入 OSC 0 title
    - [done] 在 CLI 退出路径清空受管理标题
    - [done] 不尝试读取或恢复进入 CLI 前的标题
  - [done] 补充针对性验证
    - [done] 覆盖真实标签、新建标签、溢出标签和 root-only running 计数
    - [done] 覆盖标签前缀、侧栏前缀、长名称截断和无空闲占位
    - [done] 覆盖 spinner 帧顺序、100 ms 推进与无活动时移除动画
    - [done] 覆盖 title 格式、去重写入和退出清空
    - [done] 补齐 CLI application 已使用的 Mosaic animation 依赖
    - [done] 运行 CLI application Linux 测试、IDE 检查与终端 PTY 验证

# Details

- 状态：已完成。
- IntelliJ IDEA 2026.2 正在打开本项目。
- `Kodex/` 中已有与本任务无关的用户改动；实现不得修改或还原它们。

## 已确认语义

- 顶部标签的 running 沿用现有 root-only 语义。
- 子 Agent 单独运行时，其侧栏行显示 spinner，但所属 Session 不计入 `m running`。
- `n sessions` 统计全部已打开的 persisted Session 标签，包括因宽度不足而溢出的标签。
- `New Session` 草稿、未打开的持久历史和已关闭标签不计入 `n sessions`。
- 只有 running 名称使用 `"<frame><name>"`；空闲名称保持 `"<name>"`。
- 运行切换造成的一列宽度变化是已接受行为。
- 标题始终使用复数格式，包括 `0 sessions (0 running)` 和 `1 sessions (1 running)`。

## 实现边界

- 当前顶部后缀位于 [`SessionTabBar.kt:86-107`](../../Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTabBar.kt#L86-L107)。
- 当前侧栏后缀位于 [`SessionAgentSidebar.kt:103-117`](../../Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionAgentSidebar.kt#L103-L117)。
- 在 CLI application 内建立只含 `sessionName` 和 root `running` 的 tab render state。
- 从 `RootSessionEntry.viewModel.state.value` 解析稳定 root Agent ViewModel，再对其 state 投影并去重。
- `SessionTabBar.kt` 定义该 render state 和 composable 投影，`SessionTreeCliScreen.kt` 消费结果。
- 不向 `RootSessionViewState`、`SessionTreeCliState` 或 shared Session state 增加 running 副本。
- 不修改 Agent runtime、Session repository、存储模型或 multi-agent 语义。
- `SessionTreeCliScreen.kt` 组合 root 摘要、running 计数和共享 spinner state，再分别传给标签栏与侧栏。
- spinner state 以稳定 `State<String>` 下传；顶层 screen 不读取每一帧，避免整个 screen 每 100 ms 重组。
- 标签最大宽度保持 20 列；running 前缀占用其中一列，先插入前缀再按 terminal cell 宽度截断。
- 侧栏保持现有单 Agent `running` 判断、树展开和截断规则。

## Spinner

- 帧固定为 `⠋ ⠙ ⠹ ⠸ ⠼ ⠴ ⠦ ⠧ ⠇ ⠏`。
- 帧间隔固定为 100 ms。
- 使用 `rememberInfiniteTransition` 和 `InfiniteTransition.animateValue`。
- Int 动画使用 `Int.VectorConverter`，从 `0` 线性推进到帧数量。
- animation spec 使用 `infiniteRepeatable(tween(durationMillis = 1000, easing = LinearEasing))` 和 `RepeatMode.Restart`。
- 通过动画值对帧数量取模选择 glyph，保证 duration scale 为零时仍回到首帧。
- 没有已打开 running root 且当前 Agent 树没有 running Agent 时，不组合 `InfiniteTransition`。
- 动画重新进入 composition 时从 `⠋` 开始。
- 所有帧必须验证为一个 terminal cell。
- 不实现独立 `LaunchedEffect`、`delay` 或手写 ticker。
- Mosaic frame clock 可以高频驱动动画，但离散 Int state 只约每 100 ms 改变一次。

## Terminal title

- 当前 Mosaic `Terminal` 没有 title 写入 API；本任务不扩展 Mosaic fork。
- 在交互式 CLI composition 中通过 stdout 写入 `ESC ] 0 ; <title> BEL`。
- title payload 只含本地生成的数字和固定文案，不接收 Session 名称或其他不可信文本。
- 写入 effect 以 `(sessionCount, runningCount)` 为 key，spinner 推进不会触发 title 写入。
- Mosaic 的 Ctrl-C 取消路径不会 dispose 根 composition；因此由 `Main.kt` 的 `finally` 写入空 OSC 0 title。
- tmux、screen 和终端自身的 title 策略保持外部配置责任。

## 预计文件

- 修改 `Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt`。
- 修改 `Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt`。
- 修改 `Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTabBar.kt`。
- 修改 `Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionAgentSidebar.kt`。
- 新增 `Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/RunningIndicator.kt`。
- 新增 `Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/TerminalTitle.kt`。
- 修改 `Kodex/app/cli/application/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SessionTabBarTest.kt`。
- 修改 `Kodex/app/cli/application/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SessionAgentSidebarTest.kt`。
- 新增 `RunningIndicatorTest.kt` 和 `TerminalTitleTest.kt`。
- 在 `Kodex/app/cli/application/build.gradle.kts:46-71` 声明 `libs.mosaic.animation`，同时支持现有 `animateIntAsState` 和 spinner。

## 验证边界

- 测试使用现有 in-memory Session 或纯函数，不新增 mocking framework。
- Mosaic frame-clock 测试验证 spinner 至少推进两帧，并验证 inactive 后移除无限动画。
- 记录型输出函数验证 OSC 内容、重复计数不重复写入和显式退出清空。
- `:app-cli-application:linuxX64Test` 是本任务的主要 Gradle 验证。
- 适用时运行 `:app-cli-application:jvmTest`；既有 Mosaic JDK 22 绑定问题不在本任务修复范围。
- 使用 IntelliJ 检查修改文件，并运行 `git diff --check`。
- 在实际终端验证顶部标签、展开侧栏、窗口 title、Session 切换和退出清空。

## 成本

- 预计实现与针对性验证合计 4–6 小时。
- 不包含修复既有 Mosaic JDK 22 JVM 绑定问题的时间。
- 不需要数据迁移、runtime 改造或 Mosaic API 扩展。

## 实现结果

- [`SessionTabBar.kt:25-108`](../../Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTabBar.kt#L25-L108) 建立 root-only 标签摘要；不可见的已打开标签仍参与计数。
- [`RunningIndicator.kt:16-55`](../../Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/RunningIndicator.kt#L16-L55) 使用 Mosaic Compose InfiniteTransition 提供共享十帧 spinner。
- [`SessionTabBar.kt:154-179`](../../Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTabBar.kt#L154-L179) 和 [`SessionAgentSidebar.kt:113-132`](../../Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionAgentSidebar.kt#L113-L132) 将 spinner 无空格前置。
- [`TerminalTitle.kt:7-49`](../../Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/TerminalTitle.kt#L7-L49) 管理计数 title；[`Main.kt:43-54`](../../Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L43-L54) 在退出路径清空 title。
- Mosaic 退出生命周期结论已记录在 [`shared-context/findings/cli-terminal-title-lifecycle.md`](../../shared-context/findings/cli-terminal-title-lifecycle.md)。

## 验证结果

- `:app-cli-application:linuxX64Test`：通过。
- `:app-cli-application:linkReleaseExecutableLinuxX64`：通过。
- Linux PTY smoke：观察到 `0 sessions (0 running)`、New Session 画面、Ctrl-C 返回码 0 和一次空 OSC 0 清理。
- IntelliJ 修改文件检查：无新增问题；`SessionTreeCliScreen.kt` 仅保留既有警告。
- `git diff --check`：通过。
- `:app-cli-application:jvmTest`：被既有 `Mosaic:mosaic-tty:compileJdk22KotlinJvm` 的 `Libmosaic` 未解析问题阻断；未进入本任务测试执行。
