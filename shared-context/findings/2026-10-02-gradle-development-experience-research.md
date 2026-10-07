# Gradle 开发体验：三路研究验收汇总

- 日期：2026-10-02；协调 Session 445，研究 Sessions 453/454/455。
- 状态：三项研究已交付并核对；没有执行本机生产变更、提交或推送。
- 冻结根 commit：`3008a6de9f178cda7cd63123a0261d75013f772d`。结果不能直接当正在迁移的 spec/impl 分支验收。
- 已测结论：稳定 fork 默认 binary、逐库 source opt-in 可行；显式 JVM profile 是更大的日常 Sync 杠杆，full 继续用于完整平台工作。用户随后要求原生优先，binary/GitHub 不是当前已确认的实施选择。

## 1. 真实 IDEA Sync，而非 help

- Xiaoxin Ubuntu；实际私有 GUI IDEA IC2026.2.3、Gradle 9.5.1、Daemon Temurin 25、MCP toolchain 21。
- 成功对照统一 IDEA heap 3 GiB、Gradle heap 4 GiB、1 worker、compiler in-process。各组三次热态，不是随机交错或空依赖缓存测量。

| 场景 | IDE 原生导入命令中位数 | 后续 smart 验收中位数 |
| --- | ---: | ---: |
| Source full | 108.277 s | 112.469 s |
| 完整 binary forks / full | 71.696 s | 75.631 s |
| project-local 原型 / binary full | 74.479 s | 78.232 s |
| project-local 原型 / binary JVM | 7.500 s | 10.502 s |

- smart 列包括统一 3 秒 settling；原生 import 完成提示可能比最后 dumb-mode 退出早 1–2 秒，两种边界都保留。
- Binary 在相同主模型下约减少 33% 热态耗时；local/full 未显示明确额外收益，三轮差异不足以推断回归。
- JVM profile 相比原型 full 的原生导入约快 9.9 倍；它明确隐藏 Native 等非 JVM 模型，不能称 full 等价替代。
- Source/Binary/full 的 3,437 个主 IDEA modules、295 个 repository 源码根、内部依赖边完全一致；fork sources 跳转与基础 JVM 编译通过。
- JVM 的 260 个 JVM main/test 祖先闭包与 full 一致；不删除或合并 spec/impl 项目，Native 编辑/分析需 full。
- 热态 full 的根项目配置跨度约 4–5 秒，而模型回调约 57–87 秒；不能把整个 Sync 都归咎于 settings 扫描或 buildSrc 编译。
- 冷 IDE、部分冷依赖、脚本增量另列；未测完全空 Gradle/Native 依赖缓存，源码脚本增量也有 workspace 重开混杂。
- 证据：[Sync 报告](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/REPORT.md#L54)、[计时 JSON](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/evidence/measurements.json#L1)、[实际 GUI 模型对照](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/evidence/model-comparison.json#L1)。

## 2. 内存与 CPU：不能外推早期 CLI 结论

| 热态原生导入窗口的采样 peak RSS | Source full | Binary full | Binary JVM 原型 |
| --- | ---: | ---: | ---: |
| IDEA | 4,915 MiB | 5,106 MiB | 4,838 MiB |
| Gradle | 4,942 MiB | 5,195 MiB | 1,837 MiB |

- Binary 没有证明降低完整 IDE 场景 RSS；早期 CLI 的内存收益不能套到实际 Sync。
- JVM profile 显著降低 Gradle RSS，但 IDEA 仍接近 5 GiB；不能称总开发环境降到 1.8 GiB。
- 最后 60 秒稳态的 Gradle 平均 CPU 约 0.4–0.6%，IDEA 约 4.4–5.6%（100%=一核）；没有证明无限持续满核。
- 初期 Gradle 2 GiB 的 Source 导入 GC thrashing 失败；IDEA 2 GiB 也发生独立 OOM。不能把失败和成功场景比速度，或认为盲目减堆就能解决体验。
- RSS 不是独占物理内存或 heap；1 Hz 采样可能漏短峰，场景仍有用户桌面/服务背景负载。
- 证据：[资源及曲线说明](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/REPORT.md#L100)。

## 3. 稳定 fork 可以不启动源码 Gradle

- 交付包：`kodex-forks-r1`，Mosaic 6 模块、MCP core/client/server、Lucene core，共 60 required publications。
- 完整支持 profile：JVM、linuxX64、linuxArm64、macosArm64、mingwX64；保留 KMP root metadata、`.module`、POM、目标 payload、cinterop 和 sources。
- Manifest SHA-256：`46a2c88ea51b9fef84f2aaf1876d1c5ffb67301c1e31b77cb67a3e2e5955b14a`。
- Repo：Xiaoxin `~/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/repo`，本机研究目录和授权 Mac 有同字节副本。
- 协调验收重新核对 manifest pin，并在 Xiaoxin 共享锁内通过全部 1,620 个文件 SHA-256；`graphErrors=[]`。
- 无冻结 fork 源码时，三个 JVM 探针、四 Native target 的消费者编译、Mac/Linux x64 Native 链接运行通过；逐库 source opt-in 的新增 API 联调通过。
- linuxArm64/mingwX64 实际运行未测；SDK 额外 JS/Wasm/Apple mobile/Android 等声明平台不属 r1 支持范围。
- Manifest 发布时的 Linux pending 是历史快照；后续 Linux 成功日志补证，不覆盖不可变 manifest/pin。
- Bootstrap 完整包存在时只校验，不依赖 Java/Gradle；损坏/缺件/错 pin 明确失败，不回退上游或自动启源码 build。
- 发布成本移到 fork 变更时；实际有 Zig/SDK 链接、Lucene heap、MCP version task input 等维护门槛。
- 证据：[发布矩阵及边界](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/REPORT.md#L43)、[不可变 manifest](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/fork-publication/packages/kodex-forks-r1/manifest.json#L1)。

## 4. 主构建原型与长期路线

- 142 个子项目中有 130 个 KMP 模块；不是 142 个 KMP 模块，更不是只有 142 个 IDEA 模型。
- Full/JVM 配置对象：targets（含 metadata）880/260，source sets 3,292/550，configurations 44,370/9,410。
- 原型 full 配置图等价；JVM 保留实际 source-set 祖先与产品源字节，69 个现有 JVM 测试通过，不撤销 spec/impl 边界或机械收紧 `api`。
- Project-local kRPC rule 移除一个根跨项目状态问题，保持 patch substitution；目前是隔离/维护性收益，不是已证实 full Sync 提速。
- 原 Gradle 9.5.1 的 Isolated Projects 探针失败，阻塞涉及 TestBalloon、KGP JS/Wasm/root 状态和 kRPC。原报告 HTML 自带去重行 2,047；历史终端约 747 是另一归并口径，50,158 为总记录，均不是独立 bug 数。新版 Gradle 改变 parent lookup，不能直接外推新版仍有同样错误。
- Declarative Gradle 独立 KMP spec/impl + serialization 原型和契约测试通过；没有完整 Native/插件生态或速度优势证据，不选作本期直接生产迁移。
- 证据：[架构报告及生产批次](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/configuration-architecture/REPORT.md#L130)、[模型与源字节 JSON](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/configuration-architecture/results/summary.json#L1)。

## 5. 待审阅的落地顺序与门禁

- 历史原生优先暂停已被后续用户决策替代：GitHub Packages 和默认 binary
  现已落地；[剩余原生IP复核](../../kanban/planning/2026-10-02-verify-native-gradle-sync-mechanisms.md)
  延期为可选研究。下方实验数据仍只代表其记录的旧基线，不是当前项目收益。
- 如原生证据确实需要 binary 备选且用户确认，再在独立副本重新冻结当前 spec/impl 分支，接入已验收不可变 fork + source opt-in；不直接移植旧 main 项目路径。
- 然后加入显式 JVM/full 开发 profile，full 默认/CI/release 发布面保持明确；JVM-only 不得发布不完整正式坐标。
- Project-local convention、build-logic、扫描/按需插件分批维护；不把所有改动混在一次性能归因里。
- 生产前补同一 workspace 的 full↔JVM 切换与模型恢复、IDE 重开 artifact 冲突、当前分支回归及真实跨 host Native/CLI 门禁。
- 当前冻结原型的测试不是全仓/当前 spec/impl 回归；Desktop 若涉及仍需 JBR 25。
- IDEA 自身仍近 5 GiB 的内存应另做 retained heap/索引线程/缓存分析；尚无生产内存解法证据。
- 这份记录是研究结论与待批准路线，不是允许更改活动源码或升级 compiler/plugin 的授权。

## 6. 协调验收与清理

- 已读取三会话最终答复、checkpoint 和报告，并抽核 GUI `headless=false`、真实 start/success 事件、模型差异、测试 XML/汇总和发布 pin。
- Xiaoxin 协调只读核验：共享锁可获取，未发现研究 Gradle/Kotlin/IDE Java 进程，约 12 GiB MemAvailable；没有重启或停止用户进程。
- 子任务资源清理、增量脚本 SHA 恢复及私有 GUI 停止已有记录；研究 caches/安装/package/原始证据保留供复现。
- 三研究子任务归档到 `kanban/done/`；总任务保留 [实施 planning](../../kanban/executable/2026-10-02-rescue-gradle-development-experience.md)。GitHub 暂定方案现已暂停，生产改动与公开发布尚未执行。

## 7. 原生能力复核 — 执行中

- 2026-10-02 Session 445 核对 Gradle 9.5.1/9.7.0/9.8.0 版本源码：逐项目 intermediate tooling model cache 默认 `NONE`，非 IP 模型 invocation 强制 `configureOnDemand=false`。IP 完整 request/action 结果另有保存/加载路径；不能笼统说“开启 IP 即可缓存所有 IDE 模型”，也不能说“NONE 意味全部模型缓存关闭”。
- 9.7 指南 recent changes 说明模型缓存因正确性问题关闭；internal opt-in 不能当受支持生产开关。IP 的并行模型和兼容性仍值得研究：[版本源码](https://github.com/gradle/gradle/blob/v9.7.0/platforms/core-configuration/configuration-cache/src/main/kotlin/org/gradle/internal/buildtree/control/BuildModelParametersProvider.kt)、[指南](https://docs.gradle.org/9.7.0/userguide/isolated_projects.html)。
- 官方 [KT-87790](https://youtrack.jetbrains.com/issue/KT-87790) 记录 KGP 2.4.0 的大型 KMP Sync 回归，[修复 PR](https://github.com/JetBrains/kotlin/pull/6887) 已合并。本项目症状相符，但收益和因果尚未实测；优先核对修复版本及匹配插件，而不是引入自制缓存。
- Session 445 新真实 GUI 对照只使用源码 forks；显式 substitution + COD 请求与原 source 分开，保留全平台/测试插件。证据与未完成门禁见[原生路线报告](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/native-only-445/REPORT.md#L1)。
- source control / explicit + COD / tooling parallel + 2 workers 的热态 native 中位数 111.669/110.829/112.032s，未显示明确提速。根 Kotlin catalog 2.4.20 版本组为 101.916s，本轮约低 8.7%、仅三次顺序组，需实际 plugin codeSource 和反向复测，不能单独归因修复。四组含根主 modules 3,438、源根 295 与内部边完全相同，内存也无改善证据；升级副本基础 JVM 编译通过。
- 最小 Java Runtime canary：非 IP 模型即使请求 COD/CC 仍二者关闭；IP 模型 CC/parallel=true、intermediate cache=false，但完整结果可缓存。r4 重复 model 与同 client 重复稳定 BuildAction 命中；tracked build description 变化后 model 正确失效/返回新值。不同 client 的 action 本轮未命中；原生整体模型复用存在，尚未证明完整 KMP/fork + 真 GUI 已兼容/命中。
- Maven Local 重复 publish 仅编译/jar UP-TO-DATE，发布及 POM/metadata 仍执行。最初 IP canary 的失败是研究 observer 的不兼容监听，暂停 instrumentation 后成功并原样恢复；不是完整 KMP 项目兼容证明。Binary/GitHub 仍暂停，先过真实 IDE 的原生复用门禁。

## 8. 本机原生门禁补证 — 2026-10-03

- [本机报告与原始入口](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/local-native-20261003/REPORT.md#L1) 使用新冻结 `05a472e9` / 200 主 build scripts，不将旧 main 的提速比例外推。
- 真 IDEA 的 Java + source composite 已命中完整 Tooling 请求缓存；settings 变化正确失效、模块6→9、旧源根/依赖边保留；同 IDE 热态模型请求约0.23–0.28s。这证明原生机制/客户端存在，**不是完整 Kodex KMP 提速验收**。
- 普通完整源码 GUI 仍是 CC=false；原样 accessor 冲突复现，副本命名修复后的 in-process 冷态4GiB导入发生 Gradle heap OOM。默认 daemon/冷热控制未测，不能据此判定生产默认内存下限。
- 完整 source/IP 失败于 MCP JS 扩展和跨项目状态：final HTML1629次报告/112条去重诊断；插件适配与 root规则、版本组合仍是门禁，不能直接开开关或把 Java canary 当全生态成功。
- 自动模块发现还记录了源文件下无意义的 build-script probes；其 source增删失效影响需真实 canary验证，不能当已证实根因。
- 本机其他 Gradle 工作启动后保护程序拒绝竞争，自有资源已释放。默认策略/KGP实际版本对照、完整模型等价、跨 IDE 进程复用均未完成；原生 executable 任务保持开放，GitHub/binary 实施仍暂停。

## 9. Project-only Gradle optimization extension — planning

- 用户已要求把研究范围延伸到“剥离外部 fork 后，单纯 Kodex 项目自身如何有效优化 Gradle”。
- 该研究以已经发布的 fork binary 作为受控输入，重点调查 Kodex 自身的 settings/module registration、`buildSrc`/convention logic、KMP target/source-set/variant 模型、dependency/cinterop resolution、compiler/resource strategy 和 IDEA model import。
- 当前证据显示：binary full Sync 仍约 71.7s，主仓库配置跨度约 4.1s，Gradle model request 约57.2s，模型回调后的 IDE import 约14.0s；因此不能把 settings 扫描或 `buildSrc → build-logic` 自动当成主要收益来源。
- 新规划：[Kodex project-only Gradle optimization](../../kanban/executable/2026-10-04-plan-kodex-gradle-optimization.md)。每个候选必须单变量隔离、重复真实 IDEA Sync，并保持模块图、源码根、KMP hierarchy、`expect/actual`、cinterop、导航和代表性编译/测试语义。
- 官方 Gradle 资料支持先 profile 再改动；configuration cache 主要针对 CLI configuration phase，官方文档明确 IDE import/sync 尚不使用它；Tooling parallel、build logic lazy APIs 和 Isolated Projects 必须分开测量，不能互相冒充收益。
- Xiaoxin 项目级 CLI 首轮验证：warm `help --no-configuration-cache --parallel` 约 13.9–15.6s；显式列出同一 142 个 modules 的候选约 14.8–19.1s，project list 完全一致但没有可重复收益。Settings/buildSrc warm profile 约 0.03s，项目配置约 6–7s；因此 moduleTree 的主要价值暂时是输入确定性/维护性，不是已证实的 Sync 提速。
- 远端原始报告：[project-only Gradle optimization](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/project-gradle-optimization-20261004/REPORT.md#L1)。
- Configuration Cache follow-up — **finalized 2026-10-04**: `help` first/second was `15.13s / 3.63s`, and `:mcp-stdio:compileKotlinJvm` first/second was `18.50s / 5.54s`; a controlled build-script edit correctly invalidated the cache. CLI CC is functioning; it is not the missing optimization for repeated IDEA Sync because Gradle documents that IDE sync does not use CC.
- Declarative Gradle follow-up — **finalized 2026-10-04**: official EAP3 remains experimental with no compatibility guarantee; its own getting-started guide says it is not ready for plugin authors/build engineers/software engineers, and its unified KMP sample only covers `nodeJs`, `jvm`, and `macOsArm64`. Xiaoxin’s sample run was blocked by the absent JDK 17 toolchain before model execution. It remains research-only and is not a Kodex production migration candidate.
- This branch is closed for the current optimization phase. Reopen only after a relevant Tooling model, KMP/Native variant/cinterop, or Gradle/IDE version change.
- [Configuration Cache and Declarative Gradle report](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/project-gradle-optimization-20261004/CC-DECLARATIVE-REPORT.md#L1).
