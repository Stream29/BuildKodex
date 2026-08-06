# Task Tree

- [done] 完善 Session 工具栏操作与选中态
  - [done] 接入用户触发的强制 compact 操作
  - [done] 统一 runtime 控制按钮样式
  - [done] 加粗当前 Session 标签
  - [done] 补充针对性验证

# Details

- 用户明确要求补充强制 compact 按钮、统一 Stop/Clear pending/Resume 的样式，并加粗当前 Session。
- IntelliJ IDEA 2026.2 正在打开本项目。
- 在共享 Agent ViewModel 中调用既有 `forcedCompact()`，由状态栏按 `canCompact` 控制按钮可用性。
- runtime 控制和 compact 按钮复用现有 Session 按钮颜色；通用按钮增加空闲文本样式参数，供选中标签使用 Bold。
- `app-shared-agent:linuxX64Test`、`app-cli-components:linuxX64Test`、`app-cli-application:linuxX64Test` 通过。
- `app-shared-agent:jvmTest` 通过；其余 JVM 测试仍被 Mosaic 既有的 `Libmosaic` JDK 22 绑定缺失阻断。
- IDE inspection 未发现本次变更引入的问题；`git diff --check` 通过。
