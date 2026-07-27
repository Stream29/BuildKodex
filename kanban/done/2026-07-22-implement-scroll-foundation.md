# Task Tree

- [done] 实现通用滚动基础
  - [done] 定义整数行滚动方向和实际消费量语义
  - [done] 在`cli:components`实现`ScrollableState`
  - [done] 在`cli:components`实现`ScrollState`与`rememberScrollState`
  - [done] 定义独立的`ScrollInteractionSource`及输入来源
  - [done] 实现`Modifier.scrollable`
  - [done] 让现有fixed-row viewport复用通用滚轮输入
  - [done] 覆盖方向、边界、禁用、反向和interaction测试
  - [done] 运行相关格式化与多平台测试

# Details

- 状态：已完成。
- 滚动单位固定为终端行。
- state返回实际消费量；边界零消费不得吞掉输入或发出interaction。
- `Modifier.scrollable`只解释输入，不负责移动、测量或裁剪内容。
- 首版不实现像素滚动、fling、惯性、动画和部分delta的nested-scroll remainder。
- `:cli-components:jvmTest`、`:cli-components:linuxX64Test`和远端`:cli-components:macosArm64Test`通过。
- Node.js目标不承载`mosaicMain`源码，对应测试任务无可执行测试。
- IDEA检查与`git diff --check`通过；该模块没有独立Spotless任务。
