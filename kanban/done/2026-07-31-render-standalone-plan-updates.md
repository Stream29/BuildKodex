# Task Tree

- [done] 将 CLI 的 `update_plan` 改为独立展示
  - [done] 拆出专用计划更新 renderer
  - [done] 直接展示说明与步骤状态
  - [done] 覆盖已完成与进行中的渲染快照
  - [done] 运行 history 模块的相关测试

# Details

- `update_plan` 不再复用通用工具事件的折叠式 `Arguments` 展示。
- 专用 renderer 直接显示计划标题、可选说明和状态清单。
- 已验证：`./gradlew -Dorg.gradle.java.home=/home/stream/.jdks/openjdk-26.0.2 :cli-history:jvmTest`。
