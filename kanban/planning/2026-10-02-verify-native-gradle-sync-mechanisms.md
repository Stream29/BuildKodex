# Task Tree

> Deferred research: complete full-source/IP cache reuse is not a prerequisite
> for the now-approved binary rollout. Resume requires the user's later IP
> decision. Non-IP genuine IDE/model/resource acceptance is owned by the active
> [implementation parent](../executable/2026-10-02-rescue-gradle-development-experience.md).

- `Reconcile prior findings with version-specific Gradle source`()
- `Locate native configuration, tooling, and Kotlin regression boundaries`()
- `Measure explicit composite substitution in genuine GUI imports`()
- `Measure native tooling parallelism and the KGP repair version`()
- `Verify complete model and action cache reuse with tracked input invalidation`()
- `Await renewed authorization for full-source IP cache research`()
- `Verify whole-request reuse in genuine full-source KMP imports`()
- `Compare product source and build-logic changes in genuine imports`()
- `Attribute remaining CPU and memory without introducing a replacement cache`()
- `Return the native-first implementation decision and unresolved gates`()

# Details

## Authorization and ownership

- Parent：[Gradle 开发体验规划](../executable/2026-10-02-rescue-gradle-development-experience.md)；Owner Session 445。
- 用户要求原生方案必须弄清；继续先前授权的 Xiaoxin 隔离研究，不改本机活动 Kodex，不发布 GitHub，不实现自定义 SHA→publish/bootstrap 系统。
- 沿用父任务重型资源锁和已有私有 GUI 安装；每轮停止自有进程后释放锁。研究 root 与原始证据不覆盖。
- 新证据本机目录：`~/ACodeSpace/demo/kodex-gradle-research-445/native-only-445/`；实际 GUI 新场景保存在远端 `idea-sync/runs/native445-*`，与旧 Session 454 headline 分开。
- 2026-10-03 用户追加授权本机验证，明确本机空闲；本机 Gradle/私有 GUI 实验允许，但仍不修改现有 Kodex 源码或用户 IDE 配置。新本机实验目录为 `~/ACodeSpace/demo/kodex-gradle-research-445/local-native-20261003/`；用户重新占用 IDE/Gradle 时停止自有操作。
- 本机新基线：初检为 `7310a445` 加未提交文件；实际冻结前其他开发已提交，snapshot.json 记录干净 HEAD `05a472e90f6e3c0275c0448c396e05b17be0cac4`。冻结 6,237 文件，200 个主 build scripts。不是旧 main 142 项目的复测；输入字节、fork HEAD 与工作树状态以 manifest 为准。
- 本机 IDEA IU2026.2.3、Java25.0.3，无运行中的 IDEA/Daemon，初始约 19GiB 可用。使用私有 IDE config/system/logs 与 Gradle home；seed 模块依赖、复用既有 SDK/distribution/Native cache，不改全局设置。实际 KGP jar/运行参数追踪用于避免仅凭 catalog 判定升级已生效。

## Version and invocation audit

- 原基线 Gradle 9.5.1、KGP 2.4.0、IDE IC2026.2.3；原架构 IP 探针仅配置代理，不是真实 GUI import 或全部 fork 兼容性验证。
- 官方源 `BuildModelParametersProvider` 的 9.5.1/9.7.0/9.8.0 tag 均默认 `isolatedProjectsCaching=NONE`，指逐项目 intermediate 模型缓存；普通模型 invocation 禁用 configure-on-demand。IP 完整 Tooling request/action 结果另有保存/加载路径，不能将这个 flag 外推到整次缓存。
- 9.7.0 指南的概述仍有模型 cache 表述，但 recent changes 记录 9.3 起关闭；结论以版本源码和 runtime 参数为准，不照抄概述。
- internal tooling-cache opt-in 可以作为诊断候选，但正确性风险、兼容输入和发布级别必须标明；不以 dangerous-ignore/关闭测试插件获得的成功作为完整方案。
- TestBalloon 当前官方源仍用 `project.findProperty`；9.6+ IP 改变 parent lookup，必须复测，不能把 9.5 重复报告数量等同补丁规模或推断新版一样失败。
- Kotlin 官方 `KT-87790` 的 2.4.0 Sync 回归与 2.4.20 修复线索值得验证；KGP 2.4.20 fully-supported Gradle 上限为 9.7.0。不把 9.8+旧 KGP 任意组合当作受支持生产建议。

## Genuine GUI comparison

- 保留三个 forks 的源码/提交、全部 target 和插件，baseline 与 explicit 只改变完整 substitution / configuration-on-demand 请求；不使用发布物、目标收窄或删除插件来掩盖差异。
- 显式映射 Mosaic 六实际消费模块、MCP core/client/server、Lucene core；记录 root settings 及控制参数 diff。
- 两组使用相同 JDK/IDE/Gradle/KGP、heap/workers、私有缓存、采样和脚本；prime 不进入热态，至少三次 reload，记录真实 external callbacks/native import/smart、CPU/RSS。
- 采集 runtime operational model 参数来判断 COD/model cache 是否实际启用；独立 CLI canary 只能证明按需配置语义，不当 Sync 提速证据。
- 对比 GUI 导入后的主模块、根/内部边、fork 源码导航；增量改 impl/spec/convention/fork 的失效与缓存正确性需后续分开验证。

## Native gate and completion

- 明确原生机制的生产可用程度、适用 invocation、收益与插件适配清单；无法提供某项时报告“未验证/阻塞”，不承诺原生一定能完全跳过整个 build。
- 默认不升级活动项目；新版本和插件补丁先做匹配版本、最小重现，再进入完整 GUI/编译/测试回归。
- IP 开启后仍有问题就定位具体 stack/API/版本；并行不足或缓存 disabled 属产品能力边界，与插件错误分开。
- 实际实验输出、版本官方源链接、资源清理和剩余 gates 回写本任务；任务全部语句完成前不归档。

## Handoff to the implementation tree

- 本任务只负责原生机制、版本兼容、真实 IDEA Sync、模型/导航等价、缓存失效和资源证据；不直接执行生产迁移。
- 研究结果交给父规划任务的 [Integrated implementation order](../executable/2026-10-02-rescue-gradle-development-experience.md#integrated-implementation-order)；当前 IP 已降为未来可选研究，不沿用下方历史暂停状态阻塞 binary 落地。
- 若 KGP 2.4.20 + Gradle 9.6.1、完整 Native 模型或真实 IDEA Sync 任一门禁失败，父任务停止后续批次并回用户；本任务不自行降级、绕过、启用 `dangerously-ignore` 或恢复 GitHub/binary fallback。
- 只有父任务取得生产执行授权后，研究证据才可作为实施批次的验收输入；本任务的完成状态不等同于源码变更已授权或已通过生产回归。

## Checkpoint — Session 445

- explicit/COD 的两组真 GUI 导入与采样完成，各三次热态 native 中位数 111.669/110.829 s，约 0.75% 差异未证明提速；主模型与源根/内部边完全等价。原始证据与分段见[报告](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/native-only-445/REPORT.md#L1)。
- 本轮主 modules 3,438 包含 `Kodex` 根模块；旧 3,437 统计不含根。没有改变 target 或隐藏项目来获得等价。
- Gradle 9.4+ 的官方 `org.gradle.tooling.parallel` 与任务 parallel 独立控制；原 source/full + 2 workers 真 GUI 三次中位数 112.032s，主模型等价，未显示明确收益。更多 workers/其他 clients 的收益不据此判死。
- 同 Daemon 的最小 Java canary 实测：task 的 COD=true，非 IP 模型的 COD/CC/intermediate cache 均 false；IP 模型并行和 CC=true、intermediate cache=false，但仍记录完整 entry stored。r4 重复 model、同 client 重复稳定 BuildAction 均命中完整缓存；修改 build description 后 model 正确失效/返回新值。不同 client JVM 的 action 本轮未命中，client/action identity 必须验证。
- probe 第一轮被旧测量 observer 的 buildFinished 监听阻塞；r2 仅暂停该 observer 后成功，结束按 hash 原样恢复。失败证据保留，不能把它错误归因到产品插件。
- 隔离 Maven Local canary 的第二次 publish：compile/jar UP-TO-DATE，但 publication/POM/metadata 仍运行；没有自动“省去启动 fork Gradle”的发布判断。
- 根 Kotlin catalog 2.4.20 GUI 热态三次 101.916/100.400/109.459s、中位数 101.916s、smart 107.029s，主模型完全一致；本轮约低 8.7%，不能单独归因 KT-87790，须反向复测及实际 plugin codeSource 追踪。buildSrc 显式版本不等于最终选择结果（transitive BOM/约束可能提升）；不未经验证断言修复版已加载或未加载。热态 IDEA/Gradle RSS 没有收益证据。
- 新 Native 版本下载/脚本编译单列为准备成本；首个 20min 窗口未完成热态，后续缓存就绪的全新 GUI system 场景成功，不拿超时窗口当热态成功或回归。
- 根 Kotlin 升级副本 `:mcp-stdio:compileKotlinJvm` 成功：53s、84 tasks（57 executed/4 cached/23 up-to-date）。仅基础编译，不是全平台/全测试/当前活动分支验收；源码 fork 与产品源仍保留。
- IP 新版完整兼容、GUI 自身运行参数 trace、变更失效与内存归因仍未完成。原生任务保持 executable，binary/GitHub 规划继续暂停。
- 本轮进程已全部释放，共享重型锁可获取、约 12GiB 可用；私有 Gradle properties 和 observer 字节/hash 恢复，失效 display/auth 临时文件删除。当前暂停于完整 source/KMP + 真 IDE 的复用门禁，保留缓存和证据供续测。

## Local checkpoint — 2026-10-03 / Session 445

- [本机报告](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/local-native-20261003/REPORT.md#L1) 对应 frozen clean `05a472e9`/200 主 build scripts；之后活动源码又有别人的未提交开发，本轮并未修改它。
- 原样真实 GUI 再现 duplicate `getKotlinSdk()`；仅在副本改 MCP root name 后，in-process/4GiB 冷态 Sync 948.04s 因 Gradle heap space 失败，RSS peak 6191MiB。未测默认 compiler daemon 控制，不称默认策略必然失败；partial 模型不可验收。
- 同修复副本开 IP：29.03s 失败，MCP NpmResolverPlugin 缺 `kotlinNodeJs`；完整 entry discarded。最终 HTML 1629 次问题、112 去重诊断（早期 console 为102）；涉及 Mosaic/MCP 跨项目配置及 JS/Wasm/其他插件，不能靠一个开关通过。
- 相同真实 IDEA 的 Java + source composite canary 已直接证明客户端完整请求复用：warm0.229/0.247/0.278s，entry reused；settings 增模块使缓存正确失效、6→9 modules，旧模型行无删除、fork 源根/内部边保留，再导入继续命中。变更首次由自动 import 处理，随后手动 import 是 hit；没有偷换成手动 miss。
- 该 canary 为 Java，不是全部 KMP/fork 兼容；仅同一 IDE 进程。`cachingModelBuilding=false` 不妨碍完整 entry stored/reused；settings 变更后 unchanged fork 的 beforeSettings 仍重新执行，不能承诺默认逐 fork 复用。
- root discovery 的 file walk 对 Kotlin 源文件也探测 `build.gradle.kts`，HTML8962 input records 包含 `.kt/build.gradle.kts` 等无意义项。源码增删导致真实 cache 失效仍是待测假设，优先用同算法 canary 复现，不直接改活动 settings。
- 默认 daemon 策略控制组准备后检测到 unowned 全局 JDK26 Gradle 正在使用本机，保护程序拒绝启动；无 control GUI/Daemon/数据。Kotlin2.4.20 artifact/SDK已seed，升级对照未运行；9.6.1 也未运行，不能当完成。
- 自有进程全清、锁可获取、expired display/auth/socket/short alias 和只读 probe 清理；用户进程未动。当前 bold 语句继续暂停于完整源码门禁，不归档或切回 GitHub 发布。

## Native-first decision after this round

- Gradle 原生确实能缓存完整 Tooling 请求/稳定 action 结果；`NONE` 只证明逐项目 intermediate 缓存默认关闭。原生路线必须先评估完整 Kodex + IDE action 是否兼容/稳定，不先造 SHA 检测/发布切换系统。
- 已完成的四组 GUI 都是非 IP；不把最小 Java canary 命中称为实际 Kodex Sync 已命中。
- 全部输入未变的完整请求复用，与主构建配置变更后只复用未变 fork 的逐项目模型是不同收益；后者本期版本的默认限制仍需说明。
- 若原生 full/source 路线被具体门禁阻塞，才比较简单属性控制的 selective includeBuild/标准本地 Maven publication；GitHub 分发不是前置条件。

## Pre-change research checkpoint — 2026-10-03 / Xiaoxin Ubuntu

- [变更前调研报告](../../shared-context/findings/2026-10-03-gradle-prechange-research.md)；原始日志在 `file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/buildlogic-research-20261003/results/`。
- 直接把 root `buildSrc` 搬成 `build-logic` 后，`build-logic` 自身能编译，但完整 `help` 因 Kotlin Gradle Plugin 在不同子项目重复加载而失败；当前有 130 个 convention consumers，其中 36 个同时声明 convention plugin 和直接 Kotlin plugin。
- 在 root `build.gradle.kts` 顶层用 `org.jetbrains.kotlin.multiplatform`、`org.jetbrains.kotlin.plugin.serialization`、`org.jetbrains.kotlin.plugin.compose` 的 `apply false` 预解析统一 classpath 后，完整 root `help` 成功。该变体在 Gradle 9.4.0、Temurin 25.0.4、2 workers、Xmx2g 下约 4:22；它是迁移兼容性前提，不是 Sync 提速证据。
- 原 buildSrc 副本约 3:31；这组冷态时间不具备严格 A/B 条件，不能据此选择 build-logic。生产迁移还需 root wrapper 9.5.1、真实 IDEA Sync、模型/源码导航、基础编译和 convention 变更失效门禁。
- 当前 fork 矩阵已核对：root/MCP/Lucene 为 KGP 2.4.0，Mosaic 为 2.3.21；root composite 实际由 root Gradle 9.5.1 驱动，included build 自己的 wrapper 不替换它。按 Kotlin 官方表，2.4.0 fully supports 到 9.5.0，2.3.21 到 9.3.0；版本升级必须按 Kodex 专有维护线研究，不直接改未来可能用于上游 PR 的公共 fork。
- Xiaoxin 上所有本轮 Gradle/Kotlin daemon 已停止，heavy lock 可获取，约 12 GiB 可用；没有改活动源码、提交、发布或 GitHub。
