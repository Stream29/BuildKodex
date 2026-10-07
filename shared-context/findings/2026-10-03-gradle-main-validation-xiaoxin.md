# Gradle 主分支验证：Xiaoxin Ubuntu

- 日期：2026-10-03
- 设备：Xiaoxin Ubuntu，14 GiB RAM / 16 vCPU
- 根 main：`3008a6de9f178cda7cd63123a0261d75013f772d`
- fork gitlinks：
  - Mosaic `7b1a412918f7bc24db847343480f010242939788`
  - LuceneKmp `549f8afc9570bd4289f1047653d4cd6dc9665bbd`
  - KotlinMcpSdk `53f717661712d618cd4822d1760392e903989ead`
- 远端报告和原始日志：`file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/main-validation-445/REPORT.md`
- 所有实验均在隔离副本执行，没有 checkout、修改或运行本机 `refactor/spec`。

## 已通过

- 干净 main `help`：42 秒。
- MCP root name：
  - 外层 `includeBuild` name 改为 `kotlin-mcp-sdk-fork`；
  - fork 自身 `KotlinMcpSdk/settings.gradle.kts` 的 `rootProject.name` 也必须改为 `kotlin-mcp-sdk-fork`；
  - 两者同时修改后 `help` 通过。
- 显式 `moduleTree` 原型：
  - `help` 通过；
  - baseline 与原型均为 142 个项目；
  - project path 集合无差异。
- KGP `2.4.20` + root Gradle `9.6.1`：CLI `help` 通过。
- `buildSrc → build-logic`：
  - root `apply false` 统一 KGP/Serialization/Compose classpath；
  - `help` 通过；
  - `:mcp-stdio:compileKotlinJvm` 通过。
- Native-only 原型在不启用 IP 时 `help` 通过。

## 阻塞与设计修正

- 最初的 module tree 原型把无 `build.gradle.kts` 的中间物理目录也当成 project，导致 `:tool-unified-exec-impl` 等叶子依赖缺失。
- 修正版只让 root tree 自动包含自身；nested `moduleTree` 只有目录存在 build script 时才注册 project，非 project 目录只作为命名空间。
- Kotlin settings script 中，顶层无 receiver `moduleTree` 会遮蔽 receiver scope 的同名成员；原型需要写 `this.moduleTree(...)` 才能保持父路径。这个行为说明 DSL 还不是生产可交付语法，需要单独解决。
- Native-only profile 消除了已知 MCP `kotlinNodeJs` 阻塞，但 IP 随后暴露 Mosaic 的 `allprojects` 跨项目访问：
  - `GROUP`
  - `Project.group`
  - `VERSION_NAME`
  - `Project.version`
  - `Project.repositories`
- Native profile 的 JVM + Linux Native 编译在弱设备上约十分钟仍停留在配置阶段，Gradle daemon RSS 约 2 GiB，尚未开始目标 compile task；已停止自有进程，没有用加 heap/worker 强行掩盖。

## 当前判定

- root name、KGP/Gradle、build-logic 的 CLI/JVM 门禁已通过。
- moduleTree 的项目集合已通过，但 receiver 语法和“非 project 中间目录”语义仍需设计修正，不能直接进入生产。
- IP、完整 Native 编译和真实 IDEA Sync/cache 仍未通过，因此按用户此前规则暂停后续生产迁移；不自动启用 IP、不恢复 binary/GitHub fallback。
- 原始日志目录：`file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/main-validation-445/results/`
