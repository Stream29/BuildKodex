# Task Tree

- [done] 完整review现有代码变更
  - [done] 按Mosaic、CLI和Agent核心模块检查当前变更
  - [done] 修复能够明确判定的实现与建模问题
  - [done] 运行IDE检查、格式化和相关测试
  - [done] 将需要进一步决策的清理项拆成活跃任务

# Details

修复了物理光标显隐恢复、终端Unicode宽度双重实现、按钮拖拽捕获、菜单宽度、历史硬截断和设置弹窗残留菜单。Linux与macOS Native的Mosaic、CLI组件和CLI应用测试均通过。

组合回滚语义已在[`review-storage-revert-atomicity`](2026-07-21-review-storage-revert-atomicity.md)完成复核。Mosaic的cklib任务仍不兼容Gradle configuration cache；命令保持启用该选项，但Gradle在执行后丢弃缓存。
