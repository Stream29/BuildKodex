# Task Tree

- `Read existing spec and implementation boundaries and Gradle work gates`()
- `Check official KMP source set and compilation semantics`()
- `Confirm commonMain as implementation with an upstream specMain`()
- `Record the confirmed design and update canonical boundary rules`()
- `Inventory every spec project and freeze the planning baseline`()
- `coroutineScope {`
  - `val backend = async {`
    - `Audit backend source-set consolidation`()
  - `}`
  - `val frontend = async {`
    - `Audit frontend source-set consolidation`()
  - `}`
  - `val protocols = async {`
    - `Audit protocol source-set consolidation`()
  - `}`
  - `val tools = async {`
    - `Audit tool source-set consolidation`()
  - `}`
  - `val utilities = async {`
    - `Audit utility source-set consolidation`()
  - `}`
  - `Project the proposed merge graph and identify cycles`()
  - `awaitAll(backend, frontend, protocols, tools, utilities)`
- `}`
- `Reconcile complete coverage and propose migration batches`()
- `Present the scoped migration plan for user review`()
- `Check pre-refactor contract boundaries and correct retained-project migration`()
- `Freeze confirmed design and assign global checklist ownership`()
- `Confirm implementation authorization and the upgraded production baseline`()
- `Freeze the source-set convention and representative pilot migration units`()
- `Implement source-set wiring and the pilot hard cutovers`()
- `Validate controls, compilations, tests, metadata and independent consumers`()
- `Update pilot checklists and preserve evidence while releasing validation resources`()
- `coroutineScope {`
  - `val backendImplementation = async {`
    - `Migrate backend contracts and approved consolidation units`()
  - `}`
  - `val frontendImplementation = async {`
    - `Migrate component contracts and application ownership units`()
  - `}`
  - `val protocolImplementation = async {`
    - `Migrate protocol models, transports and OpenAI units`()
  - `}`
  - `val toolImplementation = async {`
    - `Migrate tool contracts and safe handler consolidation units`()
  - `}`
  - `val utilityImplementation = async {`
    - `Migrate utility contract and helper responsibilities`()
  - `}`
  - `Update settings and all consumer IDs from the shared manifest`()
  - `awaitAll(backendImplementation, frontendImplementation, protocolImplementation, toolImplementation, utilityImplementation)`
- `}`
- `Validate whole-repository model, dependency graphs, compilations and tests`()
- `Update affected checklists and release isolated build resources`()
- `Record acceptance gaps, commit/push authorization and implementation checkpoint`()
- **`Await user acceptance of the migration and documented baseline gaps`()**

# Details

## 全仓实施授权（当前）

- 用户进一步明确“全仓迁移执行”；不再以首批检查点作为本轮结束范围。
  保留既有首批变更，在同一 `450e3bd7` 工具链基线上完成全部 91 个原 spec 的处理。
- 定稿采用前述 50 个安全合并单元（包含首批两个合并），其余 40 个保留独立项目；
  images-codec/ktor-client-ext/patch 不额外合并，但在项目内部按声明职责迁移。
  原 spec 中的具体 transport/算法/helper 留或移 commonMain，真实契约进入 specMain；
  不为凑层次造空接口，不修改 API 或新增平台实现。
- 五条实施 Session 已经用户接受：后端 779、前端 780、协议 781、工具 782、
  基础库 783。每条只修改自己的产品域与独占 executable 子任务，
  协调者统一 settings、跨域消费者重定向、全局规范及串行构建验证。
- 子任务：`2026-10-12-implement-kmp-spec-{backend,frontend,protocols,tools,utilities}.md`。
  全仓目标 149 个有构建脚本的项目、162 个注册项目，已由最终 settings 和
  实际模型确认；模型检查不代替下述编译、测试和 IDE 验收。
- 本机 IDEA 不操作；继续 Xiaoxin 独立目录、共享 device-heavy.lock、
  单 worker、4 GiB available 内存与 512 MiB swap 增量守卫。
  用户现已追加提交/推送授权；不包含发布，不修改 fork、工具链版本或无关并行任务。
- 协调者整合补充：旧 spec 名称中有九个纯实现/helper 项目
  （context-window、contract-ext、OpenAI JSON codec、两个 MCP transport、
  utility coroutines/kodex-home/kotlinx-io-serialization/logging）。
  其实现按职责保留 commonMain，不把具体函数体伪称新契约，也不新造抽象。
  所有原 spec owner 仍统一 opt in 真正的源集层次；这些项目 specMain 没有业务声明。
  utility 子线原先暂不接入四个纯 helper，协调者已补齐 convention，
  原 source/test/依赖 scope 不变；子线静态计数按交接时基线理解。

### 全仓集成中的验证发现

- settings/消费者统一重定向后，静态 149 项、553 条生产边、0 个非平凡 SCC，
  没有旧 ID、未知项目或 self edge。真实模型得到 162 个注册项目和 90 个 owner。
- 203 个原项目逐映射核对，目标集合一致；149 个目标项目每个平台的 Test
  compilation 输入等于原输入并集，associated main 保持，commonTest 根边仍为空。
  合并引入原 filesystem 的 posixTest 层次；模板可能去掉冗余直接边，
  不能用每个输入的单条直接边都相同来判断合并后是否等价。
- 原 1074 个 Kotlin 文件 → 1085 个；11 个输入因职责拆为两个文件，
  拆分时所有 package 下的非 header 业务/KDoc 行多重集合保持。
  四个非 Kotlin src 输入（含 interop/resource）字节不变。
- 首次 Test 编译暴露一个真实迁移适配：
  `ComposerViewModelTest.kt:223` 的 Pending 显式 cast 在合同与 VM 合项目后
  可被 smart cast，产生 redundant cast 且被原 Werror 阻止。
  仅删除该 cast，前面的 assertIs 与 callId 相等断言保留；不关闭 Werror。
  因此最终是 432 个 Kotlin 测试文件字节不变，一个测试有此等价语法调整。
- Agent View 原 `composerHostRepairTest` 真实运行超时，在干净 `450e3bd7`
  同 JVM/预算下重复出现相同失败；保留迁移/基线两份日志和 XML，
  不修改业务或扩大 timeout 来通过。本项属于当前工具链基线已有验收缺口。
- 全仓 `jvmTest --continue` 还收集到 Root View 两个文档渲染失败：
  `large patch reveals the next page through the actual control` 与
  `pending patch expands a real structured diff without executing it`。
  干净同基线、同 JVM/预算重跑得到相同名称、异常类型和位置。
  连同 Agent View 超时共三个基线已存在的失败，不借本轮迁移修复业务。
- 初轮 root task selector 会包含 integration-test 的默认 live-provider suites；
  30 项因缺少真实 Codex 认证前置条件失败，日志表明失败在凭据加载阶段。
  发现后仅尝试停止本轮标识的 JVM（此时该轮已退出），不读取/导入凭据、
  不重试真实端点。后续隔离验证通过 task-local init 排除十一个 live suite 源文件，
  覆盖认证、Responses、conversation、images、model catalog、web.run 和 MCP。
  初次隔离少排 MCP，因它引用被排除的 real-client helper 导致测试编译失败；
  这是验证脚本问题，已补全排除列表，不修改产品源码解决隔离问题。
  全部原测试代码另跑不排除的 compileTestKotlinJvm，只编译不执行；
  产品测试/启用政策不改，保留失败证据，不称 live 测试已通过。
- 全仓 `allMetadataJar` 已通过（2626 tasks），包括 specMain 序列化/模型和
  原 intermediate metadata；当前 Linux 宿主的 macOS cinterop 限制仍会告警，
  不能据此宣称其他宿主链接或 Native 测试已通过。
- 全部原测试代码不排除的 `compileTestKotlinJvm` 通过（583 tasks），
  live suites 仅编译没有执行。最终隔离 JVM 回归为 2571 次执行、
  3 个失败、0 error/skip；只剩上述已由干净基线复现的三个 UI 失败。
  Root task 整体因此仍返回失败，不把“未发现新增运行时失败”写成全绿。
  其余 library/backend/VM/fixture/CLI/RPC/transport 的 JVM 编译与可执行
  回归通过；共享插件的 serializer/Koin/Compose/kRPC 参与真实消费者编译。
- JS Node 执行 812 次、1 个失败、0 error/skip；失败是 Tool Runtime
  `deferred search output and handler use the active catalog` 中 `.single()` 得到
  多元素。干净同基线在原项目 ID 重跑复现相同测试/IllegalArgumentException。
  另有 integration-test 的 JS 测试编译缺 Compose Runtime，干净基线也复现；
  原脚本仅为 JVM/Native Test 添加 Mosaic Runtime，本轮只重定向 project ID，
  不为使验证全绿临时添加产品依赖或裁掉 JS target。
  全仓 JS main 编译未发现错误，root portable task 因这两个基线缺口返回失败。
- Wasm Node 执行 56 次、0 fail/error/skip。JVM/JS/Wasm 可运行隔离集合合计
  3439 次、4 个运行时失败，均有对应干净基线对照；真实账号 suites 未验收，
  browser、IDE 与其他宿主也不因 Node 通过而视为完成。
- 全仓 Linux x64 main 编译通过；Test 编译只有 Root View 的
  `mosaicTest/SessionTabBarTestAdapter.kt:33` 使用 JVM `System` 在 Native
  unresolved。干净同基线同任务复现该错误（812 tasks），不是迁移新增问题。
  全仓 Native 编译轮仍返回失败，不能声称全部 Native 测试编译通过；
  不改适配器平台职责来隐藏原基线缺口。
- 额外 Linux x64 链接/执行 storage core、context prefix、RPC models、
  tool search、images 五组测试，112 次、0 fail/error/skip。IO-spec 的原项目
  无测试源码，所请求 task 是 NO-SOURCE/SKIPPED，不计作通过的测试。
  本轮四种平台合计 3551 次执行、4 个已复现基线失败，没有新增运行时失败；
  不与上文历史首批 76 次重复合并计数。

### 全仓交付检查点与证据

- 代码、settings 与所有消费者 ID 已切换完毕；保留独立契约、原平台和测试输入。
  没有兼容转发项目，没有改版本/fork/发布配置；提交/推送状态见下文。
- 全仓 1241 个最终源码/构建输入与实际验证副本 SHA-256 一致；
  最终业务/KDoc 行多重集合核对保留唯一等价 cast 规范化。
  当前文档的本地链接及产品/文档 whitespace 已检查；
  历史审查旧路径不当作当前源码链接，也不改无关并行任务。
- 可直接查看本机保存的
  [验证说明](file:///home/stream/ACodeSpace/demo/kmp-full-772-evidence/README.md#L1)；
  该目录包含完整日志、最终 JUnit/metadata、干净基线对照、逐项 manifest、
  模型对照、源码保留核查、最终 patch/new-files 及精确验证脚本。
  Xiaoxin 同份证据留在 `~/ACodeSpace/local/kmp-full-772/results/`。
- 重型验证只在 Xiaoxin 私有副本执行，使用既定锁与资源守卫；
  完成后清理本轮构建进程、source/baseline 副本、复制的 uv 和本地临时脚本。
  轻量证据与历史首批结果继续保留；未操作本机 IDEA 或外部 Session 的资源。
- 仍待人工验收：四个基线运行时失败、JS integration/Native View 的基线编译
  缺口；真实账号/原禁用 IO、browser、其他宿主、外部 artifact/IDE/性能未验收。
  本轮执行交付完成，不把这些缺口写成通过；主任务与实施子线留 executable 等待验收。

### 用户追加授权的提交与推送

- 用户明确要求“提交推送”，授权本次迁移代码、父仓库子模块指针、
  对应 checklist 和看板记录进入版本历史；不包含发布或修复无关基线缺口。
- Kodex 已提交 `bbbc9525d5aec1bab6244ad44760dbeea9939e80`：
  `refactor: consolidate spec and impl with KMP source sets`，已成功推送 `origin/main`。
  提交内容与本轮验证 patch、375 个新增文件及 1241 个最终输入一致。
- 父仓库同步该产品 revision 与迁移规范、五份静态审查和五份实施子任务、
  本主任务；排除并行 Session 773 的磁盘清理/目录退役记录。
- 提交推送不等同于关闭四个基线运行时失败、两个测试编译缺口或完成
  IDE/性能/其他宿主验收；任务继续保持 executable，证据和剩余门槛不变。

## 首批实施授权与范围（历史）

- 用户已明确“可以推进到实现阶段了”；任务进入 executable。
  原 planning/static-only 描述是前期边界，不再阻止本轮产品实现；
  不包含提交、推送、发布、fork 修改或无关工具链调整。
- 实施基线是升级已提交后的 `450e3bd7eeef460c7293d6f6fedb55cc1a4242c5`，
  开始时 Kodex 工作区干净；KGP 2.4.20、Gradle 9.6.1、Koin compiler 1.2.1。
  原 43888711 的图审查和 2.4.0 源码证据保持历史标签。
- 首批包含构建接线、两个合并试点及一个独立契约试点，
  先获得工程证据再扩大迁移：
  - `utils/read-write-mutex/{spec,impl}` → `utils/read-write-mutex` /
    `:utils-read-write-mutex`。契约移 specMain、实现和两个测试保持 commonMain/
    commonTest；shared 八目标及 JS/browser+Node/wasm/iOS 覆盖不裁剪。
  - `app/component/session-delete/spec` → 现有 `impl/viewmodel/src/specMain`；
    保留 VM 项目 ID、原五个 CLI targets、Koin 插件及 commonTest。
    View 保留独立项目，原 application/catalog 契约改依赖唯一 VM 项目。
  - 引用上述旧项目 ID 的真实消费者全部同批更新；没有转发项目或类型复制。
  - `hook/spec/notification` 保留原项目 ID 与目录，契约移入其自身 specMain，
    serialization API 依赖移到 specMain；实现项目、RPC models 的依赖边不变。
    保留五个 CLI targets、原 commonTest 与 wire 格式，验证空业务 commonMain
    的独立契约模型及生成 serializer。
- 构建约定使用显式 opt-in 插件，创建真正 specMain 与 commonMain→specMain，
  不给其余业务项目批量创建空层。实现时保留目标层次并使用既定精确抑制。
- 已识别本地 IDEA 打开 BuildKodex；当前无可发现的 IDEA MCP 工具，
  不接管用户窗口或触发本机 Sync。重型验证继续遵守用户游戏期间的本机暂停，
  仅在 Xiaoxin 独立本轮目录、共享 device-heavy.lock 下串行执行。
- 后续单元仍需在各批次开始前按冻结规则定稿；前批验收暴露结构问题时
  停止扩大范围，保留证据，不把“已进入 executable”写成全仓已完成。

## 首批实施检查点（2026-10-12）

- 构建接线和上述三个单元已落地；仅这三个项目 opt in
  `kodex.kmp-spec-impl`。插件先显式应用默认层次再建 specMain，
  项目原目标/renderer convention 随后接入；根属性只抑制既定诊断。
- 七个移动的 Kotlin 文件与实施基线逐字节一致；只修改构建脚本、
  settings 与消费者项目路径，不修改契约、实现或测试的业务内容。
  旧 mutex 两个 ID 和 session-delete spec ID 的产品引用已清零。
- 实际 Gradle 模型：注册项目 **216 → 214**，有构建脚本的业务项目
  **203 → 201**；不是历史 50 单元候选的完成计数。
  **211 个未迁移项目的目标、源集边、源码目录、compilation 输入和测试关联模型
  与同工具链干净基线相同**；此对照不包含全部 resolved variants。
- mutex 八目标、Delete VM/Hook contract 各五目标保持；main compilation
  均包含 commonMain/specMain。原 native/apple/ios/web 层次保留，
  测试关联 main，commonTest 未增加 dependsOn，未新增 specTest。
- 精确抑制对照通过：清空属性并执行
  `checkKotlinGradlePluginConfigurationErrors` 时出现预期 ERROR；
  开启精确属性后实际编译成功。单独 `help` 会输出 ERROR 但返回成功，
  不能当作此控制的通过/失败判据。
- 在隔离副本注入反例：specMain 引用 commonMain 的 `SafeRw`，
  metadata 与 JVM 编译都以 unresolved reference 失败；反向在 commonMain
  使用 specMain 的 internal 常量，JVM 和 metadata 编译通过。
  探针随后移除，正常 JVM/metadata 重新构建通过，未进入产品工作区。

| 单元 | 已执行测试任务 | 每任务测试数 | 执行合计 |
| --- | --- | ---: | ---: |
| read-write-mutex | JVM、JS Node、Wasm Node、Linux x64 | 15 | 60 |
| session-delete VM | JVM、Linux x64 | 6 | 12 |
| 独立 Hook contract | JVM、Linux x64 | 2 | 4 |
| 合计 | 8 个平台任务 | — | **76** |

- XML 汇总：76 次测试执行，**0 failure / error / skipped**；
  是跨平台执行数，不是 76 个独立测试用例。
  Hook serialization 测试覆盖现有 wire 形状，Koin 编译插件保持启用。
- 三个 metadata jar 均含真实 specMain 内容和 project structure metadata；
  mutex、Hook 的 Kotlin Multiplatform publication metadata 生成通过。
  未发布仓库产物，未完成外部仓库消费验收。
- 真实 JVM 消费者编译通过：Delete View、Catalog VM、in-memory storage、
  RPC models；不是只有试点自身通过。原 Werror、compiler execution strategy
  与 fallback 政策未放宽。两合并试点的成功完整构建复用 configuration cache，
  Hook/消费者构建保存 configuration cache；不据此宣称 Sync 或增量更快。
- 全局源集规范补充 convention 使用顺序和控制方法；
  Hooks、前端边界、JVM 工具链 checklist 按实际路径同步，其余领域待对应批次。

### 验证资源、证据与未覆盖项

- Xiaoxin 使用独立 source/baseline 副本，同一 Java 25 / Gradle 9.6.1 /
  KGP 2.4.20；持共享 `device-heavy.lock` 串行运行，单 worker、
  Gradle 4 GiB/Kotlin 2 GiB heap，保留 4 GiB available 内存，
  swap 增量上限 512 MiB。没有触发本机重型构建或操作用户 IDEA。
- 初期两次守卫把本轮 daemon 误判为外部 owner 并中止验证：
  任意 `-D` 标记不会留在 Gradle daemon 命令，后续 compiler 命令也可能缺标记。
  改用 JVM diagnostic path 加继承的环境 owner 标识后，完整验证重跑成功；
  只清理本轮拥有的进程。这些是守卫中断，不是产品编译失败或 OOM。
- 证据在 Xiaoxin SSH 路径
  `~/ACodeSpace/local/kmp-source-sets-772/results/`：
  `summary.json`、`baseline-model.json`、`model.json`、
  各轮构建/负例日志、`junit/` XML、`artifacts/` metadata。
  复现脚本、基线 ID 与源变更补丁一同保留；临时副本/探针与本轮进程清理。
- 未验收：用户 IDEA Sync/导航/索引、同条件性能与增量失效测量，
  浏览器 runtime、Linux arm64/Windows/macOS/iOS 编译或宿主 link/runtime，
  全仓组合/端到端测试。模型保留目标不等于这些宿主构建都通过。
- 首批结构与上述实际构建门槛通过；这些结果不外推
  Compose/kRPC/interop/生成源等未覆盖单元。主任务留在 executable，
  下一步按已获实施授权定稿下一批 manifest，遇到需改冻结设计的证据再请求决定。

## 当前决定与授权

- 用户认可 spec/impl 的职责区分，但希望减少 Gradle 项目带来的开发成本；
  已明确确认本部分设计：`platformMain -> commonMain (impl) -> specMain`。
- 已固化到 [KMP 源集设计](../../checklist/kmp-spec-impl-source-sets.md)，
  并更新 [spec/impl 边界](../../checklist/spec-impl-module-boundaries.md)：
  有意义的契约仍然必要，但不再要求每个实现都对应一个独立 spec 项目。
- `commonMain` 保留真实名称并承担 impl 职责；另建真正的 `specMain`。
  此决定取代此前空 commonMain 根、commonMain 承担 spec、
  或另外插入 implMain 的候选；独立 compilation/variant 不是所选路线。
- 用户明确实现依赖可接受且经常必要；不要求所有消费者只获取 spec 产物。
  这不自动改变契约代码自身的职责与依赖规则。
- 用户进一步明确：必要的独立 contract 项目保留，在其中使用 specMain。
  “不合并项目”不等于“不迁移源码层次”；旧报告的“原位保留”按此修正解释。
- 用户现已要求冻结这部分，并预计全局 checklist 写入范围。本轮冻结架构及迁移规则，
  不把旧候选清单、精确新路径或未执行验证一并视为已批准。
- 用户此前授权静态扫描与并行审查，任务曾进入 planning；
  现已另获上述实现阶段授权。`kanban/Draft.md` 无待提取条目。
- [构建开发体验任务](../executable/2026-10-02-rescue-gradle-development-experience.md)
  及 [项目级优化任务](../executable/2026-10-04-plan-kodex-gradle-optimization.md)
  的既有批次仍独立受其模型等价约束，不因本设计自动扩大实施范围。

## 已冻结部分与全局 checklist 落点

### 冻结基线

- 2026-10-12 冻结；后续执行清单以
  [全局 KMP 源集设计](../../checklist/kmp-spec-impl-source-sets.md) 为准，
  不再将 commonMain 的职责或独立 contract 是否使用 specMain 列为待选路线。
- 可合并的 spec/impl 进入同项目；必要的独立 contract 保留项目边界，
  其中契约同样迁入 specMain。历史 contract 边界与当前真实依赖共同决定保留项。
- commonMain 是实现层，specMain 是契约层；标准边为 commonMain → specMain，
  原平台/中间层继续向下连接。不另建 implMain，也不以 expect/actual 替代普通接口。
- 允许消费者依赖公开实现；这不允许契约源码反向引用同项目实现层，
  不要求独立 spec-only artifact。
- 迁移保留现有 targets、API/package、KDoc、wire/持久化格式、资源与取消语义；
  不把业务重构或平台裁剪混入源集迁移。
- KGP 采用既定精确诊断例外，不使用 fork 或广泛关闭检查。
  编译、metadata、IDE 与性能属于实施验收，未验证不表示路线仍待选择。
- 若实际验证暴露与冻结规则无法同时满足的约束，记录具体证据并提出变更，
  不静默改路线；这不是提前声称全部工程接线已经通过。

### 全局写入安排与实际落点

已确认规则与最终路径已同步，下表保留各主题内容归属；
具体验证结果留在本任务，不转写成跨平台或 IDE 已通过的全局结论。

| 全局文件 | 应维护的内容 | 时机与本轮状态 |
| --- | --- | --- |
| [kmp-spec-impl-source-sets.md](../../checklist/kmp-spec-impl-source-sets.md) | 源集职责/方向、独立 contract、依赖可见性、平台保持、精确 KGP 例外、通用迁移不变量 | **已冻结并写入**；首批实现已补充 convention 顺序与负例控制方法 |
| [spec-impl-module-boundaries.md](../../checklist/spec-impl-module-boundaries.md) | 逻辑边界不等于项目边界，保留契约也迁 specMain；沿用 KDoc、兼容和 hard-cutover 规则 | **本轮已同步**；作为领域规范的上位入口，不复制全部 KGP 细节 |
| [jvm-toolchain.md](../../checklist/jvm-toolchain.md) | 检查一起参与 JVM compilation 的 specMain/commonMain/platform 文件门面冲突 | 首批已扩充既有规则；不改 JDK/JBR 版本政策 |
| [agent-state-and-runtime.md](../../checklist/agent-state-and-runtime.md)、[hooks.md](../../checklist/hooks.md)、[rpc-timeline.md](../../checklist/rpc-timeline.md) | 保留根契约/模型边界，准确记录新源集和获批路径；状态、生命周期与缓存保证不变 | 已同步真实落点；保留并行工作确认的行为，不改无关语义 |
| [frontend-application-boundary.md](../../checklist/frontend-application-boundary.md)、[path-picker.md](../../checklist/path-picker.md) | 更新真实 spec/VM/View 落点；保持 View 单向依赖和 framework-free 契约 | 已同步合并与保留项目；公开实现可消费不等于契约可依赖 UI/DI |
| [cli-session-view-models.md](../../checklist/cli-session-view-models.md) | 现行规则引用新全局设计；旧装配记录不得被当成新的目录约定 | 已同步现行引用，历史装配段保持历史语境 |
| [openai-module-boundaries.md](../../checklist/openai-module-boundaries.md)、[codex-cli-storage.md](../../checklist/codex-cli-storage.md) | 物理路径/API 依赖描述改为实际 specMain/commonMain 或保留契约；wire/认证/只读导入职责不变 | 已同步；连同 model-catalog 的真实路径核对 |
| [tool-handler-decisions.md](../../checklist/tool-handler-decisions.md)、[clean-model-rust-alignment.md](../../checklist/clean-model-rust-alignment.md) | 去掉必须按两项目组织的目录假设，保留 DTO/handler、共享契约和行为边界 | 已同步；image-generation/tool-search 的契约和实现路径也已核对 |
| [mcp-client.md](../../checklist/mcp-client.md)、[kodex-home.md](../../checklist/kodex-home.md)、[external-url.md](../../checklist/external-url.md)、[rpc-state.md](../../checklist/rpc-state.md) | 检查受影响的独立契约、平台/interop、生成版本及 sealed 异常的路径/源集表述 | 前三主题中 MCP/Home 及 RPC 已同步；external-url 原路径规则无须修改，平台行为/异常格式不变 |

- 具体 checklist 是否改动，由获批映射与实际 diff 决定；本表是影响预估，
  不是授权现在重写所有领域文档，也不是断言每个文件都必须修改。
- 91 项去向表、候选计数、批次进度、具体文件冲突和测试结果继续留在 kanban；
  稳定规则归 checklist，不往 AGENTS.md 复制 SOP，不新建重复总规范。
- 领域文档的最终路径与接入示例随实现一起核对。迁移未发生前保留真实旧路径，
  不能把未来 specMain 路径写成已存在源码；必要时显式标作迁移目标。
- 后续每个批次交付同时检查受影响 checklist 的目录描述、逻辑边界和相互引用；
  暂留契约也在覆盖清单中，不能只更新被合并的项目。

## 规划阶段仓库观察（历史基线）

- 只读检查的 Kodex HEAD 为 `43888711`，内层工作区干净。
  排除外部 fork 和 build-logic 后，静态扫描得到 203 个非 root
  `build.gradle.kts`；不是本轮 Gradle/IDE 实际模型测量。
- 原静态审查基线的 [KGP 版本](../../Kodex/gradle/libs.versions.toml#L2) 为官方 `2.4.0`；
  既有工作明确不采用 KGP fork、不进一步裁剪 targets。
- 冻结文档时的交付复核发现并行工具链修改：工作区 KGP 改为 `2.4.20`、
  Koin plugin 改为 `1.2.1`、Gradle wrapper 改为 `9.6.1`，并有脚本与 yarn.lock
  变化。本轮未修改这些文件，不回退或接管该升级；原静态数据与抑制源码证据仍标注
  原基线。升级后来已提交；当前实施基线和 2.4.20 对照结果见首批检查点，
  不将 `2.4.0` 的机制证据写成当时已完成的 `2.4.20` 验证。
- [Host convention](../../Kodex/build-logic/src/main/kotlin/KodexHostKmp.kt#L22)
  与 [Shared convention](../../Kodex/build-logic/src/main/kotlin/kodex.kmp-shared.gradle.kts#L19)
  的 targets 不同；还有 Mosaic、posix 等共享源集层次。
  不能假定任意 spec/impl 配对具有相同平台覆盖。
- 当前 [Unified Exec spec](../../Kodex/tool/spec/unified-exec/build.gradle.kts)
  和 [impl](../../Kodex/tool/impl/unified-exec/build.gradle.kts) 是独立消费入口；
  impl 增加具体 Shell 依赖与 posix 层次。这种依赖区别不能只靠目录更名保留。

## 调研证据与纠正

- 同项目 `dependsOn` 带来上游声明可见性与依赖继承，不是跨项目的 spec-only
  消费入口。平台主编译会包含上游共享源码。
  见 [项目结构](https://kotlinlang.org/docs/multiplatform/multiplatform-advanced-project-structure.html)
  与 [Compilation 配置](https://kotlinlang.org/docs/multiplatform/multiplatform-configure-compilations.html)。
- 先前“commonMain 不能依赖其他源集，无法绕过”的表述过于绝对。
  正常配置确实产生 ERROR，但 KGP 内置的精确诊断抑制可绕过它；
  所选例外和官方 v2.4.0 fixture 链接已放入 canonical 源集设计。
- 本地官方 sources jar 的 `PropertiesProvider` 读取错误 ID 列表，
  `ToolingDiagnosticRenderingOptions.isSuppressed` 允许抑制该 ERROR，
  collector 在存储前返回；不是只隐藏日志。
  [官方抑制逻辑](https://raw.githubusercontent.com/JetBrains/kotlin/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/diagnostics/ToolingDiagnosticRenderingOptions.kt)。
- [底层 dependsOn 实现](https://raw.githubusercontent.com/JetBrains/kotlin/v2.4.0/libraries/tools/kotlin-gradle-plugin/src/common/kotlin/org/jetbrains/kotlin/gradle/plugin/sources/AbstractKotlinSourceSet.kt)
  检查生命周期与环后记录边，没有按 commonMain 名称硬拒绝。
- 关闭默认模板本身不能绕过诊断。KGP 2.4.0 在未显式应用模板的该分支中，
  仍为对应 compilation 的默认源集接入 commonMain/commonTest；
  不能把关闭模板理解为清空一切自动接线。
- 以上是规划阶段文档、源码与 fixture 核验；该阶段没有实际构建证据。
  后续首批编译/metadata 证据见实施检查点，IDE 与性能仍未验收。

## Parallel review ownership

- [Backend](../done/2026-10-12-audit-kmp-spec-backend.md)：agent-context/runtime/session/state/storage。
- [Frontend](../done/2026-10-12-audit-kmp-spec-frontend.md)：全部 app，含组件、根、迁移、持久化。
- [Protocols](../done/2026-10-12-audit-kmp-spec-protocols.md)：openai、rpc、mcp、hook。
- [Tools](../done/2026-10-12-audit-kmp-spec-tools.md)：全部 tool。
- [Utilities](../done/2026-10-12-audit-kmp-spec-utilities.md)：全部 utils。
- 协调者独立盘点全部注册项目、构建约定、跨领域消费者与候选合并图；
  子线只修改各自报告，不修改产品、主任务或其他子线文件。
- 用户已接受五条独立 Session：backend 774、frontend 775、protocols 776、
  tools 777、utilities 778。五份报告均已完成，协调者复核后归档；
  done 仅表示静态审查完成，不表示迁移或构建完成。

## Static inventory checkpoint

- 固定 `4388871137c56bd1b73d5b739cc98f036a790f3c`，内层 clean。
  解析 settings 的实际 moduleTree 声明，与磁盘 build 脚本交叉核对：
  203 个有 build 脚本的项目、12 个无脚本根命名空间，加 root 共 216。
- 91 个 spec 项目全部分配给审查线，当前没有发现遗漏或未注册 build 文件。

| 审查线 | Spec 项目数 | 合并候选涉及 spec | 保留独立项目 | 候选减少项目 |
| --- | ---: | ---: | ---: | ---: |
| Backend | 16 | 7 | 9 | 10 |
| Frontend | 31 | 28 | 3 | 28 |
| Protocols | 13 | 5 | 8 | 5 |
| Tools | 13 | 9 | 4 | 9 |
| Utilities | 18 | 2 | 16 | 2 |
| 合计 | 91 | 51 | 40 | 54 |

- 静态提取 896 条 project 引用及 convention substitution 记录；
  按源集分离测试边，生产关系去重后 652 条，包含 kRPC substitution。
  当前生产静态图无非平凡强连通分量；不是 resolved 依赖图证明。
- 其中 879 条是脚本文本的显式项目引用，另 17 条是使用
  `kodex.krpc-utils-patch` 的项目到 patch 项目的保守候选边。
  对 main/intermediateMain 与 Test 分开投影，以新 owner 替换顶点、
  删合并自边、去重，再求强连通分量。未求值插件或 Gradle variant；
  统计不能覆盖其动态生成边。
- 仅删目录中的 spec/impl 段、合并完全同名的候选会得到 167 个项目，
  但出现三组强连通分量。这个反例图不是最终迁移映射：
  - storage clean-models → tool image-generation/unified-exec/view-image →
    tool contract → storage clean-models。
  - app settings spec → app session → app rpc → app settings spec。
  - hook notification → rpc models → hook notification。
- 更粗的“同一个 spec/impl 父目录整体折叠”反例还会引入 runtime/session、
  app agent/history、RPC/settings 和 backend/tool 的环。
  不把该粗略 64 节点图当作可落地目标或项目数承诺。

## 独立 contract 与源集迁移的纠正

- 前次方案将“保留项目边界”和“保留原 source set”混在一起，过于保守。
  用户确认的策略是：独立 contract 仍独立，契约进入其自身 specMain；
  实现项目继续正常依赖它，不为合并而改写这些依赖关系。
- 核查基线为第一笔 spec 拆分提交 `f1a90006` 的父提交
  `66e6ba707771d1603ffbb2164c2d5f349102b40f`。以下历史路径均实际存在
  `build.gradle.kts`，不是根据当前目录名推测：

| 已确认历史独立 contract | 当前相关契约项目 | 处理原则 |
| --- | --- | --- |
| `agent-runtime/contract`、`agent-session/contract`、`agent-state/contract` | 各域 `spec/contract` | 保留根契约边界，内部契约迁 specMain；不并入组合实现 |
| `hook/contract` | `hook/spec/notification` | 保留独立契约/模型，内部迁 specMain |
| `tool/{image-generation,unified-exec,view-image}/contract` | `tool/spec/<同名>` | 三组保留独立契约，内部迁 specMain；实现仍独立 |
| `app/contract/agent`、`app/contract/session` | `app/component/agent/spec`、`app/spec/session` | 保留共享根契约，内部迁 specMain；重构中类型有调整，不声称逐文件完全相同 |

- 这验证了用户对主要成环项目的判断。历史上存在 contract 是边界依据，
  但不能据此反推当前每个“暂留项目”都有相同历史，也不等于所有历史 contract
  一律不能合并；具体合并仍以用户选定范围和当前依赖职责为准。
- 在独立 contract 项目中，标准 `commonMain.dependsOn(specMain)` 仍成立；
  没有实现时 commonMain 可以没有业务源码，不新造实现，不另建 implMain。
  这与“所有项目都用空 commonMain 绕开设计”的旧备选不同。
- 保留两个项目时，原 spec 与 impl 的不同 target 覆盖可各自保持，
  因此平台差异阻止的是无条件合并，不阻止 contract 项目内部改用 specMain。
- 剩余工作应区分：项目合并的依赖/平台约束、specMain 的依赖/插件接线、
  真正合并才出现的同名 internal/JVM 门面，以及 mixed 模块的声明归属。
  已归档五份审查的静态事实仍有效，其“保留旧源集”建议由本节取代。

## 联合项目合并候选（未冻结的执行清单）

- 本节只保留既有静态审查候选。架构冻结不等于这些单元、命名或统计已定稿；
  后续须逐项按冻结规则确认，不能直接将 50 单元/149 项写入全局规范。
- 首轮项目合并候选为 **50 个单元，涉及 51 个 spec**，另 40 个保留独立边界。
  40 不是“不迁移 specMain”的数量；其契约同样纳入源集迁移，混合代码按职责放置。
  Frontend application 合并单元含两个 spec。
- 按下表同时折叠所有领域：有构建脚本的项目 **203 → 149（减少 54，约 27%）**；
  静态生产图为 557 条不同项目边，**0 个非平凡强连通分量**。
  同路径文件检查无覆盖冲突；这不代替 Kotlin 符号、JVM 门面和编译器检查。
- 计数不含 12 个无脚本顶层命名空间和 root；保持这些注册时总数为
  216 → 162。它是规划图投影，不能推导 Sync/内存或编译耗时按比例改善。

### 精确合并单元

路径相对 `Kodex/`；Gradle ID 依现有规则为 `:` 加路径各段以 `-` 连接。
同一行的每个名字独立合并，只有明确列为多输入的单元一起合并。
未列入下表的项目不因这张表合并；独立 contract 的源集迁移按上一节处理。
逐 spec 完整清单及消费者见五份报告。

| 领域 | 旧输入 → 新目录 / 保留目标 | 单元数 / 减少项目 |
| --- | --- | ---: |
| Context | `spec/agents-md + impl/agents-md-filesystem → agent-context/agents-md`；`spec/skill + impl/skill-filesystem + impl/skill-render → agent-context/skill`；`spec/prefix + impl/prefix + impl/prefix-render → agent-context/prefix`；输入省略 `agent-context/` | 3 / 5 |
| Runtime | `agent-runtime/{spec,impl}/decorator/{compact,tool}` 各并为 `agent-runtime/decorator/<名字>`；`agent-runtime/{spec,impl}/steer → agent-runtime/steer` | 3 / 3 |
| Storage | `agent-storage/spec/contract + agent-storage/impl/filesystem + agent-storage/impl/in-memory → agent-storage/core`；不含 contract-ext、filesystem-layout | 1 / 2 |
| UI 常规 | `app/component/<名字>/spec → 同组件 impl/viewmodel`，原 VM 项目/目录保留；24 个名字见下一段，View 不并 | 24 / 24 |
| UI 特例 | `session-tab-bar/spec → 同组件 impl/view`；`app/spec/application + app/component/session-sidebar/spec → app/impl/application` 同批；`app/settings/spec/persistence → app/settings/impl/filesystem` | 3 / 4 |
| OpenAI | `openai/{spec,impl}/<名字> → openai/<名字>`；名字为 account-usage、client、client-test、codex-cli-storage、model-catalog | 5 / 5 |
| Tool | `tool/{spec,impl}/<名字> → tool/<名字>`；名字为 apply-patch、builder、current-time、get-context-remaining、multi-agent、plan、request-user-input、tool-search、web-run | 9 / 9 |
| Utility 首轮 | `utils/<名字>/{spec,impl} → utils/<名字>`；名字为 filesystem-lease、read-write-mutex | 2 / 2 |

- UI 24 个名字：account-usage、application-preferences、authentication-settings、
  composer、context-source-settings、history、history-index、hook-settings、
  mcp-settings、new-session、new-session-defaults、openai-login、path-picker、
  request-user-input、runtime-configuration、session-catalog、session-delete、
  session-rename、session-settings、session-title-settings、settings、
  suggest-subagent-task、usage-reset、working-directory。
- Utility 的 images-codec、ktor-client-ext、patch 虽平台一致、合并图无环，
  但原 spec 已含算法/SSE/IO helper，先确定哪些属于契约默认行为、哪些进入实现。
  保守首轮不计入它们；若后续批准三对合并，联合图可至 146 项且仍无生产 SCC。
  不因此制造空接口，也不将有向下依赖的函数留在 specMain。
- RPC contract + client 在子线中是可选项；首轮不采用，
  先保留多端口契约和 client/server/in-memory/patch 边界。
  所有新目录命名及上述合并范围均为提案，不变成 canonical 已确认规则。

### 必须保留或先处理的边界

本表的“保留”仅指项目边界；契约移入 specMain 不因此取消。

| 风险 | 已观察到的具体问题 | 首轮处理 |
| --- | --- | --- |
| 持久化/工具环 | `clean-models → 三种工具 spec`，handler impl 又 `→ tool-contract → clean-models` | image-generation、unified-exec、view-image 三对均不并；不是删多余依赖可解决 |
| 后端组合环 | runtime composition 经 decorator/session 回到 runtime contract；session impl 经 composition 回到 session contract；state impl 经 plan/context 工具回到 state contract | runtime/session/state 根契约保留；两个工具可并，但 state 不同时并 |
| 前端环 | Agent spec → History spec，History View → Agent spec；persisted Session impl → app RPC → Settings spec → Session spec | History 仅 S+VM，View 保留；Agent 和 app Session spec 保留；Agent spec 也不塞进 app RPC |
| Hook/RPC 环 | Hook notification impl → RPC models → Hook notification spec | Hook spec/impl 与 RPC models 保留 |
| MCP 混合职责 | stdio/streamable-http 已是具体实现；只并 contract+composition 会经 transport 回环；全部合并又有 Host/CLI target 差异 | 四个项目保留，不按名字臆造 spec/impl 对 |
| 平台不等价 | utility 六对 shared→host/cli；app migration host→cli；MCP host/cli | 保留独立项目，不删 JS/browser/wasm/iOS，不为迁移补新平台实现 |
| 源码冲突 | IO 两侧同包 `internal CoroutineIoSegmentByteCount`；Shell 两份 `StdoutBuffer.kt` 均有顶层成员，触发同名 JVM 门面条件 | 两库不入首轮；后续 IO 保留上游唯一常量，Shell 实现文件改名是候选修复，仍需编译确认 |

- utility 六对是 external-url、kotlinx-io-coroutines、logging、process-client、
  search-index、shell-client。原相同目标名也要检查 JS 环境和自定义 Native group；
  不能让 shared 的 iOS 因 `withNative()` 自动进入原 host 专用实现层。
- 无独立 impl 的 spec 不等于空契约：models、JSON codec、prompt DSL、
  coroutine/terminal helpers 等已有实际函数体。保留其项目边界，
  按实际契约/实现职责分配源集，不为凑层次拆出无意义接口。
- 多实现与 fixture 各有真实职责：storage 两实现同单元，client-test 仍是可消费
  的 main fixture 库；session-test、state-test、app-test-support-rpc 不并进 commonTest。
- 证据索引：后端报告 H1–H4；前端报告 §3、§5；协议报告逐项条目；
  工具报告 B1/B2；基础库报告 B1–B3。各报告均包含具体构建行与源码引用。

## 建议实施顺序与原子边界

原批次表是实施顺序提案，首批历史范围和结果见上文。
当前全仓 manifest 已定稿并执行，50 单元及 40 个保留项目都已处理，
每个合并单元包括所有输入和消费者 ID 更新，没有兼容转发项目。
新增结构设计仍须另行确认；全仓构建证据和剩余验收见当前实施章节。

### 每个合并单元的固定操作

- 原契约 `src/commonMain/kotlin` → 新 `src/specMain/kotlin`；
  原实现 commonMain、平台/intermediateMain 源码保持各自层次，不平铺。
- 原 spec 的依赖、必要 opt-in/语言设置配置给 specMain，
  impl 的库和生成源留 commonMain/平台层；保留插件并集但核查配置冲突。
- 两侧同名 test 源集汇入目标同名 test 源集，先检查文件/声明冲突；
  不新增 specTest，不把平台测试上移，不复制 build 下的生成文件。
- 整个项目图重定向旧 spec 和 impl ID，删除合并自依赖；更新 settings、
  测试任务路径及文档，不留下转发项目。
- 验证实际 source-set/variant/任务图、原 consumer 与测试集合；
  构建通过后再考虑独立批次，不靠批量替换后的静态图代替验收。

### 批次

| 批次 | 范围 | 进入下一批前的检查 |
| --- | --- | --- |
| 0 构建接线 | 已实现显式 opt-in `kodex.kmp-spec-impl`，根属性加入精确诊断例外 | 首批诊断对照、实际层次/metadata/测试接线通过；无契约的原 spec owner 不造接口，允许空业务 specMain |
| 1 最小真实试点 | 已迁移 read-write-mutex、session-delete VM 与独立 Hook contract，更新消费者 | shared/CLI、两种项目边界形式及上述实际构建检查通过；IDE/性能和其他宿主门槛仍保留 |
| 2 低耦合单元 | runtime steer；tool multi-agent/request-user-input、builder/current-time/apply-patch；OpenAI account-usage/model-catalog；其余简单 UI leaf | 序列化插件、规范 KDoc、原测试与所有直接调用者；先 leaf 再组合者 |
| 3 多实现和 I/O | filesystem-lease；context agents-md/skill 后 prefix；storage core；OpenAI client/codex-cli-storage/client-test；runtime compact/tool；其他获选工具 | 资源/取消/关闭、StorageUri expect/actual、平台库、fixture；根契约继续保留 |
| 4 组合 UI | 余下 24 组中的组合者；tab-bar；application+sidebar 同一单元；persistence+filesystem | 先 delete 再 catalog 等依赖顺序；RPC/CLI/integration/test-support 消费；History 不并 View |
| 5 保留边界及有条件后续 | 独立 contract 内部迁 specMain；三对 mixed utility 与额外合并需求另审 | 保留契约项目无需先解决合并环；不借源集迁移裁平台或重写 API |

- application 与 sidebar 两个 spec 必须与目标 application impl 原子合并，
  避免中途形成 application spec↔impl 环；storage core 两实现同理一起重定向。
- 整个首轮只改构建拓扑、源码所在 source set 和消费者项目路径。
  保留 package、公开签名、序列化名称、KDoc、数据布局、资源所有权及取消语义；
  不趁迁移重构业务、替换 RPC/DI，或统一 View/VM 工厂。

## 实施检查门槛

- 在现有官方工具链上选择一个真实领域与独立消费者，保持当前平台覆盖。
- 先投影候选合并后的项目依赖图，确认无新环；合法 impl 组合不作为失败项。
- 核对 commonMain 仅新增指向 specMain 的预期边、specMain 无反向边，
  且未因相同诊断 ID 被抑制而遗漏 commonTest 或其他异常接线。
- 做诊断控制：无抑制时报预期错误，有精确抑制时只解除该错误，
  其他必要配置检查仍生效。显式层次应保留全部原平台与中间源集。
- 按随后确认的契约规则，用反例验证同项目 spec→impl 的可见性、
  消费者 spec 对具体实现的引用、spec internal 与库依赖传播。
  不把合法 impl 依赖纳入禁止门禁，明确编译器与额外门禁的职责。
- 单独核对 specMain 的依赖、语言设置与生成符号；不假定原先注入
  commonMain 的依赖会逆向供给 specMain。
- 保留原 api/implementation 与 test/main 所在层，仅去掉合并后自指与同配置重复边；
  静态主图和测试图分别检查，不把 `test → fixture → main` 当生产环。
- serialization、Koin compiler、Compose、kRPC 在新增源集上分别验收；
  `rpc-impl-krpc-utils-patch` 的 substitution 与 Native moduleName 不改。
  migration 生成源/测试排除，既有 Native cinterop、平台 linker 参数原位保留。
- 对照同领域有多个 impl、spec/impl 平台覆盖不一致、expect/actual、
  serialization/RPC 生成、测试与 cinterop 的工程接线，不外推简单 JVM 示例。
- 分开测真实 IDEA Sync、导航、索引与内存，以及改 spec/impl 的编译失效；
  不以 `help`、项目数或 configuration-cache hit 代替开发体验验收。
- 第 0/1 批完成前后使用同一基线、JDK/Gradle/KGP、缓存条件和资源预算采样；
  分开记录首次 Sync、暖 Sync、impl 改动、spec 改动。结构合并可能扩大
  compilation/失效范围，项目数少不保证所有增量任务更快。
- 不在用户使用中的设备跑实验；构建前另确认隔离环境、锁与资源预算。
  默认测试通过不能证明被禁用的 unified-exec PTY/进程集成测试通过。

## 规划交付核对与当前剩余项

- [x] 91/91 spec 有归属、逐项清单、候选/保留理由与消费方审查。
- [x] 五份子报告完成，联合候选投影与反例、平台和符号风险已交叉核对。
- [x] 审查基线 `4388871137c56bd1b73d5b739cc98f036a790f3c` 未变；
  审查时工作区干净。本轮交付发现上述并行工具链改动，已保留且未介入。
- [x] 静态审查阶段仅文档规划，未运行构建；后续已实际实施并远程验证首批。
- [x] 用户明确必要的独立 contract 仍保留，其契约内部改用 specMain。
- [x] 架构与迁移规则已冻结；全局两份核心 checklist 已同步，
  领域 checklist 的预期改动和写入时机已列明。
- [x] 已获实施授权，首批 convention/目录/机器/预算已确定并验证。
- [x] 定稿并执行 50 个合并单元、40 个保留项目，全覆盖 91 个原 spec。
- [x] 完成全仓 compiler plugin/interop/真实消费者与测试的可执行验证，
  记录基线失败、真实凭据测试及不可用平台，不等同于全绿验收。
- [ ] 在用户允许的资源窗口完成 IDEA、性能和其他平台宿主验收。
