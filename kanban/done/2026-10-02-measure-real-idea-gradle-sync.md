# Task Tree

- `Read parent task and claim IDE measurement ownership`()
- `Specify genuine Sync stages and comparable environments`()
- `Prepare a private IDE configuration and frozen checkout`()
- `for (scenario in syncScenarios) {`
  - `Acquire the shared heavy-operation lock`()
  - `try {`
    - `Measure IDE import and dependency resolution`(scenario)
    - `Measure indexing and steady-state resources`(scenario)
    - `Repeat controlled warm and cold scenarios`(scenario)
  - `} finally {`
    - `Stop owned IDE and Gradle resources and release the lock`()
  - `}`
- `}`
- `Attribute costs and report genuine Sync evidence`()
- `return ResearchEvidenceComplete`

# Details

## Ownership and authorization

- Parent：[本期 Gradle 开发体验](../executable/2026-10-02-rescue-gradle-development-experience.md)。
- 状态：Session454 研究测量完成，协调 Session445 已核对 C12、原生 GUI 事件、测量/模型 JSON 和清理证据并归档。下方 C0–C11 保留过程，不把当时“待验收”当作当前阻塞；生产迁移仍需另行授权。
- 实验目录：Xiaoxin `~/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/`。
- 仅维护本任务及自己输出，不改本机 Kodex 工作树或现有用户 IDE 配置；原基线不可覆盖。
- 可在授权 Xiaoxin 准备独立 IDE 环境；IDE config/system/logs/project/cache 隔离，不重启设备，不操作用户原有窗口。
- 遵循 IDEA、IDE collaboration、device、JDK 和 GUI 技能；需要用户图形交互/设备授权时明确提出，不能偷用其他机器现有 IDEA 窗口。
- IDE 安装、启动、import/index、Gradle 与性能采样阶段持有父任务共享 heavy lock；持锁直到自己的资源停止。

## Starting evidence and distinction

- [CLI 对照](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/REPORT.md#L37)仅证明配置成本，不能当 Sync 基线。
- main `3008a6de9f178cda7cd63123a0261d75013f772d` 的原CLI source曾因SDK accessor重名失败，仅命名绕过成功；旧记录保留。本次实际GUI与原样基本JVM编译未复现，不把旧CLI绕过改称原样成功或宣称问题已根治。
- Source `help` 中位数 6.06s，移除 includeBuild 的配置代理 4.09s；该代理未完整接入三个 fork 发布物。
- 不称 Tooling API `GradleProject`/`IdeaProject` 模型请求或 headless CLI 为真实 IDEA Sync。缺真实 IDE 验证时任务不算完成。

## Measurement design

- 固定源 SHA、Gradle 9.5.1/JDK 25、IDE/Kotlin plugin 版本、堆、workers、网络及 target 集；若升级版本另列实验。
- 分开记录：IDE 启动、Gradle settings/buildSrc/configuration、Tooling API/IDE model import、依赖/Native 工具下载、source attachment、索引/分析、稳态。
- 记录 wall time、Gradle/IDEA/Kotlin Daemon PID/JVM、CPU 曲线、RSS 峰值和同步后稳态；不将 heap limit 当 RSS。
- 冷缓存、热缓存和构建脚本变化后的增量 Sync 分开，至少重复热态场景并保留日志。
- Source baseline 可以先完成；接收 fork 发布任务完成的 staged repo/manifest 后补真实 binary baseline。
- 不拿“删除所有 includeBuild 但无法解析依赖”的 Sync 失败状态，和正常 source 成功状态比速度。
- 验证 source attachment、KMP common/JVM/Native model、编辑跳转和基础编译；不能为了更短 Sync 丢失必要模型。
- 确认测量没有兄弟任务后台编译、IDE/indexer 或用户负载；有外部负载时标注并重测，不停止他人的进程。

## Completion and handoff

- 输出真实 Sync/索引分段时间表与 CPU/RSS 曲线，附 IDE logs、版本、复现步骤及实际消费 repo manifest。
- 区分证实瓶颈与推断，指出 binary forks、target profile、Isolated Projects 哪些确实减少了 Sync 而非仅 help。
- 缺 Native publication、GUI 条件或插件兼容性时逐项记录并解决/请用户协助；不以代理测量关闭任务。
- 将可复现测量场景和主要瓶颈交接给配置架构任务；允许研究并行，最终比较需等待完整 artifact。
- 清理本任务 IDE/backend、Gradle 和采样进程，只保留可复用环境、报告和实验快照。

## Session454 checkpoints

- C0：领取完成；本机 `Kodex/` 只读，不运行本机 Gradle/Sync，不提交或推送。
- 本机交接目录：`file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/`；远端同名 `~/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/`。报告与原始证据仅写此目录，后续 checkpoint 记录具体路径。
- Source 成功测量前必须单独保留原样 accessor 失败，并明确标注命名绕过；binary 等待兄弟任务完整 staged repo/manifest，不使用现有 partial Maven repo 作成功对照。
- C1：Xiaoxin 无现有 IDEA/Gradle/Kotlin Daemon；已有用户 GNOME 和他用 `:99` Xvnc，均不借用。host 无 Xvfb/xdotool/grim，正在准备专用 GUI 路径。官方 IDEA 2026.2.3/build262.10968.63 安装包约1.62GB；首次持锁下载54秒仅约15MiB，已停止自有下载并释放锁，避免长期占用兄弟资源。再次原子申请锁返回 `LOCK_BUSY`，目前转做场景/采样设计。
- C1 交接：[测量协议与当前证据](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/REPORT.md#L1)；[逐PID采样脚本](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/scripts/sample-proc.py#L1)。尚无 Sync 数据；任务继续执行，GUI/插件兼容与完整binary发布物仍待验收。
- C2：第二次持锁180秒续传官方 unified 包，收到132,886,932 bytes后超时，已清理自有下载/子进程并释放锁。准备改用同版本官方 GitHub 开源包（872,276,272 bytes，有官方SHA-256）；再次申请锁忙时不启动重任务。未改网络/代理策略，仅显式使用设备已有 `127.0.0.1:7897`。
- C2：准备脚本已具备原子锁/进程组清理、私有认证 Xvfb display、config/system/plugins/log 隔离、PID/CPU/RSS/host/network采样及冻结Git blob校验复制。仅语法验证通过；不是已完成IDE导入。远端日志为 `~/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/logs/{install.log,lock-history.log,inventory-454.txt}`，后续实际场景输出在 `idea-sync/runs/`。
- C3（03:28 +08:00）：已在锁内逐文件校验Git blob并建立独立 `idea-sync/project-source`，6059个冻结tracked文件/139MiB；只读复制原Gradle缓存种子至自有home（4.8GiB），不复制Daemon registry、project缓存或IDE索引。原source/binary/results/cache均未覆写。日志 `idea-sync/logs/prepare-project.log`，固定Gradle9.5.1、JVM25、2GiB/2workers/in-process，S0仍无命名绕过。
- C3：官方开源IDE包下载仍不完整，当前用严格校验 `Content-Range` 的单连接串行续传和最终官方SHA-256作为安装门禁；每次重型锁窗口限时，网络EOF/超时准备数据不计入Sync。IDE被动observer与真实GUI `.ijperf` 导入场景模板已准备，待安装后对照实际API编译/启动验收；并非Tooling API代理结果。
- C4：官方IC2026.2.3/build262.10968.63安装包最终SHA-256通过；JBR25.0.4，bundled Kotlin262.10968.63-IJ、Gradle/Performance262.10968.63。私有observer按实际API编译通过。默认auto toolkit准备失败已单列，复验强制XToolkit＋私有runtime/session D-Bus/认证Xvfb，真实独立1600×1000 GUI已显示首次 `IntelliJ IDEA User Agreement`。
- C4授权门禁曾停在首次协议；**用户已在Session454结构化问答授权“接受并继续”，包括关闭数据上报**。只在私有官方开源安装包配置中处理，不借用账号/许可证，不激活试用/付费功能；现在继续执行，不再等待协议授权。尚未执行真实Gradle import，不关闭任务。[协议截图](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/runs/welcome-agreement/welcome.png)；实际窗口树/IDE日志/准备采样在远端 `idea-sync/runs/welcome-agreement/`；原始安装/锁/复制日志已回传本机 `idea-sync/logs/`。
- C4清理：自有IDE/Xvfb/D-Bus/采样与子进程全部停止后释放锁；等待协议答复不持锁。Source源码与原基线均未变；binary仍等Session453完整staged repo/manifest。
- C5（04:51 +08:00）：用户授权已落实，私有home中接受开源terms1.3并在实际Data Sharing窗口点击`Don't Send`，随后Welcome界面可见。隔离`user.home`、Java prefs及XDG config/data/cache；首次授权处理前曾将terms接受版本写入共享Java prefs，已记录副作用，未修改共享数据上报偏好。只读Swing树用于定位按钮，未触发模型请求，不计入测量。
- C5：开始`runs/source-original/`实际GUI场景，原样源码/无命名绕过、新IDE system、Gradle缓存种子；真实native `%importGradleProject`、observer与1Hz采样待运行日志验证。授权证据及截图在远端`runs/welcome-private-finish/`；任务仍未完成。
- C5运行核验：observer记录`headless=false`、IDE PID196836、实际`RESOLVE_PROJECT:0`开始于04:52:18.209，Daemon197952为Temurin25/Gradle9.5.1/Xmx2g；IDE使用自己的native init、sources download、source-set依赖与parallel model fetch。Gradle root/Mosaic/SDK/Lucene配置均已过，原样未加命名绕过；当前尚未复现旧CLI的accessor失败，也未完成依赖模型下载，不能预设最终结果。启动含人工Trust等待，须从Sync分段时间剔除。
- C5兄弟交接已读取：Session453完整`kodex-forks-r1`已原子安装Xiaoxin，manifest pin `46a2c88ea51b9fef84f2aaf1876d1c5ffb67301c1e31b77cb67a3e2e5955b14a`；B1不再缺发布物，待本任务校验/接入。Session455的联测recipe已交付，但其原型仅配置代理，需应用到完整binary副本再实测。
- C6（05:07 +08:00）：真实`RESOLVE_PROJECT:0`在784.707s后失败，Daemon197952日志明确`JVM garbage collector is thrashing`，heap2GiB；模型快照为0，未取得成功Sync/源码导航。触发Native bundle、cinterop/commonizer及included-build metadata工作；不与成功binary比速度。原样accessor旧CLI失败未在该次IDE配置阶段复现，未加命名绕过。
- C6：为避免600s监控窗口截断正在执行的首次模型下载，曾只暂停自有monitor shell、保留IDE/Gradle/sampler与原子锁，新增有界1200s窗口；记录在`runs/source-original/monitor-deadline-extension.txt`。观察到失败后停止后续自动/脚本重试，05:07:19确认自有进程清理再释放。observer/Gradle phases/1Hz原始样本、索引diagnostics、截图与SVG全部保留；不把索引初始化当成功模型索引。
- C6：初版采样按`GradleDaemon`子串分角色会将Native子进程继承的`sun.java.command`误归为Daemon；已按实际类token/Native标志修正后续采样，离线summary/curve纠正旧样本，原始TSV不覆盖。统计分IDE、Gradle、KotlinDaemon、Native，各PID可追溯。
- C6（05:10 +08:00）：完整r1 package本任务verifier通过，建立无fork源码`project-binary/`，只改includeBuild/独占repo路由与三组不可变版本；recipe在`manifests/binary-recipe.json`。为成功对照统一改为4GiB/1worker/parallel=false/in-process，profile SHA-256 `bc9c635828f2050b87cd72812cea09912ca5591bb5e9a46c3d591658a07a15c7`；Source正在`runs/source-4g-warm/`新IDE system运行prime＋三次Reload，未提前认定成功。
- C7：4GiB实际Source首轮回调成功，05:10:51.631→05:41:18.680约1827.049s，后续完整4429模块/索引smart marker在05:44:19.510，不能把回调结束当全部Sync/索引完成。首个热态05:44:20.224→05:45:59.207约98.983s，后续smart marker05:46:26.141。全程同IDE209880/Daemon210378、原SDK名`kotlin-sdk`、没有命名绕过。
- C7语义：[实际modules-16模型核验](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/runs/source-4g-warm/observer/modules-16.validation.json)在远端已生成，本机待回传；总4429，主工程3437，commonMain/jvmMain/linuxX64Main/linuxArm64Main/macosArm64Main/mingwX64Main模块及实际源根均存在，含mosaicMain/blockingMain/fileLoggingMain/luceneMain/processClientMain。仅模型/源码根通过，导航/JVM编译仍待实际验收。
- C7采样注记：prime中两份jcmd thread/heap快照用于归因，证明至少一次Daemon worker在HTTP SSL读取等待；IDE2GiB在模型导入/索引阶段有LowMemory信号但最终进入smart。warm2期间误插入一次约10s的模型JSON核验，须标诊断干扰，不作为quiet热态中位数；原始样本保留，下一次quiet重复补齐。
- C8纠正与阻塞：`RESOLVE_PROJECT:2`在05:48:52.693以`Java heap space`失败；**实际OOM是IDE PID209880/2GiB**，其launcher.log及2.2GiB私有heapdump确认，不能因Gradle BuildIssue通用提示而误归Gradle4GiB。Daemon210378于05:49:07接收StopWhenIdle正常退出；手动清理后05:55:47释放锁。此失败后的4429模型是旧模型，不代表该次成功。
- C8场景编号：4GiB run的`RESOLVE0`为启动自动Sync，`RESOLVE1`为脚本prime/无改动Reload，98.983s仍是一个真实成功热态Reload；`RESOLVE2`才是脚本第一个warm。此前checkpoint的warm编号依task ID描述，后续最终表按实际脚本命令和回调对齐，不制造三次成功中位数。
- C8：现在统一IDE3GiB、Gradle4GiB/单worker，在`runs/source-ide3g-warm/`复测，复用`system-source-4g`已暖索引，源代码/targets不变。后续Binary/架构对照同IDE heap；launcher加入实际Sync失败即停止重复的门禁。Heapdump仅留远端诊断，不回传2.2GiB大文件或冒充benchmark结果。
- C9：IDE3GiB Source三次脚本warm成功，Gradle/model回调约87.867/84.003/84.950s；native命令触发至`Import has been finished`约113.037/105.580/107.344s，人工`%waitForSmart`的3s settling与60s稳态不算Sync。三轮同Daemon233571，测量期间不运行模型JSON分析。
- C9语义：`runs/source-navigation`原生`GotoDeclaration`跳到冻结SDK `CoroutineStdioIo.kt`，文件URL事件、assert与截图通过；重新打开已有workspace时出现重复JS artifact Symbolic ID/配置文件冲突SEVERE，保留日志，不能称无IDE缺陷。Source主模型保留，但derived/source-attachment modules从4429增长至7445后稳定；最终对照按主模型/源根而非总module数验证，需单列重开与缓存状态。
- C9编译：隔离原样Source `:mcp-stdio:compileKotlinJvm`成功（84 tasks，61 executed/23 up-to-date，约78s）；无SDK命名绕过。它只验收基础JVM编译，不作为Sync收益或Desktop/全仓测试证据。
- C9：06:19开始完整Binary真实GUI导入（IDE3GiB/Gradle4GiB/1worker，新system与无fork源码副本，完整manifest pin已验证）。启动含Trust准备等待，分段剔除；只有成功Sync＋同主targets模型＋源码导航后才与Source计算收益。Source缓存重开冲突还需清洁workspace对照，不提前关闭任务。
- C10（07:06 +08:00）：完整Binary三次真实热态Reload成功；Gradle/model约57.175/56.864/58.684s，native命令到完成约71.696/70.844/72.240s。主模型3438 modules，fork源码目录不存在；`binary-navigation`实际跳到不可变MCP client sources JAR内`CoroutineStdioIo.kt`，URL/截图/assert通过；基础JVM编译成功（62 tasks，53 executed/9 up-to-date，约43s）。完整模型差异及资源汇总仍待离线核验，不仅依据回调计算结论。
- C10架构：将Session455的15项build logic差异应用到完整Binary副本，保留本任务不可变repo/catalog，`project-architecture-full/jvm`真实GUI三次热态均成功。JVM profile实际模型696，raw热态约6.845/6.009/5.583s；它显式隐藏Native模型，不能称与full目标等价。Full profile仍约一分钟模型阶段，不将兄弟CLI `help`的收益冒充IDE收益。
- C10控制：旧Source `.idea`仅在自己的隔离副本中归档到`evidence/source-oom-workspace/`，新最小workspace和`system-source-clean3g`实测中；原tracked源码及其他任务目录未动。Source原样启动模型回调成功、完整4429模型已smart，正在同进程prime＋三次quiet warm，随后同IDE/Daemon脚本变更增量。日志在远端`idea-sync/runs/source-clean3g-warm/`；本机尚未回传全部证据。任务继续执行。
- C11：清洁Source三次quiet warm成功，raw84.117/88.404/87.063s，native102.873/108.971/108.277s。原生`Import has been finished`有时早于最后dumb-mode退出1–2s，已离线精化计时：显式`%waitForSmart`后的边界107.546/113.481/112.469s（包括明确3s settling）；Binary75.631/74.938/76.111s。最终headline同时列native与smart验证，不以回调冒充索引完成。
- C11增量：自己的root脚本一行注释在07:18:26应用，07:19:25实际Reload，同IDE287088/Daemon287799；raw93.931s、native121.880s、显式smart127.026s。before/after/恢复SHA及时间在`runs/source-incremental/delta/`，冻结原脚本SHA已恢复。增量组重开derived模块增加，不据此计算纯脚本变化的因果开销。
- C11架构联测：full三轮native74.479/83.221/71.108s，未显示相对完整Binary明确收益；JVM8.322/7.500/6.934s，显式smart11.439/10.502/9.935s。实际GUI260个JVM source-set祖先闭包无变化，JVM原生导航到pinned JVM sources JAR通过；JVM/full基础编译均成功（22/19s，有build-cache命中，非Sync数据）。
- C12最终语义：Source/Binary/full均3437个`Kodex.`主modules、295个repository源码根，module/源根/内部依赖边差异0；含root是3438，与旧checkpoint粗计数区别已解释。JVM为695个主modules/253源码根，有意隐藏Native。Binary实际JVM6个、四Native各9个不同fork class/klib URL均为r1，完整manifest验证＋无源码副本＋导航/编译通过。
- C12资源：成功热态Source/Binary IDE峰RSS约4915/5106MiB、Gradle4942/5195MiB，Binary不是本次RSS优化；架构JVM Gradle约1837MiB而IDE约4838MiB，不能称整体只占1.8GiB。完整CPU/RSS曲线、host/network/cache状态、逐PID、分段及失败日志已回传本机；没有测全空Gradle/Native依赖冷缓存，冷IDE与部分冷依赖明确分列。
- C12交接：[最终报告](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/REPORT.md#L1)、[完整测量表](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/evidence/measurements.json#L1)、[实际GUI模型比较](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/evidence/model-comparison.json#L1)、[Source曲线](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/runs/source-clean3g-warm/resource-curves.svg)、[Binary曲线](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/runs/binary-ide3g-warm/resource-curves.svg)。远端同目录副本同步；原始传输bundle约594MiB已SHA校验，最新smart边界派生summary单独更新，不改raw。
- C12边界：完整Native发布物/GUI/插件门禁已通过本研究范围；未替代跨host runtime、Desktop/全仓测试、生产IDE重开修复或同workspace full↔JVM切换恢复。实施路线已交接445/455，只允许后续用户独立授权的生产变更。任务不冒充总任务完成，也不改本机活动项目、父/兄弟文件或其他看板。
- C12清理（07:45:07 +08:00）：最终原子持锁核验无自有IDE/Gradle/Native/sampler/display/D-Bus进程存活；增量恢复SHA正确，143MiB废弃unified partial已删。核验释放后原子锁再次可获取，证据：[final-cleanup.txt](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/evidence/final-cleanup.txt#L1)、[lock-history.log](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/idea-sync/logs/lock-history.log#L1)。只保留复现环境、缓存、原始证据及远端OOM诊断dump；本机/远端最终报告SHA一致。
