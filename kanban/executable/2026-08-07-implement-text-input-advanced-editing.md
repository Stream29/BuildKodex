# Task Tree

- 实现第一阶段输入框高级编辑
  - 扩展编辑状态与选区不变量
    - 以锚点和活动光标表达单一连续选区
    - 让插入、删除、粘贴和重置遵守选区语义
  - 完成光标移动与键盘选取
    - 实现硬行上下移动和首尾移动
    - 实现`Shift`扩展选区和无修饰移动折叠选区
    - 实现编辑器词法分组的`Ctrl+W`
  - 实现有界撤销与重做历史
    - 记录并恢复文本、光标和选区快照
    - 合并连续输入和同向删除事务
    - 在分叉编辑和重置时清理对应历史
  - 扩展输入布局与源位置映射
    - 保留每个可见硬行的源范围和终端cell边界
    - 反显当前选区并保持物理光标位置
    - 将前缀、横向裁剪和宽字符坐标映射回源偏移
  - 接入鼠标点击与主键拖拽选取
    - 点击放置光标并建立拖拽锚点
    - 使用现有指针捕获持续更新活动端
  - 保持调用方状态同步边界
    - 防止共享草稿的回声更新清除本地选区和历史
    - 让真实外部替换清除本地瞬态编辑状态
  - 补齐决策说明与自动化验证
    - 明确输入框选区与终端原生屏幕选择的边界
    - 覆盖状态、布局、键盘、粘贴、鼠标和调用方同步
    - 验证相关JVM与本机Kotlin Native目标

# Details

本任务的实现边界、顺序和验收条件已经确定；后续仍需用户明确要求后才能开始实现。

## 范围

- `TextInputValue.cursorOffset`继续表示活动端，新增选区锚点；两者相等表示无选区。
- 选区仍使用当前UTF-16偏移和Unicode标量移动边界，不在本期改为字素簇模型。
- `Left`、`Right`按Unicode标量移动。
- `Up`、`Down`按硬行移动，并跨长短行保留期望的终端cell列。
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

## 鼠标与显示

- 主键点击输入框时把光标放到最近的合法源边界并清除选区。
- 主键拖拽以按下位置为锚点，以当前指针位置为活动端。
- 拖拽离开输入框后继续接收现有捕获事件，并把位置钳制到可映射边界。
- 行前缀不属于可编辑文本；点击前缀映射到该硬行开头。
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
- `reset`和真实外部草稿替换清空撤销、重做、选区及期望列。
- 历史最多保留100个事务，避免草稿长期编辑导致无界增长。

## 调用方边界

- 选区、期望列和撤销历史只属于frontend的`TextInputState`，不进入共享`ComposerViewModel`。
- 共享草稿仍只保存文本和活动光标。
- 当前输入产生的文本/光标更新回流时，不调用`reset`。
- 提交、切换草稿来源或其他真实外部替换仍通过`reset`建立新的编辑会话。
- 预计只修改Kodex组件和调用方，不需要扩展Mosaic输入或指针协议。

## 非目标

- 不新增`Ctrl+C`、`Ctrl+X`或主动处理`Ctrl+V`。
- 不新增原生剪贴板、OSC 52、tmux或WSL剪贴板支持。
- 不新增`Ctrl+A`、`Ctrl+Left`、`Ctrl+Right`或`Ctrl+Backspace`别名。
- 不新增双击选词、三击选行、`Shift+Click`或多选区。
- 不新增通用历史区域选择、软换行、IME、字素簇编辑或鼠标边缘自动滚动。
- 终端的`Shift+拖拽`继续用于原生屏幕复制，不进入输入框编辑选区。

## 主要文件

- `Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TextInputState.kt`
- `Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TextInput.kt`
- `Kodex/app/view/components/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/components/TextInputTest.kt`
- `Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt`
- `Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeUiPrimitives.kt`
- `Kodex/app/view/application/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/ComposerInputTest.kt`
- `checklist/tui-interaction-components.md`

## 验收

- 正向、反向和跨锚点的键盘选区行为一致。
- 上下移动在空行、短行、宽字符和多行文本中保持合法偏移与期望列。
- 首尾移动及其`Ctrl`、`Shift`组合符合已确定语义。
- 删除、输入、换行和粘贴都正确替换选区。
- `Ctrl+W`覆盖空白、Unicode词组、标点组和跨行边界。
- 撤销分组、重做、分叉编辑、容量上限和重置行为可重复验证。
- 鼠标点击、正反向拖拽、跨行拖拽、前缀和横向裁剪坐标均有组件测试。
- 选区反显和禁用样式有ANSI快照覆盖。
- Composer状态回声不清除选区或历史，真实外部替换会清理两者。
- 相关JVM和本机Kotlin Native测试通过，变更文件通过IDE检查。

## 估算

- 总成本约`6–9`人日。
- 最大风险集中在源偏移与终端cell坐标映射、宽字符拖拽以及外部草稿回声同步。
