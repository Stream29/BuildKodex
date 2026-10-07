# GitHub Packages CI fork route

## Decision

- 用户在 2026-10-04 将 fork binary 路线改为 GitHub Packages CI：三个 fork 的 `kodex-submodule` 分支发布 clean、可复现、不可变 package；BuildKodex/Kodex 默认消费固定 package version。
- 用户进一步决定由 `Stream29/Kodex` CI 作为发布编排入口：其 nested fork submodule gitlink 更新触发 package pipeline，package version 对应被 pin 的 fork hash。
- 发布时序为两个主分支步骤：先完成 gitlink 更新并等待 package 发布/验证，再完成依赖版本更新；不让未完成的 CI job 直接把半成品依赖写入主分支。
- 三个 fork 各自使用一套独立 CI workflow；每套只发布对应 fork package，允许内容重复，不发布三方 bundle。
- package CI 失败时保留 gitlink、继续 pin 旧 package；修复并验证 CI 后再由维护者手工更新依赖版本。
- 三个 package 集中归属 `Stream29/Kodex`，由中央 CI 使用 GITHUB_TOKEN 单写入；coordinates 保持上游 group/artifact，版本使用 `-kodex.<short-sha>`。
- package version 采用各 fork 上游版本加 `-kodex.<short-sha>`；manifest 保存完整 40 位 hash，避免版本覆盖并保留可追溯性。
- 不新增独立 fork lock 重复保存 hash；BuildKodex 依赖版本声明是唯一消费 pin，CI manifest 只承担完整 hash、target closure 和发布验证记录。
- 三个 fork package 保持上游 group/artifact coordinates，只用 `-kodex.<short-sha>` 版本后缀区分，避免 source/binary 切换时改写依赖图。
- package target 采用产品闭包优先：排除 JS/Wasm 和非产品 Native targets，保留 Windows/Linux/macOS 实际闭包及真实 consumer 需要的 JVM variant；具体 architecture/host 需由 manifest 审计冻结。
- 首批 architecture matrix 已确定为 `linuxX64`、`linuxArm64`、`macosArm64`、`mingwX64`，另加真实 consumer 必需的 JVM；`macosX64` 暂缓。
- Native CI 允许交叉编译 `linuxArm64`；Apple binary、Apple SDK/cinterop、link/sign 仍必须使用 macOS host gate，不能把 library cross-compile 结果当作 Apple runtime 验收。
- dirty fork 不自动发布到 GitHub Packages；开发者显式选择 source `includeBuild`，或显式执行本地 publish workflow。
- dirty fork 暂不引入自动 source profile；维护者临时手工修改 `includeBuild`，完成 fork 调试后恢复 binary 配置。
- settings 不检查 dirty，不启动 nested Gradle，不负责远程发布。
- Isolated Projects 仍是可选后续优化，不阻塞 package consumer 主线。

## CI shape

- Repository boundary: `Stream29/BuildKodex` is the planning/workspace wrapper; the Gradle product and nested fork gitlinks live in `Stream29/Kodex`. The package workflow belongs to the latter.
- CI uses the built-in workflow token and `actions/checkout` for public nested submodules; no deploy key or additional cross-repository secret is required. The workflow must verify recursive checkout and SSH-to-HTTPS conversion.
- 只允许受保护分支、受保护 tag 或手工 dispatch 发布；普通 PR 不写远程 package。
- Linux/macOS/Windows matrix 先构建并校验临时 CI artifacts，单一 publish job 再写同一组 Maven coordinates。
- package manifest 至少包含 fork commit/tree、Gradle/KGP/JDK/toolchain、target closure、POM、`.module`、checksums 和验证结果。
- 版本不可变，包含 commit identity；相同版本禁止覆盖。BuildKodex 只 pin 已完成完整 consumer 验证的版本。
- Workflow permissions are limited to `contents: read` and `packages: write`; the central Kodex workflow uses repository-scoped `GITHUB_TOKEN` for package publication.

## Default consumer blocker

- `Stream29/Kodex/settings.gradle.kts` currently unconditionally includes Mosaic, LuceneKmp and KotlinMcpSdk source builds, and only declares Google/Maven Central. Package mode must remove/disable those default composites and add the central Kodex GitHub Packages Maven endpoint.
- Lucene's explicit project substitution and `repositoriesMode = PREFER_PROJECT` require real binary-consumer validation; neither may silently route package dependencies back to source or hide the package repository.

## Maven Local binary-consumer evidence

- 在 Xiaoxin Ubuntu 的 exact main `3008a6de9f178cda7cd63123a0261d75013f772d` 隔离副本中，已移除三个 source `includeBuild`、Lucene substitution，并通过隔离 `mavenLocal()` 消费已有 fixture artifacts。
- Mosaic、LuceneKmp、KotlinMcpSdk 的 dependency insight 均解析到相应 binary version；`:mcp-stdio:compileKotlinJvm` 成功。首次 help `186.23s` 包含 wrapper 下载和 buildSrc 初次编译，warm configuration-cache help `3.01s`。
- 该结果证明 binary consumer 路由可行，但不证明最终 `-kodex.<short-sha>`、GitHub Packages CI/auth、完整 Native matrix 或 IDEA Sync；弱 Linux 主机仍会配置大量 Kodex 主项目并发出 macOS cinterop warnings。
- 原始报告和日志保存在远端 `~/ACodeSpace/demo/kodex-gradle-research-445/package-consumer-mavenlocal-20261004/REPORT.md` 与 `results/binary-mavenlocal-*`。

## Cost and authentication

- 公开仓库的标准 GitHub-hosted runners 当前免费且不限 Actions 分钟；larger runners、private repository quota、package/artifact storage 和 retention 仍需单独控制。
- GitHub Packages 的 Gradle/Maven registry 即使 package public 也要求认证；开发者通常需要 classic PAT 的 `read:packages`，CI 使用关联仓库的最小权限 `GITHUB_TOKEN` 或已批准的发布凭据。
- 如果团队不接受 PAT/认证配置，GitHub Packages 不应作为默认新人入口，应回到 Release archive 或其他公开 Maven registry。
- 用户已接受一次性 classic PAT 配置；后续由用户级 Gradle 配置或环境变量提供 token，项目和 package manifest 不保存 token。

## Evidence and references

- [GitHub Packages CI planning](../../kanban/done/2026-10-04-plan-github-packages-fork-publication.md)
- [Canonical Gradle planning](../../kanban/executable/2026-10-02-rescue-gradle-development-experience.md)
- [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions)
- [GitHub Packages permissions](https://docs.github.com/en/packages/learn-github-packages/about-permissions-for-github-packages)
- [Gradle registry authentication](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-gradle-registry)
- [actions/checkout](https://github.com/actions/checkout)
