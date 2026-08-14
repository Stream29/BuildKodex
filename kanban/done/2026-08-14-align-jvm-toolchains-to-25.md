# Task Tree

- [done] 对齐 JVM toolchain 到 25
  - [done] 核对现有 toolchain 与本机 JDK
  - [done] 统一常规 JVM toolchain 为 Java 25
  - [done] Desktop 强制选择 JBR 25
  - [done] 记录 JVM toolchain 决策
  - [done] 验证 toolchain 选择与完整构建

# Details

- Desktop JVM toolchain 使用 JBR 25。
- 其他现有 Java 26 toolchain 改为 Java 25，避免版本冲突。
- 根构建使用 Foojay 解析 JBR；Desktop 编译、测试、运行与原生打包任务统一绑定同一套 JBR 25。
- 本机已有 Gradle 自动配置的 Temurin 25；IDE 自带 JBR 25，但缺少 `jpackage`，不能直接承担原生打包。
- IDEA Project SDK 与 Gradle JVM 使用已注册的 `temurin-25`；Desktop 仍由 Gradle vendor 约束单独选择 JBR 25。
- Gradle 自动配置了 JetBrains JDK `25.0.4`；Desktop main/test 编译与 `run` 使用该 JBR。
- Compose `checkRuntime`、`jlink`、`jpackage` 均使用同一 JBR 25；DEB 构建成功。
- Desktop、七个共享 ViewModel、application/history Desktop 测试通过，共执行 403 个 Gradle task。
- IDEA 文件检查未发现本次改动问题；IDEA MCP 构建两次超时，Gradle 完整 Desktop 测试与打包均成功。
- 持久规则记录在 `checklist/jvm-toolchain.md`。
