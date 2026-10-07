# Task Tree

> **Superseded route:** retained as historical GitHub Release research only. The current canonical route is [GitHub Packages CI fork publication](../done/2026-10-04-plan-github-packages-fork-publication.md); do not execute this plan unless that route fails and the user re-authorizes an archive-based fallback.

- **`Await native Gradle evidence and renewed fallback approval`()**
- `Review GitHub delivery contract and publication authority`()
- `Freeze fork commits, recipe inputs, and required target closure`()
- `Implement reproducible staged publication and bundle verification`()
- `Verify no-source consumers and publication maintenance cases`()
- `Await external publication and repository-setting authorization`()
- `Create draft release and upload the complete verified asset set`()
- `Publish without moving tags or replacing existing versions`()
- `Verify anonymous download and hand the immutable pin to consumers`()
- `Record the maintainer procedure and release evidence`()

# Details

## Scope and dependencies

- Parent：[Gradle 开发体验实施规划](../executable/2026-10-02-rescue-gradle-development-experience.md)；当前仅 planning，尚未授权源码/CI 修改、上传、tag/push 或仓库设置变更。
- 当前仍为备选：主线已选择 GitHub Packages，不实施 GitHub Release archive 安装/分发；[延期原生研究](2026-10-02-verify-native-gradle-sync-mechanisms.md) 不再作为 Packages 前置条件。
- 研究输入：[发布实验已验收](../done/2026-10-02-complete-fork-publication-experiments.md)、[canonical 证据](../../shared-context/findings/2026-10-02-gradle-development-experience-research.md)。
- 交接消费者：[binary/source 与新人入口](2026-10-02-plan-binary-fork-onboarding.md)；只交付完整、回验通过的 package，不交付 staging/r2 成本样本。

## Proposed GitHub contract

- 暂定公开 `Stream29/Kodex` 的 Release，tag 为 `forks/<bundle-id>`；明确不设为 CLI latest，不使用 `latest/download`。托管 repo 和公开发布许可由用户审阅后确定。
- 初期一个 bundle 包含三个 fork 的完整消费闭包，内部保留 Maven 目录结构；一库变化可复用另两库已验收 artifact，但需重新组装、审计和消费验证新 bundle，不要求重编所有库。
- Assets：Maven repo 归档、manifest、checksums、可复现 recipe/源码来源/许可证与 NOTICE。不得把自动生成的根仓库 source archive 当作三个 fork 的完整源发行。
- lock 文件提交到 Kodex，记录 schema、bundle id、tag/具体 asset URL、archive/manifest SHA-256、各 fork 完整 Git SHA、发布版本、recipe/toolchain identity 和支持 target 列表。manifest 记录每个文件 hash、闭包和验证状态；二进制不进 Git。
- 用户机器只需匿名 HTTPS 下载公开 Release；GitHub API/gh/token 不作为消费前提。发布凭据仅属于维护者/受控 workflow，不嵌入源码或下载脚本。
- GitHub Releases 每文件须小于 2 GiB；研究 r1 归档为 90,736,640 bytes，不代表未来包大小：[官方限制](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases)。
- GitHub Packages 公开 Maven/Gradle 包也需认证，不作为本期默认消费入口：[官方说明](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-gradle-registry)。

## Publisher changes and version identity

- 修改面暂定 Kodex 维护工具、`.github/workflows/`、fork recipe 及 lock/catalog；实际目录在重新冻结当前分支后确定。新建 workflow 不视为已允许付费 runner 或在个人设备安装 self-hosted runner。
- 建议先显式受控发布入口（workflow_dispatch 或维护命令），再接 fork Git SHA/recipe 变化检测；无变化只核对已验收包，不启动 fork Gradle。
- 新版本由上游版本 + fork SHA + recipe revision 标识，KGP/Native/cinterop/SDK/target 变化也需要新版本；不覆盖同坐标、不把 dirty 工作树发布成某个 clean SHA。
- 修正或隔离 MCP `generateLibVersion` 未声明 version input 的风险；版本升级测试必须验证实际生成常量，不能只看 POM。
- Mosaic JNI/Zig 的 SDK 链接选择与并发参数纳入 recipe；Lucene publisher heap/worker 预算纳入 recipe，不修改用户 xcode-select 或全局 SDK/daemon 配置。
- 初版可从授权 Mac publisher 构建统一 root/target artifact；Linux 验证其消费。host 构建与根 metadata 合并必须消除重复/冲突，不能简单并行上传同坐标。runner 型号、SDK/Zig/JDK 和成本在执行前确认。

## Publication gates and failure handling

- 重新从当前 Kodex gitlinks 审核实际消费闭包；研究覆盖 Mosaic 6 模块、MCP core/client/server、Lucene core，root + JVM/linuxX64/linuxArm64/macosArm64/mingwX64 共 60 publications。
- 根 metadata、`.module`、POM、JAR/klib、sources、JNI/C/native/cinterop payload 必须一致；fork groups 解析不得混入上游同名原版本。
- 当前 r1 的额外 JS/Wasm/Apple mobile/Android/macosX64 缺失逐项列出；审核新分支是否消费这些 variants。未发布就拒绝相关消费，不把五目标包宣传为上游所有平台的完整 SDK。
- 在源码缺席的独立 consumer 运行 fork-only API、真实 Native 编译/链接与宿主运行；再做当前 Kodex 消费回归。linuxArm64/mingwX64 runtime 尚未验证，交接时明确 gate，不暗中豁免。
- 校验许可证、NOTICE、源码提供要求与打包内容；不得上传私人日志、凭据、用户路径或未经审阅的源码。研究 r1 可作种子，不等于可直接公开上传。
- 先建立 draft、上传完整 assets、下载检查，再公开；发布后再匿名回验所有 pin。若用户批准启用 release immutability，先检查其对同仓库 CLI 发布的影响，再启用：[官方 draft-first 与不可变规则](https://docs.github.com/en/code-security/concepts/supply-chain-security/immutable-releases)。
- 不可变设置是 repo-wide 且只影响未来 Release，不能在本任务悄悄改变：[设置范围](https://docs.github.com/en/code-security/how-tos/secure-your-supply-chain/establish-provenance-and-integrity/prevent-release-changes)。
- workflow 最小写权限限发布步骤；PR/不可信 fork 只做无写凭据验证。构建失败、asset 缺失、下载校验失败均不更新消费 lock；发布后的错误用新 tag/version 修复，不覆写旧版本。

## Acceptance and handoff

- 新 bundle 可匿名下载，归档/manifest/所有 payload pin 校验通过，无源码真实 consumer 成功；发布来源与支持矩阵明确。
- “无变化”“一库变化”“recipe/target 变化”“半发布失败”“同版本重发”分别有测试与日志；无变化不调用 fork Gradle，错误重发被拒绝。
- 交接 URL/pin/versions/manifest、consumer 证据、recipe、公开 Release 链接与已知 gates；接入任务不猜测实验目录。
- 执行时沿用父任务设备资源锁、现有 Daemon JVM 检查和 cleanup；运行前加载相应 remote/device/Gradle 技能。tag、公开上传和提交/推送必须获得对应授权。
