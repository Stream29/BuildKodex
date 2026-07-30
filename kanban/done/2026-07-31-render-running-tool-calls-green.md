# Task Tree

- [done] 将运行中的 CLI tool call 标题渲染为绿色
  - [done] 为通用和流式工具卡片识别运行状态
  - [done] 为 pending `apply_patch` 卡片保留运行状态
  - [done] 覆盖绿色运行状态及既有白色/红色状态
  - [done] 运行 history 与 patch 模块的相关测试

# Details

- 工具标题继续不显示状态文字。
- `running`、`streaming`、`starting` 和本地 shell 的 in-progress 状态显示为绿色；完成为白色，失败为红色。
- 已验证：`./gradlew -Dorg.gradle.java.home=/home/stream/.jdks/openjdk-26.0.2 :cli-history:jvmTest :cli-patch:jvmTest`。
