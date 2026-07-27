# Task Tree

- [done] 将CLI重构为MVVM
  - [done] 将会话管理与流式投影整理为Model边界
  - [done] 建立纯common的ViewModel状态、事件和生命周期
  - [done] 将composer、设置草稿、菜单与滚动状态迁入ViewModel
  - [done] 将Mosaic View收敛为状态渲染和事件转发
  - [done] 拆分并迁移Model、ViewModel与View测试
  - [done] 验证未来多个前端可订阅同一后端状态的边界

# Details

Mosaic组合层只负责渲染状态和发送用户事件，不再作为会话状态事实来源。ViewModel使用热
`StateFlow`发布完整UI快照，并以同步状态归约和顺序命令队列分别处理UI状态与挂起副作用。

Linux `linuxX64Test`、macOS `macosArm64Test`和IDE增量构建均通过。
