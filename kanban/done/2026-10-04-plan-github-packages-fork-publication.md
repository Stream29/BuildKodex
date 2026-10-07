# Task Tree

- `Confirm the three fork repositories, kodex-submodule ancestry, and exact package consumers`()
- `Freeze coordinates, commit/tree identity, target closure, and publication manifest schema`()
- `Design protected kodex-submodule CI workflows with standard runner and credential boundaries`()
- `Design the multi-host build matrix and single-writer package publication`()
- `Define immutable versioning, duplicate-version rejection, checksums, and retention`()
- `Define developer authentication and first-clone Gradle repository configuration`()
- `Define explicit dirty-fork source mode and optional local publish escape hatch`()
- `Validate the package contract in isolated fixtures before authorizing workflow changes`()
- `Record the user's production deployment authorization`()
- `Implement and validate three independent single-writer publication workflows`()
- `Deploy approved workflows and publish the pinned clean fork versions`()
- `Verify remote bytes and complete default package-consumer acceptance`()

# Details

## Scope and current decision

- Current authorization: 用户已要求生产落地，并明确批准本批分批提交/推送、
  三个 `kodex-submodule` 维护线、中央 CI 部署及固定 fork 包发布；
  不发布新版 Kodex。下文原 planning 限制保留为历史，不覆盖当前授权。
- First production baseline: Kodex `83749434`；当前三个 gitlink 不变，
  已有 upstream 协作分支保持原样。不能先切到未发布的版本使 main 无法构建。
- [发布实施工作线](2026-10-07-implement-fork-package-ci.md)负责 workflow、
  发布配方和完整性验证；协调者负责权限、提交/推送、触发及 consumer 接入。
- 新包读取凭据仅配置用户指定四台机器；仅 `read:packages`、
  私有用户级配置，到期 2027-01-05。CI 发布仍使用仓库 `GITHUB_TOKEN`，
  不使用该只读 PAT 写包。
- 当前源码额外确认 MCP stdio/HTTP 的 `kodex.kmp-host` JS 目标真实依赖
  SDK JS；用户已批准 **仅 MCP SDK 额外发布 JS 变体**，保留 Node 编译与测试。
  因此首批 SDK 闭包为 JVM/JS/四 Native，Mosaic/Lucene 仍为 JVM/四 Native；
  不扩大到 Wasm/iOS。本文旧的统一“排除 JS”对 SDK 不再适用。

- Parent: [Gradle development-experience planning](../executable/2026-10-02-rescue-gradle-development-experience.md).
- 用户已决定：clean、可复现的 fork artifact 由 `Stream29/Kodex` 仓库的 GitHub Actions 发布到 GitHub Packages；BuildKodex/Kodex 默认消费固定 package。
- dirty fork 不自动发布远端 package；需要修改 fork 的开发者显式选择 source `includeBuild`，或显式执行本地 publish workflow。
- 本文只做 planning，不创建 workflow、分支、package、tag、release，不修改 gitlink，不上传 artifact。
- 旧的 GitHub Release archive、settings-triggered local publish 和全局 `mavenLocal()` 方案保留为后备/历史证据，不是本任务默认路线。

## Repository and branch model

- Repository boundary is explicit: the planning/workspace repository is `Stream29/BuildKodex`, while the Gradle product repository is the `Kodex` submodule at `Stream29/Kodex`; the three fork gitlinks are nested inside `Stream29/Kodex`.
- CI checkout uses GitHub's built-in workflow token and `actions/checkout` behavior for public nested submodules; no deploy key or additional cross-repository secret is opened. The workflow must explicitly verify SSH-to-HTTPS checkout conversion and recursive nested gitlink resolution.
- 三个 fork 都使用 `kodex-submodule` 作为 Kodex 集成维护线；上游主分支/开发分支保持独立，用于同步和准备 upstream PR。
- BuildKodex/Kodex 消费具体 package version 和 commit identity，不消费可移动分支。
- 用户决策：由 `Stream29/Kodex` 的 CI 负责发布；当其 nested fork submodule gitlink 更新时触发一次 package pipeline，package version 必须对应被 pin 的 fork hash。
- 用户进一步明确时序：先完成 `Stream29/Kodex` 主分支的 nested submodule gitlink 更新；该更新触发并完成 package 发布；之后再单独完成主分支的依赖版本更新。
- 用户决策：为三个 fork 分别维护一套独立 CI；每套 workflow 只负责对应 fork package，允许内容重复，不做三方 bundle 编排。
- 用户决策：如果对应 package CI 失败，保留已合并 gitlink，暂不更新依赖版本；默认 binary 继续消费旧 package，直到 CI 修复并由维护者手工完成依赖更新。
- CI 只响应 `Stream29/Kodex` 中受审阅的 nested gitlink 更新、受保护 tag 或手工 dispatch；普通 PR 不具备远端 package 写入路径。
- 每个 fork 的 package coordinates、repository owner、package name、版本前缀和维护者责任必须在 manifest 中唯一声明。
- 用户决策：三个 package 集中归属 `Stream29/Kodex`，由该仓库 CI 使用 GITHUB_TOKEN 单写入；package coordinates 保持各 fork 上游 group/artifact。
- 用户决策：不新增独立 fork lock 来重复保存 hash；package version 已包含 hash，BuildKodex 的现有依赖版本声明作为消费端唯一 pin。CI manifest 保存完整 hash 和 target 证据，不作为开发者维护的第二个版本来源。

## Publication contract

- 每个 package 必须包含 KMP root metadata、POM、Gradle Module Metadata、sources attachments（若保留）和实际需要的 JVM/Linux/macOS/Windows Native variants。
- 用户决策：采用产品目标闭包优先；默认 package 不发布 JS/Wasm 或非产品 Native targets，只保留实际 Windows/Linux/macOS 产品闭包，以及被真实 consumer 需要的 JVM variant。
- 具体 architecture/host 列表必须在 manifest 审计中冻结；不能用“产品闭包”作为跳过真实 host/Native artifact 验证的理由。
- 用户进一步确定首批 architecture matrix：`linuxX64`、`linuxArm64`、`macosArm64`、`mingwX64`，另加真实 consumer 必需的 JVM；`macosX64` 暂缓。
- 用户决策：允许用交叉编译构建 `linuxArm64`，不强制原生 ARM runner；涉及 Apple binary、Apple SDK/cinterop、link 或 sign 的任务必须单独在 macOS host 上验收，不能用交叉编译结果冒充 Apple runtime 证据。
- 每个 package manifest 记录 fork repository、branch、commit、full tree hash、Gradle/KGP/JDK/toolchain、target closure、publication inputs、checksums 和验证结果。
- 版本必须不可变，至少包含 commit identity；相同版本禁止覆盖。BuildKodex 只 pin 已经完成完整验证的版本。
- 用户决策：每个 package 版本采用各自上游版本加 `-kodex.<short-sha>`；manifest 额外保存完整 40 位 fork hash，版本字符串不重复覆盖。
- 用户决策：保持三个 fork 的上游 group/artifact coordinates 不变，只通过 `-kodex.<short-sha>` 版本后缀区分 package；source composite 与 binary consumer 共享依赖坐标。
- 每套 fork workflow 内部的 Native matrix 先分别构建并上传临时 CI artifacts；该 fork 的单一 publish job 校验闭包后写入自己的 package coordinates，避免多个 job 同时修改同一 Maven metadata。
- package 进入可消费状态前必须验证 metadata、`.module`、POM、每个实际 target artifact、checksum 和一个真实 consumer task。

## GitHub Actions cost and runner boundary

- 公开仓库使用标准 GitHub-hosted runners 时 Actions 分钟免费且不限；larger runners 不在该保证内。
- private repository 使用账户计划额度；GitHub Free/Pro/Team/Enterprise 的标准分钟额度和超额计费必须在实施前记录实际账户归属。
- GitHub Packages 与 Actions artifact storage、旧版本保留和清理策略需要独立盘点；不能因为公开仓库标准 runner 免费，就假设 package storage 和所有 runner 类型都免费。
- 默认矩阵只使用标准 Linux/macOS/Windows runners；任何 larger/self-hosted runner 需要用户单独授权和预算。
- CI workflow 要设置 concurrency、超时、取消策略和失败清理，避免重复发布占用 runner 或留下不可消费 package。
- 同一 fork/hash 的重复触发：已有完整且字节相同的 package 时校验并跳过；
  部分发布、checksum 不完整或字节不同则失败封存，不覆盖、删除或盲目续传。
  用户本轮明确选择失败封存人工处理；旧依赖不变，保存产物与失败证据，
  精确版本归属、删除权限和竞态另行审计并授权后再处理。
- 每套 workflow 只监听对应 nested gitlink；必须先校验 submodule commit、`kodex-submodule` ancestry、manifest 和 target closure，再进入该 fork 发布，不能只根据 gitlink 文本变更而盲目发布。
- `linuxArm64` 可以在 x64 host 上走已验证的 Kotlin/Native cross-compile；Apple binary/cinterop/link/sign 保留 macOS-host gate，library metadata/klib 交叉编译与真实 Apple binary 验收必须分开记录。
- Workflow permissions are limited to `contents: read` and `packages: write`; package publication uses the repository-scoped `GITHUB_TOKEN`, while public nested fork checkout uses the same workflow checkout path rather than a deploy key.

## Authentication and onboarding

- GitHub Packages 的 Gradle/Maven registry 即使 package public 也需要认证；开发者需要 classic PAT 的 `read:packages`，CI 使用关联仓库的最小权限 `GITHUB_TOKEN` 或另行批准的发布凭据。
- 不把 token 写入仓库、Gradle lock、manifest、日志或 package；Gradle credentials 通过用户级安全配置或环境变量注入。
- 首次克隆文档必须说明：PAT 创建/权限、Gradle property/env 名称、失败诊断、撤销 token 和 CI 与本地权限的区别。
- 如果团队无法接受 classic PAT 或 package 认证门槛，任务必须回到用户重新选择 Release archive、另一个 Maven registry 或显式本地 bootstrap。
- 用户决策：接受一次性 classic PAT 配置，只授予 `read:packages`；后续由 Gradle 从用户级配置或环境变量读取，项目不保存 token。

## Dirty fork policy

- settings 不执行 `git status`→publish，也不启动 nested Gradle；普通 IDEA Sync 不解释开发者 fork 的 dirty 状态。
- dirty fork 默认进入显式 source-debug mode；需要 binary 测试时使用明确的本地 publish 命令和项目隔离 repository。
- dirty local versions 不得上传 GitHub Packages，不得覆盖 clean package，也不得改变 BuildKodex 默认 pin。
- 工作树恢复 clean 后，BuildKodex 仍只解析锁定的远端 package；本地 dirty repository 不得隐式继续生效。
- 用户决策：dirty fork 暂不设计自动 source profile；维护者需要调 fork 时临时手工修改 `includeBuild`，完成后恢复 binary 配置。

## Default binary consumer gate

- 当前 `Stream29/Kodex/settings.gradle.kts` 仍无条件 `includeBuild("Mosaic")`、`includeBuild("LuceneKmp")` 和 `includeBuild("KotlinMcpSdk")`；这些必须从默认 binary path 移除或变为仅 source-debug 手工修改。
- 当前主 settings 的 dependency repositories 只有 Google/Maven Central；必须增加 `Stream29/Kodex` 的 GitHub Packages Maven endpoint，并用用户级 PAT credentials 解析 package。
- Lucene 当前存在显式 project substitution；binary mode 必须验证它不会继续把上游坐标劫持到不存在的 source project。Mosaic/MCP 也必须分别验证 package coordinates、传递依赖和 KMP metadata。
- `repositoriesMode = PREFER_PROJECT` 必须在真实 consumer 上验证；如果某个 convention/project repository 覆盖 settings repository，必须改成可审计的 repository policy，不能靠“依赖恰好能解析”通过。
- 只有默认 package consumer 验证完成后，才进入 root name、moduleTree 和 build-logic 变更批次。

## Maven Local consumer evidence

- 2026-10-04 在 Xiaoxin Ubuntu 的 exact main `3008a6de9f178cda7cd63123a0261d75013f772d` 隔离副本中，移除三个默认 source `includeBuild` 和 Lucene substitution，增加隔离 `mavenLocal()`，消费已有 fixture package。
- Mosaic、LuceneKmp、KotlinMcpSdk 的三组上游 coordinates 均解析到 fixture binary；`dependencyInsight` 未出现 source project substitution；`:mcp-stdio:compileKotlinJvm` 成功。
- binary consumer 的首次 `help` 为 `186.23s`，包含 Gradle wrapper 下载和 `buildSrc` 首次编译，不能当稳态对照；配置缓存复用的 `help` 为 `3.01s`。弱 Linux 主机仍配置大量 Kodex 主项目，并报告 macOS cinterop cross-compilation warning，因此移除 fork composite 并不等于已经移除主仓库模型成本。
- 证据目录：`~/ACodeSpace/demo/kodex-gradle-research-445/package-consumer-mavenlocal-20261004/`；报告为 `REPORT.md`，原始日志和 wall/RSS 为 `results/binary-mavenlocal-*`。
- 该验证只证明 Maven Local binary consumer 路由和 JVM 编译可行；最终 `-kodex.<short-sha>` 版本、GitHub Packages authentication/CI、完整 Native closure 和 IDEA Sync 仍未验收。

## Acceptance and rollback

- 新人验收：有 GitHub account、配置 `read:packages` 后，Linux/macOS/Windows 能直接执行 CLI 和 IDEA Sync；没有 token 时要得到清晰失败而非递归或长时间等待。
- CI 验收：三 fork 至少各有一次受保护分支/手工发布演练；重复版本拒绝；失败 job 不留下可消费的半包；manifest/checksum 可复验。
- Consumer 验收：默认 package 模式保留项目模型、源码导航、编译/测试和 Windows/Linux/macOS Native target 语义；显式 source mode 可定位到 fork 源码。
- 回滚：恢复旧 package pin、禁用新 repository route、删除未采用的 package versions 或切换显式 source mode；不重写已发布版本。
- package 发布与依赖更新必须是两个可审阅步骤；依赖更新只能在 package manifest/consumer 验证完成后发生，不能由未完成的发布 job 直接改写主分支。
- 当前 planning 完成不授权创建 CI、远端分支、package、tag、release 或修改 BuildKodex 依赖。

## Official references

- [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions)
- [GitHub-hosted runners](https://docs.github.com/en/actions/reference/runners/github-hosted-runners)
- [GitHub Packages permissions](https://docs.github.com/en/packages/learn-github-packages/about-permissions-for-github-packages)
- [Working with the Gradle registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-gradle-registry)
- [actions/checkout](https://github.com/actions/checkout)

## Completed first production rollout

- Three workflows are deployed; SDK453 and Lucene133 original uploaded files
  were independently read-audited byte-exact, preserving their failed writer
  statuses. Mosaic's reviewed new-pin run37678581820 succeeded and verified803
  immutable files after real three-host/JNI21/FFM25/Native gates.
- Default consumer is now signed main `c0ba86cc`, separately reviewed and
  [accepted on Xiaoxin](2026-10-07-consume-verified-fork-packages-by-default.md).
  Ordinary authenticated resolution, exact binary/source hashes, 132 tests,
  Linux release CLI and isolated PTY pass; both negative inputs fail honestly.
- Historical “current settings,” planning-only and automatic-cleanup statements
  above describe their earlier stage; current approved policy is fail-sealed
  manual remediation, no delete/overwrite/blind resume.
- This closes first publication and consumer wiring, not full cross-host Kodex
  CLI/IDE acceptance. Those remain in the
  [parent production task](../executable/2026-10-02-rescue-gradle-development-experience.md)
  and [model/resource gates](../executable/2026-10-02-plan-gradle-model-resource-optimization.md).
  The newly approved SDK source-root maintenance uses a new commit/version and
  separate publication, never rewrites these completed package identities.
