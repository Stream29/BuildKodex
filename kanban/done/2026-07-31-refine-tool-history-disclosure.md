# Task Tree

- [done] 调整 CLI 工具 history 的语义摘要与分层折叠
  - [done] 让 MCP 与未知工具在收起时显示原始工具名称
  - [done] 将 apply_patch 摘要改为已编辑文件数
  - [done] 为工具详情增加默认收起的二级折叠
  - [done] 更新快照与展示模型测试
  - [done] 运行相关验证

# Details

- 用户确认：过久的 unified-exec 历史失去原始命令上下文可以接受。
- 用户要求：没有可靠语义的 MCP/未知工具直接显示原始名称；`apply_patch` 显示 `Edited n files`；工具外层展开后，内部详情默认保持折叠。
- 验证：`./gradlew --quiet :cli-history:jvmTest :cli-patch:jvmTest`（使用项目指定 JDK，并跳过 Mosaic JDK 22 编译目标）通过。
