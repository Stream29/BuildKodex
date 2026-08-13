# Task Tree

- [done] 移除 cwd 路径前缀
  - [done] 修改展开状态的按钮标签
  - [done] 更新对应视图测试
  - [done] 运行相关 Native 测试

# Details

- 宽度足够时 cwd 按钮只显示路径，不显示 `cwd: ` 前缀。
- 窄布局继续使用 `cwd` 占位标签。
- 已使用 JDK 26 通过 `RuntimeStatusBarTest` 的 Linux Native 测试。
