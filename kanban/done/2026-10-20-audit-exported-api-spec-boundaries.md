# Task Tree

- `Review the previous specMain migration and representative external calls`()
- `Record the confirmed exported API boundary for every business module`()
- `Evaluate lint integration against the cleaned product baseline`()
- `Configure and benchmark the authorized detekt experiment`()
- `Commit and push the configured lint and its validation record`()
- `Plan dependency-safe migration batches and acceptance checks`()
- `Migrate shared context and steer APIs and validate consumers`()
- `Migrate component creation functions and typed factory properties`()
- `Verify the first batches with lint and platform compilation`()
- `Inventory remaining modules and partial-platform declaration constraints`()
- `coroutineScope {`
  - `val ui = async {`
    - `Migrate public UI components with subagent`()
  - `}`
  - `val tools = async {`
    - `Migrate tool modules with subagent`()
  - `}`
  - `val rpcFrontend = async {`
    - `Migrate RPC frontend and NewSession with subagent`()
  - `}`
  - `awaitAll(ui, tools, rpcFrontend)`
  - `Review changes and run centralized serial validation`()
- `}`
- `Complete the uncontested migration subset and validation`()
- `Discuss and confirm client ownership of external URLs`()
- `Align external URL and dependent View contracts with client targets`()
- `Confirm independent backend logging modules and strict frontend boundaries`()
- `Split backend logging and preserve internal TUI diagnostics`()
- `Confirm MCP contract backend declaration and target manifest`()
- `Split MCP projects and verify frontend isolation`()
- `coroutineScope {`
  - `val toolSearch = async {`
    - `Split ToolSearch contract backend with subagent`()
  - `}`
  - `val imageCodec = async {`
    - `Withdraw five concrete image codec exports with subagent`()
  - `}`
  - `val sidebar = async {`
    - `Migrate Sidebar const declarations with subagent`()
  - `}`
  - `val cancellableInline = async {`
    - `Separate cancellable inline API with subagent`()
  - `}`
  - `Discuss remaining platform and declaration shape gaps`()
  - `awaitAll(toolSearch, imageCodec, sidebar, cancellableInline)`
  - `Run centralized serial dependency API and platform validation`()
- `}`
- `Discuss the remaining 34 API migration gaps with the user`()
- `Migrate four RPC inline APIs and validate consumers`()
- `Discuss the remaining 30 API migration gaps with the user`()
- `Migrate CoroutineCloseable use and validate cleanup consumers`()
- `Record the short declarative inline extension exception`()
- `Discuss the remaining 29 API migration gaps with the user`()
- `Review and validate approved Native and API batches`()
- `Discuss remaining 18 exported API gaps and the SafeRw contract shape`()
- `Migrate approved SafeRw inline extensions and validate on xiaoxin ubuntu`()
- `Review the remaining seventeen RPC patch and CLI exports`()
- `Implement the final three groups with subagents and accept centrally`()
- `for (batch in approvedRemainingMigrationBatches) {`
  - `Freeze dependency and platform preserving declaration shapes`(batch)
  - `Migrate declarations and validate actual consumers`(batch)
- `}`
- `Record zero structural findings and separate independent quality debt`()
- `Commit and push coherent product batches and scoped context`()

# Details

## 最终检查点与迁移记录 — 2026-10-22 / Session998

- 用户已明确要求剩余17项“subagent去实现，你验收”。
  分为[RPC server7项](2026-10-22-migrate-rpc-server-exports.md)、
  [kRPC补丁9项](2026-10-22-migrate-krpc-patch-exports.md)和
  [CLI入口1项](2026-10-22-migrate-cli-entrypoint-spec.md)三个独立写入范围；
  子任务不构建/提交推送，父Session复核后在xiaoxin-ubuntu集中串行验收。
  三组已交付并经父Session集中验收；旧存储失败与JS锁门禁不顺带修复。
- 当前结构迁移检查点：严格checkSpecApi和报告均17→0，未增加baseline/豁免、
  修改规则或恢复JVM产品目标。metadata/完整CLI metadata、Linux真实编译/调试链接、
  集成测试源码通过；165项Native全部通过（保留迁前162项并新增3项补丁边界回归）。
  53规则回归结果缓存命中且全部成功，不声称本轮重跑了规则测试。
- 补丁Native4 ABI逐字匹配迁前及固定0.10.3上游基线，RPC server Linux ABI也不变；
  opt-in消息/级别和marker计数/值保持。真实CLI隔离环境的两种help返回0、未知参数返回2，
  输出符合原入口且无环境写入；未启动TUI/后端或安装/发布release。
- 本轮1538份远端源码快照构建前后匹配；相对1529份基线只变13份旧路径、
  新增12份文件（含3份整文件迁移的目标），1516份范围外旧文件不变。
  验收时迁移工作树尚未提交推送。完成的是本轮源码契约分离与结构门禁，
  不证明结构lint未覆盖的全部语义、IDE导入或其他系统/架构运行验收。
  两项已复现的存储测试失败和JS锁门禁继续按独立讨论卡跟踪。
  汇总：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/final17-validation-summary.json`。
- 用户随后明确授权分批提交推送，产品已按依赖边界完成三批：
  `ee25c2898` 修复源码跨模块移动后的 lint 缓存失效；
  `564267468` 提交相互依赖的 Native 契约与模块迁移；
  `06deba22f` 更新开发说明。三批均已正常推送，远端 main 与最后提交一致，
  1538份产品文件仍与验收快照匹配，未为提交重跑构建或改源码。
  外层仅随迁移提交相关规范、任务记录及产品引用；其他任务改动留在工作区。
- 用户统一 [inline extension 例外](../../checklist/kmp-spec-impl-source-sets.md#short-declarative-inline-extensions)：
  非常短、代码几乎声明式地说清用途的 inline extension 可以直接实现在 specMain，
  不要求显式 contract，也不为分离额外加转发层。此口径替代此前“轻量封装类”的较宽建议，
  不自动放行整类及状态。初次讨论SafeRw时，readSession/writeSession是成员而非extension，
  最初按用户要求交由 subagent 尝试保留成员形状；
  不能自动改 API 形状或放宽例外，遇 contract 保留阻塞需回报。
  Session1036 初次返回阻塞回执，原生产代码未改，新7项 Native 消费者回归通过。
  用户随后批准[SafeRw任务](../done/2026-10-22-migrate-safe-rw-contract-api.md)将两个成员改为
  spec inline扩展，类型/构造入口spec声明、状态common actual；
  接受该项符号/import/显式泛型实参变化，不搬整类或底层锁算法。
  用户指定xiaoxin-ubuntu验收，后续本机未启动Gradle。
- SafeRw扩展迁移已完成：仅3份旧产品文件和1份新spec，原函数体、初始化、
  消费算法及断言保留。远端真实provider/消费者metadata通过，
  spec-only和spec+common外部val初始化证明通过；Linux15+契约7、Wasm15通过，
  Native4 provider及CLI/集成Native源码编译通过，53规则样例通过。
  严格/报告18→17项/3模块，只移除SafeRw；余下RPC server7、kRPC补丁9、CLI入口1。
  全存储116项仍有2项与既有精确基线一致失败，JS运行受锁文件校验阻塞；
  不称整仓全绿，未改断言/锁或绕过校验。本轮未提交推送。
- 用户明确 JVM 也移除，当前 CLI/backend 产品目标冻结为
  linuxX64/linuxArm64/macosArm64/mingwX64；五/六/八只是源码中的遗留配置，
  不是待确认的产品选项。不再以 JVM 或 Node 验收当作产品支持依据；
  Gradle/detekt 的 JVM 工具运行保留，Web/Wasm 前端规划另计。
  用户现已授权 subagent 实施相关 convention、实现库和消费变体链的目标收敛，
  并分离搜索索引工厂；与 SafeRw 及下一项 app/migration API 互斥写入，
  Session1040 已完成编辑，父 Session 已完成[集中 Native 验收](../done/2026-10-22-align-search-index-backend-targets.md)：
  143个KMP项目无JVM产品目标，135个Host/CLI/Client均Native4，
  8个Shared项目保留独立非JVM portable覆盖，12个前端Native依赖图隔离通过。
- 用户已确认并完成 [Home 准备与版本入口的2项迁移](../done/2026-10-22-migrate-home-preparation-api.md)：
  只分离 app/migration 导出 API，不执行或改写 Home 数据迁移。
  Native 四目标统一后不需要为此拆项目；子任务与 Session1040 互斥写入，
  Session1042 已交付，128项Linux Native测试通过，provider四Native编译通过。
  用户已批准并完成 [MCP backend 剩余8处导出迁移](../done/2026-10-22-migrate-mcp-backend-exports.md)，
  其契约留 backend 自己的 spec，不再扩大共享 mcp/contract。
  MCP31项Linux Native测试全通过，内部资源构造器不进入spec，公开构造与成员保留。
- 上一批严格/报告均29→18项/4模块，逐条只移除MCP8、Home2、Search1，
  无新增发现；393个真实源码输入、空分析classpath，53个规则样例通过。
  剩余是 rpc/server 7、rpc/krpc-utils-patch 9、app/cli 1、SafeRw 1；
  严格入口仍正确失败，整仓迁移尚未完成。
  完整CLI metadata、Linux编译/调试链接、RPC与集成Native测试源码编译通过。
  新debug CLI隔离HOME下 --help 返回0且无文件写入，未替换/发布release。
- 本批369项Native测试为367通过、2项存储失败；
  两项在1525份源码哈希匹配的本批迁移前工作区快照上精确复现，
  不称作干净HEAD。已单独[记录待讨论缺陷](../discussion/2026-10-22-investigate-native-settings-rescan-failures.md)，
  原存储算法/断言未改。搜索与Home四目标编译通过不代表其他系统运行测试通过。
  未提交推送；完整证据：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/native4-validation-summary.json`。
- 当时下一组只读核对为[RPC server的7处导出](2026-10-22-migrate-rpc-server-exports.md)：
  5个后端类型与2个资源作用域入口，候选为本模块自己的spec/actual，
  不再次拆共享协议项目；保持公开构造器、完整成员和资源生命周期。
  此组尚未实施，讨论时检查点为18项；随后SafeRw单项验收后为17项。
- 用户要求一次性看完剩余项，已完成[全部18项复核](2026-10-22-review-final-spec-api-gaps.md)：
  RPC server7、kRPC补丁9、CLI入口1均有不改规则的候选路线；
  SafeRw1现已确认成员改扩展，不采用整类例外。
  同版入口小样链接/运行通过，kRPC迁移前ABI基线与固定上游一致，
  SafeRw原类/Native/公共metadata消费对照完成；均非实际迁移验收。
- [CoroutineCloseable.use 已补齐并验收](../done/2026-10-22-migrate-coroutine-use-api.md)：
  用户以“那补”授权，expect suspend inline / actual suspend inline 保留完整关闭算法，
  接口及构建脚本不变。51项JVM/Node/Linux测试全通过，metadata、Wasm、
  CLI JVM/Linux及集成测试源码编译通过；53个规则样例通过。
  严格/报告实际30→29项/7模块，逐条只移除use，1521份范围外文件未变。
  当时剩余18项窄平台及关联API、1项SafeRw contracts、10项CLI/kRPC桥，
  不把严格仍失败的整仓迁移写成完成；未提交推送。证据：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/coroutine-use-validation-summary.json`。
- [四项 RPC 内联 API 已完成验收](../done/2026-10-21-migrate-rpc-inline-api.md)：
  函数体、重试/取消/返回值及异常还原语义保留，expect inline / actual inline 无转发层。
  两模块及真实消费者 metadata 通过，280 项 JVM、169 项 Linux 测试全通过；
  Native 消费测试源码、CLI JVM/Linux 与集成测试源码编译通过。
  严格/报告实际 34→30 项/8 模块，无新增发现，53 个规则样例通过；
  1467 份范围外文件未变，未提交推送。证据：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/rpc-inline-validation-summary.json`。
  当时剩余为18项窄平台及关联API、2项inline/contracts（use/SafeRw）、10项CLI/kRPC桥，
  不把严格仍失败的整仓迁移写成完成。
- 四组子任务已完成集中验收并分别归档：严格/报告 lint 42→34 项/10 模块，
  恰好移除图片 5、Sidebar 2、取消助手 1，无新增发现；53 个规则样例通过。
  12 项目24个 JVM 生产配置已排除搜索 backend、索引和 Lucene。
  JVM 729 项为726通过、3项已复现旧失败，JS/Wasm/Linux 共46项通过，
  metadata、Windows/Linux codec klib、MCP Native测试源码和CLI编译通过。
  保护检查1476份范围外文件未变；完整证据在
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/parallel-validation-summary.json`。
  未提交推送，整仓迁移未完成；剩余为18项窄平台及关联API、6项inline/contracts、10项CLI/kRPC桥。
- 已完成 [runCatchingCancellable 内联 API 分离](../done/2026-10-21-migrate-cancellable-inline-api.md)：
  expect inline / actual inline 通过真实metadata及跨模块挂起/非局部return验证；
  实现体不搬spec、不加转发层。其他带 contracts 的 6 项继续保留。
- 已完成 [Sidebar 两个布局 const 按源码兼容迁移](../done/2026-10-21-migrate-sidebar-layout-constants.md)：
  保留前端 View 的公开 Kotlin API、Int 类型、3/1 的值及 const/布局语义，直接迁至现有 spec。
  用户允许这两个旧 SessionSidebarKt 字段承载符号变化，不为其额外增加兼容层；
  此决定不扩展到其他 API。Session1030 编辑交付后集中验收通过，2项实际消除。
- 已完成[图片编解码 5 个具体平台对象收回导出](../done/2026-10-21-clarify-image-codec-exports.md)。
  用户明确选择五个实现对象改 internal，统一入口、Korim、注入及行为保持；
  Session1028 交付后由父 Session 集中验证，5项已实际清零；
  四文件仅五处可见性替换，剩余源文件与快照一致。
- 已完成 [ToolSearch contract/backend 拆分](../done/2026-10-21-clarify-search-index-ownership.md)：
  用户要求先按用途划边界，不先按技术可行性补 Wasm/iOS。
  实际服务后端 Agent 的 deferred 工具发现；前端只需共享事件数据，
  拆分前 clean-models 依赖混合的 tool/tool-search，运行图传递带入搜索索引和 Lucene。
  用户确认拆为 tool/tool-search/contract 与 backend：共享参数/结果等事件数据归 contract，
  引擎、索引输入/构建及 search-index 依赖归 backend，父目录不保留项目。
  Session1026 编辑交付后，由父 Session 集中完成两新模块和真实消费者验收。
  修正嵌套 moduleTree 误用顶层参数后，注册与实际生产闭包验证通过。
  平台覆盖保持原状，逐声明分类，父 Session 不并行修改其产品范围。
  utils/search-index 工厂的 1 项源集缺口仍需单独收口，不以此确认授权补 Wasm/iOS 或收回导出。
- MCP 项目拆分完成：用户确认并授权拆为 `mcp/contract` 与 `mcp/backend` 两个模块，
  mcp 仅作目录分组；前端和 rpc/models 只消费共享 contract，backend 单向依赖 contract。
  共享层不带 SDK、transport/process 机制或后端日志；后端自有 API 仍在 backend/specMain 声明。
  需逐声明划分共享模型/协议/纯逻辑与后端运行时，不能整目录搬现有 specMain；
  保留类型身份、wire/持久化形状及公开语义，不复制前端 DTO。
  [拆分验收](../done/2026-10-21-split-mcp-contract-backend.md)保留原 Host 六目标
  及 cliHost 五目标差异；8 个共享/前端项目生产编译和运行图无 MCP backend、SDK 或后端日志。
  492 项定向 JVM 为 491 通过、1 项在干净基线精确复现的字段清单旧断言失败，
  18 项 Node 及 metadata/CLI JVM/Linux 编译通过。
  lint 严格/报告仍为 42 项/13 模块；MCP contract 清零，8 个 cliHost 导出缺口继续保留。
  同时修正同内容跨模块移动的 lint 缓存路径输入，53 个样例及失效/命中检查通过。
- 已完成日志拆分：用户确认 logging 下放两个独立后端模块，按仓库现有根目录记为 `utils/logging/`，
  `backend-scope` 提供业务日志上下文扩展，`backend-file` 提供文件输出配置；
  两者不是 contract/impl，也不存在相互生产依赖，不沿用“通用日志抽象/文件后端实现”的建模。
  logging 仅为目录分组，删除原父项目而不保留兼容转发模块。
  现有文件日志测试组合调用扩展只是测试接线，不能据此制造生产依赖。
  用户选择同时收紧前端依赖，已完成[日志拆分验收](../done/2026-10-21-split-backend-logging-modules.md)。
  scope/file 分别保留原可移植/桌面能力覆盖；TUI 另有内部接线，不再调用后端日志初始化。
  两库无相互生产依赖，前端无直接日志库依赖或 scope 调用；
  日志项结束时混合 MCP 仍给应用/RPC 带来传递后端依赖；该缺口现已随 MCP 项目拆分解除，
  不将它继续列作当前债务。双模式 CLI 整体分发包及后端测试接线仍可带后端依赖。
  日志验收检查点 lint 42 项/13 模块，新模块均清零；412 项定向 JVM 测试 407 通过，
  5 项与已验证基线旧失败一致，Linux file 另 1 项通过。
- 用户逐项确认 external-url 按客户端目标实施，见
  [已完成子任务](../done/2026-10-21-align-external-url-client-targets.md)；
  不采用按后端宿主拆 system 的方案。本轮客户端沿用已落地 JVM/四 Native，
  View 与外链共用客户端配置；浏览器目标和真实实现留 Web 前端工作。
  外链项结束时严格/报告 lint 为 43 项/14 模块；该项三个模块无发现。
  外链 JVM/Native 6 项和登录 View 9 项通过，根 View 仍是两项完全匹配干净基线的旧失败。
  以下 46 项为无争议补迁结束时的历史检查点；外链项结束时剩余分类为
  窄平台及关联依赖 24、inline/contracts 7、CLI/kRPC 桥 10、Sidebar const 2。
  日志项后窄平台及关联依赖降为 23，其余分类不变，总计 42。
- 用户进一步要求先实际修好无争议部分，再讨论剩余部分；本轮不止规划或派发。
  已接受四个并行编辑 Session，均禁止构建、提交推送及越界修改；
  父 Session 处理 MCP 共享声明和集中串行验收；该子批次已交付并完成集中验证。
- [无争议子批次](../done/2026-10-21-migrate-uncontested-spec-api.md)：
  Views [1013](file:///home/stream/.kodex/sessions/1013)、
  utils/tool [1014](file:///home/stream/.kodex/sessions/1014)、
  Agent/OpenAI [1015](file:///home/stream/.kodex/sessions/1015)、
  RPC/应用装配 [1016](file:///home/stream/.kodex/sessions/1016)。
- 用户选择暂缓平台拆分。少量受限 system 子模块仅为候选方案，未获确认；
  不新增此类模块，除已单独确认的 external-url 客户端边界外不调整目标集合，
  不以 stub、baseline 或 lint 豁免处理窄平台导出。
- 无需改变上述边界的普通声明、共享 provider 和组件 View 已完成本轮补迁。
  共处理 86 个模块；结构发现 367/93 模块降至 46/17 模块，
  工具自有导出已清零。详细编译、测试限制和剩余分类见子批次回执。
- 实现体引用未迁移 provider 是既有依赖债务，不自动阻止本地声明分离；
  签名和默认值需要的类型/入口则须先保证 spec 可见、依赖和目标闭合。
  不因暂时没有调用者就收回已有导出。
- inline/contracts 单列声明形状与消费者编译验证，保留非局部返回、
  callsInPlace、取消及异常语义；不为过 lint 改为普通回调或搬入完整机制。
  CLI/框架桥接入口单独核查启动和符号兼容。
- host 搜索工厂等其余窄平台缺口继续显式保留；
  这不是已通过检查。两个已复现的旧录制失败不混入本轮 API 分层整改。
- 无争议补迁结束时剩余 46 项：27 项窄平台及关联签名/defaults、7 项 inline/contracts、
  10 项 CLI/kRPC 框架兼容入口、2 项 Sidebar const JVM 字段承载类兼容。
  严格 lint 仍失败；下一步讨论这些声明形状，不把本子集归档写成整仓完成。
- 本轮定向 1778 项测试最新结果为 1773 通过、5 项已在干净基线复现的失败；
  通知 Hook 测试编译错误也已复现。租约首次超时重跑通过，原因未定；
  完整证据和非 JVM 编译覆盖见子批次，不宣称全仓测试全绿。

## 并行子任务 — 2026-10-21 / Session998

- 本节保留上一轮 Sessions1004–1006 的历史检查点；工具后续补迁与当前
  剩余状态以上节及已归档子任务为准。
- 用户要求用 subagent 迁移，已接受三个普通 Session；独立上下文提示已包含
  规范、脏工作树保护、路径所有权、禁提交推送和禁并发构建约束。
  三个 Session 的最终回执已读取；均未运行构建，不将编辑交付当作验收通过。
  UI/RPC 交付范围内迁移，工具组只完成安全子集，剩余 provider 依赖债务仍需处理。
- `Migrate public UI components with subagent`：
  [UI 子任务](../done/2026-10-21-migrate-spec-api-ui-components.md)，
  [Session1004](file:///home/stream/.kodex/sessions/1004)；
  唯一写范围 `Kodex/app/view/components/**`。
- `Migrate tool modules with subagent`：
  [工具子任务](../done/2026-10-21-migrate-spec-api-tools.md)，
  [Session1005](file:///home/stream/.kodex/sessions/1005)；
  唯一写范围 `Kodex/tool/**`。
- `Migrate RPC frontend and NewSession with subagent`：
  [RPC 前端子任务](../done/2026-10-21-migrate-spec-api-rpc-frontend.md)，
  [Session1006](file:///home/stream/.kodex/sessions/1006)；
  唯一写范围 `Kodex/app/rpc/**`、`Kodex/app/component/new-session/viewmodel/**`。
- 三者可读全仓，但不改共享根配置、convention、lint、版本、目标或其他任务记录；
  先前 26 模块的改动保持。子任务只编辑和静态检查，不运行 Gradle/编译器，
  不操作 IDE/进程，不再派生 subagent。
- 父 Session 在编辑稳定后集中复核及串行验证 metadata、实际消费者和测试，
  再更新 lint/性能与剩余缺口；窄平台或跨范围架构阻塞先记录，不放宽规则或补 stub。
- 父 Session 复核修正了无初始化 const 的不可编译 expect 声明、Compose getter
  注解目标、sealed DSL 的源集归属；把 popup 定位机制移回 mosaicMain，
  去掉 RPC spec 中的私有 owner/widths/invalidation 构造细节。
  常量保留在公开 spec 对象，运行逻辑经 internal expect/actual 实现绑定，
  不把 const 降级为普通 val；增加编译期常量回归样例。

## 子任务集中验收 — 2026-10-21 / Session998

- 父 Session 已修正并验收本轮十个变更模块：354 项 JVM 测试全部通过，
  无失败/错误/跳过；其中 UI 167、RPC 前端 107、NewSession 5、七个工具模块 75。
  新增四个常量回归样例，覆盖常量仍可用作 const 初始化表达式。
- 全部变更模块的 spec/common metadata 编译通过；
  UI、RPC、NewSession、WebRun、UnifiedExec 的 Linux X64 klib，
  CurrentTime 的 JS 编译通过。不冒充其他 Native 目标的运行验收。
- `app-view`、`app-application`、`app-session`、`agent-runtime-composition`
  和 integration-test 测试源码编译通过，没有运行真实模型集成测试。
- `app-view:jvmTest` 为 68 项、2 项旧 patch 录制失败。对干净 `d212c43cf`
  用临时源码导出重跑旧 suite，6 项中同样 2 项失败；名称、类型、消息、源码位置
  完全一致，测试源码哈希与基线一致。不修改旧测试，不把该全量任务写成成功。
  [逐项对照](file:///home/stream/ACodeSpace/demo/kodex-detekt-998/subagent-baseline-comparison.json#L1)。
- 该轮结构 lint 为 367 项/93 模块，比子任务派发前 535 项减少 168 项；
  UI、RPC、NewSession 及四个完整工具子集均 0 项。工具另有三个模块仅部分迁移，
  provider 和窄平台声明债务继续由本卡后续批次与工具子任务跟踪。
  严格入口退出 1，报告/严格结果一致，无自动 suppress/baseline。
- 最新缓存命中的两次强制全扫：墙钟 2.520/2.072 s，分析 1.499/1.198 s；
  无变化报告加规则测试 1.325 s、分析 UP-TO-DATE。首轮含配置/工具冷加载
  17.617 s（配置 6.639、分析 5.998）；与历史基准不作受控回归比较。
- 最终命令及资源回执为 `subagent-jvm-final`、`subagent-metadata-fixed`、
  `subagent-native-js`、`subagent-lint-*`，位于已有 demo 证据目录。
  所有构建复用 JDK25 daemon 并单 worker；临时基线目录已清理，
  不修改全局配置、停止用户进程或新建 worktree。迁移仍未提交推送。

## 状态与确认

- 2026-10-20 / Session998：用户关切所有对外 API 是否已尽可能通过 specMain
  与实现分离，并明确即使承担 impl 职责的模块，也要在模块内部区分
  specMain/commonMain，相当于导出与非导出 API。
- 用户进一步明确：只有 specMain 可以声明 public API，其他生产源集
  要么是对应的 actual 实现，要么是 internal/private。隐式 public 和暂无
  调用者的导出同样纳入检查；撤回此前“不普遍采用 expect/actual”的限制。
- 用户提出优先考量 detekt 降低实现复杂度，并要求检查性能不要太差。
  将框架复用、配置开销、暖执行耗时、无变化跳过和内存作为方案验收项。
- 已更新[源集职责](../../checklist/kmp-spec-impl-source-sets.md#source-set-responsibilities)
  和[模块边界](../../checklist/spec-impl-module-boundaries.md)。
  用户已授权本机配置与测试 lint，见
  [已完成的配置与测量子任务](../done/2026-10-20-configure-detekt-spec-api-lint.md)。
- 后续用户明确要求先提交并推送本轮 lint 配置，成功后立即开始 API 迁移。
  按依赖与平台边界分批实施并验证，不以 lint 问题数归零替代编译验收；
  不包含版本发布或其他并行任务。
- 配置提交与推送已完成并核对远端：产品 `d212c43cf`，外层 `178910e`。
  本卡进入 executable，首批冻结为 `agent-state/context-window`、
  `agent-runtime/steer`、`agent-context/prefix`：共享层覆盖原模块全部目标；
  保持现有包名、构造入口、签名、KDoc 与行为，不新增适配接口或改变调用者。
  函数使用 expect/actual；现有具体类使用 expect class 与 actual class，
  私有状态与算法留在 commonMain，契约依赖上移 specMain。
- 首批验收：三个模块 metadata/JVM 编译、现有 JVM 测试与实际消费者编译，
  整仓报告确认首批无结构违规；任何编译失败区分本轮引入与干净基线失败。
  其他批次按依赖和平台范围继续细化，不承诺首批即完成整仓迁移。
- 第二批为已有独立 spec 模型的组件 ViewModel 创建函数与工厂属性：
  Composer、RequestUserInput、SuggestSubagentTask、HistoryIndex，以及 Settings
  和其函数式子组件入口。只提取声明与原 KDoc，维持函数/属性名和包名；
  不替换具体实现或引入额外接口。验收 scoped JVM 测试、metadata 和组合消费者编译。
  同批覆盖 DirectoryPicker、WorkingDirectory、UsageReset、RuntimeConfiguration、
  SessionCatalog 和 History 的现有入口；NewSession 依赖尚未迁移的 app-rpc
  具体类型，留待 RPC 契约批次处理，不通过 internal 化隐藏现有 API。
- 旧迁移结项说明源集和布局的历史完成状态，不证明已经满足本轮更严格的
  导出 API 覆盖要求；不改写旧验收证据。

## 首批迁移检查点 — 2026-10-20 / Session998

- 已迁移 26 个模块：三个共享 Agent 模块和 23 个组件 ViewModel 模块。
  新增 29 份 spec 文件、40 个顶层 expect 声明；公开类/对象的成员也显式匹配。
  原包名、入口、参数与行为保留，默认参数移到 expect，私有状态与算法仍在 commonMain。
- 三个 Agent 模块及上述 23 个组件的 spec/common metadata 编译全部通过；
  现有 JVM 测试合计 411 项，失败、错误、跳过均为 0。
  `agent-runtime-composition`、`tool-get-context-remaining`、`app-rpc`
  三个真实消费者的 JVM 编译通过。
- 补充非 JVM 编译通过：Steer 的 JS 与 Linux X64 klib、Composer 的 Linux X64 klib。
  这不是所有目标的运行测试或完整 CLI 链接验收。
- 去掉迁移标记及移动的 KDoc 后，回查全部已修改 commonMain 文件：
  除已确认迁往 expect 的 Catalog 构造默认值外，实现 token 流未变。
  类/对象使用文件级 beta-warning 确认，不压制 actual 匹配诊断或 lint 规则。
- 整仓结构问题从 575 降为 535，仍涉及 100 个模块；已迁移 26 个模块均无发现。
  严格入口实际退出 1，不创建 baseline 或添加 lint suppress。
  规则测试 53 项保持通过；报告任务成功不代表整仓整改完成。
- 迁移后单次性能回归：configuration cache 命中的强制全扫墙钟 1.178 s、
  分析任务 0.703 s；无变化报告加规则测试 0.516 s、分析 UP-TO-DATE。
  首次组合命令含配置与分析冷加载为 9.159 s；这些是单次回归，不替代原中位数。
- 完整日志与任务参数在 `~/ACodeSpace/demo/kodex-detekt-998/migration-*.{log,json}`。
  重点回执为 `migration-factories-retry`、`migration-core-metadata`、
  `migration-components-metadata-retry`、`migration-non-jvm`、`migration-lint-strict`。
- 四次验证因可用内存低于约 1.5 GiB 自动取消，均不计通过；其中两次受 IDE Sync
  并发影响。后续单 worker、复用 IDLE JDK25 daemon；一次在空闲时触发 GC
  释放本轮 in-process 编译临时内存后重试成功。不停止 IDE/JDK26 或改全局堆设置。
- 配置提交已推送；本批迁移仍是未提交工作树改动。无临时产品探针或新增工作树。

## 迁移前只读抽查

- `agent-state/context-window` 的 `tokensUntilCompaction` 在 commonMain，
  被 `tool/get-context-remaining` 和 `agent-runtime/decorator/compact` 调用。
- `agent-context/prefix` 的 `AgentContextPrefixResolver` 在 commonMain，
  被 `agent-state/core` 直接构造；spec 中的结果模型不替代服务契约。
- `agent-runtime/steer` 已有 `SteerProvider` spec，但 composition 模块仍直接
  构造 commonMain 中的 `StateFlowSteerProvider`，创建入口需分离。
- Composer 已有 factory spec，但 app/rpc 仍直接调用 commonMain 中的
  `createComposerViewModel`；需要审查实际调用入口，而不只检查接口是否存在。
- 以上是迁移前的生产调用抽查；这些入口已纳入首批迁移，
  不是完整符号解析或全仓缺口清单。

## 待确定与审查检查

- 盘点模块自有公开入口及真实跨模块引用，覆盖工厂、扩展、属性、类型成员、
  签名依赖、规范性 KDoc、隐式 public 和暂无调用者的导出；检查生成的公开
  声明及平台源集，不以仅扫描显式 public 关键字替代可见性检查。
- 逐类验证 spec 声明与 actual 的匹配方式和工具链支持；在 specMain 中
  已完整声明的接口/值模型不机械改成 expect。内部类型实现接口的 override
  不构成独立导出；公开实现类或独立工厂则不能只以“已有接口”为由豁免。
- 规范已确认，但本轮不以文档决定代替编译验证，也不通过批量制造接口、
  转发层或搬入实现体来满足目录检查。
- 保留已有 partial-platform 目标范围，为窄平台导出明确契约声明位置；
  不把全部 API 无条件扩到 specMain 的宽目标，不补无能力的 stub。
- 完整方案需包含引用回查、可见性检查、metadata/真实消费者编译和适用测试；
  单纯 convention 覆盖或文件迁移数量不能证明导出契约已闭合。
- 普通 RPC、存储、Mosaic View、工具和基础库已在后续子批次补迁；仍保留的窄平台入口如
  `createSearchIndex` 的现有实现目标
  小于其模块 specMain 目标，不能直接上移 expect 后声称保持覆盖；
  后续需冻结保目标的模块/声明边界，不能静默新增平台 stub 或放宽 public 规则。

## Lint 配置调研 — 2026-10-20 / Session998

### 基线与范围

- 用户清理后的产品子模块工作树干净，HEAD 为
  `fc36bce2b1d20f1ea1bd5b18db6053714c4740b6`；外层仍有既有文档改动，
  不视为用户授权清理或重写。
- 清理后的初始业务 convention 已启用 explicit API，但当时没有产品 Kotlin lint。
  四份现有 GitHub workflow 都是 fork package 工作流，不把本门禁塞进发布流程。
- Git 跟踪输入中有 668 份生产 Kotlin 文件，分布在 20 种 Main 目录，
  其中 specMain 为 250 份。这是静态目录盘点，不是已求值的 Gradle 源集模型。

### 已验证的配置与覆盖边界

- 已用 detekt 自定义规则承载 `checkSpecApi`，复用其规则加载、测试、报告和
  Gradle 任务设施，不自建 parser runner。实际接线集中在根 build.gradle.kts，
  读取真实 KMP main 模型，不给每个业务 convention 注入一套 detekt 插件。
- 根入口使用一个聚合 Detekt 任务，生产文件去重后批量分析；
  不在每个业务模块机械套用整套插件、为每个 source set/target 生成重复扫描。
- 结构规则先使用 light/no-type-resolution 模式，只启用自定义边界规则：
  `disableDefaultRuleSets=true`、`allRules=false`、`ignoreFailures=false`。
  不提供产品 analysis classpath；detekt 工具及自定义规则依赖仍需正常解析，
  不能误把它们清空。初始 `parallel=false` 仅关闭并行 AST 构建，不代表框架
  完全串行；实测后保留串行默认和显式并行开关。
- 固定 detekt 版本并保留其匹配的 compiler runtime，不强制对齐到产品
  Kotlin 2.4.20。先验证本仓语法和规则测试兼容性；确有阻塞才评估独立检查器。
- 输入来自真实 KGP 生产 compilation/source-set 图及其文件过滤规则，而不是
  `src/commonMain` 的固定 glob。覆盖 mosaic、cliHost、posix 等中间层；
  消费型 fixture 项目的 Main 也属于生产 API，不按模块名含 test 排除。
- 工具输入包括文件内容、模块/源集归属、规则及工具版本；优先复用 detekt 的
  控制台诊断和一种机器可读报告，不默认生成全部报告格式，
  适配 configuration cache、up-to-date/build cache。任务执行期不访问 Project 模型，
  不解析产品依赖或触发产品编译/link。生成源码须关联实际 producer，不能把不存在的
  生成文件记成已检查；编译器插件合成的声明另列覆盖范围。
- lint 检查源集/公开声明边界；编译器校验 expect/actual 的语言匹配及目标覆盖。
  有效可见性、继承、actual 成员和 expect 来源若需要符号解析，应明确实现与覆盖，
  不以“出现 actual 关键字”冒充完整对应关系验证。源文件结构检查与需要完整
  classpath 的语义检查分开；后者不能借用“不解析产品依赖”的轻量任务承诺。
- 独立任务与正反例已完成，测量和缓存检查见上述子任务；首轮报告为 575 个结构问题，
  不是完整 API 审计结论。完成迁移后再单独确定 check/独立 CI job 接入；
  不自动添加 suppress 或生成掩盖现状的 baseline。
  测量阶段未迁移业务 API 或接入 CI；后续迁移状态见本卡检查点。
- 默认 invoker 只按路径缓存类，已用 Gradle 内容寻址 artifact transform 保证规则重载；
  不用每次都启动 Worker JVM。最终默认串行，保留并行开关；详见
  [测量回执](file:///home/stream/ACodeSpace/demo/kodex-detekt-998/README.md#L1)。

### 性能验收

- 分别测量依赖就绪后的冷执行、暖执行、单文件改动后执行和无变化 no-op；
  下载/规则工程首次编译、Gradle 配置、实际分析耗时分项记录，同时保留端到端数据。
- 无变化应复用任务结果而不再次分析；配置/规则/源集归属变化必须正确失效。
  up-to-date 不等于单文件改动时工具会只分析该文件，后者不作未经验证的承诺。
- 暖执行的分析部分以数秒级为优化目标，不作为已验证数字或用户确认的硬阈值。
  在本仓规模下比较并行 AST 开关、内存和 CPU 开销，控制启动和重复解析成本。
- 结构检查保持轻量；完整语义验证若需要产品 classpath 或编译，单独列出成本与
  覆盖门禁，不通过跳过难例来换取“全规则高速通过”。

### 方案取舍与验证证据

- detekt 能承载自定义规则，但当前[兼容矩阵](https://detekt.dev/docs/introduction/compatibility/)
  列出的最新组合为 2.0.0-alpha.6 / Kotlin 2.4.10，而产品固定 2.4.20；
  这不是不兼容证明，但需要额外验证。其默认 KMP
  [类型解析任务](https://detekt.dev/docs/gettingstarted/type-resolution/)
  只覆盖 JVM/Android，不能据此声称已语义检查 Native/JS/Wasm。
  这不阻碍所有生产源集使用结构规则，亦不足以证明自建框架更合适。
- detekt 的[自定义规则设施](https://detekt.dev/docs/introduction/extensions/)
  和[Gradle 配置](https://detekt.dev/docs/gettingstarted/gradle/)
  可复用，不再把框架启动、报告和隔离机制作为必须自建的工作。
- 以下独立 PSI 探针保留为历史技术证据，不代表当前优先方案仍是自建 parser。
- 早期探针阶段本机已有用户 IDE/Gradle/Kotlin 进程，未构建产品或停止这些进程。
  使用已缓存的 Kotlin 2.4.20，在独立 Java 进程、`-Xmx256m` 下解析 6 组样例：
  显式/隐式 public、内部类 override、actual 函数、actual 类的额外公开成员、
  构造属性与 companion。全部完成 PSI 解析；未执行规则或语义匹配。
- 探针前两次启动分别误取 stdlib 的 metadata variant、缺少新版 extensions storage
  初始化；改用 JVM runtime variant 并按 2.4.20 API 初始化后成功。
  无仓库文件或临时文件写入，进程已退出，环境已 dispose；这些结果只证明解析入口，
  不证明实际 Gradle 接线、规则准确性、配置缓存或任何目标编译已经通过。
