# Task Tree

> **Current boundary:** resource/model work follows the completed package-consumer gate. Targets are fixed; the user rejected target profiles. Isolated Projects remains optional.

- `Review the fixed target model, resource goals, and validation scope`()
- `Await the package-consumer gate before production changes`()
- `Freeze the current project graph and genuine GUI baseline`() // failed full import preserved
- **`Diagnose IDEA import retention without changing the existing target model`()**
  - [`Measure isolated build-cost ablations`](../done/2026-10-08-measure-fixed-model-build-cost-ablation.md)() // configuration savings measured, GUI remedy not accepted
  - **[`Repair the full-model import failure`](2026-10-08-repair-full-model-idea-import-oom.md)()**
- `Preserve every existing target and publication variant`()
- `Verify unchanged source hierarchy and navigation semantics`()
- `Move kRPC substitution into project-local convention`()
- `Benchmark independently approved resource controls`()
- `Apply independently measured build-logic improvements`()
- `Validate current-branch, cross-host, and genuine GUI scenarios`()
- `Report remaining gates and approve the measured rollout`()

# Details

- Current prerequisite is complete: fixed remote packages, binary consumer,
  equivalent moduleTree and SDK maintenance version are adopted. The current
  CLI/model evidence does not complete this task's genuine GUI/resource baseline.
  Compatibility and build-logic remain separately gated in the active parent.

## Current execution gate

- 2026-10-11 用户否决修改或维护 KGP。保留源码实验、对照 OOM 和导航失败
  证据；仅推进[项目内测试所有者配置](2026-10-11-adopt-project-owned-gradle-test-configuration.md)
  的分批提交。不采用临时插件仓库，不把该配置当作完整 OOM 修复。
- 本地产品配置提交 `44aac502`、开发说明 `f47f4198`；官方 KGP2.4.0、
  Gradle9.5.1、完整目标与资源上限不变，未推送或发版。
- 用户已授权本期生产落地；本任务进入 executable，保持先 package-consumer、
  后固定完整目标模型、kRPC local convention 与资源验证的执行依赖。
- 主任务：[Gradle 开发体验实施](2026-10-02-rescue-gradle-development-experience.md)；
  初始基线是 Kodex v0.4.10 / `83749434`，当前已上线基线为 `5b41be52`；
  旧 main 的测量不当作本期结果。
- 不占用用户打游戏的本机，不使用其 IDE 窗口；重型操作只在 Xiaoxin
  独立目录持 `device-heavy.lock` 串行运行。缺少真实 GUI 测量时只交付
  CLI/模型证据，不宣称 IDEA Sync 已提速。
- 当前 package-consumer 门槛已通过并以 `c0ba86cc` 推送；
  [验收与失败预算记录](../done/2026-10-07-consume-verified-fork-packages-by-default.md)。
  模型仍为216注册项目，无fork included builds；Native release链接需要本轮
  原预算4g，实验2g OOM不能外推为全局资源优化失败或已证明IDE收益。

## Scope and evidence

- Parent：[实施规划](2026-10-02-rescue-gradle-development-experience.md)；依赖 [binary/source 接入](../planning/2026-10-02-plan-binary-fork-onboarding.md)，当前仅 planning。
- 最新顺序：先完成 [GitHub Packages CI fork 发布规划](../done/2026-10-04-plan-github-packages-fork-publication.md)及其 package-consumer gate，再做固定目标的 binary consumer 与资源实验。IP/KGP 版本实验可在隔离研究内做，生产升级仍待审阅。
- [真实 GUI 研究](../done/2026-10-02-measure-real-idea-gradle-sync.md)、[架构研究](../done/2026-10-02-research-gradle-configuration-architecture.md)、[验收汇总](../../shared-context/findings/2026-10-02-gradle-development-experience-research.md) 是冻结 main 的证据，不是当前分支验收。
- Source full / binary full / local-binary full / local-binary JVM 热态 IDE 原生 import 中位为 108.277/71.696/74.479/7.500s；“原生 import”指 IDE 自身命令，不是 Kotlin/Native。
- binary 未证明真实 IDE RSS 降低；JVM profile 将研究 Gradle RSS 从约 5GiB 降至 1.8GiB，IDEA 仍近 5GiB。本期必须保留 IDE 自身内存诊断，不能以 Sync 提速代替该问题。

## Invariants and current-branch rebaseline

- 遵守 [spec/impl 边界](../../checklist/spec-impl-module-boundaries.md) 与 [JVM toolchain](../../checklist/jvm-toolchain.md)；保留有意义的模块契约、KDoc/API、serialization/RPC/persisted fixtures，不合并模块或机械 `api`→`implementation`。
- 从获准当前 commit 的独立副本盘点项目、targets、source-set ancestry、实际源字节、内部边、生成符号与 fork variants；旧 main 的 142/130 项目数不是当前验收目标。
- 记录 IDEA/Kotlin plugin/Gradle/JDK、系统背景负载、cache/network、heap/workers/parallel、Daemon JVM/PID。源码图变化与工具链升级分别做 A/B，不同时改一堆后归因。

## Batch A — fixed target model

- 2026-10-08 用户明确否决 `kodex.targets` 和开发裁剪模式；现有 targets
  已削减过，本期不再调整。上述候选未实现，现已撤回。
- 固定全部现有 KMP target、source-set ancestry、实际源覆盖、生成符号、
  expect/actual 和 cinterop；不以仅禁 task、按 OS 隐式选择或隐藏模型绕过。
- 开发、CI与完整发布保留同一目标配置。IDE restart/分支重开及
  source/binary 组合仍需验证模型、平台导航与 artifact roots 一致。
- 历史 JVM-only 实验保留为研究记录，不作为当前方案或验收替代。

## Batch B — project-local build logic

- 先只迁 kRPC 0.10.3 utils substitution；当前观察路径 `:rpc-impl-krpc-utils-patch`，执行冻结后再次确认。保持 compiler/core/transport/wire 的原版本和 patch Native 行为。
- 根规则移到实际消费者的 project-local convention，核对现有全部平台解析、RPC serializer/round-trip、基础插件入口覆盖；不声称该维护性改动必然提速。
- 后续 buildSrc→build-logic、settings 有界扫描、按需 publishing/test plugin、task avoidance 各自独立试点；测普通 Sync、改 spec、impl、convention 的失效范围和开销，catalog/插件 classloader/version generation 不丢失。
- 仅将有证据且通过图/功能验证的方案纳入本期落地；其余记录明确不采用原因，不把候选整改变成无界全构建重写。
- IP 当前有 TestBalloon/KGP/kRPC 等插件阻塞，DCL 仅小原型通过；本期默认不启 IP、不迁 DCL、不顺带升级 compiler/plugin。后续匹配版本实验另行审阅。
- 当前单因素实测已筛出 kRPC project-local 候选：配置后堆约降低62MiB，
  模型与68个JVM/Linux解析图相等；插件删减收益很小，暂不推进。
  但候选真实GUI初次导入仍报告heap OOM，未通过生产门槛。
  下一诊断聚焦实际观察到的旧式源码HEAD查询和元数据/诊断保留；
  不关闭必要源码导航，不把配置阶段收益等同完整Sync已解决。

## Batch C — memory and CPU, not only wall time

- 新受控 IDEA 实例采集模型导入/下载/索引/分析/稳态阶段、IDEA/Gradle/Kotlin/Test/Native 进程 RSS、CPU、GC 和 wall；不用原用户窗口或后台有负载的样本做对照。
- IDE heap/retained objects、模块/库模型、索引缓存和后台线程是诊断方向，不是已确认根因。heap dump 可能含源码/路径/凭据，仅在授权隔离工程生成、受限保存、不公开上传。
- 对照合理 workers 1/2/4、Kotlin Daemon 与 in-process、tooling model parallel 等选项；限制总资源，避免 GC thrashing。控制组参数一致，heap 上限不等于 RSS 或独占物理内存。
- 区分高 CPU 导入阶段与持续空闲负载；研究稳态不是满核，不能编造 idle 热点。若本期不能安全降低 IDEA RSS，交付 retained-heap/线程证据、已排除项和具体后续整改，而不宣称问题已解决。

## Integrated validation and acceptance

- 真 GUI IDEA 冷启动/首次包下载、热态至少三次、改 spec/impl/convention 增量、重开与模式切换分开测；Tooling API/help/cache-hit 只作辅助，不冒充实际 Sync。
- 量化目标在新 baseline 后固定：完整图与全部平台语义等价，减少无效配置/解析保留而非平台模型；报告 native import 与 smart/index 完成两个时间点及 CPU/RSS。旧 main 的 33%/9.9×不是当前分支承诺。
- 当前分支定向 spec/impl JVM 测试、storage 旧 fixtures、RPC round-trip、生成 serializer/RPC 符号、Mosaic UI/CLI 与下游编译；模块名从新冻结清单确定。
- Linux x64、macOS arm64、Windows x64 真实 Native CLI compile/link/run；Linux arm64 在对应授权资源完成 runtime。主仓库 shared JS/Wasm/iOS 与 Desktop/JBR25 如属现有支持面，逐项验收，不从 JVM 推断。
- 任何缺 host 或授权的项显式阻塞/提请用户裁定，不把 compile-only 写成 runtime 或关闭完整生产验收。
- 每批独立补丁和 rollback：kRPC 回根规则、build-logic 回原结构、资源参数回 baseline，目标配置始终不变。真实 GUI/Daemon/采样结束并清理资源后才释放父任务锁。
