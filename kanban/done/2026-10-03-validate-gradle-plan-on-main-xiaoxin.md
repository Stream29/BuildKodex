# Task Tree

> Historical diagnostic invocation completed at its recorded stop/rollback
> boundary. It did not complete production Native, IP or IDEA acceptance.
> Current compatibility, model and genuine IDE gates live in the active parent;
> no remaining gate is closed by this archive.

- `Confirm origin main and fork gitlinks without touching refactor/spec`()
- `Clone a verifiable main baseline on Xiaoxin and freeze its manifest`()
- `try {`
  - `Acquire the shared heavy lock and reject unrelated JVMs`()
  - `Capture the baseline configuration and project graph`()
  - `for (batch in orderedCompatibilityBatches) {`
    - `Prepare the isolated batch from its validated predecessor`(batch)
    - `Validate the batch and record failures without hiding them`(batch)
    - `if (!batchGatePasses(batch)) {`
      - `Persist evidence and suspend dependent migration pending user decision`()
      - `return`
    - `}`
  - `}`
  - `Validate genuine IDEA Sync and native cache correctness`()
- `} finally {`
  - `Stop only owned IDE/compiler/Gradle processes before releasing the lock`()
  - `Preserve reproducible patches, logs, manifest, and resume point`()
- `}`
- `Report passed gates separately from unverified runtime and current-branch gates`()

# Details

## Authorization and scope

- 用户明确授权先在主分支验证整套方案，设备为 Xiaoxin Ubuntu；本机 `refactor/spec` 仍在开发，不 checkout、修改或运行它。
- Parent：[开发体验实施规划](../executable/2026-10-02-rescue-gradle-development-experience.md)；原生研究证据：[原生机制复核](../planning/2026-10-02-verify-native-gradle-sync-mechanisms.md)。
- Owner：Session 445；仅授权隔离验证，不 commit/push、改变公共 fork、gitlink 或发布。
- 当前 GitHub `origin/main` 为 `3008a6de9f178cda7cd63123a0261d75013f772d`；本机 local main 较旧。Xiaoxin 的 HTTPS submodule clone 发生 TLS 中断，因此最终使用本机 exact main/gitlink commit 做 `git archive` 传输，避免活动 `refactor/spec` 和未经核对的旧归档。
- 研究目录：Xiaoxin `~/ACodeSpace/demo/kodex-gradle-research-445/main-validation-445/`；共享锁沿用父任务的 `device-heavy.lock`。
- 正常构建 JVM 为明确的 SDKMAN Temurin 25.0.4；MCP 21 toolchain 单独声明。若已有不能复用的用户 Daemon，拒绝竞争，不停止用户进程。
- 初始安全测量条件为 Gradle heap 2GiB、2 workers，Kotlin daemon-first/禁止 fallback；该设置不是最终产品资源预算，采集各进程 RSS 而不是把 launcher `/usr/bin/time` 的 RSS 当总峰值。

## Batch interfaces and stop conditions

- 批次顺序：MCP **自身** root name → 显式 moduleTree → KGP 2.4.20/Gradle 9.6.1 → root build-logic → Native 产品模型 → IP 兼容 → 真实 Sync/cache/资源。
- 将 `includeBuild.name` 和 fork `rootProject.name` 区分；只改前者不足以消除 fork 自己的 accessor 冲突。
- DSL 使用 root 无 receiver 与 nested `ModuleTreeScope` receiver 两个重载，根和嵌套树均自动声明自身，叶子用 `module`；必须验证 Kotlin settings 脚本的作用域选择，不能交付需要 `this.moduleTree` 才正确的绕过。
- 旧 main 含无 build script 的 root 聚合项目；新增显式中间树若自动包含自身，必须分类记录新聚合项目，保证旧模块、physical directory、依赖边不丢失，不能虚报完全相同项目数。
- KGP 兼容线遇到真实版本/编译失败，按用户决策停止；研究脚本/fixture 错误单独记录并修正，不归因 KGP，也不越过失败继续集成。
- 真实 Sync、Native 编译/测试和 IP/cache 通过前，`help`/`projects` 仅代表 CLI 配置门禁通过，不等同整套验收。
- Windows/macOS runtime 不能在 Linux 上声称完成；可用 metadata/跨编译证据与需匹配 host 的 cinterop/runtime 门禁分开。

## Exploratory checkpoint before verified clone

- `prechange-main-20261003/` 中的旧归档探针不作为本任务最终基线；其 root help 成功不能替代干净 main 冷态结果。
- 探针发现两个 DSL 实现错误：漏建中间节点会丢掉真实叶子项目；同脚本 root 函数会遮蔽 scope 内同名成员，导致路径丢父级。`this.moduleTree` 临时诊断恢复了原 142 个项目，但不是用户要求的最终语法。
- 探针仅改外层 composite name 后，Gradle 9.6.1 仍出现 `getKotlinSdk()` 重复；改 MCP 自身 `rootProject.name` 后 help 通过。
- 旧归档 build-logic/help 与基础 JVM compile 成功；旧全目标 IP 仍失败于 MCP `kotlinNodeJs`。这些仅用于准备可追溯的 main 批次，不宣称 IP/IDE Sync 已验收。

## Verified main checkpoint — 2026-10-03

- 干净 main 已通过本地 `git archive` 加 exact gitlink commit 转移到 Xiaoxin，避免远端 GitHub submodule clone 的 TLS 中断；manifest、日志和报告在 [远端验证报告](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/main-validation-445/REPORT.md)。
- main baseline `help` 约 42 秒；正确修改 MCP 自身 root name 后 root-name `help` 通过；KGP 2.4.20 + root Gradle 9.6.1 `help` 通过；build-logic prototype `help` 和 `:mcp-stdio:compileKotlinJvm` 通过。
- moduleTree prototype 的 project graph 与 baseline 均为 142 项且无 path 差异，但必须处理两个 DSL 语义：非 project 中间目录不能自动 include；settings script 中 receiver 同名成员需要避免被顶层函数遮蔽。当前只算诊断原型，不算生产门禁通过。
- Native-only profile 的非 IP `help` 通过；IP 继续被 Mosaic 的 `allprojects` 访问 `GROUP`、`Project.group`、`VERSION_NAME`、`Project.version` 和 `Project.repositories` 阻塞。MCP `kotlinNodeJs` 已不再是第一阻塞。
- Native profile 的 JVM + Linux Native compile 在弱设备上约十分钟未进入目标 task，Gradle daemon RSS 约 2 GiB；已停止自有进程。完整 Native 编译、IP、真实 IDEA Sync/cache 仍未验收。
- 这次结果应作为当前阶段的暂停点：root name/KGP/build-logic 的 CLI/JVM 门禁可继续评审；moduleTree、IP、Native/runtime 和 IDE 门禁必须在下一批次单独解决，不得用 heap/workers 或 binary fallback 绕过。
- 详细复用结论：[共享研究记录](../../shared-context/findings/2026-10-03-gradle-main-validation-xiaoxin.md)。
