# Task Tree

- `Record the report and spontaneous recovery`()
- `Inspect current backend logs without changing user state`()
- `Reproduce real native TUI history startup in an isolated Home`()
- `Identify the bottleneck through an isolated socket-option ablation`()
- `Confirm the dependency repair and consumption boundary`()
- `Implement and validate the authorized repair`()
- `Verify history timing and regression behavior`()
- `Preserve evidence and release owned resources`()
- `Prepare the user-selected temporary dependency consumption proposal`()
- `Record authorization to implement the smallest practical repair`()
- `Verify the actual baseline and Native-only artifact substitution`()
- `Implement and publish the reviewed Native-only dependency closure`()
- `Run exact-source regression and real native history acceptance`()
- `Commit and merge the validated consumer pin`()
- `Hand off the validated repair to the authorized release task`()
- `Hand off release caches and release repair resources`()

# Details

## 已完成修复与合入 — 2026-10-16

- 维护源码`64cf7fdbcf65`已由中央
  [38786634541](https://github.com/Stream29/Kodex/actions/runs/38786634541)
  完成三宿主构建、归一元数据合并、三宿主真实binary smoke及不可变发包。
  正式版本`3.5.0-kodex.64cf7fdbcf65`仅有四Native network坐标；
  独立回下载22份实际载荷及manifest，字节/大小/哈希一致。
- 产品修复已签名提交为`c811eb739877b9380d3e2ee18e322b2c2d473c31`，
  推送feature并快进合入已推送main。仅新增限定仓库路由、单一Native消费pin、
  四叶坐标force和完整192项首屏回归，不改历史分页或流式订阅。
  四Native图中其他Ktor组件保持原值；产品JVM Network实际仍为官方3.5.1，
  不是把独立probe的官方3.5.0假定为产品JVM图。
- 正式包消费的产品JVM15、Mac Native12、Linux Native12项通过；
  Linux Release真实TUI首屏：240条0.603秒、1000条0.613秒，
  均无注入、未缩减192项首屏；原始发行二进制为18.682/20.746秒。
  Ktor维护源码重跑JVM40、Mac Native32项通过，另保留一个原有UDP忽略项；
  format/lint通过，生产及测试源哈希仍等于已验证补丁。
- 证据入口：
  [正式消费汇总](file:///home/stream/ACodeSpace/local/kodex-history-loading-787/formal-consumer/summary.json)、
  [公开包回读](file:///home/stream/ACodeSpace/local/kodex-history-loading-787/ci-ktor-06/public-readback.json)。
  1,314份产品源文件与Mac输入逐项一致；早期脚本/CI失败留存，不计为通过。
- 所有修复演示Home及活动构建进程已结束，临时凭据由验证器清理，
  共享设备锁已释放；私有验证缓存已由
  [v0.4.16发行任务](2026-10-16-release-kodex-0-4-16.md)
  复用并在发版后统一删除。该版本已发布，但未替用户切换现用后端。

- 用户进一步要求“好，那就这么做吧，你去修”：现在实施最小可行修复，
  优先验证四个Native产物替换、JVM和其他Ktor保持官方；
  如需复杂的元数据覆盖才成立，则按消融结果退回完整 `ktor-network` 发布。
  不修改整套Ktor，不改变历史首屏/订阅，不切换867的fork工作区。
  随后用户明确允许本次所需提交/推送/合入及依赖发布，并要求“顺手发个版”；
  当前按v0.4.16修复版推进，仍不自动停止或重启现用Daemon。
  产品main有其他Session的model-catalog/OAuth未提交改动，保持不动；
  本轮继续使用 `fix/daemon-history-loading` 隔离worktree。
- 分发由[独立v0.4.16发布任务](2026-10-16-release-kodex-0-4-16.md)跟进；
  本卡以依赖发布、修复代码验收合入和发布交接为完成边界，不冒充用户已经切换后端。
- 最初用户选择“修依赖并验证”时仅授权隔离Ktor Native补丁和Mac构建验证，
  不包含当时的公开PR、远端fork/依赖发布或新版产品Release；
  后续扩大授权以本节首项为准。
  867另行与用户处理贡献线的公开提交，不将该线授权自动继承到产品消费。
  补丁以官方3.5.0提交 `3ccad96f` 为基线；仅使INET接受端对齐已有JVM
  TCP_NODELAY=true行为，不对Unix-domain Socket设置TCP选项。
  在Ktor原有buildOrCloseSocket内设置选项，失败沿用原FD释放边界。
- Native getsockopt真实回归按原代码→补丁执行红绿对照，
  不将LD_PRELOAD消融作为修复后验收。
  Ktor遵循自身JDK21/Kotlin2.3.21/Gradle9.4.1配置，产品验证仍Java25；
  不在本机运行Gradle，不修改公共缓存或偷偷升级产品依赖版本。
- 用户另行要求由subagent负责上游贡献，已接受创建
  [Session867：Ktor Native TCP_NODELAY上游贡献](file:///home/stream/.kodex/sessions/867)。
  贡献路线与公开Issue/PR由用户和867讨论；本主线继续依赖与产品验证，
  867不写当前上游候选目录或占用正在运行的Mac验证根。
- 用户确认每次Loading history仍会等待很久，并要求自行启动复现；
  现在执行隔离真实TUI与合成持久历史的测量，不改真实Session、
  不重启现用后端。需要编译时仍只用remote MacBook。
  优先核验首屏逐项RPC读取和真实网络往返，不预设根因或直接加超时。

## 隔离复现与归因 — 2026-10-15

- 以已发布Linux x64 Release原字节、独立PTY/Home和纯合成持久历史复现；
  无真实Session数据或付费请求。240条全为可解码的UserMessage，
  打开目录/选择Session到历史首屏约18.68秒，Loading marker约18.08秒。
- Native系统调用观察有大量约40ms网络间隔。仅在本轮隔离进程的accept返回处
  设置TCP_NODELAY，同一二进制/240条历史首屏约0.716秒；1000条约0.587秒。
  注入库仅为消融诊断，不是产品修复，不安装或影响用户后端。
- Curl poll timeout降到1ms仍约19.58秒，否决通过该环境变量提速的猜测。
  不将无效对照藏掉或写成优化成功；最初PTY缺控制终端、合成settings字段
  拼写错误的脚本失败也保留，不算产品错误。
- 锁定Ktor3.5.0源码中Native TCP accept只设置nonBlocking，再构造Socket；
  未设置TCP_NODELAY。缓存3.5.1及核对时上游main同一路径也没有该设置。
  [上游接受路径](https://github.com/ktorio/ktor/blob/3.5.0/ktor-network/posix/src/io/ktor/network/sockets/TCPServerSocketNative.kt#L57-L67)；
  [隔离时序与实录](file:///home/stream/ACodeSpace/local/kodex-history-loading-787/README.md)。
  此前真实新会话/空历史与JVM网络验收没有覆盖原生非空首屏的时间预算，
  发布冒烟的12项也未打开非空历史，是验收覆盖缺口，不称此前已经测过。
- 历史初始页保留192个结构项，逐次RPC元数据/邻项查询放大小包延迟；
  不因此减少首屏大小、隐藏Loading、伪造Ready或改变订阅幂等性。
- 修复应优先在实际Socket owner/依赖层完成，并覆盖TCP与非TCP地址、
  配置失败清理及Native真实往返；不能全局扫描FD、反射Ktor私有Socket、
  用生产LD_PRELOAD或修改真实Home。新增依赖维护/公开上游提交需明确授权。
- 反向对照：1000条原版首屏约20.746秒，对比只加TCP_NODELAY约0.587秒；
  页数、数据、订阅和二进制保持不变。已明确这是本机Linux Native实测，
  不泛化为JVM/Mac/Windows均已复现或依赖补丁已经通过。
- Mac `red-native-03` 已真实执行新回归，原生产代码返回TCP_NODELAY=0而失败。
  应用修复后的 `green-native-01` 六项中五项通过，新断言因Mac返回非零bitmask 4
  而非整型1失败；这是测试把boolean选项写死为1的错误，不能当补丁无效。
  已改为非零断言，并计划同一新断言再跑原代码红/修复代码绿。两轮XML保留，
  供867贡献线参考，不把首轮绿测试失败藏掉。
- 同一非零断言的 `red-native-04` 已在原代码失败（flag=0）；
  修复版 `green-native-02` 的Native 5项TcpSocketTestNix与1项UnixSocketTest
  均通过，XML已保留。格式/完整模块及消费验证仍在后续门槛内，
  不把这6项称作产品已安装修复或全部平台验收通过。
- Ktor完整网络模块回归：JVM 40项通过；Mac Native 32项通过，
  1项原有 `@Ignore` 的UDP descriptor测试跳过。`formatKotlin`、
  `lintKotlin` 和模块 `assemble` 通过；桌面验证通过官方私有
  `target.androidNative=false` 开关避开冷下载的Android NDK，
  不宣称Android完整验证。首次全目标准备触发30分钟守卫，证据保留。
- 生成唯一实验版本 `3.5.0-kodex-nodelay-787` 的本地Maven格式产物，
  根/JVM/四桌面Native均指向该版本；仅用于隔离消费验证，
  未上传外部仓库、未覆盖官方坐标缓存或改产品持久依赖配置。
- 新增产品192项非空历史真实网络首屏预算回归；原依赖在JVM及Mac上
  已通过，不能把名为 `product-red-01` 的批次说成红测试成功。
  后续Linux测试及源码修复TUI结果见下节，不以测试命名替代实际结果。
- 原依赖Linux测试在正确TMPDIR下执行12项，仅首屏5秒预算失败；
  测试框架虽返回进程码0仍发出失败事件，已改用事件判定的独立runner。
  修复依赖的Debug CLI真实首屏约5.91/5.08秒（240/1000条），
  Debug测试同样未通过5秒预算；不据改善就称回归全绿。
  随后构建与已发布基线同为Release的候选，同时将Debug回归预算校准为
  10秒，对原依赖/修复依赖在同一预算下重新红绿对照。
  预算仅作用于测试，不改产品超时、首屏大小或订阅。

## 隔离修复验证完成 — 待确认正式消费方式

- 同一10秒测试预算：Linux原依赖12项中只有历史首屏失败；
  修复依赖12项全通过。最终JVM 15项、Mac Native 12项全部通过，
  均为定向回归，不称全仓全绿。
- 修复版真实Linux Release首屏，各3次：240条中位0.761秒，
  范围0.665–1.486秒；1000条中位0.960秒，范围0.686–1.492秒。
  原发布版基线分别18.682/20.746秒。无注入库或Curl poll覆盖，
  保留全部192项首屏和原订阅语义；不是任意规模/平台的时延保证。
- 已核对实际选中的私有network KLib/cinterop坐标及43份产物哈希；
  最终1312份消费源码与本机工作区一致，其中原1311份均未改变，
  只新增历史回归测试。Release生产输入不受测试预算调整影响。
- [完整结果及复现资料](file:///home/stream/ACodeSpace/local/kodex-history-loading-787/README.md)；
  [机器可读统计](file:///home/stream/ACodeSpace/local/kodex-history-loading-787/verification-summary.json)。
  本轮未改产品持久依赖配置、未提交或推送修复分支、
  未安装候选二进制、未重启真实后端、未发产品修复版。
- 向867提供了[Mac非零选项断言证据](file:///home/stream/ACodeSpace/local/kodex-history-loading-787/UPSTREAM-HANDOFF.md)：
  Darwin启用标记实测可为4而非1，贡献分支须自行核对其断言。
  本主线未修改867的fork、PR或讨论卡。
- 临时Home、连接凭据和本轮进程已清理；Mac私有源码/构建缓存、
  工具链复制品及下载中转文件已删除，共享锁原inode保留并确认释放。
  原验证seed的1311份源码哈希仍一致。仅保留证据、小型私有测试仓库、
  本机候选源码和最终测试/预览产物，待正式消费决定；任务不提前归档。

## 正式接入实施边界

- 用户已批准实施、提交/推送/合入、中央依赖发包及产品修复版发布；
  真实Daemon切换仍由用户在维护窗口处理。

### 源码和版本

- 维护分支与867的上游贡献分支分离，不切换或重置用户正在使用的fork工作区。
  拟使用 `Stream29/ktor` 的 `kodex-submodule` 维护分支，按实际消费基线加最小
  TCP_NODELAY补丁及非零getsockopt回归；不直接消费上游main PR分支。
- 已在Mac冻结原依赖图：四Native目标的network/root实际均为3.5.0；
  部分其他Ktor组件因MCP解析为3.5.1。正式补丁沿用已验证的3.5.0基线，
  不改变其余组件的实际版本；此前仅凭MCP元数据的疑点已核实。
- 隔离的3.5.0补丁已证明路线可行，不等于不同基线的正式包已验证。
  版本采用既有 `<upstream-version>-kodex.<12位commit>`，
  manifest记录完整commit/tree和recipe/toolchain，不沿用会话号实验版本。

### 发布与消费

- 复用现有中央 `Stream29/Kodex` GitHub Packages路由、不可变版本与CI单写入协议；
  现有认证入口不变。按既有模式拟增加Ktor源码gitlink和最小发布适配，
  不另设fork lock或默认 `includeBuild`，不把整个Ktor加入产品Gradle构建。
  当前三fork脚本并非已经支持Ktor，新增适配须实现及验收，不能直接调用现有writer。
- 消融确认只替换四个Native目标可行，root/JVM继续官方。
  维护提交 `64cf7fdbcf65eac0186df0954a03a98b922a52e3` 已签名推送，
  包版本为 `3.5.0-kodex.64cf7fdbcf65`。
  只发布四桌面Native闭包，
  保留真实KLib/cinterop、sources、POM和`.module`。
  Native GMM归一为自包含组件，删除对未发布fork root的父URL；
  不生成新root、不修改官方root的variant路由。
- 保持group/artifact身份；只为实际需要的四个Native network坐标及fork版本
  增加exclusive repository路由，其他 `io.ktor` 组件继续走官方仓库。
  不设置 `includeGroup("io.ktor")`，也不把全局Ktor版本改成fork版本。
- 消费采用已验证的四Native坐标单一force规则，并通过实际依赖图验证；
  正式consumer仍需确认归一后元数据、链接和真实历史均通过。
  不把实验init的 `force` 加 `eachDependency` 两套覆盖一起搬进产品。
- 发布分两阶段：先完成干净源码的不可变包构建、校验、上传及回下载验证，
  再更新产品消费pin。不能重命名实验包充当正式产物，不能让包发布中途切换主线。
  本机不跑Gradle；重型开发验证仍在remote MacBook，部署CI另行授权。

### 验收与实际启用

- 对照原依赖图，除network闭包外其他Ktor组件版本应保持不变；
  以干净消费缓存证明真正下载正式坐标，不回退到官方network或`mavenLocal()`。
- 在正式基线重跑Socket红绿、Unix-domain和相关JVM/Native回归，
  保留完整192项历史首屏回归及240/1000条真实Native TUI时序。
  实际发版另按release skill完成四Release产物与Home门槛，不能用本轮Linux预览替代。
- 安装新二进制不会替换旧Daemon；启用修复须由用户选维护窗口，
  先处理运行任务，再显式 `kodex backend stop`。
  stop返回只表示接受，确认旧实例退出及锁释放后再运行新 `kodex`。
  旧TUI遇到实例更换按既定规则报错并需重开，不增加自动restart或跨实例重连。

### 回退、退出和当前门槛

- 保留旧pin及不可变包；问题回退通过恢复依赖配置或修复版构建处理，
  不覆盖已发包版本、不重写真实Home版本、不为本补丁新增no-op migration。
  替换运行实例仍须显式维护窗口，并核对Home/业务协议兼容。
- 上游正式发行版实际包含修复并通过同一验证后，统一升级官方Ktor依赖，
  删除network临时pin、限定仓库路由及临时发布适配；贡献跟踪仍归867。
- 早前查询因Mac共享锁占用而拒绝；新实施授权后重新检查设备空闲再运行，
  未争抢资源。原图与四目标替换均已实际解析，不再以静态MCP元数据代替事实。
- 发包首次CI停在guard：Ktor版本来自`VERSION`而非其他fork使用的
  `gradle.properties`。已增加对应读取和离线回归；未进入上传，未留下半成品包。
- 正式维护源码为`Stream29/ktor`的`kodex-submodule@64cf7fdbcf65`，
  版本`3.5.0-kodex.64cf7fdbcf65`，仅发布四个Native network坐标。
  Mac从该提交完成四目标真实KLib发布到私有文件仓库，退出0且无资源守卫触发。
- 中央流水线准备中另外暴露配置顺序问题：configuration-on-demand下原
  `projectsEvaluated`过早，随后提前读取KGP artifacts又违反FinaliseDsl阶段；
  已改为network项目的发布插件回调，并保留Ktor原生附件、不提前读取集合。
  Native元数据仍查询Java 8工具链，CI显式提供JDK8与JDK21。
  前三轮均未进入上传。
- `38780740014`三宿主构建成功，但合并校验拒绝额外Dokka附件；
  `38783852366`去除该附件后，通过产物结构校验，又拒绝Windows私有
  Gradle配置追加时产生的CRLF输入差异。两个拒绝均发生在上传前。
  已保留真实三宿主stage并核对差异仅是追加行的换行字节，
  改为二进制追加以保持同一输入，不放宽跨宿主哈希检查。
  随后的`38786634541`已完成完整重建、smoke及公开发包，结果见本卡顶部。
- 来自第二轮真实CI stage的22份Native载荷经独立结构校验；
  Mac以其归一后的Native元数据消费，实际TCP交换及清理通过，
  JVM同时消费官方3.5.0成功。这是私有预检，不冒充中央发布回执。
  当前离线发布协议回归88项通过。

### 提案依据

- [已确认的中央fork发布协议](../../shared-context/findings/2026-10-04-github-packages-ci-route.md)。
- [现有发布入口](../../Kodex/scripts/fork-packages/pipeline.py)，
  [消费者仓库配置](../../Kodex/settings.gradle.kts)，
  [MCP的Ktor版本声明](../../Kodex/KotlinMcpSdk/gradle/libs.versions.toml)。
- [GitHub Gradle registry认证与发布](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-gradle-registry)，
  [Gradle解析规则](https://docs.gradle.org/current/userguide/resolution_rules.html)；
  文档依据不等于候选规则已在产品Gradle 9.6.1验证。

## 最初只读观察 — 后续已补隔离归因

- 用户在升级v0.4.15后报告所有打开的Session停在“Loading history…”，
  随即确认已经加载出来。两次反馈之间没有重启、修复或修改会话数据，
  不将自行恢复称作修复通过，也不认定历史损坏或永久死锁。
- 产品源码/发行tag为 `082bfc66`；最初阶段只读查看源码、
  进程及当前后端日志，未调用模型、迁移真实Home或使用共享GUI。
- 后端PID3780096在用户报告期间持续存在；当时日志记录Session839、843、787
  于UTC08:50:56、08:51:10、08:51:28打开，随后Session787正常处理请求。
  TUI日志为空，所查后端片段无历史加载相关错误。
  Session打开时间不等于历史首屏完成时间，不能据这些间隔推算加载耗时。
- 当时日志没有页面请求开始、Session激活、binding完成及首屏渲染的关联计时，
  仅凭日志不能区分首次激活/文件读取慢、RPC初始化往返、订阅等待或渲染积压。
  不把“新进程预热”当作已确认根因，不根据症状添加超时/重试或改动订阅。
- 若复发，先核对同一实例下新旧页签、等待持续时间和错误日志；
  如需加诊断或复现，用隔离历史与真实网络RPC，重型Gradle仍只在Mac。
  不为复现反复打开真实会话，不干预运行中的后端。
