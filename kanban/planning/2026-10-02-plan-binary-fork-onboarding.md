# Task Tree

> **Superseded route:** retained as historical binary/source onboarding research only. The current default is GitHub Packages CI for clean fork artifacts with explicit source mode for dirty forks; do not execute this plan as a GitHub Release/bootstrap implementation without renewed user authorization.

- **`Await native Gradle evidence and renewed fallback approval`()**
- `Review first-clone and source-development experience`()
- `Freeze the current dependency graph and shared configuration inputs`()
- `Implement pinned package installation and local repository routing`()
- `Make CLI and direct IDEA import use the same bootstrap`()
- `Implement explicit per-library source mode and diagnostic checks`()
- `Await the verified GitHub release and pin`()
- `Verify first-clone, incremental, offline, corruption, and switch cases`()
- `Document the ordinary and fork-maintainer workflows`()

# Details

## Scope and contract

- Parent：[实施规划](../executable/2026-10-02-rescue-gradle-development-experience.md)；发布依赖：[GitHub fork 包](2026-10-02-plan-github-fork-artifact-publication.md)。当前仅 planning，不修改活动 Kodex。
- 当前仍为备选：默认已采用固定 GitHub Packages binary，本地自动变化检测/引导不实施；[延期原生研究](2026-10-02-verify-native-gradle-sync-mechanisms.md) 不再作为主线前置条件。
- 默认体验：克隆 → 项目 JDK 25 → 直接 IDEA 打开或 gradlew；不要求初始化三个 fork、配置 token、安装 gh/Python 或先构建发布包。其他必需子模块在冻结当前图时单独列出。
- 首次 Sync 可自动下载一个固定包；它不是自动源码编译。额外 setup/doctor 命令可用于预下载/修复，但不能成为隐藏的强制前置步骤。
- 研究验证过 file repo 安装/校验和逐库 source mode；**尚未实现 IDE 与 CLI 通用入口或 Windows installer**：[研究证据](../../shared-context/findings/2026-10-02-gradle-development-experience-research.md)。

## Proposed implementation surfaces

- `settings.gradle.kts`：先读取受版本控制的 lock 与本地开发选项，准备已发布包，再配置 fork 排他本地仓库和按库条件 includeBuild。不在 settings 内调用 Gradle、git fetch、publisher、Native compiler 或子模块初始化。
- bootstrap 候选为最小 JDK/Gradle API 实现；避免为了准备依赖先编译复杂 buildSrc/build-logic 或引入新运行时。CLI wrapper-only hook 不足以覆盖 IDEA Tooling API，必须测试直接 IDE 导入。
- `gradle/` 下单一 lock 与 catalog 坐标映射；版本、URL/pin 与三个 gitlinks 一致性检查有唯一输入来源，不让 settings/catalog 各维护一份版本事实。
- 仓库级本地、Git-ignored 开发选项文件由 CLI/IDE 共读；定义优先级与生效时机，用户全局属性不默默覆盖项目 lock。正式属性名沿用候选 `kodex.forks.source`，最终文件位置/格式在实施审阅时固定。
- 审核当前 `PREFER_PROJECT` 与所有 project repositories；fork 路由须在真正解析位置排他生效，不能仅在 settings 添加一个可能被忽略的 repo。plugin repositories 与普通非 fork Maven Central/Google 依赖保留。
- publishing、测试源码、Native cinterop 与 sources attachments 全部消费相同 pinned package；full/JVM target profile 属另一条显式选项，不由 source mode 推断。

## Download, installation, and fast-path rules

- 根据 lock 中的固定 HTTPS asset URL 下载；明确连接/读取超时、有限重试、进度、取消、代理及离线行为。匿名公开 asset 下载不依赖 API rate-limit 查询；私有托管不符合本期无 token 目标。
- 下载和安装使用跨平台进程锁；临时文件与最终目录在可原子安装的持久磁盘，不能用 tmpfs 存 cache。处理 Windows rename/路径/锁/杀进程与半包恢复。
- 归档 SHA、manifest pin、每文件 SHA、目标闭包与 consumer 验收状态校验后原子提升；拒绝路径逃逸、越界软链接、重复路径和解压超量。只有完整成功才写完成标记。
- 快路径读取 schema/pin/完成标记，不联网、不启动 fork Gradle、不在每次 Sync 重扫整个发布物目录；完整校验在安装、显式 doctor 或异常恢复时做，说明 marker 信任边界和损坏发现机制。
- 包/缓存按 immutable bundle identity 分目录，允许多个 checkout/分支共享且不互相覆盖；先不自动删除仍被其他 checkout 使用的旧包。
- `--offline` 或断网：完整已安装包正常使用；没有对应 pin 的包则快速给出具体缺项与修复入口。主仓库其他依赖/JDK 仍可能未缓存，不承诺整个项目无网开箱即用。
- bootstrap 外部状态与 Gradle configuration cache 输入/重用行为必须测试；cache 命中不能绕过新 lock、模式变化或包失效。首次下载、后续检查与 Gradle 配置开销分别采样。

## Explicit source mode

- 空选项为全 binary；`mcp`、`mosaic`、`lucene` 或合法组合只 include 指定 fork，覆盖对应完整模块映射。非法库名、缺源码目录、坐标映射错误明确失败。
- 按源 build 的 group/artifact/project identity 显式 substitution，避免默认发现配置无关项目；保留 Kotlin MCP root/accessor 命名约束，先复现再决定是否需要改名，不照搬实验绕过。
- 显示当前 mode、实际 fork 版本/源码 SHA 和 profile。没有选中的源码目录不应被扫描、初始化或启动构建。
- 当用户编辑 fork 但仍 binary 时提供准确提示：本地改动不会生效，需明确启用对应 source mode。不能仅凭 Git dirty 自动切换，不让一次 commit 暗中切回 binary。
- 修改 fork gitlink 却没有匹配新 binary lock 必须明确失败/引导 source 联调；未提交 fork 修改不冒充已发布版本。诊断 Git 不可用时提示受限，不默认重建。
- source→binary、binary→source、新 checkout、分支切换、单库/组合模式及 IDE 重开均检验 classpath/模型/源码导航；不得残留旧 source 模块或绑定错误 sources。

## Acceptance matrix and rollback

- Linux/macOS/Windows：独立用户目录/空项目缓存，无 token/gh/Python、fork 子模块未初始化，CLI 与实际 GUI IDEA 首次导入成功；Windows 不是仅校验 POSIX 脚本。
- 并发启动两个 checkout、安装中止、wrong pin、损坏文件、缺件、网络超时、404、offline、lock 更新与 schema 不兼容均有可操作错误且不会 source/upstream fallback。
- 第二次 Sync 无 fork 项目配置、无包下载/重发布；验证 root/.module/目标 payload、sources/jump、fork-only APIs 和当前分支主要消费者。
- binary/full 主模型/根/内部边等价；资源收益按真实 IDE 测量，不把旧 CLI RSS 结果作为验收。与 [模型/资源任务](../executable/2026-10-02-plan-gradle-model-resource-optimization.md) 共享固定环境。
- 回滚为独立 settings/repository/catalog/本地配置补丁，恢复原显式 source build；不删除用户 fork 修改或共享缓存，不覆写公开版本。README 普通流程短，维护者发布细节另页链接。
