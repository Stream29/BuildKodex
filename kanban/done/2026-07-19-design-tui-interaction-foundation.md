# Task Tree

- [done] 定义TUI交互基础的组件边界
  - [done] 区分Mosaic终端输入基础设施与Kodex控件语义
  - [done] 定义键盘焦点、模态焦点与物理终端光标的职责
  - [done] 定义鼠标点击、滚轮和键盘操作的统一行为
  - [done] 列出首批需要实现的基础组件与产品组件

# Details

Mosaic只提供Compose Runtime、自有布局和终端事件解析；它没有控件、焦点、鼠标分发或惰性列表。
本任务只记录组件与交互边界，不实施对应组件。
