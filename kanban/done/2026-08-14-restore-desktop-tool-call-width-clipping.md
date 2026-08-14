# Task Tree

- [done] 恢复 Desktop tool call 行宽裁剪
  - [done] 对照 TUI 明确单行裁剪语义
  - [done] 梳理 Desktop tool call 实际布局约束
  - [done] 实现宽度感知的单行省略
  - [done] 补充宽度变化回归测试
  - [done] 运行 Desktop 历史回归验证

# Details

- 用户指出 Desktop tool call 不再按可用行宽裁剪。
- TUI 的折叠 tool call summary 使用实际可用宽度进行单行省略；Desktop 应保持相同的响应式语义。
- Desktop history 在有限宽度的 `LazyColumn` 内渲染；修复应直接消费标题自身的布局约束，不使用窗口宽度或固定字符数。
- 仅 tool call 标题保持单行；普通可展开历史标题和详情继续按实际宽度换行。
- 计划将单行省略设为 tool call 标题的显式渲染策略，并用 `TextLayoutResult` 覆盖宽窄容器切换。
- 修改范围限定为 Desktop history renderer 与其 Desktop UI test。
- Tool call 标题现在关闭软换行并按自身有限宽度单行省略；其他可展开历史标题恢复普通换行。
- 新增回归测试覆盖宽容器完整显示、窄容器溢出，以及运行时宽度变化后的重新布局。
- IDEA 定向构建通过，修改文件无 error 级 inspection。
- `:app-view-history:jvmTest` 与 `:app-desktop:test` 通过，`git diff --check` 通过。
