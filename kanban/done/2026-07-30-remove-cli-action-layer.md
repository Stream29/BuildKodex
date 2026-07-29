# Task Tree

- [done] 移除 CLI action 抽象层
  - [done] 将 Dialog 与 PopupMenu 的 Escape 路由内收到组件
  - [done] 删除生产代码中的 Action host、scope 和按钮重载
  - [done] 删除 `cli/action` 模块及 Gradle 依赖
  - [done] 调整组件测试并保留弹层键盘语义覆盖
  - [done] 更新 TUI 交互组件决策
  - [done] 运行定向编译、测试和依赖残留检查

# Details

- 用户明确要求执行迁移。
- 保留现有未提交改动，不创建 Git commit。
- `cli-components` 与 `cli-app` 的 Linux x64 测试通过。
- Mosaic metadata 与 CLI Linux x64 生产代码编译通过。
- JVM 测试受 Mosaic 既有 `Libmosaic` 生成绑定缺失阻塞。
- 项目没有适用的格式化任务；`git diff --check` 与残留引用检查通过。
