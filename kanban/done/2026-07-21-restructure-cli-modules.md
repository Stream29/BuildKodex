# Task Tree

- [done] 重组CLI模块目录
  - [done] 将顶层`TUI`容器改为`cli`
  - [done] 将现有CLI入口模块落到`cli/app`
  - [done] 将`global-settings`改为`cli/settings`
  - [done] 更新Gradle include、依赖、包名和文档引用
  - [done] 验证Linux与macOS构建测试

# Details

最终目录以`cli`为顶层容器，应用入口和设置不再作为demo或顶级孤立模块存在。

Linux X64与macOS ARM64上的`cli-settings`、`cli-action`、`cli-components`和`cli-app`测试均已通过；IDE检查未发现迁移错误。
