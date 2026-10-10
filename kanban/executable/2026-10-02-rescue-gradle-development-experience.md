# Task Tree

- `Confirm the current branch, fork relations, and frozen publication contract`()
- `coroutineScope {`
  - `val forkAudit = async {`
    - `Audit kodex-submodule ancestry, gitlinks, coordinates, target variants, and commit/tree identity`()
  - `}`
  - `val publicationEvidence = async {`
    - `Review the completed fork publication package and local-consumer evidence`()
  - `}`
  - `forkAudit.await()`
  - `publicationEvidence.await()`
- `}`
- `Review the GitHub Packages CI publication route with the user`()
- `Record production implementation and scoped deployment authorization`()
- `Freeze the current production baseline and publish the three pinned forks`()
- `Freeze the approved branch and exact fork commits in isolated workspaces`()
- `Design and validate the protected GitHub Packages CI publication flow`()
  - `if (cleanForkPublicationIsCompleteAndImmutable()) {`
    - `Consume the pinned package from the normal binary-consumer path`()
    - `Keep source includeBuild as an explicit fork-maintainer mode`()
  - `} else {`
    - `Stop package rollout and return the missing target, metadata, auth, or reproducibility gate`()
    - `Do not publish dirty local state or silently fall back to source composite`()
  - `}`
- `Apply the approved sequential production batches`()
  - `if (githubPackagesContractIsReviewable()) {`
    - `Define immutable coordinates, commit/tree manifest, target variants, credentials, retention, and single-writer publication`()
  - `} else {`
    - `Rollback the CI publication design and return the unresolved contract`()
  - `}`
  - `if (rootConsumerCanResolvePinnedPackages()) {`
    - `Apply and validate the MCP root name kotlin-mcp-sdk-fork`()
  - `} else {`
    - `Rollback the isolated batch; halt and return to the user`()
  - `}`
  - `if (rootModelAndSyncRemainEquivalent()) {`
    - `Apply and validate the explicit moduleTree settings DSL`()
  - `} else {`
    - `Rollback the isolated batch; halt and return to the user`()
  - `}`
  - `if (moduleGraphAndNavigationRemainEquivalent()) {`
    - **`Validate the kodex-submodule KGP 2.4.20 and Gradle 9.6.1 compatibility line`()**
  - `} else {`
    - `Stop the migration and ask the user to decide`()
  - `}`
  - `if (kodexCompatibilityLinePassesAllVersionGates()) {`
    - `Migrate buildSrc to build-logic with one root plugin classpath`()
  - `} else {`
    - `Stop all version and build-logic migration; return the concrete failure`()
  - `}`
  - `if (buildLogicPreservesPluginIdsTargetsSourceSetsAndTasks()) {`
    - `Freeze and verify the Windows Linux macOS Kotlin/Native product model`()
  - `} else {`
    - `Rollback the isolated batch; halt and return to the user`()
  - `}`
  - `Set Kotlin daemon-first resources with fallback disabled and measure the control group`()
  - `Validate normal package mode, explicit source opt-in, first clone credentials, dirty fork, and clean package pin behavior`()
  - `Validate CLI and genuine IDEA Sync, model/navigation, compile/test, cache invalidation, and host matrix`()
  - `Review measured results and approve rollout, rollback, or another user decision`()
  - `Record the developer bootstrap procedure and close the task`()
- `if (userLaterApprovesIsolatedProjectsResearch()) {`
  - `Run IP diagnostics as an optional, separately gated optimization`()
  - `Never make IP, jextract, or dangerouslyIgnore a prerequisite for binary local-publication mode`()
- `}`

# Details

## Current production execution

- 2026-10-11 用户否决修改或维护 KGP；不采用实验 Kotlin 插件、不新增
  Kotlin fork 或包发布流水线。仅分批提交已验证的
  [实际测试所有者配置](2026-10-11-adopt-project-owned-gradle-test-configuration.md)。
  保持官方 KGP2.4.0；源码实验成功、两个未修改控制 OOM 和 Json 导航失败
  分别保留，完整开发体验验收继续阻塞。
- 用户明确要求优先落地 Gradle 开发体验改进，并同步纠正忘记更新的看板状态。
  用户随后授权按验证门槛分批提交、推送，建立三个 fork 的
  `kodex-submodule` 维护线，并部署 `Stream29/Kodex` 的三个包发布 workflow；
  本批不发布新版 Kodex。
- 生产基线为 Kodex `837494340036f433e5320fabedccd874b0f320e7`
  （v0.4.10），内层工作区干净。三个当前 gitlink 与下方关系审计表一致。
  外层其他用户文件不得夹带进本批提交。
- 本机构建与 CLI/IDE 运行仍暂停：用户使用本机打游戏。重型验证仅在
  Xiaoxin 的本轮独立目录，持原 `device-heavy.lock` 串行执行；Apple 专属
  构建门槛使用标准 GitHub macOS runner，不接管用户设备窗口。
- 包读取凭据只配置本机、remote-macbook、xiaoxin-ubuntu、aliyun-ecs。
  用户已完成浏览器本人验证；新 classic PAT 仅有 `read:packages`，
  到期日为 2027-01-05。凭据只进入用户级私有配置，不写入仓库、任务、
  manifest 或构建日志，不替换原 `gh` OAuth 登录。
- 执行顺序采用 `Integrated implementation order`，先完成发布/consumer
  门槛，再进入结构、固定目标模型与资源批次。后文“仅 planning”“待执行授权”
  是历史边界，已被本节当前授权替代；产品语义及兼容失败停止条件保持不变。
- [包发布实施](../done/2026-10-04-plan-github-packages-fork-publication.md)、
  [默认 binary consumer](../done/2026-10-07-consume-verified-fork-packages-by-default.md)、
  [模型与资源实施](2026-10-02-plan-gradle-model-resource-optimization.md)、
  [项目级受控优化](2026-10-04-plan-kodex-gradle-optimization.md)；
  [旧看板对账](../done/2026-10-07-reconcile-completed-migration-task-status.md)。
- 本期不重启 Isolated Projects、Declarative Gradle 或已否决的 settings
  自动发布；不以旧分支研究结果代替当前 main 的验收。
- Initial execution checkpoint: 已在三个公开 fork 创建 `kodex-submodule`，
  分别准确指向上表现有 gitlink，不修改源码、上游协作分支或产品 gitlink。
  本轮 Kodex 工作线为 `build/gradle-development-experience`，
  生产 main 在 consumer 门槛通过前继续使用原可构建配置。
- Current checkpoint: 三套固定包完整核验（Mosaic803、SDK453、Lucene133），
  默认 binary 消费四文件提交 `c0ba86cc` 已推送 main。无 fork 源码的
  Xiaoxin 验证通过132项 JVM/Node、CLI/Integration编译、Native release链接、
  39个实际包摘要、9个源码附件和隔离CLI启动/Settings/退出；
  缺凭据401、缺pin只查指定仓库，未回退源码。模型仍为216注册项目，
  KGP2.4.0/Gradle9.5.1不变；Native实验2g OOM和随后原预算4g成功分别保留。
  SDK root-name 维护提交51ca6e已完成真实source/API门槛，新包CI37696609442
  三宿主和453远端文件核验SUCCESS；
  [根consumer回归与只读审计](../done/2026-10-08-maintain-mcp-sdk-composite-root-name.md)
  通过，catalog-only提交2e5c0df4采用51ca6e包，未覆盖旧版本。
  [显式moduleTree](../done/2026-10-08-integrate-explicit-module-tree.md)
  已以settings-only提交9458be6f推送：216实际模型等价、8负例、
  定向测试与完整CLI链接通过，独立审查无确认B1。
  下一步是[工具链兼容性门槛](2026-10-08-validate-kodex-toolchain-compatibility.md)；
  真实IDE收益、build-logic与资源门槛仍开放；不引入目标profile。
- 隔离Koin1.0.1警告失败后，用户授权1.2.1重验；现110项Jvm结果与in-process
  控制通过，Werror未关闭。Node停在旧JS锁后，用户授权标准锁更新与差异审查；
  标准任务28s和Node重验52s通过，完整Linux CLI链接与64项Native回归507s通过，
  新二进制隔离PTY正常启动、Settings交互与退出通过。
  版本/build-logic未集成，生产main仍2.4.0/9.5.1。
- Xiaoxin 干净 detached baseline 已 checkout v0.4.10 与全部 nested gitlink；
  public SSH URL 的 clone 首次失败后，用本次命令级 HTTPS rewrite 修正，
  不修改设备全局 Git/SSH 配置。基线 CLI 配置测量持共享重型锁进行。
- 原样 baseline `help` 失败：生成 `RootProjectAccessor.getKotlinSdk()` 重复方法，
  与原 MCP root-name 调查一致；39s Gradle / 40.44s wrapper wall。
  原失败保留在远端 `results/baseline-help-cold.log`，不能作为完整 Sync
  成功样本。已批准的 root-name 修正及 binary 消费后需分别重验。

## Authorization and scope

- 2026-10-02，用户确认三条均为本期必须完成，并要求“记入看板作为并行任务开工”。
- 本任务授权研究、远程隔离实验、必要的实验环境准备及任务记录；不授权修改本机正在开发的 `Kodex/`、提交或推送。
- 最新授权：用户要求进入 **planning**，先整理 fork 分支关系和 Gradle 修复路线；此次只编写计划，不创建实施会话、不修改活动源码、不上传包或修改 GitHub 设置。
- 用户随后重新评估了 Isolated Projects 的投入产出，决定把它从当前主线降为可选后续优化；先形成稳定 fork 的 binary 消费路线。
- 用户最新决策：clean、可复现的 fork artifact 由 `Stream29/Kodex` 仓库的 GitHub Actions 发布到 GitHub Packages；BuildKodex/Kodex 默认消费固定 package；dirty fork 不自动发布到远程，改走显式 source mode 或显式本地发布。
- 用户进一步决定：由 `Stream29/Kodex` CI 负责发布；其 nested fork submodule gitlink 更新时触发 package pipeline，package version 对应被 pin 的 fork hash。
- 用户明确发布时序：先合并主分支 submodule gitlink 更新，CI 发布对应 hash package；package 发布并验证完成后，再单独更新主分支依赖版本。
- 三个 fork 各自拥有一套独立 CI workflow；每套只发布对应 fork package，允许 workflow 内容重复，不做三方 bundle 编排。
- package CI 失败时保留已合并 gitlink，暂不更新依赖版本；默认 binary 继续使用旧 package，CI 修复后由维护者手工更新依赖。
- 用户进一步决定：三个 package 集中归属 `Stream29/Kodex`，中央 CI 使用 GITHUB_TOKEN 单写入；package coordinates 已确认保持上游 group/artifact。
- 用户已确定 package 版本格式：各 fork 上游版本加 `-kodex.<short-sha>`，manifest 保存完整 40 位 fork hash。
- 不新增独立 fork lock 重复保存 hash；BuildKodex 现有依赖版本声明作为唯一消费 pin，CI manifest 只保存完整 hash/target 验证证据。
- 三个 fork package 保持上游 group/artifact coordinates 不变，只通过 `-kodex.<short-sha>` 版本后缀区分；source/binary 切换不改依赖坐标。
- dirty fork 暂不引入本地开关文件或自动 source profile；维护者临时手工修改 `includeBuild`，完成后恢复默认 binary 配置。
- 用户已确定 package 采用产品目标闭包优先：不发布 JS/Wasm 或非产品 Native targets，只保留实际 Windows/Linux/macOS 闭包及真实 consumer 需要的 JVM variant；architecture/host 仍需 manifest 冻结。
- 首批 architecture matrix 已确定为 `linuxX64`、`linuxArm64`、`macosArm64`、`mingwX64`，另加真实 consumer 必需的 JVM；`macosX64` 暂缓。
- Native CI 允许交叉编译 `linuxArm64`；Apple binary、Apple SDK/cinterop、link/sign 必须保留 macOS host gate，不能用交叉编译替代真实 Apple 验收。
- 用户已确认接受 GitHub classic PAT 的一次性开发者配置，仅授予 `read:packages`；token 通过用户级 Gradle 配置或环境变量注入，不写入项目。
- 原生机制复核和既有发布实验仍作为证据输入；settings 内 dirty 检查和嵌套 Gradle publish 不再作为默认方案。GitHub Release archive、全局 Maven Local 和 SHA 自动切换仍是后备选项。
- 2026-10-03 用户明确本机空闲、授权本机验证；允许私有 GUI/Gradle 和冻结源码副本的诊断改动，仍不修改活动 Kodex、用户 IDE 配置或运行生产发布。实际冻结为当前干净 `05a472e9`，200 个主 build scripts；本机证据不套用旧 main 的比例。
- Xiaoxin Ubuntu 已获用户授权做实验和使用 SDKMAN 安装 JDK。其图形会话负责 Caffeine 保活，不远程重启。
- 不复用或操作用户现有 IDEA 窗口；用户接管资源时停止相关操作。
- 保留正在推进的 spec/impl 项目边界；不能为优化 Sync 而撤销该用户设计。

## Execution orchestration

- 这棵树是“先研究、后授权、再逐批实施”的单一入口；planning 文档本身不授权修改活动源码、fork、gitlink、提交、推送或发布。
- 只有只读 fork 关系审计和已有证据整理可以在 `coroutineScope` 中并行；任何 Gradle、Kotlin/Native、IDEA Sync、索引或资源测量都必须在 Xiaoxin 的 `device-heavy.lock` 下串行执行。
- 本期主线从 source composite/IP 优先改为“GitHub Packages clean binary 优先”；普通开发不默认配置三个 included build，source composite 只作为显式 source-debug 模式。
- GitHub Actions 只发布 clean、可复现、不可变的 `kodex-submodule` commit；不得从普通 PR 或开发者本地 dirty 工作树自动发布远程 package。
- CI 实际配置在 `Stream29/Kodex`，因为三个 fork gitlink 嵌套在该仓库；公开 nested submodule 使用 GitHub-provided checkout/token，不引入 deploy key 或额外跨仓库 secret。
- 每个实施批次都在独立工作树和受保护 CI 流程中完成；Native matrix 产物必须经过校验后由单一 publish job 写入同一坐标，避免半完成 Maven metadata。
- GitHub package 缺失、认证失败、metadata/variant 不完整、commit/tree 不匹配或 CI 发布失败时，必须明确失败；不能偷偷回退到 `includeBuild` 或在 settings 中启动 Gradle。
- Windows/Linux/macOS Kotlin/Native 是产品模型边界；JS/Wasm 只有在确认存在真实源码或构建依赖时才作为异常处理，不为任何缓存或 IP 通过而静默删除语义。
- Kotlin compiler 采用 daemon-first 且 `kotlin.daemon.useFallbackStrategy=false`；具体 heap、workers 和 IDEA heap 必须在 binary/source 路线稳定后测量，不先用堆或并发数掩盖配置成本。
- Isolated Projects 从主线降级为独立后续优化：不阻塞本地发布物消费，不使用 `dangerously-ignore`，不继续把 Mosaic/jextract 兼容迁移当作当前发布路线前置条件。
- Gradle/IDE 原生 Tooling cache 仍验收完整请求、输入失效和 IDE 重开正确性，但不再承担 fork 变化检测或自动发布职责；失败时保留证据，不发明第二套缓存。
- GitHub Release archive、全局 `mavenLocal()`、SHA 自动切换和默认 `includeBuild` 均不是当前默认方案；只有 GitHub Packages 路线明确失败并经用户重新决策后，才恢复相应后备路线。

### Integrated implementation order

1. 冻结当前分支、三个 `kodex-submodule` fork 的具体 commit/tree 和跨平台 baseline；不在活动工作树上直接试错。
2. 复核完整 fork publication package 的坐标、manifest、KMP metadata/POM 和 Windows/Linux/macOS 所需 Native variants，明确 clean commit、target closure 和 artifact identity。
3. 设计三个 fork 的 GitHub Actions workflow：只允许保护分支/tag/manual 触发，使用标准 runner，禁止 dirty/PR 自动远程发布。
4. 在隔离副本验证 CI 的 build matrix、single-writer publish、manifest/checksum、重复版本拒绝和失败清理；公开仓库免费分钟、Packages 认证、存储与 retention 单独记录。
5. 以 GitHub Packages binary consumer 为默认接入，保留显式 source opt-in；验证首次 clone credential、正常 pin、package 缺失、认证失败、离线和并发 Sync。
6. 在 binary consumer 已可解析后再应用 MCP root name `kotlin-mcp-sdk-fork` 和显式 `moduleTree` DSL；每批验证 project graph、源码导航和模型等价，失败只回滚该批。
7. 在 `kodex-submodule` 隔离维护线上验证 KGP 2.4.20 + Gradle 9.6.1；实际 plugin codeSource、Native 模型和编译必须通过，否则按用户约定中断，不自行降级或继续。
8. 在版本线通过后迁移 `buildSrc` 到 `build-logic`，以 root `apply false` 统一 plugin classpath；逐项验证 convention plugin、KMP target/source set 和任务语义，不能把维护性收益冒充 Sync 提速。
9. 固定 Windows/Linux/macOS Kotlin/Native 产品模型，优先减少非产品模型输入；这一步不删除仍有真实源码/构建依赖的 target，也不修改 API/spec/impl 语义。
10. 测量 daemon-first 控制组，再验证 CLI/真实 IDEA Sync、原生 Tooling cache、tracked input invalidation、IDE reopen、模型/导航/编译/测试和三 host 矩阵。
11. 最后由用户审阅 rollout/rollback；若 GitHub Packages 路线没有通过认证、发布完整性或新人验收，回到用户重新选择 Release archive、显式 bootstrap 或本地发布，而不是在 settings 中继续加黑魔法。

## Fork maintenance decision — blocking prerequisite

- 三个 fork 仓库都采用同名的 `kodex-submodule` 分支作为 Kodex 集成维护线；该分支的上游关系明确指向各原始仓库的主分支或开发分支。
- 各 fork 的原始仓库主分支/开发分支继续保持可用于同步上游和准备 PR；当前只读审计显示 Mosaic 上游为 `trunk`、LuceneKmp 上游为 `master`、KotlinMcpSdk 上游为 `main`。Kodex 专属的 KGP/Gradle 兼容修复、API 改动和构建脚本变化只进入 `kodex-submodule`。
- BuildKodex 将来如切换维护线时固定消费 `kodex-submodule` 上的具体 commit；分支名只用于维护和同步，不作为可移动的构建输入。
- 当前不修改任何 gitlink；本次决策只梳理命名和上游关系，不改变当前源码、依赖解析或 IDEA/Gradle 行为。
- 三条维护线分别同步上游、记录差异、回合并路径和发布责任；不要求三个 fork 共用同一 Kotlin/Gradle 版本。
- 这个决策只确定分支模型，尚未授权创建远端分支、推送、修改 fork 内容或发布 artifact。后续所有 fork 变更先在隔离副本验证，再按 fork 单独授权。
- 关系审计和具体 commit 冻结是实施树的前置门；完成后按上方 `Integrated implementation order` 顺序先验证本地发布，再处理 root 命名、`moduleTree`、版本线、build-logic、Native 和可选 IP/cache，不在维护线未冻结时改 gitlink。

### Read-only fork audit — 2026-10-03

| Submodule | BuildKodex gitlink | Current local branch | Public fork | Original upstream default |
| --- | --- | --- | --- | --- |
| Mosaic | `7b1a412918f7bc24db847343480f010242939788` | `trunk` | `Stream29/mosaic` | `JakeWharton/mosaic:trunk` |
| LuceneKmp | `549f8afc9570bd4289f1047653d4cd6dc9665bbd` | `fix/host-logging-configuration` | `Stream29/lucene-kmp` | `nehemiaharchives/lucene-kmp:master` |
| KotlinMcpSdk | `53f717661712d618cd4822d1760392e903989ead` | `main` | `Stream29/kotlin-mcp-sdk` | `modelcontextprotocol/kotlin-sdk:main` |

- 审计只读取 `.gitmodules`、submodule 状态、local remotes 和 upstream `HEAD`，没有 fetch、checkout、branch、gitlink 或 push。

## Completed parallel research children

- [Fork 发布实验](../done/2026-10-02-complete-fork-publication-experiments.md)：研究验收完成，完整不可变包、实际 target 消费和逐库 source mode 已交付。
- [真实 IDEA Sync 测量](../done/2026-10-02-measure-real-idea-gradle-sync.md)：研究验收完成，真实 GUI 导入、模型、导航、CPU/RSS 与清理证据已交付。
- [主仓库配置架构](../done/2026-10-02-research-gradle-configuration-architecture.md)：研究验收完成，JVM profile、local convention、插件隔离诊断和 DCL 小原型已交付。
- 三个方向可以同时调研、阅读和准备；资源敏感的构建和测量不是无节制同时运行。
- 子会话各自维护自己的任务文件和实验输出；仅协调会话维护本总任务，不改兄弟任务或用户其他看板记录。

## Dispatch checkpoint

- 用户已接受三会话派发批次；三个最终答复和任务 checkpoint 已由协调会话读取，研究成果已核对并归档。
- Fork 发布：[Session 453](file:///home/stream/.kodex/sessions/453)，负责 publication 子任务及 `fork-publication/`。
- 真实 Sync：[Session 454](file:///home/stream/.kodex/sessions/454)，负责 IDEA 子任务及 `idea-sync/`。
- 配置架构：[Session 455](file:///home/stream/.kodex/sessions/455)，负责 architecture 子任务及 `configuration-architecture/`。
- 协调会话为 [Session 445](file:///home/stream/.kodex/sessions/445)。子会话不继承源历史，已分别收到明确授权、基线、设备约束、锁、验收与任务文件位置。
- 回收接口为各子任务文件的 Session/checkpoint/证据链接和各自实验输出；发布任务将完整 staged repo/manifest 交给 Sync 任务，架构任务与 Sync 任务共享验证场景。
- 新普通会话的结果不会自动回流；本次通过文件主动回收并验收，不以派发 metadata 宣称完成。

## Research acceptance — Session 445

- Canonical 汇总：[三路研究验收](../../shared-context/findings/2026-10-02-gradle-development-experience-research.md)；三个研究子任务已完成，生产迁移未执行。
- 完整 fork r1 package 的 manifest pin 与 1,620 个文件 hash 已重新校验；支持 JVM/linuxX64/linuxArm64/macosArm64/mingwX64，实际 ARM Linux/Windows runtime 尚未测。
- 真实 GUI 热态原生导入中位数：Source full 108.277s、Binary full 71.696s、local/binary full 74.479s、local/binary JVM 7.500s。
- 后续 smart 验收分别为 112.469/75.631/78.232/10.502s，含统一 3 秒 settling；不能将导入提示当全部索引结束。
- Binary 保持主模型/源码根/内部依赖边并缩短约 33%；local/full 无明确额外速度收益。
- JVM profile 保留 JVM 祖先闭包但隐藏 Native 模型，不是 full 等价替代；142 个子项目、130 个 KMP 模块和 spec/impl 边界保留。
- 真实 IDE 场景的 Binary 没有证明 RSS 下降；JVM profile 降 Gradle RSS 到约 1.8GiB，IDEA 仍接近 5GiB。早期 CLI 内存结果不外推。
- IP 仍有插件阻塞，DCL 小原型成功但无主仓库完整生态/提速证明；这两项不直接投入生产。
- 原研究曾优先建议 binary/source→JVM/full→local build logic；用户最新要求改为原生机制与版本/插件行为优先，下述 binary 落地步骤仅保留为暂停的备选。
- 生产前门禁：同 workspace full↔JVM 恢复、IDE 重开缺陷、当前活动分支回归、跨 host Native/CLI，以及 IDE 自身内存诊断。
- 远端共享锁可获取，无研究 Java/IDE 残留，约 12GiB MemAvailable；原始证据、private IDE/cache 与发布物保留。
- 研究验收后进入下述实施规划；已验收证据仍对应冻结旧 main，不代表当前分支生产验收。

## Native-first gate — historical research input

> 本节记录原生方案研究及已完成的 canary；在用户把本地发布物路线设为主线后，它不再决定是否进入 binary/local-publication 方案。完整 Tooling cache 仍需独立验收，IP 不再是当前生产前置门。

- [原生复核任务](../planning/2026-10-02-verify-native-gradle-sync-mechanisms.md) 的剩余 full-source/IP cache 研究已延期，不阻塞 binary 主线；非IP的真实Sync、模型/导航与资源门槛仍在本期。
- [旧主分支 Xiaoxin 验证任务](../done/2026-10-03-validate-gradle-plan-on-main-xiaoxin.md) 已按当时 `origin/main` 与 exact gitlinks 重跑 root name、moduleTree、KGP/Gradle、build-logic 和 Native profile；已完成暂停/回滚诊断，但通过项不能替代当前203项目、完整Native或真实IDE验收。
- 严格区分任务增量/cache、配置缓存、tooling model cache、project configuration parallelism、IDE model parallel fetching；不将任一 cache 名称当作另一个的收益。
- 先保留三个源码 builds，验证显式完整 substitution 与 configuration-on-demand 对实际 IDE 的作用；随后测官方 `org.gradle.tooling.parallel` 和有界 workers，再按受支持版本组合验证 KGP 修复与 IP 兼容，不先造替代缓存。
- 新 native GUI 对照已完成：source control / explicit + COD 的三次热态中位数 111.669/110.829s，主模型完全一致，尚无提速或内存收益证据。该结果不是对选择性 publish 的否定，而是说明这两个开关没有解决现有完整源码导入的成本。
- 已核对 Gradle 9.5.1/9.7.0/9.8.0 源码：逐项目 intermediate 模型的 `isolatedProjectsCaching` 默认 `NONE`，普通模型请求 `configureOnDemand=false`。IP 完整 Tooling request/action 结果另有缓存路径，不能从 `NONE` 外推为全部模型结果都不能缓存；需重复请求/变更输入与实际 IDE 客户端验证，internal opt-in 不当生产原生解法。
- Kotlin 官方 `KT-87790` 记录 2.4.0 的 Sync 回归、2.4.20 修复线索；症状相似但不证明本项目根因，需独立版本对照。
- 原生 canary 已实测 IP 的完整 model 和同 client 的稳定 BuildAction 结果命中；改变 tracked build 输入后 model 正确失效。逐项目 intermediate 默认关闭不等于完整结果缓存关闭；下一门禁是完整 source/KMP 和实际 IDE action 的兼容与复用。
- 同轮根 Kotlin catalog 2.4.20 的 GUI 热态中位数 101.916s（control 111.669s），主模型等价，基础 JVM 编译通过；收益约 8.7% 仅属小样本版本组观察，需实际 KGP codeSource 及反向复测，不能单独归因修复或认为 RSS 已改善。
- 原生路线只在已测收益、兼容性代价或产品限制明确后与 binary 备选比较；即使探针失败也须归类到具体代码/API，不以笼统“插件不支持”结案。
- 当前本机补证见[本机原生报告](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/local-native-20261003/REPORT.md#L1)：真 GUI Java composite 已复用完整缓存并正确失效；完整 source/KMP 的 accessor、冷态 OOM 与 IP 插件/跨项目门禁未通过。默认策略及实际版本对照因本机新共享工作而在启动前暂停，生产规划不据此解锁。

## 修复方案与实施批次 — 已决策，待执行授权

以下是根据本机原生验证整理的实施批次。用户已逐项确认 root 命名、模块 DSL、fork 维护线、KGP/Gradle 版本、build-logic、daemon、IP 和原生 cache 边界。进入计划不等于授权修改活动源码、fork、发布或提交；每一项仍应单独验证、单独回滚，避免把 Sync、KMP 版本、内存和 Isolated Projects 的收益混在一次改动中。

1. **修复 composite root 命名冲突**
   - 将 MCP fork 的 included build root name 与其内部 `:kotlin-sdk` 项目区分，避免生成重复 `getKotlinSdk()` accessor。
   - 验收：原样 IDEA Sync 不再出现 `RootProjectAccessor.java` 重复方法；主项目、MCP 项目和源码跳转模型保持等价。
   - 证据：[本机完整源码尝试](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/local-native-20261003/REPORT.md#L23)。
   - 用户决策意见：没有问题，合格，进入 plan；root project name 采用 `kotlin-mcp-sdk-fork`。

2. **显式声明 root 模块树**
   - 不再让 `walkTopDown` 自动猜测模块；在 `settings.gradle.kts` 定义 settings-local DSL，根节点本身也统一使用 `moduleTree`：
     ```kotlin
     moduleTree("app") {
         moduleTree("shared") {
             module("session-title")
         }
     }
     moduleTree("rpc") {
     }
     ```
   - 当前已确认实现：根为 `fun Settings.moduleTree(String, block)` 扩展，nested 为 `ModuleTreeScope` 的同名成员；调用仍统一写 `moduleTree(...)`，分别表达 root tree 和 nested tree。原无 receiver 的 script 函数会遮蔽 nested 成员，实际验证失败；用户于 2026-10-08 批准上述声明级调整，不使用 `this.moduleTree`、上下文栈或额外调度器。
   - root/settings scope 的 `moduleTree(String)` 自动包含 root tree 本身，即使它只是没有 build script 的聚合目录；nested `ModuleTreeScope` 的 `moduleTree(String)` 只有在目录存在 build script 时才包含自身，纯物理中间目录只作为命名空间。两者 body 只声明子树。`module(String)` 是 `ModuleTreeScope` 中的叶子模块，表示不会再嵌套子模块的目录。
   - 所有树节点和叶子目录都由 DSL 显式写出，但 tree node 不需要再重复写一个同名 `module(String)`；模块路径由树上下文和叶子名称组合。
   - 自动把路径规范化为现有 project path 规则，例如 `app/shared/session-title` → `:app-shared-session-title`；重复物理目录、重复 project path 或命名冲突直接失败。
   - 首批 DSL 保持 KTS-only、使用 Gradle 原生 `include` 与 `project(...).projectDir`，不扩大到 Groovy、不引入外部插件、不改变现有模块语义。
   - 验收：完整模块图、源码导航、模型和增量 Sync 结果与当前实现一致；配置输入不再包含普通源文件下的 `build.gradle.kts` 探测。
   - 证据：[自动模块发现问题](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/local-native-20261003/REPORT.md#L76)。
   - 用户决策意见：根节点也使用 `moduleTree`，root tree 自动包含自身，nested tree 对应真实 build-script module 时包含自身，纯中间目录不额外生成 project；body 只写子树，叶子使用 `module(String)`。根声明的 Settings 扩展调整已单独批准，原调用形式和上述模型语义不变。

3. **统一并验证 Kotlin Gradle Plugin 版本**
   - 去掉 `buildSrc` 中独立写死的 KGP 版本来源，统一 catalog、convention/build logic 和各 fork 的实际 plugin 版本声明。
   - 以 Kotlin 2.4.20 + Gradle 9.6.1 作为 Kodex 专属兼容线目标；逐项记录实际 plugin `codeSource`，不能只依据 version catalog 判断版本已生效。
   - 验收：默认 Kotlin compiler daemon 与 in-process 两种策略分别完成完整 Sync、基础编译和模型等价；跨 fork 的 KGP 支持范围明确。
   - 证据：[版本依据与待补门禁](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/local-native-20261003/REPORT.md#L83)。
   - 用户决策意见：我不确定各fork的处理侧栏这样做是否合适，因为这个fork我未来可能还有用来提PR，一定要改fork的对应版本的话，建议使用kodex专有分支
   - 变更前调研已确认：root/MCP/Lucene 当前为 KGP 2.4.0，Mosaic 为 KGP 2.3.21；root composite 实际使用 root Gradle 9.5.1，不会使用 included build 自己的 wrapper。官方 fully-supported 上限分别为 9.5.0 与 9.3.0，候选升级不能绕过 Mosaic 的独立维护线。[版本矩阵与原始证据](../../shared-context/findings/2026-10-03-gradle-prechange-research.md)
   - 用户决策意见补充：采用 Kodex 专属兼容线，目标统一验证 KGP 2.4.20 + Gradle 9.6.1；Mosaic 在 `kodex-submodule` 上单独适配。若该兼容线验证失败，视为异常，立即中断迁移并回到用户决策，不自行降级、绕过或继续集成。

4. **将 Isolated Projects 降为本地发布路线之外的可选优化**
   - 既有研究显示 IP 会放大插件兼容、跨项目状态和内存风险；Mosaic 的 `allprojects`/jextract 兼容原型虽可继续研究，但不是稳定 fork 二进制消费的前置条件。
   - 主线先冻结 Windows/Linux/macOS Kotlin/Native 产品模型并减少不必要的模型输入；只有主线稳定后，用户再次确认投入时才运行 IP diagnostics。
   - 在完整模型、真实 IDEA Sync、编译和测试通过前，不使用 `dangerously-ignore`，也不把 Java composite canary 的缓存命中外推为 Kodex 已兼容。
   - 证据：[IP 的实际阻塞](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/local-native-20261003/REPORT.md#L46)。
   - 用户新决策：当前任务不以 IP 兼容为主线；先验证“clean fork 由 CI 发布 GitHub Packages、BuildKodex 消费固定 package、dirty fork 显式 source/local”的路线。

5. **先验证 Kotlin compiler daemon 与资源上限，再调整内存**
   - 复跑默认 daemon 策略、固定 workers 和统一 JDK 的控制组，区分编译器策略、KMP 模型构建和 Gradle heap 的贡献。
   - 不先盲目提高 `Xmx` 或 workers；记录 Gradle、Kotlin daemon、IDEA 和 Native 进程的 RSS/CPU，并设置停止条件。
   - 验收：完整 Sync 不发生 OOM，且资源峰值和热态耗时均有可复现改善；否则保留失败证据，不宣称“加堆优化”。
   - 证据：[OOM 的准确范围](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/local-native-20261003/REPORT.md#L37)。
   - 用户决策意见：没有问题，合格，进入plan
   - 用户决策意见补充：采用 Kotlin daemon-first，并设置 `kotlin.daemon.useFallbackStrategy=false`；daemon 启动或通信失败时直接暴露异常，不静默切换到 in-process。具体 `Xmx`、workers 和 IDEA heap 等实测后再定。

6. **将完整 Tooling 请求缓存作为独立验收，而非 fork 发布机制**
   - 保留 Gradle/IDE 原生 cache 的输入失效语义；不额外实现第二套模型缓存。fork 是否发布由受保护 CI、manifest、commit/tree 和 package target closure 决定，不把 Tooling cache 当作发布或 dirty 检测器。
   - 明确完整请求复用与“主构建改变时只复用未变 fork 的逐项目模型”不是同一能力；完整 Kodex KMP 和跨 IDE 进程复用仍需单独验收。
   - 验收：同一 IDE client 的完整请求、settings/源文件/模块变更失效、重开 IDE 后模型等价分别有证据。
   - 证据：[真 GUI 原生缓存 canary](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/local-native-20261003/REPORT.md#L54)。
   - 用户决策意见：只使用 Gradle/IDE 原生 cache；不启用内部 intermediate model cache，不实现自定义替代缓存。GitHub Packages 的 package identity 和 CI manifest 独立于 Tooling cache；dirty fork 不由 settings 自动判断或发布。
   - 验证应区分普通源文件编辑与设置、模块声明、build-logic、fork 构建脚本变化；不预设所有源码变化都会使 Tooling cache 失效，也不为提高命中率漏报配置输入。沿用原生任务完成真实 Sync、模型、导航及编译验收，决策通过不代表缓存已验收。

- **当前不作为默认方案**：GitHub Release archive、全局 `publishToMavenLocal()`、根据 fork SHA 自动切换 `includeBuild`；它们保留为 GitHub Packages 认证/发布完整性门禁失败后的备选，不应绕过本任务的失败门禁。

- **已确认的用户取舍**：三个 fork 仓库都建立同名的 `kodex-submodule` 分支作为 Kodex 集成维护线；公共 fork 的上游协作线不直接承载 Kodex 专属版本、Gradle 兼容或 API 修改。BuildKodex 后续 pin `kodex-submodule` 上的具体 commit，而不是依赖可移动的分支引用。三条维护线分别同步上游、分别记录差异和回合并路径；本决策只确定分支模型，尚未授权创建远端分支、推送或修改 fork 内容。

## Project-only Gradle optimization extension

- Fork publication is now a prerequisite/controlled input, not the whole performance solution. The remaining full binary Sync cost is primarily Kodex’s own model construction and IDE import.
- New planning task: [Kodex project-only Gradle optimization](2026-10-04-plan-kodex-gradle-optimization.md).
- This extension first freezes a fork-independent baseline, then investigates settings/module registration, build logic realization, the unchanged target model, dependency/variant resolution, compiler/resources, and native Gradle mechanisms one variable at a time.
- It must preserve the current project graph, source roots, KMP hierarchy, `expect/actual`, cinterop behavior, API/spec/impl boundaries, navigation, representative compile/test tasks, and package consumer behavior.
- Configuration cache, Tooling parallelism, and Isolated Projects are measured as distinct mechanisms. A CLI cache hit, `help` timing, or a faster JVM-only profile cannot be presented as full IDEA Sync improvement.
- The new task does not authorize production code, fork, gitlink, CI, package, or IDE changes; each candidate requires isolated validation and a user-reviewed rollout/rollback decision.

## GitHub Packages CI publication route — new canonical plan

- **默认行为**：BuildKodex 正常开发消费三个 fork 的 GitHub Packages binary，不在每次 IDEA Sync 中默认配置完整 source `includeBuild`。source composite 只作为明确的 source-debug 入口，供需要修改 fork 的开发者使用。
- **fork 身份**：每个 package 必须携带 fork 坐标、`kodex-submodule` commit、完整 tree hash、发布配方/toolchain、KMP metadata/POM/`.module` 以及实际 Windows/Linux/macOS Native variants 的 manifest；BuildKodex pin 具体 package version，不依赖可移动分支。
- **CI 触发边界**：只允许受保护的 `kodex-submodule` branch、受保护 tag 或手工 workflow dispatch 触发发布；普通 PR 和开发者本地 dirty 工作树不得自动写入远端 package。
- **发布结构**：Linux/macOS/Windows 的构建 job 先生成并校验 target artifacts，单一 publish job 再写入同一组 coordinates，避免 Maven metadata 被并发写成半完成状态。
- **不可变版本**：版本至少包含 fork commit identity；相同版本禁止覆盖。manifest、checksums、recipe/toolchain、target closure 和发布结果必须随 package 记录。
- **认证门禁**：GitHub Packages 的 Gradle/Maven registry 即使 package public 也需要认证；开发者需要 `read:packages`，CI 使用最小权限的 `GITHUB_TOKEN` 或明确的发布凭据。认证缺失时要给出可执行配置说明，不在 settings 中调用 `gh` 或启动 Gradle。
- **成本门禁**：公开仓库使用标准 GitHub-hosted runners 时 Actions 分钟免费；private 仓库、larger runners、package/artifact storage 和旧版本 retention 必须单独测量和控制，不能假设所有 CI 成本为零。
- **dirty 语义**：dirty fork 不自动发布 GitHub Packages；开发者显式选择 source `includeBuild` 或显式本地 publish。这样不会把个人实验版本污染远端，也不会让 settings 解释 Git 状态。
- **本地仓库边界**：普通开发不依赖 `~/.m2`；本地 repository 只作为显式 dirty/source workflow 的实现细节，未经另行决策不写全局 `mavenLocal()`。
- **跨平台门禁**：至少验证 JVM/Linux/macOS Native 的实际 consumer；Windows/mingw 由对应 host/CI gate 验收，不能用“能生成 metadata”替代真实 target artifact。
- **安全回滚**：回滚为恢复旧 package pin、禁用新 package route 或切换显式 source mode；不修改 gitlink、不重写已发布版本、不在 settings 中 silently fallback。
- 第一规划子任务：[设计 GitHub Packages CI fork 发布](../done/2026-10-04-plan-github-packages-fork-publication.md)；当时只定义 workflow、凭据、版本、target 和验收门禁；现有部署与发布结果以本文件 Current checkpoint 为准。
- 既有完整包和验证证据：[fork 发布实验](../done/2026-10-02-complete-fork-publication-experiments.md)；其 immutable file repository 结果可作为 package 内容和 manifest fixture，但不代表 GitHub Packages 已验收。

## Superseded settings-triggered local publication route — retained evidence

> 这条路线已被用户的新 GitHub Packages CI 决策取代。保留它用于解释为什么不在 settings 中执行 dirty 检查或嵌套 Gradle publish；相关 fixture 任务不再是当前执行入口。

- 原设计要求 settings 根据 artifact 缺失或 fork dirty 自动发布到项目本地 Maven repository。
- 研究结论：settings 处于初始化阶段，而 publish 属于后续 task execution；嵌套 Gradle 会引入递归、锁竞争、IDE Sync 重入、configuration cache 输入污染和半发布仓库风险。
- 如果未来重新考虑本地发布，必须由独立 preflight/CLI 负责，不得把 `git status`、dirty 版本计算和 publish action 放进普通 settings Sync。
- 旧任务：[settings 触发本地 fork 发布](../done/2026-10-04-validate-settings-triggered-local-fork-publication.md)；当前仅作失败案例记录，不进入本期默认执行树。

## buildSrc → build-logic 研究 — 已完成变更前原型，生产方案待审阅

- Gradle 官方建议把共享构建逻辑放到显式的 `build-logic` composite，而不是长期依赖 `buildSrc`；`buildSrc` 本质上也是 Gradle 自动管理的 included build。
- 这不是“少启动一次完整 Gradle”的开关：显式 `build-logic` 仍是 included build，会被独立配置；迁移本身可能增加一个 build 的 settings/plugin 配置边界。
- 可能的收益主要是失效范围和结构：`buildSrc` 任意改动会让整个构建的配置重新执行；拆成 `build-logic` 的独立子项目后，理论上可以只让真正消费变更产物的项目失效，也更容易独立测试、独立打开和按 convention plugin 拆分。
- 当前 Kodex 的 root `buildSrc` 只有 8 个源码文件、6 个 precompiled convention plugins，但它同时承载 KGP、Compose compiler、Koin、TestBalloon 等构建插件依赖；因此不能仅凭目录迁移宣称会降低 IDEA Sync 的 CPU/RSS。
- Xiaoxin 隔离原型已验证：直接搬目录后 `build-logic` 自身可以编译，但完整 root `help` 会因 KGP 被不同子项目重复加载而失败。当前 130 个 convention consumers 中有 36 个同时声明 convention plugin 和直接 Kotlin plugin。
- 原生修复是 root `build.gradle.kts` 顶层以 `apply false` 预解析 Kotlin Multiplatform、Serialization、Compose 三个插件，让 root/subprojects 复用同一插件 classpath；该 root-only 变体的完整 `help` 成功。它是兼容性前提，不是性能收益证据。
- 实验使用 Gradle 9.4.0 而非生产 wrapper 9.5.1；buildSrc 约 3:31、build-logic + root apply false 首次约 4:22，不能据此宣称 build-logic 更快。当前可交付判断是“可继续做维护性/失效范围原型”，不是“已证明降低 Sync”。
- 用户决策：现在立即迁移 root `buildSrc` 到显式 `build-logic`，把它作为当前变更主线中的独立批次；迁移不得隐藏 KGP classpath 统一、插件 ID、KMP target、source set 和任务语义的兼容性门禁。
- 研究任务：
  1. 盘点各 convention plugin 的消费者闭包、共享 classpath 和跨项目状态。
  2. 在隔离副本把 root `buildSrc` 等价迁移为 `build-logic`，保持插件 ID、KMP target、source set 和任务语义不变。
  3. 对比 buildSrc / build-logic 的冷/热配置、IDEA Sync、变更失效范围和 Daemon/IDEA RSS。
  4. 若收益只体现在增量失效而非 Sync，按维护性整改处理，不把它列为性能银弹。
- 官方依据：[Sharing build logic using buildSrc](https://docs.gradle.org/current/userguide/sharing_build_logic_between_subprojects.html)、[Favor build-logic composite builds](https://docs.gradle.org/current/userguide/best_practices_structuring_builds.html)、[Gradle performance: Settings and buildSrc](https://docs.gradle.org/current/userguide/inspect.html)。
- 变更前原型：[Gradle 变更前调研](../../shared-context/findings/2026-10-03-gradle-prechange-research.md)
- 用户决策意见：现在立即迁移；按独立批次执行，先验证 root `apply false` 与 build-logic classpath，再进入完整模型/IDEA Sync/编译验收。

## Superseded GitHub Release/archive fallback planning — retained historical evidence

> 本节保留 2026-10-02 的 GitHub Release、下载 bootstrap 和 binary onboarding 研究，作为 GitHub Packages 认证或发布完整性失败后的候选材料；它不是当前默认执行树。当前 canonical route 见 [GitHub Packages CI publication route](#github-packages-ci-publication-route--new-canonical-plan)。

### Confirmed requirements and proposals

- 需求：普通开发者不应先学习发布流程或 Gradle 模式切换；默认消费固定 fork，修改依赖时才显式启用对应 source build。
- GitHub 是用户暂定托管平台；**具体使用 Releases、公开可见性、仓库、首次自动下载实现及发布权限均为待审阅方案**。
- 建议 GitHub Releases 承载一个完整 Maven 仓库归档，客户端校验后作为本地文件仓库消费。Release URL 不是 Maven endpoint；不直接把附件地址写进 `maven { url = ... }`。
- GitHub Packages Maven/Gradle 下载公开包也要求认证，故不作为无 token 新人体验的默认入口：[官方认证要求](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-gradle-registry)。
- 暂定发布目标 `Stream29/Kodex`，独立 `forks/<bundle-id>` tag，不替换 CLI 的 latest；2026-10-02 只读 API 确认该仓库公开。若用户改选专门仓库，只修改托管地址，不改变消费坐标和 pin 协议。
- 固定 fork Git SHA + 发布配方/工具链 + target 矩阵决定新发布版本；开发者明确选择决定 `includeBuild`。不依据 dirty/clean 自动改变 IDE 模型，不追踪 `latest`。

### Implementation children and dependency boundaries

- `Publish immutable fork artifacts on GitHub`：[发布与维护流程](../planning/2026-10-02-plan-github-fork-artifact-publication.md)；产出已验证 Release、归档、manifest/pin 和可复现 recipe。
- `Implement transparent binary consumption and explicit source mode` / `Verify anonymous first-clone onboarding`：[新人入口与依赖接入](../planning/2026-10-02-plan-binary-fork-onboarding.md)；本地 fixtures 可先开发，公网验收依赖完整已发布包。
- `Optimize target models and diagnose resource retention`：[模型、资源与真实 Sync](2026-10-02-plan-gradle-model-resource-optimization.md)；先独立 baseline，binary 接入后单变量联测。
- 三研究方向仍属本期；上述 binary 发布/接入实施暂停，模型/资源方向优先参与原生复核。备选重新授权后才可派发；重型实验串行持锁，共用文件顺序集成。

### Fallback integration batches — only after renewed approval

| Batch | 改动与交付 | 进入下一批的门禁 |
| --- | --- | --- |
| 0 | 授权后冻结当前分支及三个 gitlinks，盘点项目、ABI/序列化/RPC fixtures、target/源集与真实 GUI baseline | 当前 commit、依赖闭包和平台范围明确；不直接复制研究 main 的旧路径 |
| 1 | 验证发布配方，组装完整 fork Release；新版本审计、无源码消费、下载回验 | 所有实际消费 variants 完整；匿名可下载；pin 匹配；不交付 staging |
| 2 | 默认 binary、逐库 source、settings 轻量 bootstrap、repository 路由和首次克隆文档 | CLI/IDE 同配置；未初始化 fork 也成功；缺件/断网明确失败而不偷偷编译 |
| 3 | 固定现有全部 targets/source sets，核对开发、CI与发布模型一致 | 不引入目标开关或裁剪模式；完整模型、生成符号和发布闭包保持 |
| 4 | project-local kRPC convention；按数据小步处理 build-logic/扫描/插件与内存 | 每项独立 A/B 和回滚；不把维护性收益冒充提速，不预设 IP/DCL 升级 |
| 5 | 当前分支集成回归、真实 GUI 冷/热/增量/稳态资源测量、跨 host CLI smoke | 新人流程与功能/模型一致性通过；残余内存问题有定位证据和明确处置 |

### Acceptance and rollback

- 新人验收：Linux/macOS/Windows 的独立空缓存环境，无 GitHub token/gh/Python、不初始化三个 fork，配置正常项目 JDK 后直接 IDEA 打开；Gradle CLI 走同一入口。缺其他必要非 fork 子模块须明确记录。
- 初次允许一次有进度、超时和取消能力的发布包下载；重复 Sync 无下载/发布/fork Gradle，也不每次扫描完整文件包。下载时间与模型导入时间分别报告。
- 最新只读观测：`Kodex/` 为 `refactor/spec`、HEAD `7310a4452db4ea45539581b155332dbe23f10387`；这是观察值，不是已批准冻结基线。新执行前再次确认分支、工作树与协作状态。
- 新基线重新验证完整模型、内部边和源码导航等价；所有现有 targets/source sets 保留。保留所有有意义 spec/impl、KDoc、RPC/持久化协议与 API 导出。
- JVM 25、Desktop 若涉及则 JBR 25；MCP publisher 的 JDK 21 与应用项目 JVM 要求分开。跨平台矩阵与未测 gates 见子任务，不以旧 main 的 smoke 代替当前分支回归。
- 回滚采用 reviewable 独立补丁：恢复默认 source 配置/原 catalog；kRPC/build-logic 单独恢复，目标模型始终不变。已发布包/tag 不覆写，旧 pin 可重复使用。
- 发布、源码生产变更、合并/提交/推送和仓库全局设置更改均需明确授权；planning 文档存在本身不授权执行。

### Review focus

- 是否采用公开 Release 归档作为本期默认托管；发布到 `Stream29/Kodex` 还是独立仓库。
- 是否接受 IDE 首次 Sync 自动安装**固定发布物**，而不是额外强制 setup 命令；实现必须先通过跨平台/缓存兼容验收。
- 用户已否决目标开关与开发裁剪模式；现有 targets 已削减过，不再调整。
- linuxArm64/mingwX64 真实 runtime 和 IDE retained heap 诊断的资源/授权，不能以编译成功或 profile 提速替代。

## Original CLI baseline — historical

- 已完成报告：[REPORT.md](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/REPORT.md#L1)，同目录 `results/` 保存原始证据。
- Xiaoxin 可复现基线：`~/ACodeSpace/demo/kodex-gradle-research-445`；报告、source/binary archives、consumer、Maven repo、测量脚本和 Gradle cache 均保留。
- 根仓库 main：`3008a6de9f178cda7cd63123a0261d75013f772d`；fork gitlinks 与该提交一致。
- Gradle 9.5.1；Daemon SDKMAN Temurin 25.0.4；MCP toolchain Temurin 21。测量为 Xmx2g、2 workers、Kotlin compiler in-process、各组新 Daemon 后热态重复。
- 根 `help` 中位数 source 6.06s / 移除 included builds 4.09s；该 binary 组只是配置成本代理，没有完整接入三个 fork artifact。
- MCP 小型 JVM consumer 真实 binary/source 对照中位数 0.60s / 1.63s；源码移开后 binary 仍可运行 fork-only API。
- 历史 CLI source 成功对照使用实验性 MCP 根名称绕过重复 `getKotlinSdk()` accessor；原始失败保留。2026-10-02 的后续 GUI 和基础 JVM 编译未复现；2026-10-03 本机冷态 GUI 再次复现，见当前门禁，不以历史成功判断命名已根治。
- 原样 source 的 `help` 额外配置 Mosaic/MCP；Lucene 的显式替换使其项目在本轮 help 未配置。CLI 结果不等同于 IDEA Sync。
- 移除 included builds 后仍有 142 个主仓库子项目。9.5.1 IP 探针失败，终端约 747 为近似归并口径；后续 HTML 分类使用自带 2,047 去重行，均不是独立缺陷数。
- 先前 SSH 故障为 Tailnet ACL 所致，不是 Gradle 压垮设备；网络已恢复，不再修改 Tailscale。
- 本轮原实验 Daemon 已停止，源码已恢复。新任务使用各自子目录，不覆写上述基线及原始证据。

## Resource coordination

- Xiaoxin 上共享重型操作锁：`~/ACodeSpace/demo/kodex-gradle-research-445/device-heavy.lock`。
- Gradle、Kotlin/Native 编译、IDE 安装/启动/import/index 与性能测量必须先通过 `flock` 原子获取该锁；不要仅检查文件是否存在。
- 保持锁直到任务自己的 Daemon、IDE backend、采样进程等停止，资源回到可比较的基线；不要后台化重型任务后释放锁。
- 禁止并发重型操作污染测量；锁忙时做只读分析或明确等待，不抢占或停止其他任务进程。
- 锁仅协调本期研究，不替代检查用户其他工作、系统内存、磁盘和当前设备使用状态。
- Gradle 调用前检测可复用 Daemon JVM，显式传入；不因无法复用而另起第二个 Daemon 抢资源。遵守 Gradle/device/IDE 相关 skills。
- `/tmp` 为 tmpfs，实验源码/cache/artifact 必须放持久磁盘；各自使用独立输出目录和可识别的采样文件。
- 每组记录 commit、Gradle/JDK/IDE/plugin 版本、cache/网络状态、Daemon、workers、堆、任务和 model 参数；失败准备耗时与成功测量分开。

## Completion gates

- 三个子任务均有可复现证据、失败/平台边界、资源清理记录和可执行后续方案。
- GitHub Packages CI 必须单独通过：clean commit 发布不可变 package，target closure/metadata/checksum 完整，single-writer 不把部分上传宣称完成；按用户确认策略失败封存、人工审计，不删除或盲目续传。开发者认证和首次 clone 可复现。
- 真正 Sync 测量必须使用实际 IDE import；Tooling API/headless 模型代理只能作为补充，不冒充 Sync。
- 完整发布物、manifest、target variants、认证配置或 package identity 缺失时不能称三个 fork 已可生产迁移；缺失的 host/target/依赖矩阵逐项列出并解决或提请用户提供资源。
- 默认 package mode、显式 source-debug、首次克隆认证、dirty fork、package 缺失/损坏、CI 取消和并发 Sync 都有独立验收；不能用 `help` 成功代替发布消费验证。
- 汇总发布物、source opt-in、固定完整模型与构建逻辑整改的收益、维护成本及兼容性；用户审阅后另行授权生产变更。
- 三研究子任务完成不等于生产变更验收；总任务已获用户生产授权并进入 executable，已落地批次分别归档。关闭时仍须区分研究、发布、生产变更与剩余 gates。

## Current genuine GUI stop boundary

- 当前完整项目 GUI 基线与工具链候选均未通过初始原生导入：基线报告
  Java heap space；候选 daemon 在 KGP IDE metadata 解析/依赖解析中记录 OOM，
  IDEA 同时收到日志 Throwable 反序列化的 UTFDataFormatException。
  [确切失败与资源记录](2026-10-08-validate-kodex-toolchain-compatibility.md)。
- 保留失败现场，不宣称三次热导入、导航或当前项目 IDE 收益通过。
  生产保留5b41be52的已验收包消费和模块树，工具链仍2.4.0/9.5.1；
  版本、fork 统一与 build-logic 集成暂停，不自动扩大默认堆。
- 同4GB、无额外模型探针的真实导入仍 OOM；随后按用户授权取得私有堆，
  已定位到配置/解析和源集保留图的主要占用，但未证明唯一根因。
  用户已否决 JVM 开发 profile，现有 targets 不再调整；
  后续只在完整模型不变的前提下处理保留问题，堆和原始报告不公开上传。

## Confirmed target boundary — 2026-10-08

- 用户明确：“我不要这个开关，我们的target是已经削减过一轮的，不要再动了。”
- 撤回 `kodex.targets`、full/JVM 开发模式和 host-only 裁剪候选；
  不以按 OS 隐式选择、禁用任务或隐藏源集变相实现。现有全部 targets、
  hierarchy、expect/actual、cinterop、导航和发布语义作为固定验收输入。
- 历史研究中的 JVM-only 数据保留为历史证据，不是后续执行方案。
  配置/依赖解析保留、按需配置、build-logic 与工具链门槛继续分别处理，
  不用减少平台模型换取通过，也不自动扩大生产默认堆。
