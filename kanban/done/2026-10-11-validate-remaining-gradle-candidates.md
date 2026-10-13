# Task Tree

- `Freeze clean production and the user decision boundary`()
- `for (candidate in independentCandidates) {` // executable checks finished; blocked checks preserved
  - `Prepare isolated source and preserve the official toolchain`(candidate)
  - `Validate model, classpath, actual functions and measured costs`(candidate)
  - `if (blocked) { recordEvidence(); restoreBaseline(); continue }`
- `}`
- `Run bounded complete IDEA import and navigation checks where resources permit`() // NOT_RUN: original disk preflight
- `Release owned processes and private credential copies`()
- `Report every result`()
- `Hand off the approved A/B/C/D candidates to the adoption task`()

# Details

## Current authorization

- 用户睡前授权逐项做候选与验证；卡点记录后跳过，继续下一项，最后逐项汇报。
  隔离验证阶段不修改生产或提交；随后用户明确授权采用A/B/C/D并提交推送，
  见[采用记录](2026-10-11-adopt-retained-toolchain-gradle-improvements.md)。不发版。
- 固定产品 `1b6ad5d3922fdfec63a472e01304ecf58d160577`，
  外层 `d9e6330fd179fd947e0d2da693d434d1328ded2f`；开始时两侧干净。
- 保持官方 KGP2.4.0、Gradle9.5.1、Koin compiler1.0.1 和现有全部
  targets/source sets；不使用实验 KGP、不维护 Kotlin fork、不增加最大堆。
- 构建/测试/私有 IDEA 仅在 Xiaoxin，持共享重型锁串行执行；本机只做静态
  准备。复制凭据不进入证据，结束移除。资源阈值不足也是明确的阻塞结果。

## Independent candidates

| 项目 | 本轮门槛 | 当前结果 |
| --- | --- | --- |
| 插件版本单一来源 | 原版本 catalog alias、实际插件 classpath 与全模型等价 | PASS：模型/生成任务及独立84项JVM测试 |
| buildSrc → build-logic | 明确 included build、共享插件 classpath、生成任务与代表编译测试 | PASS（CLI范围）：模型、218项JVM、9项Node/Linux及代表编译、生成任务CC |
| kRPC 规则仅真实消费者 | 原 Native patch 保留、完整反向消费闭包、实际解析图等价 | PASS（语义）；重复收益测量 BLOCKED：17消费者/306图等价、149项测试 |
| 禁止编译器失败回退 | daemon 正常编译与故障时诚实失败，最大堆不变 | PASS：正常编译及同故障的允许/禁止回退对照 |
| 完整 IDEA 与导航 | 原预算完整模型、真实导入/源码跳转；不以 help 代替 | NOT_RUN：原26GiB私有GUI启动空间不足 |

## Related work

- [主任务](../done/2026-10-02-rescue-gradle-development-experience.md)
- [完整模型与资源](../done/2026-10-02-plan-gradle-model-resource-optimization.md)
- [项目受控优化](../done/2026-10-04-plan-kodex-gradle-optimization.md)
- 本轮 retained-toolchain 候选当时不以独立暂停的工具链升级为前置；升级现已按自身证据阶段结项。
- v0.4.11 已完成官方工具链四平台发布；早期链接资源中止是历史证据，
  不能写成当前已发布版本未链接，也不能反过来证明完整 IDEA 已修复。

## Evidence checkpoint

- Xiaoxin 隔离根：`~/ACodeSpace/demo/kodex-gradle-remaining-20261011`。
  1312源文件、7,355,578字节；传输归档SHA256
  `3eaaa25d4114fb02ae6de8dfbd78ce86ef66451494c89cbf9a5bc0b325900a88`。
- `baseline-model` 38.813s，`catalog-model` 83.105s：216项目、
  1344 targets、5026 source sets。两份项目记录（包括已应用插件产物名、
  source roots、任务与 configurations）完全等价，included builds 均为空。
  这不是耗时收益对照：后者首次重编 catalog accessors，缓存状态不同。
- catalog 只把 buildSrc 中 KGP/Koin 的两个重复版本字符串改为原版本
  library alias；`generateKodexVersion` 实际输出仍为0.4.11。
- `buildlogic-model` 87.145s通过：主项目全记录与 baseline 完全一致，
  仅增加预期 `build-logic` included build。原九个 Kotlin 文件不改实现，
  根预先声明三种 Kotlin 插件 `apply false`，目录改名并显式接入。
  原 `GenerateKodexVersion` 类型和任务仍直接可用，不增加转接生成器。
- 为继续 CLI 门槛，核对并封存前一轮已结束的隔离集成工程源码、测试结果
  与 reports，逐字节复核归档后释放该工程生成物；未删除共享缓存或旧实验堆。
- build-logic 原有三种根 Kotlin `apply false` 声明消融：
  `buildlogic-unanchored-r02` 30.710s明确失败于 `agent-context-spec-contract`，
  `NodeJsRootPlugin` 报告 KGP 在不同项目重复加载。不是版本升级失败，也不在
  生产发生；正向候选保留根共享 classpath，已通过227次真实测试执行。
  更早一轮因4GiB启动磁盘门槛未启动，未将该中止算作语言/插件失败。
- 当前实际依赖反向闭包仍为17消费者，306个主/测试JVM、四Native、metadata
  与编译插件配置的已选中组件、variants/attributes/capabilities完全等价；
  原外部 `kotlinx-rpc-utils` 没有在这些图中出现，仍选择真实 patch。
- 以上累计460次强制测试执行，零失败、错误或跳过；不是460个去重用例。
  各门槛目录保持独立，未运行 integration-test 中的付费/live 模型用例。

## Final per-item results

### A — canonical plugin versions

- 固定原版本，只增加 `kotlin-gradle-plugin`、`koin-compiler-gradle-plugin`
  library aliases，并替换 buildSrc 的两个实现依赖字面量；四项现有实现依赖
  继续由同一根 catalog 提供。没有把 Gradle 自带 Kotlin DSL 编译器说成KGP2.4.0。
- `catalog-functional` 66.976s：Migration与PathPicker的84项JVM测试真实执行。
  类路径/模型门槛通过；收益是消除重复版本来源，不是已证实的时间或内存优化。

### B — explicit build-logic

- 九个 Kotlin 实现文件逐字节保留；原 `buildSrc` 的两个构建文件迁入
  `build-logic`，显式接入 `pluginManagement.includeBuild`。根声明三种
  Kotlin插件 `apply false`，保留共同类路径，避免已实测的重复加载故障。
- `buildlogic-jvm` 121.464s：218项测试与完整CLI JVM编译；
  `buildlogic-platforms` 83.254s：8项Node与1项Linux Native测试，
  RPC contract/Migration的实际Linux Native编译通过。
- `cc-buildlogic-generator` 同一命令两轮成功：先保存配置缓存，
  后实际复用，第二轮Gradle报告477ms、版本生成任务UP-TO-DATE。
  只证明这个真实任务的CC兼容；没有测修改后的失效范围，不把477ms
  当成全工程编译、GUI Sync或相对buildSrc的收益。
- 主工程216项目/1344 targets/5026 source sets保持；只新增预期的
  build-logic included build。没有产生Generator转接类、模块代理或新公开业务契约。
- 尚未验证当前候选的Mac/Windows本机执行、四平台完整release链接或真实GUI。

### C — consumer-local kRPC replacement

- 根全项目替换改为一个真实规则convention，明确应用到17消费者。
  主/测试反向依赖闭包来自当前Gradle声明，不沿用旧清单而不检查。
- 配置名称及数量不删：规则作用范围由68,008个现有subproject configurations
  缩到5,271个，减少62,737个作用点；不是减少KMP targets或模块。
- `krpc-model-routing` 83.138s；306组主/测试JVM、四Native、
  metadata/编译器插件解析的组件、variant attributes/capabilities等价。
  这是图等价，不是全量文件SHA或所有符号导航等价。
- `krpc-functional` 61.054s：149项JVM/Linux测试真实执行，
  in-memory RPC的Linux Native编译通过，实际utils patch保留。
- 本轮计划三组交替重复；触发原4GiB启动空间阈值后未继续启动。
  保留首次中止，再封存本轮已不用的隐式构建生成物后，只完成两组control和
  一组candidate，仍不足三组。`cost-baseline-r02-1`与`cost-krpc-r02-1`
  配置后强制GC观察分别777,775,248与731,382,800字节；
  其他control到809,496,576字节，不能当成稳定百分比/峰值RSS/GUI收益。
  [旧基线的重复62MiB结果](2026-10-08-measure-fixed-model-build-cost-ablation.md)
  不冒充本轮完成的重复结果。
- 新增消费者需要应用规则；今后应保留闭包/解析检查，不能把17视为永久名单。

### D — explicit no-fallback policy

- 只在候选 `gradle.properties` 增加
  `kotlin.daemon.useFallbackStrategy=false`；原Gradle4GiB、Kotlin2GiB不变。
- `fallback-positive` 40.836s：完整模型等价、KMP实现类计数1，
  强制执行真实JVM编译，采样实际观察到Kotlin daemon。
- 注入相同不可用JVM启动选项、不改源码：`fallback-daemon-fault`
  20.689s明确在 `utils-coroutines-spec:compileKotlinJvm` 失败并报告回退关闭；
  默认配置 `control-daemon-fault` 22.664s实际输出
  “Compile without Kotlin daemon”并成功。
- 这是故障行为改善，不是性能改善；采用后daemon故障会变成明确构建失败，
  不能声称比默认配置始终更快或解决已有GUI OOM。
- 未将10秒daemon退出、额外GC参数、堆/worker调小或in-process策略加入生产候选。

### E — complete GUI and navigation

- 原私有GUI入口要求26GiB启动空间，本轮初始4.5GiB，使用后只有约4GiB；
  `summary.json.gui` 记录27,917,287,424字节要求与4,291,207,168字节观察值。
  本轮没有启动IDE，不改阈值、不伪称冷/暖/编辑后重导入通过。
- 完整initial/repeat/import-after-edit/reopen、expect/actual和库源码实际跳转，
  以及既有stock KGP/Json K2解析阻塞，继续作为采用后的独立门槛；
  CLI成功和source roots等价不能代替它们。
- 工具链2.4.20/9.6.1及fork统一仍暂停，不在本轮偷偷恢复。

## Isolated evidence and adoption handoff

- 固定候选保留在Xiaoxin隔离根的 `overlays/{catalog,build-logic,krpc,fallback}`；
  `summary.json` 含每文件SHA256、模型断言、每阶段结果及未运行记录。
  `results/` 保留控制组失败、测试XML归档、私有日志和采样；不提交原日志或凭据。
- 全部测试计数为执行次数：84 + 218 + 9 + 149 = 460，零失败/错误/跳过。
  两次故意失败是诊断对照，不混进测试绿灯；磁盘预检中止也不算插件失败。
- 本轮没有构建/使用修正KGP，没有改变任何产品或fork源码、版本、targets，
  没有提交/推送/发版。生产仍为 `1b6ad5d3`。
- 可供用户选择：A消除版本漂移，B完成结构迁移，C收窄规则作用范围，
  D强化故障可见性；B/C没有被认证为OOM修复或GUI提速。E仍明确阻塞。
- 父Gradle任务继续打开；A/B/C/D已获采用授权，不能按本报告整体归档为开发体验验收完成。
- 清理前1312个产品源文件逐一恢复并核对原SHA；复制凭据与自建JVM均已释放。
  本轮临时checkout已移除，固定源码/candidate overlays、测试归档与控制证据保留；
  `SEALED.sha256`封存1513文件并回读验证。没有删除用户共享缓存或私有旧堆。
