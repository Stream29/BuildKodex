# Task Tree

- `Read parent task and claim publication research ownership`()
- `Audit fork coordinates and complete transitive publication graph`()
- `Prepare isolated staged Maven repositories and immutable manifests`()
- `for (fork in requiredForks) {`
  - `Acquire the shared heavy-operation lock`()
  - `try {`
    - `Publish required metadata and target artifacts`(fork)
    - `Verify fork-only APIs and binary consumers`(fork)
    - `Compare binary and selective-source modes`(fork)
  - `} finally {`
    - `Stop owned resources and release the lock`()
  - `}`
- `}`
- `Measure new-version publication resources and validate bootstrap`()
- `Record a reproducible publication and bootstrap design`()
- `Report platform gates and evidence to the coordinator`()

# Details

## Ownership and authorization

- Parent：[本期 Gradle 开发体验](../executable/2026-10-02-rescue-gradle-development-experience.md)。
- 状态：**Session 453 研究已完成**；协调源 Session 445。研究执行已获授权，所有 scoped 实验与清理已结束。
- 协调 Session 445 已核对报告、完整 manifest pin、文件 hash 和消费/清理证据，完成研究验收并归档；生产接入另需授权。
- 本任务只维护本文件和自己实验目录；不改本机 Kodex 工作树、其他看板任务或本期原始基线。
- 在 Xiaoxin 持久磁盘上的 `~/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/` 工作；源 archive 和隔离 cache 可作为种子。
- 所有重型操作遵循父任务 `device-heavy.lock` 和 cleanup 协议；如需 Mac host，先按技能确认资源与授权，不操作用户现有 IDEA/Daemon 或其主工作树。

## Checkpoint — Session 453

- 发布图已审核：[GRAPH.md](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/GRAPH.md#L1)。Mosaic 6 模块、MCP 3 模块、Lucene core；源码为冻结 Git archive。
- Xiaoxin MCP 首轮因重复下载已有 LLVM 受控取消并停止自己的 Daemon；部分 staging 只留原始证据，不供消费者使用。
- Mac Mosaic 6 模块的 root/JVM/linuxX64/linuxArm64/macosArm64/mingwX64 发布成功（36 publications），无源码消费者已通过。
- Mosaic 首次失败为 Zig 0.15.1 / Xcode SDK26.5 链接 libSystem；实验 JNI 任务专用 `DEVELOPER_DIR=/Library/Developer/CommandLineTools`（SDK26.2）+ `-j2` 后成功。原始失败日志保留。
- Lucene Mac 初始 2 GiB/2workers 因 GC thrashing 失败；单 worker/4 GiB/nice 10 重试已成功（6 publications），成本不可混入 2 GiB 对照。
- MCP 完整 required 发布在 Mac 成功（18 publications）；原 Xiaoxin 部分发布只留作原始失败/准备证据，不混合两个 host 的根发布。
- Mac 全部 60 required publications 已由 [manifest checkpoint](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/results/mac-full-artifact-manifest.json#L1) 审核，`completeKodexArtifacts=true`、`graphErrors=[]`。未消费的 JS/Wasm/iOS/tvOS/watchOS/Android/macosX64 仍有声明但未发布，不能称全 SDK 所有平台已完成。
- 首次实验 consumer applied Kotlin script 缺 KGP classpath，已改为 Groovy 动态共享脚本并重新验证。属于实验脚本准备失败，不是 fork artifact 缺陷。
- 本机实验输出：[fork-publication](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/)；远端同名子目录。
- **已原子完成可交接 binary package（仅 Kodex 实际 5-target profile，不是所有额外声明平台）**：
  - 本机 repo：`/home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/repo`。
  - [已验证 manifest](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/manifest.json#L1)，SHA-256：`46a2c88ea51b9fef84f2aaf1876d1c5ffb67301c1e31b77cb67a3e2e5955b14a`。
  - 归档：`/home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/archives/kodex-forks-r1.tar.gz`；SHA-256：`7771b9786adb308f31809aab9cee6db6fa2a0356fa02091e6f2bc2853a9ee7a9`。
  - Mac repo：`/Users/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/repo`（同 manifest，package 已只读）。
  - 3 个 JVM probes、12 个 Native consumer compile、3 个 Mac Native compile/link/run 均在冻结源码目录移开时成功；解析图覆盖全部 5 个配置，没有上游 fork 版本混入。
  - [源码指纹](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/results/source-fidelity.json#L1) 将发布 sources 中 FocusOwner/Focus、MCP async stdio、Lucene logging 与冻结 Git 对象逐字节比对，全匹配。
  - **Xiaoxin 已通过共享 flock + checksum + 原子安装完成**；完整 repo 为 `~/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/repo`，同 manifest pin。不要消费旧 `staging-r1` MCP partial。
  - Xiaoxin 3 个 JVM + 3 个 linuxX64 Native compile/link/run 均通过。首次 TLS/HEAD 外部依赖准备失败已保留；仅使用设备已有 `127.0.0.1:7897` 代理重试成功，未改网络策略。所有自有 Daemon 已停止、锁已释放。
- 自包含 verifier/installer：[repository_manifest.py](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/repository_manifest.py#L1)。完整 package 存在时只校验，不启动 Gradle；缺失/损坏不会 fallback 到上游/source。
- 三个 fork 逐库 source opt-in 的公共新函数 sentinel 均在 JVM 和 Mac Native 运行成功（仅改独立 source-mode clone）。
- 同 JVM/参数热态 JVM consume 中位秒：MCP binary/source `0.51/1.18`；Mosaic `0.69/1.20`；Lucene `0.48/1.25`。这是 CLI 探针，不是 IDEA Sync。
- r2 缓存复用发布全部 60 项通过；Mosaic/MCP/Lucene wall 为 `67.75/39.43/13.83 s`，总 Java RSS peak 为 `1.952/2.393/0.994 GiB`。仅成本采样，**r2 不交接 Sync，不改 r1 package/pin**。
- Bootstrap 完整包无 Java/Gradle 校验快路径与 5 个安装安全测试通过；缺件/损坏/错误 pin 不 fallback 到上游或源码。
- 最终报告：[REPORT.md](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/REPORT.md#L1)；[发布成本](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/results/publication-cost-summary.json#L1)；[资源停止证据](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/results/final-resource-check.log#L1)。
- 剩余 host gates：linuxArm64/mingwX64 消费编译通过，实际运行未测试；额外 SDK platforms 不在 r1 profile。IDEA Sync、完整 Kodex 生产迁移及完整 fork 回归不是本任务已验证项。
- 未启动本机 Gradle/IDEA，未改本机 Kodex 源码；仅维护本任务和独立实验输出。
- 用户在本 Session 结构化问答中授权个人 MacBook 隔离实验；已加载 remote-macbook 技能并只读检查：有用户 IDEA，无运行 Gradle Daemon，不操作其窗口/项目。
- Mac 实验使用独立目录、初始 2 workers/2 GiB、stdlib Python `fcntl.flock` 持有 Mac 同名资源锁（Mac 无 flock 命令）。Mac JVM 为已有 GraalVM 25、MCP toolchain GraalVM 21.0.8；与 Xiaoxin Temurin 数据分开记录。
- Mac 独立目录实际为 `fork-publication.noindex`，`fork-publication` 是同目录链接，避免实验 cache 被 Spotlight 大量索引；不改系统索引偏好。

## Baseline evidence

- [真实 MCP 验证](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/REPORT.md#L23)：core/client 已发布根 metadata 和 JVM artifacts，版本 `0.14.0-stream.53f7176`。
- binary consumer 的 `commonMain` 引用并在 JVM 运行 fork 新增 `CoroutineStdioSource`；移开 SDK source 后仍成功。
- **父任务原始基线**不是完整多平台仓库；当时其他 target artifact 尚未发布，server 未纳入消费验证。本子任务的后续完整结果见 checkpoint。
- 根 main `3008a6de9f178cda7cd63123a0261d75013f772d`，Mosaic `7b1a412918f7bc24db847343480f010242939788`，Lucene `549f8afc9570bd4289f1047653d4cd6dc9665bbd`，MCP `53f717661712d618cd4822d1760392e903989ead`。
- 原始 `maven-repo/` 和 `results/` 只作对照，不覆盖；完整发布试验写新 staging repo 和新版本/manifest。

## Required coverage

- 确认 direct/transitive fork 模块、group/artifact/version 与实际项目/发布坐标对应关系。
- Mosaic 审核 runtime、animation、testing、terminal、tty 等传递模块及 C/native 库；保留修改后的 API 和行为。
- MCP 覆盖 core/client/server 的实际依赖和新增异步 stdio 接口。
- Lucene 覆盖 core 及其实际发布依赖，不能把上游同版本误当修改结果。
- 按 Kodex 实际消费 JVM、linuxX64、linuxArm64、macosArm64、mingwX64 检查 variants；明确 cinterop host 限制及未完成的 target。
- 保留 KMP 根 publication、`.module`、POM、metadata、target payload、sources；不使用 flatDir/JAR-only 绕过 variant graph。
- 设计按 fork commit/recipe 不可变的版本与 SHA-256 manifest；不以 SNAPSHOT 或全局 `mavenLocal()` 污染日常消费。
- bootstrap 在发布物存在且完整时仅验证/下载，不启动 fork Gradle；缺 artifact 时明确失败，不自动回退上游或全源码构建。
- source mode 按库显式 opt-in，完整声明 module→project mappings；验证 fork 改动后可正常联调。
- 仅向隔离 file repo 发布。未经用户独立授权不上传 Maven Central/GitHub Packages，不创建公网服务或修改网络策略。

## Checks and outputs

- 用独占 repository filter 和 fork-only API/校验和排除误用上游，记录解析 graph 及每个目标消费结果。
- 至少验证无 fork source 目录时实际 binary 编译/运行；普通 help 成功不替代验证。
- 记录一次发布成本与日常消费成本、cache miss/hit、资源；对照相同 JVM、workers、堆及模型请求。
- 实验结论分别标注已验证、失败、需要其他 host、未测试，不把根 `.module` 存在当成 targets 已发布。
- 输出包括发布矩阵、manifest、复现命令、binary/source settings 示例及维护成本；保存于自己的实验目录并在本任务链接。
- 将可供 Sync 任务消费的完整 staged repo/manifest 路径写入 checkpoint；原子完成后交接，避免消费者看到半发布版本。
- 释放所有本任务创建的 Daemon/编译器/采样资源，再释放共享锁。
