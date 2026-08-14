# Task Tree

- [done] 实现第一阶段输入框高级编辑
  - [done] 扩展编辑状态与选区不变量
    - [done] 以锚点和活动光标表达单一连续选区
    - [done] 让插入、删除、粘贴和重置遵守选区语义
  - [done] 完成光标移动与键盘选取
    - [done] 实现可视行上下移动和硬行首尾移动
    - [done] 实现`Shift`扩展选区和无修饰移动折叠选区
    - [done] 实现编辑器词法分组的`Ctrl+W`
  - [done] 实现有界撤销与重做历史
    - [done] 记录并恢复文本、光标和选区快照
    - [done] 合并连续输入和同向删除事务
    - [done] 在分叉编辑和重置时清理对应历史
  - [done] 扩展输入布局、软换行与源位置映射
    - [done] 按终端cell宽度拆分带源范围和cell边界的可视行
    - [done] 反显当前选区并保持物理光标位置
    - [done] 将前缀、软换行、视口偏移和宽字符坐标映射回源偏移
  - [done] 实现主Composer有界输入视口
    - [done] 按调用方可用行预算增长并以可见高度参与屏幕布局
    - [done] 溢出时保留本地滚动偏移并始终保持活动光标可见
    - [done] 支持命中输入视口的鼠标滚轮滚动
  - [done] 接入鼠标点击与主键拖拽选取
    - [done] 点击放置光标并建立拖拽锚点
    - [done] 使用现有指针捕获持续更新活动端
  - [done] 保持调用方状态同步边界
    - [done] 防止共享草稿的回声更新清除本地选区、历史和视口
    - [done] 让真实外部替换清除本地瞬态编辑状态
  - [done] 补齐决策说明与自动化验证
    - [done] 明确输入框选区与终端原生屏幕选择的边界
    - [done] 覆盖状态、换行、视口、键盘、粘贴、鼠标和调用方同步
    - [done] 验证相关JVM与本机Kotlin Native目标

# Details

本任务已按确定的实现边界完成。

## 实现结果

- `TextInputState`现在持有单一连续选区、可视列、最多100项的undo/redo历史和本地视口偏移。
- `TextInputLayout`支持按终端cell宽度soft wrap、字素簇安全源映射、选区反显和有界可见行。
- 主会话与新会话Composer按剩余屏幕行预算增长；溢出后内部滚动，编辑和移动会重新保持活动光标可见。
- 键盘选取、可视行移动、硬行首尾、`Ctrl+W`、undo/redo、鼠标点击、拖拽和滚轮已经接入。
- Composer草稿回声保留frontend-local瞬态状态，真实外部替换通过`reset`清理这些状态。
- 现有`ComposerInput`布局参数足以完成接入，未修改并行变更中的`SessionTreeUiPrimitives.kt`。

## 验证

- 相关`jvmTest`通过：`utils-terminal-text`、`app-view-components`、`app-view-application`。
- 相关`linuxX64Test`通过：`utils-terminal-text`、`app-view-components`、`app-view-application`。
- 变更文件通过IntelliJ检查和目标文件构建。

## 调研结论

- `Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TextInput.kt:20-69`目前只按显式换行生成硬行；活动硬行做横向裁剪，其他硬行直接截断，没有软换行或垂直视口。
- `Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt:59-68`和`:190-197`使用全部硬行数计算Composer高度；长草稿会持续挤压历史区，超过终端可用高度后还可能裁掉Composer、提示或状态区。
- `Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TerminalText.kt:6-20`已有按终端cell宽度换行的字符串工具，但没有保留源偏移，且极窄宽度下遇到无法容纳的首个字符会提前停止，不能直接承担编辑布局。
- `Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/VerticalScroll.kt:13-64`已有有界裁剪和滚轮滚动能力；编辑器仍需自行维护“活动光标始终可见”的滚动不变量。
- [OpenAI Codex TextArea](https://github.com/openai/codex/blob/main/codex-rs/tui/src/bottom_pane/textarea.rs)的实现可作为参照：换行缓存保存源范围，上下键按可视行移动，渲染时钳制滚动并最小调整视口以保持光标可见。

## 范围

- `TextInputValue.cursorOffset`继续表示活动端，新增选区锚点；两者相等表示无选区。
- 选区仍使用当前UTF-16偏移和Unicode标量移动边界，不在本期改为字素簇模型。
- `Left`、`Right`按Unicode标量移动。
- `Up`、`Down`按软换行后的可视行移动，并跨长短可视行保留期望的终端cell列。
- `Home`、`End`移动到当前硬行首尾。
- `Ctrl+Home`、`Ctrl+End`移动到整个草稿首尾。
- 上述移动增加`Shift`时保持锚点并移动活动端；活动端可以越过锚点。
- 无`Shift`移动清除选区；`Left`、`Right`面对非空选区时先折叠到对应边界。
- `Backspace`、`Delete`和`Ctrl+W`面对非空选区时优先删除完整选区。
- 普通字符、业务层换行和现有`PasteEvent`面对非空选区时替换完整选区。
- `Ctrl+W`先跨过光标前的空白，再删除前一个词法组：
  - Unicode字母、数字和下划线属于词组。
  - 连续标点或符号属于独立组。
  - 例如`foo/bar|`依次变为`foo/|`、`foo|`、`|`。

## 软换行与输入视口

- 软换行以可配置模式接入`TextInput`，本期只为主Composer的现有会话和新会话输入启用；其他调用方默认维持当前行为。
- 软换行只生成显示用可视行，不修改草稿，也不插入换行符。
- 第一条可视行使用`firstLinePrefix`；其余可视行，包括同一硬行的软换行续行，使用`continuationLinePrefix`。
- 每条可视行保留UTF-16源范围和字素簇安全的终端cell边界，供光标、选区、上下移动和鼠标命中共同使用。
- 换行宽度扣除对应前缀宽度；极窄终端或单个宽字素簇无法放入剩余宽度时仍须推进源位置，不得丢字或死循环。
- 启用软换行的主Composer不再横向裁剪草稿；完整内容通过可视行和垂直视口访问。
- 调用方先为分隔线、提示、请求面板和状态栏等固定区域保留行数，再把剩余的有界行预算交给Composer；Composer可见高度为总可视行数与预算的较小值，且有空间时至少显示一行。
- 主屏幕布局只扣除Composer的可见高度，而不是总可视行数；历史区使用余量，在极小终端中允许降为零。
- 垂直滚动偏移属于`TextInputState`的本地编辑状态。编辑、光标移动、撤销、重做、重置及终端宽高变化后重新换行、钳制偏移，并做保持活动光标可见所需的最小滚动。
- 鼠标滚轮命中Composer时只滚动其内部视口，不移动光标；下一次编辑或光标移动重新建立光标可见不变量。

## 鼠标与显示

- 主键点击输入框时把光标放到最近的合法源边界并清除选区。
- 主键拖拽以按下位置为锚点，以当前指针位置为活动端。
- 拖拽离开输入框后继续接收现有捕获事件，并把位置钳制到可映射边界。
- 点击和拖拽先用视口偏移定位可视行，再映射到源边界。
- 行前缀不属于可编辑文本；点击前缀映射到该可视行的源起点。
- 选区使用反显样式；禁用状态继续拒绝编辑和选取。
- 本期不增加拖拽到边缘时的自动横向或纵向滚动。

## 撤销与重做

- `Ctrl+Z`撤销，`Ctrl+Y`重做。
- 历史只记录文本变化，不把纯光标或选区移动作为独立历史项。
- 每个历史项同时保存变化前后的文本、光标和选区。
- 连续普通字符输入合并为一项。
- 连续`Backspace`和连续`Delete`各自合并，方向或操作类型变化会切断分组。
- 光标移动、选区变化、鼠标操作、粘贴、换行、`Ctrl+W`、撤销和重做会切断当前分组。
- 粘贴、换行、`Ctrl+W`和一次选区替换各自是原子历史项。
- 撤销后发生新文本编辑时清空重做栈。
- `reset`和真实外部草稿替换清空撤销、重做、选区、期望列及视口偏移。
- 历史最多保留100个事务，避免草稿长期编辑导致无界增长。

## 调用方边界

- 选区、期望列、撤销历史和视口滚动偏移只属于frontend的`TextInputState`，不进入共享`ComposerViewModel`。
- 共享草稿仍只保存文本和活动光标。
- 当前输入产生的文本/光标更新回流时，不调用`reset`，也不重置视口。
- 提交、切换草稿来源或其他真实外部替换仍通过`reset`建立新的编辑会话。
- 预计只修改Kodex组件和调用方，不需要扩展Mosaic输入或指针协议。

## 非目标

- 不新增`Ctrl+C`、`Ctrl+X`或主动处理`Ctrl+V`。
- 不新增原生剪贴板、OSC 52、tmux或WSL剪贴板支持。
- 不新增`Ctrl+A`、`Ctrl+Left`、`Ctrl+Right`或`Ctrl+Backspace`别名。
- 不新增双击选词、三击选行、`Shift+Click`或多选区。
- 不新增通用历史区域选择、IME、字素簇编辑或鼠标边缘自动滚动。
- 不新增可见滚动条或Composer内的`PageUp`、`PageDown`滚动快捷键。
- 终端的`Shift+拖拽`继续用于原生屏幕复制，不进入输入框编辑选区。

## 主要文件

- `Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TextInputState.kt`
- `Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TextInput.kt`
- `Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TerminalText.kt`
- `Kodex/app/view/components/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/components/TextInputTest.kt`
- `Kodex/app/view/components/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/components/TerminalTextTest.kt`
- `Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt`
- `Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeUiPrimitives.kt`
- `Kodex/app/view/application/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/ComposerInputTest.kt`
- `checklist/tui-interaction-components.md`

## 验收

- 正向、反向和跨锚点的键盘选区行为一致。
- 上下移动在空行、短可视行、宽字符、软换行和多硬行文本中保持合法偏移与期望列。
- 首尾移动及其`Ctrl`、`Shift`组合符合已确定语义。
- 删除、输入、换行和粘贴都正确替换选区。
- `Ctrl+W`覆盖空白、Unicode词组、标点组和跨行边界。
- 撤销分组、重做、分叉编辑、容量上限和重置行为可重复验证。
- 长硬行按终端cell宽度软换行；宽字符、组合字符、连续空格、空硬行和极窄宽度均不丢失源内容。
- Composer随内容增长到调用方预算后改为内部滚动，历史、分隔线、提示、请求面板和状态栏不被总文本行数挤出布局。
- 编辑、移动、撤销、重做、重置和终端缩放后活动光标均可见，滚动偏移始终合法。
- 鼠标点击、正反向拖拽、跨软换行拖拽、前缀、滚动视口和宽字符坐标均有组件测试。
- 鼠标滚轮只在命中Composer时滚动其视口，并能访问光标之外的可视行。
- 选区反显和禁用样式有ANSI快照覆盖。
- Composer状态回声不清除选区、历史或视口，真实外部替换会清理本地瞬态状态。
- 相关JVM和本机Kotlin Native测试通过，变更文件通过IDE检查。

## 估算

- 总成本约`8–12`人日。
- 最大风险集中在软换行重排后的源偏移与终端cell映射、视口和光标同步、宽字符拖拽以及外部草稿回声同步。
