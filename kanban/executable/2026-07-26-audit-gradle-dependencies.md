# Task Tree

- 清理全仓 Gradle 依赖声明
  - [done] 审计旧基线的 79 个构建脚本和 495 条声明
  - [done] 复核当前 111 个构建脚本和 746 条声明
  - [done] 清理确认的无用依赖与传递桥接
    - [done] 移除当前模块结构下的无用声明
    - [done] 将 `utils/patch` 的 JSON 依赖收紧为 serialization core
    - [done] 收紧 `agent-state/test` 的依赖导出边界
    - [done] 为测试消费者补充实际使用的直接依赖
    - [done] 移除未使用的 serialization 插件
  - [done] 复核 `commonMain implementation` 的公共 ABI
    - [done] 将进入公共 ABI 的 50 条声明提升为 `api`
    - [done] 将 4 条未进入公共 ABI 的声明降为 `implementation`
    - [done] 通过 Linux KLIB ABI 复核剩余声明
  - [done] 评估 normal tool spec 与 handler 的模块拆分
  - 验证 JVM、JS 和 Linux x64 的 main/test 编译
    - [done] 验证 43 个受影响模块
    - [done] 单独验证 `app-cli-application` 的 Linux x64 测试编译
    - 验证被 Mosaic JDK 22 绑定生成故障阻塞的 JVM 测试编译
  - 运行受影响模块测试
    - [done] 运行 38 个无现存失败的 JVM 测试任务
    - 复核 4 个现存功能或真实端点测试失败

# Details

- 当前状态：依赖清理已实施；任务保留在 `executable/`，等待与本次构建脚本变更无关的验证阻塞解除。
- 当前改动覆盖 `Kodex/` 下 44 个 `build.gradle.kts`：
  - 50 条 `implementation` 提升为 `api`，覆盖协程、I/O、schema 和项目公共类型。
  - 4 条 `api` 降为 `implementation`。
  - 7 个不再需要的 Kotlin serialization 插件应用已移除。
  - `agent-state/test` 不再导出实现模块；测试消费者改为声明直接依赖。
  - normal tool 模块移除了旧 tool-builder、JSON 和插件依赖，保留既有 contract/impl 边界。
  - `utils/patch` 从 serialization JSON 收紧为 serialization core。
- 变更后共有 747 条显式 `api`/`implementation` 声明；净增 1 条来自补齐测试直接依赖，不是新增生产依赖。
- 依据 `checklist/tool-handler-decisions.md`，normal tool 的 spec、模型和实现不继续拆分；本次只清理过时声明。
- Linux KLIB ABI 复核覆盖 52 个含 `commonMain implementation` 的模块；未发现剩余直接 implementation 类型进入公共 ABI。
- `mcp/streamable-http/build.gradle.kts:12` 保留 `utils-ktor-client-ext`，用于安装平台默认 HTTP engine；该初始化依赖不会形成普通 JVM 字节码引用。
- 编译验证：
  - 43 个受影响模块的 120 个 JVM、JS、Linux x64 main/test 编译任务通过；Gradle 汇总为 801 个任务成功。
  - `:app-cli-application:compileTestKotlinLinuxX64` 通过。
  - `:app-cli-application:jvmTestClasses` 在进入应用编译前失败于 `Mosaic/mosaic-tty/src/jvmJdk22/` 的 `Libmosaic` 绑定未生成。
- 测试验证：
  - 排除现存失败后，38 个受影响模块的 `jvmTest` 通过；Gradle 汇总为 387 个任务成功。
  - `agent-runtime/decorator/subagent` 的失败响应事件测试未抛出预期异常。
  - `tool/unified-exec/impl` 的真实 shell session 测试失败。
  - `app/shared/auth/filesystem` 与 `tool/web-run` 的真实 OpenAI 端点测试失败；后者因连续真实端点调用已停止剩余执行。
  - `integration-test` 仅完成 JVM、JS、Linux x64 测试编译；真实端点测试未重复执行。
- 本次未修改 Kotlin 源码，也未创建临时文件。
