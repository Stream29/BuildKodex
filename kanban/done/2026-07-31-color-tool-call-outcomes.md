# Task Tree

- [done] 将 CLI tool call 状态改为颜色表达
  - [done] 移除工具标题中的文字状态
  - [done] 将失败工具标题渲染为红色，其余渲染为白色
  - [done] 覆盖无状态文案与 ANSI 颜色渲染
  - [done] 运行 history 与 patch 模块的相关测试

# Details

- 通用 `ToolEvent`、流式 tool call 和独立 `apply_patch` 卡片均不再显示文字状态。
- 失败标题为红色，其他工具标题为白色；详情区域仍按其原有语义展示。
- 已验证：`./gradlew -Dorg.gradle.java.home=/home/stream/.jdks/openjdk-26.0.2 :cli-history:jvmTest :cli-patch:jvmTest`。
