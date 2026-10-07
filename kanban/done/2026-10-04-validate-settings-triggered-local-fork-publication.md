# Task Tree

> **Superseded:** this feasibility task recorded the rejected settings-triggered publication idea. The current canonical route is [GitHub Packages CI fork publication](2026-10-04-plan-github-packages-fork-publication.md); do not execute this task unless the user explicitly reopens the local-publication route.

- `Confirm the frozen main baseline and fork publication manifest`
- `Prepare a tiny settings-triggered publication fixture on Xiaoxin`
- **`Validate the clean artifact-hit path without starting fork Gradle`()**
- `if (artifactIsMissingOrForkIsDirty()) {`
  - `Acquire the single publication lock`
  - `Trigger exactly one isolated local publication from settings`
  - `Verify the same CLI or IDE invocation resolves the newly published artifact`
- `} else {`
  - `Verify no publication process, source composite, or fork configuration occurs`
- `}`
- `if (publicationReentersGradleOrLeavesPartialState()) {`
  - `Stop the experiment and preserve the failure evidence`
  - `Return the lifecycle, recursion, lock, or stale-resolution failure to the user`
- `} else {`
  - `Validate missing, corrupt, dirty, clean-after-dirty, cancellation, and concurrent-sync cases`
  - `Compare a project-isolated repository with global Maven Local without changing user state`
- `}`
- `Report the exact feasible contract and keep production migration suspended pending review`

# Details

## Authorization and scope

- Parent: [Gradle development-experience planning](../executable/2026-10-02-rescue-gradle-development-experience.md).
- 用户已把主线改为“稳定 fork 默认消费本地发布物；本地 artifact 缺失或 fork dirty 时发布到本地”，并选择 **settings 内自动发布** 作为需要验证的行为。
- 这份任务只授权 Xiaoxin Ubuntu 上的隔离 fixture/副本研究，不授权修改本机活动 `Kodex/`、fork、gitlink、提交、推送、发布或全局 Maven 配置。
- 使用 exact main baseline `3008a6de9f178cda7cd63123a0261d75013f772d` 及已核对的 fork gitlinks；研究不得 checkout 本机 `refactor/spec`。
- 研究目录：`~/ACodeSpace/demo/kodex-gradle-research-445/local-publication-20261004/`；必须使用父任务的 `device-heavy.lock`，不将源码或 cache 放在 `/tmp`。
- 负责人需在任务 checkpoint 记录 Session、baseline manifest、远端路径、当前 gate、日志路径和资源清理状态。

## Feasibility contract

- 正常命中必须满足：本地 artifact、manifest、所需 KMP metadata/POM/`.module` 和实际 target variants 完整；fork worktree clean 且 commit/tree 与 pin 匹配。此时 settings 不得启动 fork Gradle。
- 缺 artifact、校验失败、版本/tree 不匹配或 fork dirty 时，settings 只能触发一次可观测的本地发布；发布必须写入隔离 staging，完成后原子提升，不能暴露半成品 metadata。
- 必须验证同一个 CLI/IDE invocation 是否能在 settings 阶段触发发布后继续解析新 artifact。若 Gradle 初始化/任务执行边界导致无法安全做到，任务立即停止，不自行改成 `includeBuild` 或隐藏 wrapper。
- 必须记录 nested Gradle 是否递归读取 settings、是否发生 daemon/锁死锁、是否污染 configuration cache、是否产生重复 publish、是否出现 stale dependency resolution，以及取消/崩溃后的清理结果。
- dirty artifact 必须和 clean pin 使用不同的本地身份，不能覆盖稳定 artifact；工作树恢复 clean 后必须重新解析 clean identity。
- 先比较 project-local Maven repository 与全局 `~/.m2`; 未经用户另行授权不得写全局 `mavenLocal()`、修改用户 settings 或安装全局 init script。

## Required scenarios

- fresh clone with no local artifact or fork checkout
- clean fork at pinned commit with complete artifact
- missing or corrupt manifest/variant
- dirty fork with changed source and changed build logic
- clean fork after a dirty publication
- two concurrent CLI/IDE requests
- cancellation and nested publication failure
- offline mode and unavailable toolchain
- explicit source-debug opt-in, kept separate from default binary mode
- JVM, Linux Native and macOS Native consumer evidence; Windows/mingw is a host/CI gate if Linux cannot run it

## Evidence and stop conditions

- Record commit/tree identity, coordinates/version, manifest hash, repository path, Gradle/JDK/KGP versions, process PIDs, lock holder, wall time, CPU/RSS and dependency-resolution outcome.
- `help` alone is not sufficient; the fixture must prove dependency resolution and at least one published consumer task.
- If settings-triggered publication cannot be safe and deterministic, preserve the smallest reproducer and mark this task blocked on user decision. Do not continue into production root settings changes.
- On completion, stop all owned Gradle/compiler processes, remove staging/temporary repositories, release the heavy lock, and leave only reproducible fixture sources and logs.
