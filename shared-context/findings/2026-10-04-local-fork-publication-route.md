# Local fork publication route — superseded

> Superseded on 2026-10-04 by [GitHub Packages CI fork route](2026-10-04-github-packages-ci-route.md). Retained as evidence for why settings-triggered dirty detection and nested Gradle publication were rejected.

## Decision

- 用户在 2026-10-04 重新编排 Gradle 任务：稳定 fork 默认以本地发布物被消费；本地发布物缺失、损坏、与 pin 不匹配或 fork dirty 时，触发本地发布。
- 用户选择“settings 内自动发布”作为待验证的行为，不等于该行为已经被 Gradle 生命周期证明可安全实现。
- Isolated Projects 从当前生产主线降级为可选后续优化；Mosaic/jextract IP 兼容不再阻塞本地发布物消费。

## Required design

- clean + matching artifact：settings 只解析本地 repository，不启动 fork Gradle。
- missing/stale/corrupt/dirty：只允许一次可观测发布；staging 完成并校验后原子提升 manifest 和 artifact 目录。
- clean and dirty identities must not overwrite each other; every artifact records fork commit/tree, publication recipe/toolchain and target variants.
- source `includeBuild` is explicit source-debug mode, not the normal IDEA Sync path.
- project-isolated repository is the first experiment; global `~/.m2` needs a separate decision and must not be polluted by a probe.

## Lifecycle risk

Gradle evaluates settings during initialization, while publication tasks run during a later execution phase. A settings-triggered nested Gradle invocation may recurse through settings, deadlock on daemon/file locks, pollute configuration-cache inputs, or leave a partially published Maven repository. Therefore the first task is a fixture-level feasibility gate, not a production settings change.

If the same CLI/IDE invocation cannot safely publish once and then resolve the new artifact, the experiment stops and returns the minimal reproducer to the user. The task must not silently fall back to source composite or invent an unapproved wrapper.

## Evidence

- [Fork publication experiments](../../kanban/done/2026-10-02-complete-fork-publication-experiments.md)
- [Canonical Gradle planning](../../kanban/executable/2026-10-02-rescue-gradle-development-experience.md)
- [Gradle Build Lifecycle](https://docs.gradle.org/current/userguide/build_lifecycle.html)
- [Gradle ProviderFactory.exec](https://docs.gradle.org/current/javadoc/org/gradle/api/provider/ProviderFactory.html)
