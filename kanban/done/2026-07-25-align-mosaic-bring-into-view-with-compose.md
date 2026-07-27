# Task Tree

- [done] 对齐 Mosaic bring-into-view 与 Compose 抽象
  - [done] 对照 AndroidX 源码确认最小 Modifier Node 基础
  - [done] 实现 BringIntoViewModifierNode 与祖先请求传播
  - [done] 将焦点获取接入挂起式 bring-into-view 请求
  - [done] 将 LazyColumn 响应实现迁移到节点契约
  - [done] 补齐生命周期、传播、取消和滚动回归测试
  - [done] 更新 ABI 并运行相关构建与测试

# Details

- 以 AndroidX Compose 当前实现为语义基准。
- 不保留现有同步 `BringIntoViewModifier` 作为第二套抽象。
- 仅引入 bring-into-view 链路必需的 Modifier Node 能力。
