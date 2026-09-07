# Task Tree

- [done] 清理已确认的 Python 和 Java 文件
  - [done] 确认删除范围与 Kotlin 替代方案
  - [done] 删除 FUSE 实验目录和两个离线迁移脚本
  - [done] 将 MCP stdio fixture 转为 Kotlin 并保留测试覆盖
  - [done] 更新 FUSE 历史共享记录并检查残留引用
  - [done] 独立编译并验证 Kotlin fixture 子进程
  - [done] 按用户确认跳过完整 MCP stdio JVM 测试并归档

# Details

- 用户已确认整个 `Kodex/experiments/sqlite-session-fuse/`、两个 `Kodex/scripts/migrate-*.py` 的删除，以及 Java fixture 的 Kotlin 转换。
- 路线已确定，按上述文件范围实施，删除实验目录配套文件并在共享记录注明移除。
- 修改计划完整，进入执行；验证包含差异检查、残留引用检索、IDE 检查及可用环境下的测试。
- 不修改嵌套子模块、真实 Home、Git 历史或既有发布任务，不创建提交。
- 保留 fixture 协议行为，子进程 classpath 加入 Kotlin 标准库；原计划运行 `:mcp-stdio:jvmTest`，最终按用户确认跳过。
- IDEA 已打开本项目；修改前相关测试已有 `StableMcpToolEvent` 未解析错误。
- 本机暂未发现运行中的 Gradle Daemon；必须复用 Daemon，无法复用时将构建验证记为阻塞。
- 已通过 Kotlin 2.4.0 缓存编译器 + 本机 Temurin Java 25 的独立 fixture 编译；未运行 Gradle 构建。
- 真实 Java 子进程使用 fixture 类目录与 Kotlin 标准库启动，验证 initialize、通知无响应、tools/list、tools/call、环境变量/工作目录、引号/反斜杠/Unicode、未知方法错误及 EOF 正常退出全部通过；临时编译和运行目录已清理。
- IDEA 新 fixture 无错误；原测试仍报告修改前相同的四项错误，涉及 `StableMcpToolEvent` 及关联引用。
- 两个仓库的 `git diff --check` 通过；Kodex 直接跟踪的 Python/Java 文件已无剩余，产品仓库内无已删除脚本名称引用。实验目录仅余既有未跟踪 `.ruff_cache`，未操作。
- `:mcp-stdio:jvmTest` 未运行：两次进程检查均无运行中的 Gradle Daemon，按环境规则不自行启动，也不切换设备；用户随后要求直接标记完成，按现有验证结果归档，不将未运行的测试记为通过。
- 验证期间检测到外部新提交 Kodex `a4cb748c`、BuildKodex `cd9d465` 已包含实现改动；本 Session 未创建提交或修改历史。
