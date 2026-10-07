# Gradle 变更前调研：build-logic、插件 classpath 与 fork 版本矩阵

- 日期：2026-10-03
- 设备：Xiaoxin Ubuntu；研究目录 `~/ACodeSpace/demo/kodex-gradle-research-445/buildlogic-research-20261003/`
- 目的：在用户已批准的生产变更开始前，验证 `buildSrc → build-logic` 的真实迁移边界，并核对三个 fork 的 Gradle/KGP 版本组合。
- 范围：只操作研究副本；没有修改本机活动 `Kodex/`、三个 fork、用户 IDE 配置或 GitHub。

## 实验条件

- 研究源快照来自本期冻结源码归档；MCP included build 的 root name 仅在副本中改为 `mcp-sdk-research-root`，用于绕过已知 `getKotlinSdk()` accessor 重复错误。
- Gradle 9.4.0、SDKMAN Temurin 25.0.4、`--max-workers=2`、`-Xmx2g`、关闭 Configuration Cache 和 build cache。
- 远端设备 heavy lock 全程按要求持有；实验结束后 Gradle/Kotlin daemon 全部停止，锁可重新获取，约 12 GiB 可用内存。
- 9.4.0 不是 Kodex wrapper 的 9.5.1，因此以下是迁移/兼容性原型证据，不是生产版本验收。

## buildSrc → build-logic 的实际边界

### 1. 直接搬目录不能直接工作

- 把 root `buildSrc` 移为 `build-logic`，在 root `pluginManagement` 中加入 `includeBuild("build-logic")`，保持六个 convention plugin 的 ID 不变：
  - `:build-logic:build` 本身成功，约 6 秒、11 个任务。
  - 完整 root `help` 在 52 秒后失败，位置是 `agent-context/prefix/agents-md/contract/build.gradle.kts`。
  - 错误是 Kotlin Gradle Plugin 被不同子项目重复加载，而不是 precompiled convention plugin 编译失败。
- 当前快照有 130 个 root convention plugin consumer，其中 36 个同时声明了 `kodex.kmp-*` convention plugin 和 `libs.plugins.kotlin.*` 直接插件。`buildSrc` 的隐式 classpath 与 included `build-logic` 的独立 classloader 在这组项目上暴露了冲突。
- 因此，`buildSrc` 改名、`pluginManagement.includeBuild` 和保持插件 ID 还不构成可交付迁移。

### 2. Gradle 原生的 classpath 解法已验证

- 在 root `build.gradle.kts` 顶层加入以下**仅解析、不应用**的插件声明：

  ```kotlin
  plugins {
      id("org.jetbrains.kotlin.multiplatform") version "2.4.0" apply false
      id("org.jetbrains.kotlin.plugin.serialization") version "2.4.0" apply false
      id("org.jetbrains.kotlin.plugin.compose") version "2.4.0" apply false
  }
  ```

- 在同一研究副本中，完整 root `help` 成功，耗时约 4 分 22 秒；root-only 方案已足以消除重复 KGP 错误。另加 `settings.pluginManagement.plugins` 的版本声明也成功，但没有证据表明它比 root `apply false` 必要。
- 这与 Gradle 官方“在 root 声明 `apply false`，让子项目复用同一插件版本/classpath”的做法一致：[Plugins intermediate](https://docs.gradle.org/current/userguide/plugins_intermediate.html)、[Best practices for structuring builds](https://docs.gradle.org/current/userguide/best_practices_structuring_builds.html)。
- 这不是性能收益证明：
  - 原 `buildSrc` 副本 `help` 成功约 3 分 31 秒；
  - `build-logic + root apply false` 的首次完整 `help` 约 4 分 22 秒；
  - 研究副本的缓存、路径和 included build 初始化不同，不能把这组数字作为最终 A/B；
  - `build-logic` 的独立编译和配置边界仍然存在，甚至可能增加首次成本。
- 生产迁移还需要保留/补齐：
  - `build-logic/settings.gradle.kts`、root version catalog 引用和插件仓库；
  - 明确的外部插件依赖；
  - root `apply false` 的完整插件清单；
  - KotlinMcpSdk 自身已有的 nested `buildSrc` 迁移边界。
- Gradle 官方确实推荐显式 `build-logic` composite，理由主要是封装、独立测试和更窄的失效范围；它不是跳过完整 Gradle 配置的开关：[Favor build-logic composite builds](https://docs.gradle.org/current/userguide/best_practices_structuring_builds.html)、[Sharing build logic](https://docs.gradle.org/current/userguide/sharing_build_logic_between_subprojects.html)。

### 3. 当前结论

- `build-logic` 可以作为维护性/失效范围整改，但必须把 root 插件 classpath 统一作为迁移的一部分。
- 本轮没有证明它降低 IDEA Sync、CPU 或 RSS；在通过完整 current-branch GUI Sync 之前，不应把它列为性能优化。
- 如果进入实施，应先做单独可回滚批次：
  1. root `apply false` 与 `build-logic` settings；
  2. full root help、全平台模型、IDEA Sync、基础编译；
  3. 修改一个 convention plugin，验证实际消费者闭包和缓存失效范围；
  4. 与原 `buildSrc` 方案对比后，再决定是否保留。

## fork 的 Gradle/KGP 版本矩阵

官方 Kotlin compatibility table 当前给出：KGP 2.4.20 fully supports Gradle 7.6.3–9.7.0；KGP 2.4.0–2.4.10 fully supports 7.6.3–9.5.0；KGP 2.3.20–2.3.21 fully supports 7.6.3–9.3.0。[Kotlin Gradle configuration](https://kotlinlang.org/docs/gradle-configure-project.html)

| Build | 独立 wrapper | 实际 KGP | 以 root composite 运行时的 Gradle | 结论 |
| --- | --- | --- | --- | --- |
| Kodex root / root buildSrc | 9.5.1 | 2.4.0 | 9.5.1 | 超过 KGP 2.4.0 fully-supported 上限 9.5.0 一个 patch |
| Mosaic | 9.6.1 | 2.3.21 | 9.5.1 | KGP 2.3.21 超过 fully-supported 上限 9.3.0；独立 wrapper 不会替换 composite 的 root Gradle |
| LuceneKmp | 9.5.1 | 2.4.0；AGP 9.0.1 | 9.5.1 | KGP 同样超过 fully-supported 上限一个 patch；AGP 9.0.1 在该 KGP 的表内范围 |
| KotlinMcpSdk | 9.6.1 | 2.4.0 | 9.5.1 | KGP 超过 fully-supported 上限；独立 wrapper 仅影响单独进入该 fork |

推论：

- 当前不是“全仓只改一个 Kotlin 版本”这么简单：Mosaic 的 2.3.21 是另一条 fork 维护线，且其公共 fork 可能仍要用于上游 PR。
- 用户已经决定：若 KGP 版本必须变化，优先做 Kodex 专有分支/维护线，不直接改未来可能向上游提 PR 的公共 fork。
- 候选矩阵应优先验证 Kodex root/MCP/Lucene 的 KGP 2.4.20 + Gradle 9.6.1/9.7；Mosaic 单独验证可升级版本或保持专有旧 KGP 的隔离方案。没有跨 host Native、完整 GUI Sync 和实际 plugin `codeSource` 证据前，不进入活动源码。
- 用户进一步决定采用统一兼容线：目标为 KGP 2.4.20 + Gradle 9.6.1，Mosaic 在 `kodex-submodule` 上单独适配；如果该路线验证失败，立即视为异常并中断迁移，返回用户重新决策，不自行降级、绕过或继续集成。
- 用户决定现在立即迁移 root `buildSrc` 到显式 `build-logic`；该迁移作为独立批次进入主线，必须先通过 root `apply false`/plugin classpath、插件 ID、KMP target、source set、完整模型、IDEA Sync 和基础编译门禁。
- 用户决定编译器采用 Kotlin daemon-first，并关闭 daemon 故障时的静默 in-process fallback；资源上限和 workers 不在本决策中预设，先用统一控制组实测。
- 用户决定投入一条独立、一次性完成的 Isolated Projects 兼容迁移线；Kodex 的真实产品 target 已明确为 Windows/Linux/macOS Kotlin/Native。JS/Wasm 不属于产品默认模型，应优先从开发配置中移除或隔离；只有发现源码/构建逻辑实际依赖时，才作为异常处理。
- 用户决定只使用 Gradle/IDE 原生 cache，不启用内部 intermediate model cache、不实现 fork SHA 自动切换或自定义替代缓存。完整 Native/source composite 的重复 Sync、跟踪输入变化、IDE 重开和模型正确性仍需验收；如果失败，回到用户决策，不自动启用 binary/source fallback。普通源文件变化与构建输入变化分别验证，不预设它们具有相同失效行为。

## Fork branch relationship decision

- 用户已确定：三个 fork 都使用同名 `kodex-submodule` 分支作为 Kodex 集成维护线。
- 每条 `kodex-submodule` 分支的上游是对应原始仓库的主开发分支；当前只读审计显示 Mosaic 使用 `trunk`、LuceneKmp 使用 `master`、KotlinMcpSdk 使用 `main`。原始分支保留上游同步和 PR 用途。
- 当前不修改 gitlink；这次只确定命名与分支关系，不改变现有源码、依赖解析或 IDEA/Gradle 行为。
- BuildKodex 将来切换维护线时固定消费具体 commit，而不是依赖移动中的分支名。

只读 gitlink/remote 审计：

| Submodule | Gitlink | 当前本地分支 | 公共 fork | 原始 upstream 默认分支 |
| --- | --- | --- | --- | --- |
| Mosaic | `7b1a412918f7bc24db847343480f010242939788` | `trunk` | `Stream29/mosaic` | `JakeWharton/mosaic:trunk` |
| LuceneKmp | `549f8afc9570bd4289f1047653d4cd6dc9665bbd` | `fix/host-logging-configuration` | `Stream29/lucene-kmp` | `nehemiaharchives/lucene-kmp:master` |
| KotlinMcpSdk | `53f717661712d618cd4822d1760392e903989ead` | `main` | `Stream29/kotlin-mcp-sdk` | `modelcontextprotocol/kotlin-sdk:main` |

## Module declaration decision

- 用户决定不继续修补“自动扫描”算法，而是定义 settings-local 的显式模块树 DSL。
- 根节点也使用 `moduleTree(String) { ... }`；实现两种同名重载：root/settings scope 中的无 receiver overload，以及 `ModuleTreeScope` 中的有 receiver overload。root tree 自动包含自身，即使它是无 build script 的聚合目录；nested tree 只有目录存在 build script 时才包含自身，纯物理中间目录只作为命名空间。body 只写子树，叶子使用 `module(String)`。
- Xiaoxin main 原型还发现：settings script 中顶层无 receiver 函数会遮蔽 receiver scope 的同名成员，导致 nested path 丢失；直接写 `this.moduleTree(...)` 可绕过，但尚未形成用户期望的最终调用语法，需单独修正。
- DSL 只负责生成 Gradle 原生 `include` 和 `project(...).projectDir`，自动按现有路径规则重整 project name，并对重复路径/名称直接失败。
- 首批保持 KTS-only，不引入外部插件、不扩大 Groovy 支持、不修改 gitlink；真实模块树和模型/源码导航等价是实施门禁。

## 证据索引

- buildSrc 失败与修复后的原始日志：
  - [`buildlogic-help.time`](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/buildlogic-research-20261003/results/buildlogic-help.time)
  - [`build-logic-only-20261003.log`](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/buildlogic-research-20261003/results/build-logic-only-20261003.log)
  - [`buildlogic-fix-root-only-help.log`](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/buildlogic-research-20261003/results/buildlogic-fix-root-only-help.log)
  - [`buildlogic-classpath-fix.diff`](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/buildlogic-research-20261003/results/buildlogic-classpath-fix.diff)
- 原 buildSrc 与迁移副本的初始耗时：
  - [`baseline-help.time`](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/buildlogic-research-20261003/results/baseline-help.time)
- 既有原生 Gradle/IDEA 证据：
  - [原生机制复核任务](../../kanban/planning/2026-10-02-verify-native-gradle-sync-mechanisms.md)
  - [Gradle 三路研究汇总](2026-10-02-gradle-development-experience-research.md)
