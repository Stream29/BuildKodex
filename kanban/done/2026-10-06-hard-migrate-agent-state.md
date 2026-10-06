# Task Tree

- `Inventory the complete AgentState root and production consumers`()
- `Compare original operations, implementations, tests, and SOPs`()
- `Specify the hard cutover and ablate the tool support project`()
- `Receive review of the AgentState engineering plan and existing compaction risk`()
- `Receive explicit implementation authorization`()
- `Cut over the real State contract and implementation`()
- `Retarget consumers and remove old project entries`()
- `Validate State, Runtime, Session, RPC, and frontend observations`()
- `Receive scoped acceptance and archive`()

# Details

## Scope and authorization

- 用户选定完整后端 `agent-state` 单根并审阅 Planning，随后明确“执行吧”。
  用户随后验收本批，并明确授权分批提交与推送。
- 基线：`Kodex/` 的 `refactor/spec`，`206266c805785189d2df009531251b5e503c809d`；
  内层工作区干净。外层其他用户任务和 shared-context 指针不属于本批。
- 包含原 `contract`、`context-window`、`tool`、`impl`、`test` 及全部消费者；
  不包含前端父级组合迁移、RPC wire 调整、Runtime 编排或存储格式改造。
- 直接接管原 `KodexAgentState`，不是新增第二套 AgentState/Aggregate/Timeline。
  此处是 Agent 的原子操作层，不是 MVVM 页面，也不新增 ViewModel 或 UI 状态。

## Project mapping and ablation

| 原项目 | 目标项目 | 真实职责 |
| --- | --- | --- |
| `agent-state/contract` | `agent-state/spec/contract` | 原 State 接口、状态联合类型、RequestFinish、语义扩展及可观察异常。 |
| `agent-state/context-window` | `agent-state/spec/context-window` | State/storage 快照与 Model Catalog 派生的共享只读预算扩展及原测试。 |
| `agent-state/impl` | `agent-state/impl/state` | 原 scope 工厂、唯一私有实现、请求/压缩投影及原测试。 |
| `agent-state/tool` | 合入 `agent-state/impl/state` | 原工具定义装配、provider call → pending event 转换及原测试。 |
| `agent-state/test` | 保留 `agent-state/test` | 仅测试装配的 context/MCP 假实现；不制造生产 spec。 |

- 五个现有项目变为四个：三个生产项目和一个测试支持项目。
  新坐标为 `:agent-state-spec-contract`、`:agent-state-spec-context-window`、
  `:agent-state-impl-state`；`:agent-state-test` 不变。
- `tool` 并非独立执行器：其工具定义函数直接依赖具体 `*Tools`，
  pending 转换也识别具体工具名称、serializer 和 patch 解析。
  把这个支撑模块命名为 spec，或另造工具 registry/resolver 接口，只会增加层次。
- 合并后原函数与包名保留；Runtime composition 直接依赖实际 State 实现项目，
  继续复用 `toDeferredToolSearchDocuments`，不建立转发项目。
- 删除合并后的 `impl -> tool` 内部项目边；这是消融一个 Gradle 层，
  不是删除功能。State 实现同时承接两组原测试，不丢失 pending projection 证据。
- 只读 context-window 扩展不依赖具体 State 实现；真实跨实现运算可留在 spec，
  无需为了目录形式造预算 provider/interface。

## Engineering work units

- [State contract and implementation](2026-10-06-cutover-agent-state-contract-and-implementation.md)：
  原类型接管、异常归属、原子操作与 scope 生命周期。
- [Context/tool support and consumer closure](2026-10-06-classify-agent-state-context-and-tool-support.md)：
  合并 tool 支撑、预算扩展、依赖方向、全部生产/测试消费者。
- 两份子计划属于同一复杂根，不把 State/CAS/streaming/compaction 算法分给互相
  修改同一实现文件的并行线。未来若使用多个 Session，仅分离契约审查与
  不重叠的 helper/test 文件；宿主 Gradle、旧入口删除和构建由一个协调者负责。

## Behavior and documentation gates

- 原签名、FQCN、sealed 分支、RequestFinish 和 serializer 形状保留。
  [原工厂与实现](../../Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt)
  继续由原 scope 工厂直接返回原接口，借用 client/storage/MCP，不加 adapter。
- Streaming 的发布、持久化、释放顺序与原 SharedFlow replay 策略不变；
  网络不占用 RequestResponse 的短写锁，逻辑状态保持请求所有权。
- Runtime 保留续跑、重试、自动压缩、工具执行和 pending steer；
  State 不承接这些编排，不复活已退役控制 Hook。
- [原语义扩展](../../Kodex/agent-state/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentStateExtensions.kt)
  保留实际分步操作。`appendPlanUpdate` 是 settings 写入与 tool completion 两次操作，
  不能把文档中的“一次事务”描述当成新增事务层的理由。
- 实现阶段修正三类已有描述冲突：request failed 实际返回 Retryable；
  injectHistory 中用户消息按历史推断轮换 turnId；compact 的网络等待实际释放 Mutex。
  以源码、既有测试和确认的协议说明对照修正文档，不倒过来改算法迎合旧文本。

## Existing compaction risk

- 当前 `compact` 捕获 settings 后释放锁；期间 settings 更新可提交；
  checkpoint 又用捕获的 `previousSettings.copy(...)` 写入新 settings。
  本批已实际复现：CAS 成功写入 `"during"`，checkpoint 后最新标题恢复 `"before"`。
- 证据：
  [现行 compact](../../Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt)、
  [checkpoint 写入](../../Kodex/agent-storage/spec/contract-ext/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/ext/CompactionStorage.kt)、
  [允许压缩中更新的原测试](../../Kodex/agent-state/impl/state/src/commonTest/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImplTest.kt)。
  原测试只断言中途 index 的标题，没有断言压缩后的最新 settings。
- [新增 gated 复现](../../Kodex/agent-state/impl/state/src/commonTest/kotlin/io/github/stream29/kodex/agentstate/impl/AgentStateHardCutoverTest.kt#L110)
  明确标记为现存缺陷，独立断言压缩后的最新值；无 sleep 或 live provider。
- 这是基线一致性风险，不是本次搬迁造成的回归。用户授权执行硬迁移；
  本批补实际复现、单列风险，但不视为授权修改压缩并发协议。
  后续若修复，先确定 checkpoint 对并发更新、turn/window 字段的合并规则，
  不能默默改成全程持锁或新事务协议。

## Validation matrix

- 静态：原声明逐项对照、唯一 FQCN、三个新生产坐标、四个旧坐标删除；
  原 `agent-state-tool` 不以空 project/alias 复活。主源码与测试依赖分开检查。
- 新项目：三个生产项目 `compileKotlinJvm`，State 实现及预算 spec 的 `jvmTest`，
  测试支持 `compileKotlinJvm`。普通 spec 无测试时不能把 NO-SOURCE 计成测试通过。
- 原 State 六个测试源文件、tool 两套、context-window 一套完整迁移；
  新增缺口用 gated fake client/storage 验证，禁止靠 sleep 或重复运行掩盖风险。
- 下游 JVM 回归：Runtime 三个 decorator、Session filesystem/in-memory、
  tool plan/get-context-remaining、RPC server、app-viewmodel-rpc、
  History/History Index ViewModel、Application View。
  直接 spec 与 composition、app-viewmodel-agent、CLI 编译也纳入。
- `integration-test:compileTestKotlinJvm`；live OpenAI/MCP integration 不自动运行。
  Session→State→Runtime→RPC 的假客户端真实装配需证明路径实际切换，而非只编译。
- 保留原 `kodex.kmp-host`；当前该插件同时声明 JVM、四种 Native 和 JS。
  先跑 Linux x64 主源码；JS 用实际解析结果判断依赖可用性，不凭目录推定支持。
  其他设备 Native、真实 CLI 与 live provider 未验证时明确列出。
- 构建集中执行，先核对已有 daemon/用户资源，再使用合适 JVM 与低 worker 数；
  不并行运行多组 Gradle 或强占 IDE/设备。
- 旧路径扫描、当前 checklist 同步、链接与两个仓库 `git diff --check`。
  历史 done 文档不重写成“今天已验证”；重验遗留风险不在本批默认关闭。

## Planning evidence

- 已读取真实五项目、原 State/预算/tool 代码、测试与生产装配。
  静态扫描发现 18 份包含四个待删除坐标的构建文件；执行时重新扫描，
  该数量不是固定修改清单。
- 合并项目的静态主源码依赖投影，在删除旧 `impl -> tool` 内部边后无环。
  这不是 Gradle 解析、编译或运行结果。
- 用户已授权执行；实现、构建和风险复现证据在本任务集中记录。

## Implementation checkpoint

- 原 contract/context-window 已进入目标 spec，原 impl 与 tool 源码/测试合并到
  `impl/state`；四个旧坐标删除，全部直接消费者切换，没有 forwarding project。
- 可观察异常移入 spec 且原 FQCN 保留；State 工厂与算法除了 KDoc、
  该异常物理移动外保持一致。原 tool 实现/测试与预算测试逐文件字节对照一致。
- 补入 `AgentStateHardCutoverTest`：构造失败的 scope/借用资源边界、
  final/commentary 后 injection 的 turnId，以及明确标记为缺陷复现的
  compact/CAS 旧快照覆盖。三项均通过；缺陷复现通过不表示缺陷已修复。
- 实际 208 个项目主源码依赖静态检查无环、无缺失项目、无四个旧 State 坐标；
  两个 State spec 不直接依赖 impl。这不替代 Gradle 和运行验证。
- 本机用户 IDE 的 Gradle 内存耗尽仍在退出，未抢占或终止其进程。
  用户明确改在 Xiaoxin Ubuntu 验证；使用独立临时目录
  `~/ACodeSpace/local/kodex-agent-state-419`，完整源码与三个固定子模块
  6270 个文件 SHA-256 核验一致，不修改原研究源码工作区。
  远端无运行 Gradle，使用实际 JDK 25 与原 Gradle 缓存，取得 device-heavy
  互斥锁后单 worker 执行；核心及下游 JVM 矩阵通过。

## Central validation checkpoint

- Xiaoxin Ubuntu：JDK `25.0.4-tem`，Gradle `9.5.1`，
  `--max-workers=1 -Dorg.gradle.parallel=false`；JVM 矩阵复用本轮同一 daemon，
  Native 下载中断后按下述记录重启，随后 JS 复用该新 daemon。
- 核心目标通过：State contract/test-support JVM 编译，
  `:agent-state-spec-context-window:jvmTest`（3 项）、
  `:agent-state-impl-state:jvmTest`（55 项），XML 无失败、错误或跳过。
- 首轮仅新增测试缺少 Job `cancelAndJoin` 导入导致测试源码编译失败，
  已修正后完整重跑通过；原预算测试及所有新生产项目编译首轮即通过。
- 最终源码清单 6270 个文件再次校验一致；本机未执行构建。
- 下游首轮卡在 Mosaic `ZigDownloadTask` 的 Java HttpClient 无超时等待；
  线程栈已确认，终止的是本轮远端 wrapper，daemon 回到 IDLE。
  从本机缓存复制同版 Zig `0.15.1`，二进制 SHA-256 两侧一致：
  `0ee27482eb2e7b19fad58579c107364aa93fbfb54be2d2b9c52cef5f955d6225`。
  后续矩阵只排除 `:Mosaic:mosaic-tty:zigDownload` 工具链下载步骤，
  不排除 JNI 编译、任何组件编译或测试；不修改 Mosaic 源码。
- 下游最终矩阵成功；合并核心结果，14 个实际测试目标共 562 项，
  当前 XML 均无失败、错误或跳过。两个最终日志内测试任务实际执行，
  不把依赖的 UP-TO-DATE 编译目标或 NO-SOURCE 目标算成新增测试。

| 实际 JVM 测试项目 | 项数 |
| --- | ---: |
| State impl / context-window spec | 55 / 3 |
| Runtime compact / steer / tool | 11 / 10 / 12 |
| Session filesystem / in-memory | 103 / 23 |
| tool plan / get-context-remaining | 2 / 2 |
| RPC server / app-viewmodel-rpc | 118 / 68 |
| History / History Index ViewModel | 37 / 16 |
| Application View | 102 |

- State contract/test-support、Runtime contract/composition、app-viewmodel-agent、
  CLI JVM 主源码及 `integration-test:compileTestKotlinJvm` 编译通过。
  Integration 只编译测试源码，未运行 live provider 集成；CLI 未做真实终端运行验收。
- 四个 State `compileKotlinLinuxX64` 目标通过；包含真实 Process/Shell cinterop
  及下游工具依赖编译，不跳过源码目标。首次 Native sysroot 下载缓慢，
  终止本轮 wrapper 后，复用本机已安装的同名依赖目录；传输压缩包两侧
  SHA-256 一致：`cb90faf963393649b0cd26da28a9a6678179e05a2e5a1127d1efa6b47616b5e5`。
  Gradle daemon 因该次中断自行退出，重启使用同一 JDK 与 Gradle 缓存。
- 四个 State `compileKotlinJs` 目标也通过；这是主源码编译，
  不代表 JS 上 Process/Shell 平台能力的实际运行支持。
- 最后再次核验 6270 个源码文件 SHA-256 与本机一致、562 项 XML 无失败。
  远端独立源码/生成目录、本机传输包与清单已清理；仅停止本轮创建的
  Gradle/Kotlin 空闲进程，保留可复用依赖缓存与原研究工作区。
  本机 IDE/Gradle 未操作；旧目录可能剩下其占用的 ignored build 缓存，
  不含活源码或 Gradle 项目入口，不在占用期间强制删除。
- 用户已接受迁移与本矩阵，主任务与两份子任务归档 done。
  其他 Native 平台、Native/JS 运行测试、真实 CLI 和 live provider 尚未验证。
  压缩覆盖并发 settings 的已有缺陷仍未修复，重验其他遗留风险也未默认关闭。

## Acceptance and delivery

- 用户确认“我看没问题，可以分批提交推送了”；范围验收不代表已有
  compact/CAS 缺陷或未验证平台自动关闭。
- 代码分为实际硬迁移和补充行为/缺陷复现测试；外层文档归档与子模块指针
  分开提交。推送先内层 `refactor/spec`，再外层 `main`。
- 内层提交：`19bad603` 为真实 State 硬迁移与消费者切换，
  `034b778c` 为所有权、turnId 与已有压缩风险的补充测试；内层工作区干净。
- 其他用户任务和 shared-context 改动不纳入；两分支此前仅本地提交的已验收
  修复随祖先历史一起推送。具体新提交与远端核验由本轮交付报告列出。
