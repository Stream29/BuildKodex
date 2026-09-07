# Task Tree

- 调查 Context Parameters 在 Kodex 中的高价值使用场景
  - [done] 完成首轮全库静态筛查与候选调用链审查
  - [done] 记录初步结论、调查边界与统一报告格式
  - [done] 提交并启动分模块并行调查
    - [done] agent-context：上下文发现、加载与 Prompt DSL
    - [done] agent-runtime：运行时装配、装饰器与日志
    - [done] agent-session：Session repository、装配与生命周期
    - [done] agent-state：原子操作、请求投影与快照
    - [done] agent-storage：时间线、文件布局与存储投影
    - [done] app：contract、ViewModel、view、shared、migration 与 CLI
    - [done] hook：事件上下文、执行与投影
    - [done] mcp：服务、客户端、认证与传输
    - [done] openai：协议、序列化、客户端与重试
    - [done] tool：全部工具、builder 与搜索
    - [done] utils：全部公共辅助模块与平台实现
    - [done] integration-test：跨模块行为与验收缺口
    - [done] Mosaic：布局、绘制、交互与测试 API
    - [done] KotlinMcpSdk：请求上下文、handler 与 DSL
    - [done] LuceneKmp：查询、索引与移植边界
    - [done] build：语言版本、编译插件与目标兼容性
  - [done] 收集各调查 Session 的最终报告
  - [done] 核实候选、去重并汇总到本文件
  - [done] 归档已汇总的 16 个调查 Session
  - 与用户讨论首批场景及未决选择

# Details

## 用户授权与范围

- 2026-09-07：用户要求先记录到 discussion，再分模块建议 subagent 并行调查，
  发掘更多有价值的场景，最后汇总到单一任务文件。
- 当前仅调查 Context Parameters；不实施迁移，不升级依赖，不涉及 rich errors。
- 本文件是唯一调查与成果汇总任务文件；不另建分模块任务或 findings 文件。
- 协调 Session：`file:///home/stream/.kodex/sessions/244`。
- 按 `Kodex/settings.gradle.kts` 的 12 个一级模块树划分，每组覆盖全部子模块、
  平台源集与相关测试；加 3 个内嵌构建与 1 个构建配置调查，共建议 16 组。
- 用户已接受全部 16 组建议；现已收齐 16 份稳定最终报告并完成本文件汇总。
- 用户追加要求：全部完成、成果汇总后归档这些调查 Session；保留协调 Session 和本讨论文件。
- 当前工作树存在其他任务的改动；只读检查当前工作树，不覆盖、不回退、不提交。
- 初步排序只是讨论输入，不是用户确认的迁移方案；调查者可以否定或重新排序。

## 首轮审查快照

- 主工程静态扫描 731 个 Kotlin 文件，其中 `*Main` 源集 474 个文件、
  65,059 行，包含测试辅助模块；排除 build、out 与三个内嵌库。
- 对候选阅读实现、调用点与相关测试；不是对全部源码逐行做语义审计。
- 内嵌库做结构、API 签名与工具链筛查；尚未完成各自深入调查。
- 未修改代码，未执行编译、单元测试或 CLI 运行验证。
- 版本及行号是首轮工作树快照；后续报告需重新核对，不沿用失效引用。

## 初步候选

- **优先：历史读取与投影**
  - `IndexedEntry.previous`、`representationAt` 与 `AnchorRepresentation.toProjection`
    反复接收只读 `KodexAgentStorage`。
  - 保留数据对象和索引作为显式输入，在读取入口内部共享 storage。
  - 不新建 `HistoryReadContext`，不隐式化 snapshot index。
  - [IndexHistoryRead.kt:54、244–310](../../Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/IndexHistoryRead.kt#L244)。
- **优先：AGENTS.md 加载链**
  - discovery、loadInstructions、readBytes 逐层传递同一 `CoroutineFileSystem`。
  - 来源计划、字节预算、去重集合仍显式；Skills resolver 已构造持有文件系统，
    不应机械采用同一改法。
  - [FileSystemAgentsMd.kt:17–123](../../Kodex/agent-context/prefix/agents-md/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/agentsmd/filesystem/FileSystemAgentsMd.kt#L17)。
- **首轮次优先、复审后收窄：迁移与文件布局操作**
  - migration → timeline migration → layout helpers 共享文件系统。
  - 先保留 `Migration.action(home, fileSystem)`，在执行时建立 context；
    注册表的函数引用需要专门核对，不能机械改签名。
  - 复审补充：已发布 migration 实现被冻结，不能修改其私有 helpers；
    首轮版本迁移子链仅保留为适配性分析，不作为当前可实施范围。
  - [KodexHomeMigration.kt:124](../../Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigration.kt#L124)、
    [FileSystemLayout.kt:10](../../Kodex/agent-storage/filesystem-layout/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystemlayout/FileSystemLayout.kt#L10)。
- **次优先：租约内部 I/O helpers**
  - heartbeat、过期检查与释放辅助操作共享文件系统。
  - 保留 owner scope 的显式表达及长期续租对象的依赖字段；context 不证明持锁、
    租约有效性或资源不逃逸。
  - [FileSystemLeaseSupport.kt:33–125](../../Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseSupport.kt#L33)。
- **次优先：live history 的 shell registry**
  - history → group → entry → content → event renderer 逐层传递 registry。
  - 独立 renderer 当前允许无 live registry；不能为省参数取消可选语义或发明空对象。
  - 保留叶子订阅与 Agent 归属；需要验证重组、延迟 lambda 与 Session 切换。
  - [AgentHistoryView.kt:313](../../Kodex/app/view/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt#L313)、
    [CleanEventView.kt:108](../../Kodex/app/view/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/CleanEventView.kt#L108)。
- **较低收益，待复审**
  - Prompt XML：领域对象 receiver + `PromptXmlBuilder` context。
  - Tool Hook：只读 storage 的调用与 invocation 投影。
  - JSON：对象 receiver + 当前 decoder 对应的 Json 环境。
  - OpenAI：重试判定共享 `OpenAiClientRetryConfig`。
  - Patch：语义适配，但局部闭包已经消除了大部分 fileSystem 传递。

## 首轮不推荐方向

- 不统一替换长期构造注入、typed factory、runtime decorator 依赖字段。
- 不用构造时捕获的 settings/cwd/model 替换执行时读取的 provider。
- source/target、Session 身份、快照、generation、布局参数仍显式。
- 不整体替换 `CoroutineContext`、`CompositionLocal` 或现有 receiver DSL。
- 不为了新语法新增全局大 Context、service locator 或通用 capability 框架。
- 这些是待进一步调查的初步判断；反例必须给出具体代码和收益。

## 内嵌构建与语言约束

- 主工程版本目录为 Kotlin 2.4.0；首轮未发现现有 Context Parameters 声明。
- Mosaic 的版本目录为 2.3.21，不直接继承主工程语言版本。
- KotlinMcpSdk 的 compilerOptions 显式指定 language/api version 2.1；
  `RequestHandlerExtra` 同时显式传参和通过协程上下文传播，值得独立审查。
- LuceneKmp 有 Java Lucene 行为/API 对齐约束，不能仅为语言风格扩大 fork 差异。
- [官方 Context Parameters 文档](https://kotlinlang.org/docs/context-parameters.html)：
  区分稳定的 context parameters 与仍标 Experimental 的 explicit context arguments。
- 函数引用、override、context function types、DSL marker、可空值、Compose 插件、
  JVM/Native/JS/Wasm 支持须按具体版本核实，不凭提案示例宣称编译通过。

## 并行调查协议

- 每个 subagent 首先读取根 `AGENTS.md`、本文件、`kanban/Draft.md`，
  并按主题加载 checklist 和相关 tracked task；内嵌库遵守各自 `AGENTS.md`。
- 从当前工作树独立审查负责子树，不只重复首轮候选；列出全部子模块的覆盖情况。
- 可以只读追踪直接调用方/被调用方；跨模块候选注明双方职责与主归属。
- 不修改代码或文档，不新建任务文件，不提交，不升级工具链，不自行派生其他 Session。
- 不运行 Gradle 构建、IDE 重构、真实业务或耗资源基准；需要编译实验时先报告
  最小方案与原因，由协调 Session 统一安排，避免并发干扰。
- 使用现有设备和环境；如使用 Python，必须通过 `uv`。
- 新特性语义或编译支持不确定时查官方来源；不能核实的明确标为待验证。
- 最终成果只通过本 Session 的最终回复返回；唯一任务文件由协调 Session 写入。

## 统一最终报告格式

- **覆盖**：检查过的子模块、平台、测试；只做结构扫描与深入阅读分别列明。
- **高价值候选**：每项提供 ID（组名 + 序号）、相对路径与行号、现有调用链、
  最小签名草图，以及与普通参数、扩展 receiver、构造注入的具体比较。
- **收益**：说明消除了什么当前问题；仅省一处参数也要如实说明，不凑候选数量。
- **约束**：生命周期、并发、快照、身份、可空/default 语义、回调捕获、
  序列化、override/ABI 和跨平台风险，只列与该候选有关的项目。
- **验证**：可复用的实际测试文件及应新增的验证；区分静态推断与实测。
- **不推荐**：看似适合但现有方式更清楚的具体例子及理由。
- **结论**：优先级、是否值得实施、阻塞或未决事项；允许“本模块无高价值候选”。
- 尽量用短条目；不要以泛泛语言介绍替代代码证据，也不要声称覆盖未读代码。

## 调查分配与结果登记

- 归档状态：246–261 全部已创建项目支持的 `archive.mark`；
  仅修改归档标记，不删除报告、不修改任何时间线，协调 Session 244 保留。
- 实现依据：[归档写入:104–117](../../Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt#L104)；
  [标记与过滤测试:238–260](../../Kodex/agent-session/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepositoryTest.kt#L238)。
- 已验证磁盘标记与原报告仍存在；未进行 UI 列表刷新验证。

| ID | 负责范围 | 状态 | Session |
| --- | --- | --- | --- |
| agent-context | `Kodex/agent-context/**` | 已汇总；最终事件 60 | `file:///home/stream/.kodex/sessions/246` |
| agent-runtime | `Kodex/agent-runtime/**` | 已汇总；最终事件 86 | `file:///home/stream/.kodex/sessions/247` |
| agent-session | `Kodex/agent-session/**` | 已汇总；最终事件 80 | `file:///home/stream/.kodex/sessions/248` |
| agent-state | `Kodex/agent-state/**` | 已汇总；最终事件 89 | `file:///home/stream/.kodex/sessions/249` |
| agent-storage | `Kodex/agent-storage/**` | 已汇总；最终事件 101 | `file:///home/stream/.kodex/sessions/250` |
| app | `Kodex/app/**` | 已汇总；最终事件 131 | `file:///home/stream/.kodex/sessions/251` |
| hook | `Kodex/hook/**` | 已汇总；最终事件 71 | `file:///home/stream/.kodex/sessions/252` |
| mcp | `Kodex/mcp/**` | 已汇总；最终事件 91 | `file:///home/stream/.kodex/sessions/253` |
| openai | `Kodex/openai/**` | 已汇总；最终事件 113 | `file:///home/stream/.kodex/sessions/254` |
| tool | `Kodex/tool/**` | 已汇总；最终事件 98 | `file:///home/stream/.kodex/sessions/255` |
| utils | `Kodex/utils/**` | 已汇总；最终事件 143 | `file:///home/stream/.kodex/sessions/256` |
| integration-test | `Kodex/integration-test/**`；跨模块现有测试只读追踪 | 已汇总；最终事件 94 | `file:///home/stream/.kodex/sessions/257` |
| Mosaic | `Kodex/Mosaic/**` | 已汇总；最终事件 119 | `file:///home/stream/.kodex/sessions/258` |
| KotlinMcpSdk | `Kodex/KotlinMcpSdk/**` | 已汇总；最终事件 91 | `file:///home/stream/.kodex/sessions/259` |
| LuceneKmp | `Kodex/LuceneKmp/**` | 已汇总；最终事件 95 | `file:///home/stream/.kodex/sessions/260` |
| build | 主工程/内嵌构建的 Gradle 配置、buildSrc 与编译相关 scripts | 已汇总；最终事件 103 | `file:///home/stream/.kodex/sessions/261` |

## 成果汇总规则

- 建议工具仅返回新 Session 元数据，不自动回流最终报告；协调者登记实际返回的
  storage URI，并只读收集这些已授权 Session 的稳定最终回复。
- 不把 Session 已创建、出现活动时间或 pending 数据当成调查完成。
- 以下按 ID 收录每组的覆盖、候选、反例和待验证项；来源见上表。
- 完整原报告位于对应 Session 的 `index/<最终事件>.json`；
  已确认类型为 `assistant_message`、阶段为 `final_answer`，不是中间计划或 pending 数据。
- 合并跨模块重复项，保留来源 ID、代码证据、不同意见和核实状态。
- 统一结论按“值得试点 / 条件成立再做 / 不值得”整理，并列实际改动范围与验收门槛。
- 未返回、失败、中断或覆盖不足的组明确列为未完成，不能默认为无候选。
- 讨论是否实施由用户决定；不得仅因调查结束自动推进到 planning 或修改代码。

## 并行调查汇总与去重结论

- 已阅读全部 16 份最终报告；以下是协调汇总，不复制数十万字的报告原文。
- 模块覆盖统计来自各组静态报告，跨模块追踪有重叠，不能相加当成全库逐行审计量。
- 协调者复核了历史读取实参、图像重载、Tool Hook 采样顺序、迁移冻结规则及官方稳定范围。
- 所有签名仍未编译；并发、生命周期疑点未经运行复现，不登记为已确认缺陷。
- 多数模块的合理结论是“不迁移”：现有构造持有、receiver、闭包或显式身份已经足够。

### 值得先讨论的局部试点

| 汇总 ID | 来源 | 范围与真实收益 | 限制 |
| --- | --- | --- | --- |
| C1：历史结构读取 | app-1、agent-storage-7、integration-test | `IndexHistoryRead.kt:54–124、244–312`；三个私有操作及两个 override，减少 4 处 storage 转交，保留条目 receiver | 不改 storage contract、入口、snapshot/generation，不宣称减少 I/O |
| C2：AGENTS.md 私有读取 | agent-context-01、integration-test | `FileSystemAgentsMd.kt:43–142`；生产 plan 入口 → loadInstructions → readBytes 共用 fs | 收益中等；保留两种 public overload、默认 fs、来源计划及预算 |

- C1 的最小方向：`context(storage: KodexAgentStorage)` 修饰私有
  `previous()`、`representationAt(upperInclusive)`、`toProjection(predecessorIndex)`。
- C2 的最小方向：public 入口不变，私有读取函数要求 `CoroutineFileSystem` context。
- C2 的 legacy overload 才经过 discoveryRoots/nearestProjectRoot；
  不能把两种入口串成一条不存在的更深生产调用链。
- C2 与普通扩展的比较还需考虑：filesystem 成员 `readBytes` 返回 ByteArray，
  私有同名 helper 返回带 warning 的结果，机械改 receiver 可能改变成员解析。
- 原有参数版本仍是可接受基线；“值得试点”是讨论建议，不是实施授权。

### 条件成立再做

| 汇总 ID | 来源 | 最小范围 | 前置条件 |
| --- | --- | --- | --- |
| C3：live registry | app-2、Mosaic-4、integration-test | `AgentHistoryView.kt:313–416、551–626` 的内部非空 group/entry/content/header 链 | 顶层显式配对 history/registry；公开 nullable renderer 不变；先验 Compose 捕获和跨 Session 隔离 |
| C4：lease 内部 fs | utils-1、agent-session-1 | `FileSystemLeaseSupport.kt:33–130`，必要时 catalog 私有 metadata helper | 先比较普通 fs 扩展；owner receiver、长期 fs 字段、默认参数不变 |
| C5：迁移协调 I/O | app-3、agent-storage-3、integration-test | 仅非冻结协调器私有 helper 或未来获准的新 migration | 不改已发布 migration/codec/fixture，不改 action 类型、注册引用或为省参数加兼容层 |

- C3 保留无 registry 的独立渲染：不创建空对象，不使 default null 意外继承外层 registry。
- C3 已提交命令无 live session 时与 pending 命令的 fallback 不同；叶子订阅不能上移。
- C4 context 不证明持锁、owner 有效、heartbeat 身份正确或清理已完成。
- C5 采用 storage 组的冻结限制，**否决 app-3 草图中修改已发布 `migrateWorkTimeline` 的部分**。
  依据：[Kodex Home:101–107](../../checklist/kodex-home.md#L101)。

### 首轮之外的候选：已发现，但当前收益低

| 来源 ID | 代码依据 | 场景与结论 |
| --- | --- | --- |
| agent-runtime-01 | `KodexToolRuntime.kt:102–141、261–275` | tool-call KLogger；只省一处参数，现有 receiver 可替代，不独立迁移 |
| agent-state-01 | `ContextWindowTokenBudget.kt:16–40` | AgentState receiver + model catalog context；仅一层传递，保留查询时采样 |
| openai-2 | `FunctionCallOutputModels.kt:61–76、112–150` | MCP 输出转换的 Json；只省一次内部转交，保留 public 参数 |
| tool-2 | `ToolSearchDocumentConversion.kt:54–99` | schema 递归文本累积器；真实转交但属于可变输出，普通列表 receiver 更直接 |
| utils-2 | `PromptImageProcessing.kt:79–115` | 图像 transformer；保留 ByteArray/String receiver，但删除参数会撞上无转换能力重载 |
| integration-test-1 | `PatchRendererPerformanceProbeTest.kt:296–331、433–583、674–724` | ThreadAllocationMeter 多层传递；仅手动 JVM 探针，低价值 |
| Mosaic-1/2/3 | `ui/Box.kt:136–148`、`focus/FocusOwner.kt:680–704`、`samples/rrtop/.../Table.kt:54–119` | placement、tree 查询、业务绘制；已有 receiver/闭包，最多省一处参数 |
| KotlinMcpSdk-5/6 | `test-utils/.../serializationUtils.kt:41–87`、`conformance-test/.../auth/discovery.kt:18–44` | 测试 Json / OAuth HttpClient；普通扩展有同等收益，不改公共 SDK |
| LuceneKmp-N-1 | `analysis/common/.../SynonymGraphFilterFactory.kt:112–195` | ResourceLoader 私有初始化链；三处实参收益不足以增加移植差异 |
| build-5/6 | `KodexHostKmp.kt:50–55`、`generatePolishDicData.gradle.kts:64–159` | Project/Writer；收益低，且构建语言环境不同，不列业务试点 |

- 此表为定位摘要，完整相对路径与调用链见各组原报告；下方关键证据索引补充常用文件链接。
- 首轮的 Hook storage、decoder Json、retry config 仍语义适配，但均未发现足以提高优先级的深链收益。
- PromptXml、MemScope 已有 receiver，Patch 已有闭包；不将 receiver 换位算作消除了依赖传递。

## 分模块成果

### agent-context

- 覆盖：11 子模块，14 生产、8 测试文件；平台声明已读，未编译。
- agent-context-01 合入 C2；02 PromptXml 降为风格选择；03–05 不隐式化 settings/plan、Skills 缓存/authority 和局部预算状态。
- Skills 延迟读取必须使用发现时绑定的 fs；`resolve(cwd)` 与 `resolve(plan)` 来源规则不同。
- AGENTS 与 prefix 的相似 discovery helper 对根冲突优先级并不一致，不随语法变更合并。
- 测试：`FileSystemAgentsMdTest.kt:14–138`；缺直接 plan overload、双 fs、取消与失败关闭的专门断言。

### agent-runtime

- 覆盖：6 子模块，9 生产、4 测试文件及直接上下游；没有独立高价值候选。
- agent-runtime-01 为低收益 tool logger；02 与 hook-1 合并；03 拒绝 decorator factory logger 环境化。
- 保留 application/session/agent/tool 日志身份、State delegate 路由和 owner/current operation Job 区别。
- Tool pre/post 各读最新 settings；TurnHook 每次 resume 捕获；Compaction pre/post 共用 request。
  不建立一个冻结规则不明的 RunContext。
- 验收：两个 Session 同 callId 的日志/Hook 隔离、取消、重复完成防护；未执行。

### agent-session

- 覆盖：contract/filesystem/in-memory/test 四模块全部生产及相关测试。
- agent-session-1 仅作 C4 边缘候选；2 装配 dependencies 已有聚合对象且长期借用，不独立迁移。
- 3–6 保留缓存 owner、Session/index、lease handle 和 fork source/target；
  未使用 helper 的递归传参不计入生产收益。
- Catalog 无租约时可扫描但不修复；环境 fs 不证明取得写入资格。
- `FileSystemKodexSessionRepositoryTest.kt:525–535` 的 failed-fork 用例在 source 不存在时失败，
  不覆盖预留 target 后复制失败清理。补测建议不是已复现故障。

### agent-state

- 覆盖：五子模块 17 个 Kotlin 文件，直接请求/预算/工具投影调用链。
- agent-state-01 catalog 预算、02 MCP service 投影均低收益；03–05 保留 mutation 目标、快照、局部状态和 owner。
- `modify` 的失败恢复不回滚已写数据；context 不能把它提升为 all-or-nothing 事务或防重入证明。
- 06–08 为独立待验证项：预算跨 timeline 采样、compaction 旧 settings 提交、
  标题/plan 准入外读取与多次操作交错。仅静态疑点，不混入迁移。
- 测试基线：`KodexAgentStateImplTest.kt:140–214、797–836、950–1029`，
  `ContextWindowTokenBudgetTest.kt:42–70`；需要可控挂起点验证，而非仅改签名。

### agent-storage

- 覆盖：六模块 55 生产、15 测试文件，含四个平台 actual；跨模块 history/cache/migration 定向阅读。
- agent-storage-7 合入 C1；3 合入受限 C5；1/2 contract 与窗口投影已有 storage receiver。
- 4–6 不改固定 codec/serializer、fork source/target 与 cache 构造持有。
- fork 底层允许两端不同 fs，不能统一为一个环境。
- retention 测试的 exactReads 计数不覆盖 `valuesIn` 的批量解码量，不能宣称证明按预算停止解码。
- 测试：`IndexHistoryReadTest.kt:28–183`、`FileSystemLayoutTest.kt:41–91`、
  `FileSystemIndexVersionedTest.kt:36–200`；原报告列有合成起点、双 fs、复制失败等缺口。

### app

- 覆盖：32 构建模块、255 Kotlin 文件结构扫描；history、registry、migration、owner/身份等重点深读。
  `app/view/session` 当前无非生成 Kotlin 源码，不记为 renderer 深审。
- app-1/2/3 合入 C1/C3/C5；4–9 保留主题 Local、标题快照、HistoryItemLoadContext、
  VM 构造注入、Auth/Settings fs、LazyColumn receiver 和 CLI 宿主资源。
- app-3 的历史 migration 私有 helper 草图受冻结规则否决，不照单采用。
- 现有 CleanEvent 测试直接传 ProcessSession，绕过 registry lookup；不能证明完整传递链。
- 验收：双 storage、同 shell ID 跨 Session、registry 替换、null renderer、延迟展开和 row 重组。
- 部分调查时正在进行的 UI/sealed 任务现已有 done 文件；实施前重新核对工作树，不把旧阻塞状态永久化。

### hook

- 覆盖：contract 全部 11 文件、tool-utils、impl/projection 全部生产和两个测试文件。
- hook-1 与 runtime-02 为同一低收益候选：ToolHooks/descriptor receiver + storage context。
- hook-2–5 不改 ShellClient receiver、固定协议 Json、Hook DTO、配置快照和错误 cwd 捕获。
- pre/post 各自采样；排除/失败事件先短路后读取 settings，不能提前读取或复用 pre invocation。
- `ProcessSession.close()` 只请求终止，不证明立即清理完成；输入构造异常也不在全部失败隔离范围内。
- 验收：共享 hooks + 两 storage；handler 中途更新 settings；失败 post 零读取；
  runtime 测试的工具 cwd 断言不等于 hook invocation cwd 断言。

### mcp

- 覆盖：四子模块全部生产、管理/服务/OAuth/transport 测试定向阅读；SDK 仅追直接边界。
- mcp-1 选中的 SDK Client 只有一个叶子调用，收益近零；2–6 保留 server client、
  OAuth attempt、事务内 latest 配置、owner 和 transport 字段。
- 基础 HTTP client、每 server 派生 client、OAuth client 同类型但不同身份，不能按类型隐式混用。
- 独立待验证：旧 owner 的 authorizer 按名称读取最新 token；同名配置身份切换时的交错，
  登录中 URL 改变后的旧结果写回及取消清理。未复现，不记录为凭据泄漏事实。
- 验收可用既有 fake store/refresher、MockEngine 和 `McpServiceImplIoTest` 的局部结构；
  不需要为 context 调查启动真实 OAuth。

### openai

- 覆盖：10 叶子模块，36 commonMain、28 commonTest、1 jvmTest 结构扫描；
  client/retry/codec/catalog/usage/storage 重点深读。
- openai-1 decoder Json、2 MCP 输出 Json、3 retry 分类均低收益；
  4–8 不改 operation 预算、metadata、expectedAccount、catalog/usage 快照与长期依赖。
- decoder context 必须来自本次 `decoder.json`，不能从全局 codec 补取。
- HTTP 与 SSE 共享 operation budget 的桥梁是 request attributes；语言 context 不能替代它。
- reset 的账号归属和 idempotency 绑定保持显式；usage snapshot 测试不覆盖账号切换交错。
- 指导中的重试次数与实现/测试默认 20 的差异仅作待决语义核查，不在本任务修改。
- 测试必须筛选，部分 client/storage 用例访问真实后端或凭据。

### tool

- 覆盖：全部工具及 builder/search，73 Kotlin 文件；平台描述和相关测试已读。
- tool-1 Patch 合并 utils-4 并降级；2 递归 searchText 累积器为新增低收益；
  3 typed handler、4 artifact fs 不推荐。
- pending/input/callId 是具体调用数据，不做 ToolContext；host-owned 两类交互保持宿主执行。
- provider 采样存在分支差异：view-image 仅相对路径读取 cwd，image edit 一批图片共用一次，
  unified-exec 两 provider 顺序读取，不是原子 settings 快照。
- 旧 raw DTO/StateFlow 指导与当前 clean-event/provider 实现有差异；图片持久化归属也需另行澄清。
- 历史图片选择参数目前仍被拒绝；不能为制造 context 场景擅自授予 history/storage 能力。
- 验收：schema 文本顺序、兄弟文档隔离、动态 cwd、完成身份；真实生成/web/进程测试不默认运行。

### utils

- 覆盖：159 Kotlin 文件，110 生产、49 测试；全部辅助模块按参数与资源边界审查，
  不等于所有算法/interop 正确性证明。
- utils-1 合入 C4；2 transformer 新增但低收益；3 MemScope、4 Patch 进一步降级。
- 5–14 不改 raw IO 的 source/sink、owner scope、SafeRw 视图、logger 字段、Curl userdata、
  cold SSE Flow、search snapshot、纯文本/环境函数和 serializer override。
- transformer 不能机械删除参数：非 suspend 无转换重载“需要转换即失败”的行为必须保留。
- Node cleanup scope、Curl 异步 close、search-index 长期资源关闭为独立验证边界；未证明泄漏。
- 验收按平台区分：`/proc/self/fd` 测试不通用，JVM CIO SSE 不验证 Native Curl，
  fake transformer 不验证实际 codec。

### integration-test

- 覆盖：唯一构建及全部四个测试文件完整阅读；相邻候选测试定向复核。
- integration-test-1 allocation meter 新增但低收益；2–4 fixture 大 Context、
  隐式 storage 断言和 mock/MCP callback 重写均不推荐。
- 贡献重点是六维验收矩阵：多实例、取消清理、快照、延迟捕获、Compose 重组、匹配平台。
- HooksIntegrationTest 只验证 prompt/stop，不等于 Tool Hook 端到端覆盖。
- 性能 renderer probe 不执行 Patch 文件应用；不能替代 FS 验收。
- 多项集成测试启用了真实 API；不将全套 integration-test 作为默认离线回归。

### Mosaic

- 覆盖：全部模块结构；runtime/layout/draw/focus、tty 生命周期、testing 与代表 sample 深读；
  未逐行读全部 parser、动画数学、C/JNI 和平台测试。
- Mosaic-1/2 私有 placement/tree helper 收益近零；3 sample 绘制仅条件示例；
  4 registry 归 app，不推动库 API 改造。
- Row/Column/Box 已有 scope dispatch receiver + Modifier extension receiver。
- 不改 MeasurePolicy/DrawModifier/SAM、CompositionLocal、动画 CoroutineContext 或 tty owner。
- 独立 2.3.21，无现成 context 开关；JVM/四 Native，未配置 JS/Wasm。
- 验收需单独检查 DSL marker、延迟 draw/placement、Compose、公共 API/klib；
  当前没有升级或编译授权。

### KotlinMcpSdk

- 覆盖：core/client/server/testing/test-utils/integration/conformance/DSL/sample 结构，
  Protocol、ClientConnection、路由和代表性测试深读，不声称逐行全库。
- KotlinMcpSdk-1/2 现有 dynamic extra 和 ClientConnection receiver 已解决主要需求；
  不替换公共 callback。4 保留 metadata DTO/builder；5/6 是低收益 Json/HTTP helpers。
- 3 为语义疑点：extra 捕获 transport，但发送 helper 再走当前 protocol；
  typed connection 不自动补 relatedRequestId。
- ChannelTransport 不保留 send options，其测试不能证明 HTTP SSE 路由。
- 7 工具链门槛：共享 convention language/api 2.1；conformance 和独立 sample 不一概受该限制。
- 验收需决定旧 extra 重连后行为，区分 progress token/request ID/session ID；
  不以 context 类型可用性代替请求归属检查。

### LuceneKmp

- 覆盖：3,823 Kotlin 文件、821,932 行结构/模式扫描；查询/merge/codec/analysis、
  资源加载和 Kodex search-index 代表链深入抽查，不是逐行审计。
- LuceneKmp-N-1 loader 私有链低收益；N-2 merge state、N-3 Random 不迁移公共 API。
- Query/Weight/LeafReaderContext 绑定 reader；Directory copy 的源 READONCE 与目标 IOContext 不同。
- 移植行为/API 对齐成本高；未逐项对照固定 Java 提交，不能宣称验证精确等价。
- Graph factory 测试缺口不能用旧 SynonymFilterFactory 的测试冒充；search-index 现有单例匹配测试也不证明资源关闭。

### build

- 覆盖：主 buildSrc 全部 convention、Mosaic build-support、SDK 共享配置、
  Lucene 生成器代表链；其余构建结构扫描。
- build-1–4 为语言/平台/插件门槛；5/6 Project/Writer 低收益；7 任务状态/长期 Project 不改；
  8 提供最小验证顺序。
- 主工程 Kotlin 2.4 业务源码可做基础试点；不推出 Gradle Kotlin DSL 同样为 2.4。
- 主 buildSrc 使用 kotlin-dsl；Mosaic build-support 是普通 Kotlin/JVM 2.3.21，二者不能混为一种环境。
- composite 从根启动时不会分别运行内嵌 wrapper；独立 wrapper 版本不是根 composite 的执行证据。
- Mosaic Kotlin/Poko 需成套核对，不能只升 Kotlin；主目录声明 Poko 不等于主 compilation 实际应用。

## 语言核实与验收门槛

- 协调者核实：[Kotlin 2.4 发布说明](https://kotlinlang.org/docs/whatsnew24.html)
  将基础 Context Parameters 稳定化，但排除 explicit context arguments 和 callable references。
- [当前文档](https://kotlinlang.org/docs/context-parameters.html)：
  类型可用性不验证对象身份；同层多匹配会歧义；constructor 不可声明 context 参数。
- [2.3.20 发布说明](https://kotlinlang.org/docs/whatsnew2320.html#changes-to-overload-resolution-for-context-parameters)
  已修改 context 重载优先级；不沿用 KEEP 旧示例保证同名重载选择。
- 首批只讨论主工程私有函数；不启用显式 context 实参，不迁移函数引用，不改公共 callback。
- context function types、override、Compose/TestBalloon/Koin 组合及各目标仍需实际编译；
  不能从版本目录或玩具示例推导项目通过。

| 候选 | 已有测试入口 | 必补的针对性验证 |
| --- | --- | --- |
| C1 历史读取 | `IndexHistoryReadTest.kt:28–183`、`AgentHistoryModelsTest.kt:439–495` | 双 storage 同 index 异内容；compaction 上界；挂起期间 replacement/取消；私有 override 编译 |
| C2 AGENTS | `FileSystemAgentsMdTest.kt:14–138` | 直接 plan overload；同路径双 fs；不回落系统 fs；warning/预算与取消关闭 |
| C3 registry | `AgentHistoryEntryInteractionTest.kt:242–277`、`CleanEventViewTest.kt:350–453` | 真实 nullable 入口；双 Session 同 ID；registry 替换；迟到展开/订阅及重组 |
| C4 lease | `FileSystemLeaseImplTest.kt:26–106`、repository 取消测试 | 创建 fs 始终用于 heartbeat/释放；旧身份不删新租约；owner 与 operation 取消分开 |
| C5 migration | `KodexHomeMigrationTest.kt:73–134、164–171` | action 保存后执行仍取本次 fs；失败/取消不提前写版本；冻结文件零改动 |
| 低收益 Json/Hook/retry | serializer、runtime Hook、retry policy 原有测试 | 两环境隔离；decoder 自身配置；pre/post 分别采样；HTTP/SSE 共用预算与取消 |

- 复用 counting/suspending filesystem、MockOpenAiClient、可替换 TestMcpService、
  CompletableDeferred 挂起点和 runMosaicTest，不引入新测试框架。
- 按顺序分别记录：编译、筛选后的离线单元/renderer 测试、匹配平台运行、实际 CLI；
  本次这四类均未执行，只有文档和源码静态检查。

## 关键新增代码证据索引

- [图像双重载与 transformer:79–115](../../Kodex/utils/images/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/PromptImageProcessing.kt#L79)。
- [搜索文本递归累积:54–99](../../Kodex/tool/tool-search/impl/src/commonMain/kotlin/io/github/stream29/kodex/tool/toolsearch/ToolSearchDocumentConversion.kt#L54)。
- [MCP 输出 Json 转换:61–150](../../Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/FunctionCallOutputModels.kt#L61)。
- [Hook pre/post 与 invocation:27–92](../../Kodex/hook/tool-utils/src/commonMain/kotlin/io/github/stream29/kodex/hook/toolutils/ToolHookUtils.kt#L27)。
- [预算采样:16–40](../../Kodex/agent-state/context-window/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contextwindow/ContextWindowTokenBudget.kt#L16)。
- [性能探针 meter:674–724](../../Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/integrationtest/PatchRendererPerformanceProbeTest.kt#L674)。
- [Lucene loader:112–195](../../Kodex/LuceneKmp/analysis/common/src/commonMain/kotlin/org/gnit/lucenekmp/analysis/synonym/SynonymGraphFilterFactory.kt#L112)。
- [SDK extra 与 transport:167–217](../../Kodex/KotlinMcpSdk/kotlin-sdk-core/src/commonMain/kotlin/io/modelcontextprotocol/kotlin/sdk/shared/Protocol.kt#L167)。
- [SDK language/api 配置:18–25](../../Kodex/KotlinMcpSdk/buildSrc/src/main/kotlin/mcp.multiplatform.gradle.kts#L18)。

## 未决事项与范围控制

- 是否先做 C1，或同时做 C2；尚未获得实施选择。
- C3/C4 相比普通参数或 receiver 的可读性收益是否值得验证投入，待用户讨论。
- C5 不包含任何已发布 migration 重构；不通过新增兼容层扩大本次范围。
- 各组发现的 settings 并发、OAuth 身份、请求路由、资源完成语义和指导漂移，
  仅保留为调查输入；不自动开新任务、不修复、不写成全局指导。
- 当前仍在 discussion；调查汇总完成不等于已决定迁移路线。
