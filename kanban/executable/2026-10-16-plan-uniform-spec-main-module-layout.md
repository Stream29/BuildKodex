# Task Tree

- `Load the confirmed source-set rules and current migration inventory`()
- `Confirm consolidation, platform research and impl path cleanup scope`()
- `Map every current project and identify cycle-breaking contracts`()
- `Project the combined production graph and inspect source collisions`()
- `Define prerequisite experiments, atomic batches and acceptance gates`()
- `Review the target boundaries and unresolved platform gates with the user`()
- `if (implementationAuthorized()) {`
  - `Freeze current inputs and acquire an isolated validation environment`()
  - `Prove partial-platform metadata and JS environment isolation`()
  - `Release idle P0 processes while preserving experiment evidence`()
  - `Approve the three system module boundary correction`()
  - `if (revisedPlatformBoundariesAreSupported()) {`
    - `Freeze the executable manifest and source-set ownership map`()
    - `Cut over the complete owner graph and source paths atomically`()
    - `for (batch in approvedBatches) {`
      - `Migrate the batch and all consumers without forwarding projects`()
      - `Validate graph, platform inputs, generated code and scoped tests`()
    - `}`
    - `Validate the whole repository and present acceptance evidence`()
  - `} else {`
    - `Preserve failed evidence and obtain a revised boundary decision`()
  - `}`
  - `Release owned validation resources`()
  - **`Await human IDE import, navigation and interactive UI acceptance`()**
- `}`

# Details

## 授权、基线与状态

- 用户已审阅规划并明确授权执行迁移，指定 Gradle 等重量级操作在 remote MacBook。
  任务进入 executable；用户随后明确授权提交、推送，仍不包含发布或接管用户 IDE/现有远端工作目录。
- 先执行 P0：在 remote MacBook 私有目录验证 B1/B2，再冻结跨平台逐文件清单。
  原“不实施/待审阅”段落是规划阶段检查点，不代表当前授权；
  若门禁要求超出已选方案的设计变更，保留证据并回用户确认，不静默绕过。
- Session 878 已冻结当前工作树的 1315 个自有文件（含未提交与未跟踪业务输入）；
  本机证据目录 `~/ACodeSpace/demo/kodex-spec-main-878/`，
  远端私有目录 `~/ACodeSpace/local/kodex-spec-main-878/`。
  本机 IDEA 打开 BuildKodex，不触发本地 Sync/构建或控制其窗口。
- remote MacBook 初检无活跃 IDEA/Gradle/Kotlin daemon，24 GiB 内存、swap 0。
  复用既有 device-heavy.lock，私有 APFS clone 依赖/Native 缓存，
  使用安装的 JBR 25.0.4、Gradle 4 GiB / Kotlin 2 GiB、单 worker；
  保留 4 GiB 可回收内存与 512 MiB swap 增量守卫。首次没有 daemon，
  仅启动此 JBR 的本任务 daemon，后续调用显式使用同一 JVM。
- 已确认：所有业务 KMP 模块使用 specMain，Gradle 模块不以 spec 分类；
  真正为破环保留的独立契约叫 contract。
- 用户选择八项归并、全仓 impl 分类目录清理和跨平台同模块深入验证。
- 规划静态基线为 Kodex `03212868100c005850d4c5583fe9278065040cc6`，
  Gradle 9.6.1、KGP 2.4.20、Java 25；未提交输入另有完整 hash 冻结。
- [前轮扫描](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-16-audit-spec-module-migration-leftovers.md#L1)
  保留历史实物清单；本任务持有后续映射、门禁证据与执行状态。

## 当前实施检查点 — 140 项迁移已落地，等待人工验收

- 已实施完整 owner 图与源码路径切换：140 份业务构建脚本统一显式接入源集
  convention，删除 15 份被归并项目的构建脚本，所有消费者直接使用新坐标。
- 1120 个模块内非构建输入逐项 SHA-256 与冻结工作树一致，900 个文件搬迁，
  包含原未提交的 model-catalog/RPC server 修改及新增测试；未改 Kotlin 源码内容。
  逐文件证据见
  [source preservation](file:///home/stream/ACodeSpace/demo/kodex-spec-main-878/source-preservation.json#L1)。
- 新增 cliHost/desktopNative 窄层 convention，并将原 CLI 的 Native network
  pin 提取为可复用 convention；版本、目标、CLI 命名和 kRPC unique name 不变。
- remote MacBook 全仓 `help` 已通过（57.6 秒，无 guard stop/swap）；
  首轮 metadata 因私有包仓库缺少用户名失败，尚未进入编译。
  后续以 secret-free init 复用设备既有包访问凭据，仅读取固定依赖，不复制或打印密钥。
- 全仓 configuration check 与 metadata 编译通过（3 分 18 秒，1695 tasks）。
  两份真实模型逐文件比较：1114 个源码/资源文件所属源集的 target/compilation 集合不变，
  另外 4 个 Native interop 定义及 patch 的 LICENSE/README 字节不变；140 个项目均为
  commonMain→specMain 且每个原 test compilation 仍关联 main。
  业务依赖 scope 按合并/窄层映射一致；KGP 自动 stdlib/DOM 与测试 convention
  输入单列归一化，不把懒注入差异误报为业务依赖变化。
  这是求值后的源集覆盖比较；历史测试的显式 exclude 仍保留，不把被排除文件记为已执行。
  [模型对照](file:///home/stream/ACodeSpace/demo/kodex-spec-main-878/model-comparison.json#L1)、
  [求值声明图](file:///home/stream/ACodeSpace/demo/kodex-spec-main-878/evaluated-production-graph.json#L1)
  为 140 节点、556 边、0 环，含 kRPC 替换规则的保守边，不冒充全部 resolved variant/task 图。
- 全仓 JVM 生产编译和除 integration-test 外的测试编译通过；
  integration-test 在
  [ApplicationRootBoundaryTest.kt:288](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/ApplicationRootBoundaryTest.kt#L288)
  仍调用旧四参数
  RpcGlobalSettings.open，产生三条参数错误。冻结的迁移前工作树在同一 Mac/JVM
  完整复现相同三条错误，属既有测试缺口，不改业务或冻结测试掩盖它。
  其余 139 个模块的默认 JVM 任务已执行：118 个项目产生报告，
  2632 个用例中 2619 通过、13 失败；失败分布在 7 个项目。
  冻结基线对这 7 项复跑 261 个用例，同样 13 失败；
  用例名、异常类型和诊断消息全部一致，仅归一化私有路径与动态 /proc FD 号。
  不运行 integration-test 的真实账号/联网模型测试，不启用原默认禁用的 real IO/性能探针。
- 140 个 allMetadataJar 均通过；JS/Wasm main/test 与 iOS main 编译已执行，
  唯一失败为 JS integration 的 Compose Runtime 缺失；冻结基线同样复现。
  首次专项 Node 任务列表误含无 JS target 的 RPC 项目，属于验证命令选项错误，
  未进入编译；已按真实 target 模型纠正，不为补验证任务扩大 RPC 平台。
- Node 专项 10 个项目共 201 用例：199 通过、IO system 的 2 项失败；
  原 IO impl 在冻结基线复跑 11 用例，同样 2 项失败，异常与诊断逐项一致。
  真实 headless Chrome 的 JS Browser 18 用例全部通过；
  Wasm Browser 的 logging 8 用例通过，Shell 的启动因
  `Cannot use 'import.meta' outside a module` 失败，旧 Shell spec 在基线同样复现。
- Mac Native CLI 编译、链接通过；13 个归并相关项目共 436 个 Native 用例，
  435 通过、RPC models 的 1 项失败。旧 RPC models 在基线复跑 31 用例，
  同一用例、异常、诊断一致。上述用例对照均不以“源码没改”替代真实复跑。
- Native CLI 在全新 HOME/USERPROFILE 中完成 help、启动 backend、等待 ready、
  多次控制连接使用同一实例、reload、stop 和退出后 not_running 检查。
  所有控制命令退出 0，backend 正常退出，无剩余 backend；
  不把这项控制面 smoke 写成实际 TUI 渲染/业务重连已验收。
- 配置缓存与增量：对 `:mcp:compileKotlinJvm --offline` 做 prime/no-op、
  utils/patch 的 specMain 与 commonMain 注释级输入变化、恢复和最终 no-op。
  5 次后续调用复用 configuration cache；两次编辑均触发对应 producer 编译，
  ABI 未变的 MCP consumer 保持 UP-TO-DATE；恢复使用 build cache，最终 no-op。
  两份探针已还原，远端全部 1302 个冻结候选输入 hash 一致。
  这是代表性接线/缓存验证，不是性能提升或完整 IDE 增量验证。
- 收尾复核：本机 1120 个搬迁/保留模块输入仍与冻结基线一致。
  唯一候选快照外的并行变化是 `gradle.properties` 的默认 Gradle 堆由 4g 改为 6g；
  本轮未修改、未覆盖此变化。全部远端回执始终以 CLI 显式指定 4g/2g，
  因此验证使用的是原定预算，而非该并行默认值。
- 远端自有 Gradle/Kotlin JVM 均已停止，剩余 owned PID 为空，共享锁已释放且未删除。
  临时 smoke Home 和重复上传 tar 已清理；私有源码/依赖缓存保留供待验收项复现。
  本机旧 build/out/.idea 等非本轮产物未删除，也未接管本机已有 IDE/Gradle 进程。
- 未验收：真实 IDE 导入与 specMain/contract/窄层声明导航、交互 TUI，
  以及 Linux/Mingw 和 iOS 设备执行。任务留在 executable 等待人工验收，
  不宣称整仓全绿；后续提交只含迁移，不混入并行业务修改，不执行发布。
- [58 份证据、复现入口与验收矩阵](file:///home/stream/ACodeSpace/demo/kodex-spec-main-878/product-evidence/README.md#L1)
  已逐项校验远端 SHA-256；21 轮 Gradle 均无守卫中止、swap 为 0，
  最低可回收内存 8068.4 MiB。
- 提交边界：产品迁移提交 `e40bce3321fa41c1c8dc0246fa769e49d3238f76`，
  900 份源码搬迁在 index 中使用原 HEAD blob，认证/model-catalog/RPC server
  的并行业务修改、新增测试和默认堆 6g 保留在工作区，不混入该提交。
  验证矩阵对应前述含并行改动的冻结工作树；提交边界另行静态核对。
  外层仅提交迁移规范、两份本轮记录和产品子模块指针，保留其他任务文件。
- 复审后用户授权修正发布 skill：版本更新入口改用 MCP cliHostMain 新路径，
  Home 生成版本检查改用 app/migration；冻结门禁仅允许明确获批、
  逐文件映射且 blob/模式一致的纯移动，并在拟发布 commit 再核验。
  同步 Home checklist，保持历史内容与 registry/step 顺序冻结；不执行版本 bump 或发布。

### P0 历史检查点与获批修订

- P0 所有重型操作均在 remote MacBook 执行；当时产品源码尚未搬迁。
  当时 1315 个冻结输入逐项核对，0 hash 变化，既有并行改动保持。
- 基本分层可行：CLI commonMain 消费 cliHostMain 工厂、Host commonMain
  消费 hostMain API，以及 provider allMetadataJar / JVM / JS 编译通过。
  Host 越界调用 CLI 工厂时 metadata 编译按预期 unresolved。
- 单 JS target 的 Browser 编译器仍能调用 Node-only 工厂，随后 Webpack
  在 node:fs 失败；契约-only bundle 可成功，但不满足“不靠 DCE 隔离”的门禁。
- 双 JS target + public Gradle attributes 能分开 compilation 与 metadata，
  Browser 越界调用 Node 工厂会按预期 unresolved；但真实运行/打包任务在
  kotlinNpmInstall 被官方 KGP 2.4.20 的 NPM resolver 拒绝：
  `Dependency to project with multiple js/wasm compilations is not supported yet.`
  不能把前面的 compile/metadata 成功写成方案已通过。
- 第一轮 gateModel 的 configuration-cache 序列化失败和早期 JS 负例注入时机问题
  均作为 fixture 错误保留；修正诊断调用/注入后复测，不算产品或架构缺陷。
  产品 configuration-cache、Werror、精确诊断例外及工具链均未改。
- [P0 证据与复现说明](file:///home/stream/ACodeSpace/demo/kodex-spec-main-878/README.md#L1)
  保存全部命令/回执、成功与失败日志、metadata jar、固定 KGP 源码和输入 hash；
  36 个远端导出证据逐文件 SHA-256 一致。
- P0 结束时远端自有 Gradle/Kotlin JVM 已停止，remaining_owned_pids 为空、共享锁已释放；
  私有 fixture/缓存暂留供下一步继续，未操作用户 IDE、工作目录或生产 Home。
- 用户通过结构化选择批准修订：IO、Process、Shell 各保留一个真实 `system` 实现项目，
  宽契约/模型在其父业务项目 specMain；二者都不叫 spec/impl/contract。
  其余归并与全仓目录清理继续，执行目标由条件 137 调整为 140；
  该替代图为 556 条生产边、0 SCC、0 平坦 ID 碰撞。
- 采用 system 分界，不继续双 JS target / NPM 替代工作流，不 fork、升级或裁平台。
  IO 的重复 internal 常量、Shell 的同名文件仍属两个独立模块，无需 B3 合并修复。
  IDE、真实 Native/Browser 执行、TestBalloon association 和产品全仓验收仍待实施验证。
- 下文映射已纳入三个 system override。140 项、13 个合并单元、8 个 contract；
  87 个目标 owner 已有源集 convention，另 53 个需补齐。全部消费者与原测试输入随迁移更新。

## 执行范围与不变量

| 范围 | 本轮盘点 / 目标 |
| --- | --- |
| 迁移前业务 KMP 项目 | 155；root 和无脚本命名空间不计业务项目 |
| 旧 spec 路径项目 | 40，全部逐项映射 |
| 旧 impl 路径项目 | 86，其中组件 View/VM 路径 51 |
| 迁移前源集接线 | 94 已应用 convention，61 尚未应用 |
| 已选八项归并 | 条件中间检查点 147 项 |
| 跨平台归并 | external-url、logging、search-index、migration 四项；IO/Process/Shell 保留 system 边界 |
| MCP 四合一 | 额外减三，最终 140 个业务项目 |
| 执行映射 | 13 个合并单元、净减 15 项；127 个旧项目的 owner 路径变化 |

- 所有保留业务项目都使用 `platform/intermediateMain → commonMain → specMain`；
  不创建额外 implMain，不重命名 commonMain，不新增 specTest。
  真实 contract 进入 specMain；实现、工厂、renderer、fixture 不因 public 而全数上移。
  没有自有契约的实现模块允许 specMain 为空，不复制其已有上游契约。
- 去掉模块目录/项目 ID 的 spec、impl 分类段；保留 Kotlin package、
  `*Impl` 类名、`kodex.kmp-spec-impl` 等插件 ID。它们不是本轮模块分类目录。
- View 与 VM、根 View、CLI、Application、backend host、RPC server 继续各有职责；
  去目录分类不等于合并 UI 层，不按 Mosaic/平台复制一套领域模块。
- 保留 API/KDoc、序列化形状、持久化格式、异常/取消/资源所有权和行为；
  不借布局重构改 Tool 与 clean-models 的现行业务关系来消灭依赖环。
- 不删除 browser/Node/Wasm/iOS、改变 real IO/live suites 的启用政策，
  不添加 UnsupportedOperationException actual 或空实现来填平台覆盖。
- 不升级工具链、fork 或库，不改变 KGP 精确例外、Werror、fallback、GC/堆预算。
  外部 fork、build-logic 自身不是业务模块统一命名的对象。
- 合并与改名同时修改 settings、所有 Main/Test 消费者和现行文档；
  不留旧项目 alias/转发层。历史 kanban/shared-context 证据不批量改写为新布局。

## 模块映射

以下是获批 140 项执行方案中的 override。其余基线项目适用统一规则：

- 对旧路径删除独立的 impl 段，其余路径段不变。
- 无 spec/impl 段、也不在 override 表的项目路径保持不变。
- 项目 ID 一律为目标路径 `/` 换成 `-` 后加 `:`，继续沿用 settings 的平坦 ID。
- 这三条规则覆盖当前全部 155 项，不按磁盘 build 产物猜测项目。
- 表中旧路径用于核对冻结输入；当前源码入口使用目标路径，不保留旧目录别名。

| 旧项目路径 | 目标项目路径 | 处置 |
| --- | --- | --- |
| `agent-context/spec/available-skill` | `agent-context/skill` | 模型并入 specMain |
| `agent-context/spec/contract` | `agent-context/settings` | 独立设置/模型业务库，不沿用无破环依据的 contract 名称 |
| `agent-context/spec/prompt-dsl` | `agent-context/prompt-dsl` | 自足 DSL；specMain 保留 marker，builder 留 commonMain |
| `agent-runtime/spec/contract` | `agent-runtime/contract` | C1 |
| `agent-session/spec/contract` | `agent-session/contract` | C2 |
| `agent-state/spec/context-window` | `agent-state/context-window` | 保留独立算法，不并 core 引回工具环 |
| `agent-state/spec/contract` | `agent-state/contract` | C3 |
| `agent-state/impl/state` | `agent-state/core` | 避免 agent-state/state 的重复命名 |
| `agent-storage/spec/clean-models` | `agent-storage/clean-models` | 共享持久化/事件模型库 |
| `agent-storage/spec/contract-ext` | `agent-storage/core` | 扩展算法并 commonMain；core 测试自边删除 |
| `app/component/agent/spec` | `app/component/agent/contract` | C4 |
| `app/migration/spec` | `app/migration` | 与实现合并，CLI 实现限 cliHost 层，受 B2 控制 |
| `app/spec/session` | `app/session/contract` | C5；app/session 自身仍是持久化实现项目 |
| `hook/spec/notification` | `rpc/models` | 已属于 rpc.models 包的前端配置 DTO/测试归入现有模型 owner |
| `mcp/spec/contract` | `mcp` | 整体合四项目，specMain 契约 |
| `mcp/spec/stdio` | `mcp` | Host transport，commonMain |
| `mcp/spec/streamable-http` | `mcp` | Host transport 与原平台 engine |
| `mcp/impl/composition` | `mcp` | CLI-only composition，cliHostMain，受 B2 控制 |
| `openai/spec/json-codec` | `openai/json-codec` | codec 实现库，specMain 允许空 |
| `openai/spec/models` | `openai/models` | 共享 wire/model 库，不塞进 HTTP client |
| `rpc/spec/contract` | `rpc/client` | 服务接口并 specMain；adapter/cache 留 commonMain |
| `rpc/spec/models` | `rpc/models` | 持久化与 wire DTO 保留所有权 |
| `tool/spec/contract` | `tool/core` | Tool/ToolName + typedTool，与 builder 合并为基础库 |
| `tool/builder` | `tool/core` | JSON builder 契约与适配实现，保留两个原测试集 |
| `tool/spec/image-generation` | `tool/image-generation/contract` | C6 |
| `tool/spec/unified-exec` | `tool/unified-exec/contract` | C7 |
| `tool/spec/view-image` | `tool/view-image/contract` | C8 |
| `utils/coroutines/spec` | `utils/coroutines` | helper 库 |
| `utils/external-url/spec` | `utils/external-url` | Shared 契约 + CLI 实现，B2 |
| `utils/images-codec/spec` | `utils/images-codec` | 同 Host 合并 |
| `utils/images/spec` | `utils/images` | 自足 images 库，不吞入 codec |
| `utils/kodex-home/spec` | `utils/kodex-home` | helper 库 |
| `utils/kotlinx-io-coroutines/spec` | `utils/kotlinx-io-coroutines` | Shared 契约；Host 实现保留独立 system |
| `utils/kotlinx-io-coroutines/impl` | `utils/kotlinx-io-coroutines/system` | 保留单 Node JS 的真实系统实现 |
| `utils/kotlinx-io-serialization/spec` | `utils/kotlinx-io-serialization` | serializer helper 库 |
| `utils/ktor-client-ext/spec` | `utils/ktor-client-ext` | 同 Host 合并，保留 Linux curl |
| `utils/logging/spec` | `utils/logging` | scoped helper + 桌面 file logging，B2 |
| `utils/os-environment/spec` | `utils/os-environment` | 原 expect/actual 与 helper 层次不变 |
| `utils/patch/spec` | `utils/patch` | 同 Host 合并，Models/parser/apply 分层 |
| `utils/process-client/spec` | `utils/process-client` | Shared 契约；Host 实现保留独立 system |
| `utils/process-client/impl` | `utils/process-client/system` | 保留单 Node JS 的真实系统实现 |
| `utils/rpc-exception/spec` | `utils/rpc-exception` | 异常/恢复算法库 |
| `utils/search-index/spec` | `utils/search-index` | Shared 契约 + Host factory/Lucene/JS 算法，B2 |
| `utils/shell-client/spec` | `utils/shell-client` | Shared 契约；Host 实现保留独立 system |
| `utils/shell-client/impl` | `utils/shell-client/system` | 保留单 Node JS 的真实系统实现 |
| `utils/terminal-text/spec` | `utils/terminal-text` | 自足 Unicode/width 算法库 |

### impl 统一规则的实际边界

- 51 个组件 `app/component/<name>/impl/{view,viewmodel}` →
  `app/component/<name>/{view,viewmodel}`；不存在的 VM 不补建。
- `agent-runtime/impl/composition` → `agent-runtime/composition`；
  `agent-session/impl/{filesystem,in-memory}` → 对应业务名；
  `agent-storage/impl/filesystem-layout` → `agent-storage/filesystem-layout`。
- `app/impl/{application,backend-daemon,backend-host,rpc,session,view}` →
  对应 `app/<name>`；`app/settings/impl/filesystem` → `app/settings/filesystem`。
- `rpc/impl/<name>` → `rpc/<name>`，其中 client 吸收服务契约；
  `rpc/krpc-utils-patch` 的 Native `moduleName = org.jetbrains.kotlinx:utils`
  必须保持，替换 convention 的项目引用随新 ID 更新。
- `tool/impl/<name>` → `tool/<name>`，同 owner 的 `/contract` 仍是独立子项目；
  这不是恢复 spec/impl 分类树。
- 三个 system override 保留；其余 `utils/<name>/impl` 和 `app/migration/impl` 与相应旧 spec 合并；
  `hook/impl/notification` → `hook/notification`，仍只负责执行通知命令。
- MCP 特例已在 override 表定义，不生成无实际职责的 `mcp/composition` 转发模块。
- app/view、app/session 和三个 tool 根项目将与其子项目共用父目录。
  settings 支持带 build 脚本的父节点及子节点；迁移按文件/源根合入，
  不能整目录覆盖既有 app/view/components 或 contract 子目录。

## 八个 contract 的必要性证据

箭头是当前生产依赖；将末端契约与开头实现合并会闭环。
这是保留该边界的理由，不是用 contract 为整片目录重新分类。

| 编号 / 新项目 | 当前回路证据 |
| --- | --- |
| C1 agent-runtime/contract | composition → decorator-compact/tool → runtime contract；另经 session contract 返回 |
| C2 agent-session/contract | filesystem/in-memory → runtime composition → session contract |
| C3 agent-state/contract | state → tool-plan / tool-get-context-remaining → state contract |
| C4 app/component/agent/contract | 真实实现所在 app RPC → Settings VM → app Session contract → Agent contract；若并 View，另经 History View 回环 |
| C5 app/session/contract | app Session → app RPC → Settings VM → app Session contract |
| C6 tool/image-generation/contract | image handler → tool/core → clean-models → image DTO |
| C7 tool/unified-exec/contract | exec handler → tool/core → clean-models → exec DTO |
| C8 tool/view-image/contract | view-image handler → tool/core → clean-models → view-image DTO |

- 当前路径证据入口：
  [runtime composition](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/composition/build.gradle.kts#L8)、
  [session filesystem](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/filesystem/build.gradle.kts#L8)、
  [state](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/core/build.gradle.kts#L8)、
  [Agent 真实 RPC 实现依赖](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/rpc/build.gradle.kts#L31)、
  [tool root](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/core/build.gradle.kts#L1)。
- context settings、RPC 服务、MCP 契约、Hook 配置、tool 根接口不因历史名叫
  contract 就获得豁免；分别归 settings、client、mcp、models、core 的实际业务 owner。
- 不继续合并这八个边界所依赖的组合根；改变 Tool/clean-models 或 Agent/RPC
  组合职责是另一轮行为/API 设计，不是此次消除目录分类的隐含工作。

## 获批的平台源集与 system 边界

- 根 specMain 覆盖原契约目标并集；commonMain 只接收所有这些目标均适用的实现。
- 原 Host/CLI 的实现、工厂及所需 expect/actual 放在有明确目标集合的中间层，
  而不是把窄平台代码全塞入 commonMain。平台分层属于 platformMain 的具体化，
  不是额外的 implMain 或第二个 spec 层。
- 层次：平台专有层 → hostMain / cliHostMain → commonMain → specMain。
  P0 已验证部分平台 metadata 语义；真实产品模型、编译和测试仍须通过。
- 官方支持为部分目标建立中间源集，也按目标集合解析依赖源集；
  这是候选机制的依据，不是本仓 KGP/IDE 验收。
  [官方层次模型](https://kotlinlang.org/docs/multiplatform/multiplatform-hierarchy.html)、
  [granular metadata 与目标匹配](https://kotlinlang.org/docs/multiplatform/multiplatform-advanced-project-structure.html)。

| 合并单元 | 宽层保持什么 | 窄层放什么 / 禁止误扩展 |
| --- | --- | --- |
| external-url | Shared DTO | cliHostMain：JVM + 四桌面 Native launcher；JS/Wasm/iOS 无占位 actual |
| IO | Shared FS/Raw IO 契约及默认行为 | 独立 system：JVM、Node JS、四桌面 Native factory/IO；blocking/posix 范围保持 |
| process-client | Shared command/session/client 契约 | 独立 system：生命周期/工厂，平台 spawn；interop 仅原桌面平台 |
| shell-client | Shared shell/settings/output 契约 | 独立 system：Shell factory、解析/执行、PTY；保留 processClient/posix 的原分组 |
| logging | Shared scoped helper | fileLoggingMain 限 JVM + 四桌面 Native；不让 withNative 顺便纳入 iOS |
| search-index | Shared document/index 契约 | hostMain factory，原桌面 Lucene、原 JS fallback；不新增 iOS/Wasm actual |
| migration | Host 契约/版本模型，含 Node | cliHostMain：迁移执行、真实 IO、生成应用版本；不是把完整迁移带进 Node |
| MCP 四合一 | Host specMain 与 transport/default helper | CLI-only composition 移 cliHostMain；Host Node 仍只取得原支持的部分 API |

- Shared = JVM、JS(browser+Node)、Wasm(browser+Node)、四桌面 Native、iOS arm64。
  Host = JVM、Node JS、四桌面 Native。CLI = JVM、四桌面 Native。
- [system IO expect](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/system/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/SystemCoroutineFileSystem.kt#L5)
  不能无条件上移宽层，否则必须给 iOS/Wasm 补 actual，偏离范围。
- 原 nativeMain 在 Host 项目里只覆盖桌面；合 Shared 后默认 nativeMain 会包含 iOS。
  必须显式建立桌面 Native 子集并重挂旧源码，不照搬 withNative。
- 每个合并 owner 只使用一套最终目标配置，不靠叠加 Shared/Host/CLI convention
  来凑目标并集。迁移时同时保留 compiler options 和非目标配置：
  例如旧 CLI convention 的四桌面 Native Ktor network 强制版本，
  不能因 MCP/migration 改用宽层配置而丢失，也不能扩到新增平台。
- 合并后的窄实现测试同步迁入 hostTest/cliHostTest/原平台 Test，
  宽契约测试保持 commonTest；不把窄实现测试扩大到所有新目标。

## 阻塞、风险与先验门禁

### B1 — JS Browser/Node 隔离：已批准 system 边界规避

- 同一个 js target 下的 browser()/nodejs() 是运行环境配置，
  不能仅凭加一个 hostMain 就认为实现已按环境隔离。
  [官方 JS 配置模型](https://kotlinlang.org/docs/js-project-setup.html)。
- 当前 IO 使用 node.fs，process 使用 node.childProcess，Shell 依赖 node-pty；
  原 Shared 契约没有这些宿主依赖。合并不能破坏 Browser 契约消费者的打包/加载。
  [IO Node imports](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/system/src/jsMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/NodeCoroutineFileSystem.kt#L21)、
  [Shell Node PTY](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/system/src/jsMain/kotlin/io/github/stream29/kodex/utils/shellclient/NodeShellClient.kt#L43)。
- 最小实验须同时覆盖 Browser 契约-only consumer、Node 实现 consumer、
  同一消费工程的普通 project 依赖、metadata、IDE 与正常包加载。
  Browser 只取得原契约能力；Node 仍取得真实工厂和实现。
- 不接受仅靠“未调用所以 DCE 会删除”、禁用 browserTest、Node polyfill 或新 stub
  作为隔离证明。若必须引入多 JS target、custom compilation 或额外 Gradle variant，
  先单独验证当前 KGP 的能力、variant 选择和 metadata/IDE，作为待批准的设计扩展；
  不静默修改 frozen source-set 方案或新增诊断抑制。
- P0 结果及用户修订见文首：三个真实 system 项目独立保留，
  不再采用双 JS target 合并设计；Browser 继续仅依赖 Shared API。
  这关闭架构分支，不代替产品 Node/Browser 构建验收。

### B2 — 部分平台 API 的 metadata/IDE 可见性：未关闭工程门禁

- 在真实 KGP 2.4.20 上验证：
  - CLI consumer 的 commonMain 能使用 provider 的 cliHostMain 工厂。
  - Host consumer 能取得 hostMain API；只支持宽目标的 consumer 不取得窄工厂。
  - iOS/Wasm 只编译原契约，没有 node/PTY/desktop interop/Lucene/file logging 污染。
  - 保留 main/test association，实际 TestBalloon 发现并执行原范围测试。
- 先用 external-url/migration 的 CLI 子集以及一个 Host 工厂做最小样本，
  再验证 MCP Host transport + CLI composition；只看项目图不够。
- 生成版本目录要随 migration 实现进入 cliHostMain，
  不把 `GeneratedKodexApplicationVersion` 当作所有 Host 平台的输入。
- B1 按已批准边界处理；逐文件表已冻结，余下四项跨平台归并与 MCP
  四合一的真实模型、IDE 和执行结果继续记入本门禁。

### B3 — 合并符号冲突：system 修订保留原模块隔离

- IO 两侧同包 `internal CoroutineIoSegmentByteCount` 与 Shell 的
  两份 StdoutBuffer.kt 仍分属两个项目；无需删除常量或改文件名。
- ExternalUrl.kt、ProcessClient.kt、ShellClient.kt 也有跨旧项目同名文件；
  对应契约文件只有类/接口，不能一概报为 JVM 门面冲突，仍需编译检查。
- 已有 Tool.kt 的 spec/common 拆分也是此类；不为消除同 basename 而改 API。
- 其他跨项目 internal/private 同名、sealed/serializer、Koin/kRPC/Compose
  生成符号由每个合并单元编译检查，不将静态 basename 扫描当完整符号证明。

### B4 — 路径/产物与并行输入：实施前可解决的集成门禁

- 当前 model-catalog 与 RPC server 有并行业务修改；RPC server 此次会从
  rpc/impl/server 移到 rpc/server。实施前协调 owner、冻结完整工作树；
  不覆盖、回退或只复制 HEAD 漏掉未提交输入。
- 全部 module ID 更名会影响任务路径、输出目录和 Native klib 标识；
  所有内部消费者重编译，不声称外部已发布库二进制兼容。
- kRPC patch Native unique name 保持原值，不能随 Gradle 项目 ID 自动改变。
- CLI baseName/entryPoint、四原生发布归档布局和 Daemon 生命周期不变；
  发布不在本任务授权范围。
- 精确匹配已找到产品侧 docs/gradle-development 的两条示例命令，
  以及 migration 性能探针 KDoc 的旧任务名；至少十份领域 checklist 有具体旧路径。
  另扫通用 impl/view 等文字，不能只做 ID 替换。
- migration 冻结的历史测试/排除政策保留；源码搬迁不重写旧版迁移行为。

### B5 — 验收资源与历史失败：以实际复跑结果为准

- 规划阶段仅有静态图；当前 metadata、JVM、Node、真实 Browser/Wasm、
  Mac Native 编译/执行与 CLI 控制面结果见文首检查点。
  已复现的基线失败、未覆盖平台和 IDE/交互验收分别记账，不写成全仓全绿。
- 旧 specMain 迁移曾有 UI/JS 运行时与 JS integration/Native View 编译缺口；
  这些只是历史参照，须在新的冻结基线重跑，不能照抄旧结论认定本次无回归。
- 实施验证前确认空闲设备/预算、检测并显式复用可用 Gradle Daemon JVM；
  不能复用按构建失败处理，不擅自换 JVM、扩堆或操作用户 IDE。
- 真实账号、联网模型调用与默认禁用 real IO 不自动启用。
  隔离验证排除项、原测试编译、平台未覆盖结果分别记账。

## 实施批次与原子边界

以下批次已获实施授权，P0 边界修订亦已批准；全量 owner 图原子切换，
再按域执行验证，避免通过保留旧别名模块维持中间可构建状态。

| 批次 | 内容 | 通过条件 |
| --- | --- | --- |
| P0 证明与冻结 | 私有 fixture/副本验证 B1/B2；冻结完整源码、构建/目标/测试输入、依赖及工具链基线 | 精确 source/variant 模型、正负 consumer、Browser/Node、metadata/IDE 证据；不通过停止跨平台 manifest |
| P1 统一接线 | 对缺失的保留 KMP owner 显式应用源集 convention；保留现有 default/Mosaic/posix/Native hierarchy | commonMain→specMain 单向、原目标及 test association 保持；不以创建空层为由移动实现 |
| P2 低耦合业务改名 | context settings/prompt DSL、OpenAI models/codec、独立 utility/helper、clean-models | owner ID/路径、全部消费者、metadata 与 scoped tests 同批更新 |
| P3 八项归并 | 三对同 Host utility、storage ext、available-skill、RPC client、Hook→RPC models、tool/core | 删除合并自边/原注册；serialization/kRPC/真实消费者编译；契约与实现/KDoc 保留 |
| P4 跨平台链 | IO/Process/Shell 的 system 改名；external-url/logging/search-index/migration 归并 | B1 以 system 分界规避；窄层 API/测试/interop 不扩宽；不执行无必要的 IO/Shell 合并修复 |
| P5 MCP 四合一 | Host contracts/transports 与 CLI composition 进入一个 mcp 项目 | Host Node transport 与 CLI factory 各自真实消费；消除 test self edge，不把 CLI fixtures 放 commonTest |
| P6 其余 impl 改名 | 后端组合/存储/RPC、Application/Daemon、51 个组件 View/VM、根 View/CLI 相关引用 | View→VM 方向不变，renderer/生成/fixture source scope 不变，八个 contract 回路边界保留 |
| P7 整仓验收 | 当前冻结基线对照、全仓模型/metadata/可用宿主编译测试、IDE 导航、CLI/Daemon smoke | 新问题为零或逐项阻塞；未验收平台明确列出，资源清理后交人工验收 |

- P1 采用现有显式 convention 接入方式，避免同时重写全部 target convention 的应用顺序。
  完整 140 项投影中，已有 opt-in 合并成 87 个 owner，另 53 个 owner 需补齐；
  不是直接对当前 61 个全部机械加插件后就宣布范围已完成。
- 每个批次包含该批所有跨域消费者更新，由一个协调 owner 持有 settings/manifest；
  子线只能做约定域内搬迁，不能并发各自修改全仓 ID 或启动重型构建。
- 每个合并单元原子执行：冻结输入 → 逐文件 source-set 去向 →
  合并脚本插件/依赖/测试/生成/interop → 删除自边 →
  settings/消费者硬切 → 静态检查 → 编译/测试 → 同步现行文档。
- 同一批内按依赖先基础 owner 后消费者验证；实现过程中发现新环或平台限制，
  停在该单元，不用 alias/复制 DTO/业务 adapter 掩盖。

## 验收检查

- 静态：155 个旧项目有且只有一个目标；当前 40 spec/86 impl 都覆盖；
  最终 module ID/目录不含 spec/impl 分类段；无未知 ID、非合并自边或生产 SCC。
- 模型：逐原项目比较 target、每个原生产/测试文件的目标集合、
  编译器/plugin/options、依赖 scope、main/test association、生成输入和 interop；
  合并 owner 目标并集不等于每份实现都自动进入并集。
- 源码：API/package/KDoc/wire shapes 保持；system 修订无需 B3 符号改动；
  不能只看 Git rename 检测或文件数量。
- 接线：全仓 allMetadataJar、所有受影响 JVM main/test 编译、适用的 JS Node/Wasm、
  各兼容宿主 Native 编译/link 与原测试；Browser 不能由 Node 通过替代。
- 生成/消费者：serializer、kRPC、Koin、Compose、独立模型/fixture/RPC/backend/UI
  真实消费都覆盖，不只编译新根项目。
- IDE：真实完整导入和新 specMain/contract/窄层工厂的精确声明导航；
  CLI 成功不当作 IDE 成功。增量/no-op 至少做代表性 spec/impl 编辑与重置检查。
- 产品 smoke：隔离 Home 中 CLI 启动退出、Daemon bootstrap/重连/停止；
  不动用户生产 Home，不联网发模型请求，不因布局迁移改变生命周期。
- 每轮记录基线 revision、未提交输入 hash、命令/结果/排除项/失败及资源回执；
  真正执行的测试才记通过，历史失败与未覆盖项不涂绿。

## 规划阶段静态证明（历史检查点）

- 按上述 override + impl 删除规则投影当前 155 份业务构建脚本：
  得到 137 个条件目标、16 个合并单元、8 个 contract、127 个 owner 路径变化。
- 纳入已知 kRPC patch convention 边，分开 Main/Test：
  当前生产 584 条去重边、投影后 547 条；两图无非平凡 SCC。
  目标平坦 ID 无碰撞；这不是求值后的 Gradle variant/task 图。
- 八个 contract 都在最终投影图中逐一复核了实现经第三方回到该契约的路径，
  不是仅凭合并前的历史图或名字推断保留。
- MCP 根从命名空间变业务项目，settings 不能重复注册 :mcp；
  简单以“删 18 个 include”推导注册总数会漏掉这个变化。
- 规划阶段检查跨旧项目同包 basename、定位 B3，没有运行编译或 fixture。
  用户随后已批准清单及 system 修订进入实施；当前结果以文首实施检查点为准，
  不把历史图无环当作平台门禁通过。
