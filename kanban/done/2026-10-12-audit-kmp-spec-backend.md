# Task Tree

- `Inventory and review every backend spec project`()
- `Propose consolidation mapping and identify concrete hazards`()
- `Record source evidence and validation requirements`()

# Details

- Parent: [KMP source-set migration plan](2026-10-12-discuss-kmp-spec-impl-source-sets.md#L1).
- Scope: all `agent-context`, `agent-runtime`, `agent-session`, `agent-state`,
  and `agent-storage` projects in `Kodex/`; include their implementations and test support.
- Baseline: `4388871137c56bd1b73d5b739cc98f036a790f3c`; verify HEAD before/after review.
- Read [confirmed design](../../checklist/kmp-spec-impl-source-sets.md).
  Target is platform/intermediate -> commonMain (impl) -> specMain.
- Static audit only. No build, product edits, commits, device/IDE control, or new Sessions.
- Only this report is writable by this review line.
- Deliver a complete per-spec inventory, concrete old->new project/source mapping,
  mechanical moves vs decisions, local and cross-domain edges, and file/line evidence.
- Review one-to-many/pure specs, platform coverage, source/dependency scope, cycles,
  plugin/generated code, expect/actual, tests/resources, and native interop.
- Distinguish observed blockers, plausible unverified risks, and checks needed.
  Do not claim compilation or performance results from source inspection.

## Review result

### 审查范围、基线与结论

- 静态审查完成；本子任务脚本已完成，供协调者汇总，**不是迁移授权**。
- 开始核对：Kodex HEAD = `4388871137c56bd1b73d5b739cc98f036a790f3c`，源码工作区干净。
  结束核对见文末。外层已有其他任务修改，未触碰。
- 按 `git ls-files` 扫描本领域全部 180 个跟踪文件：
  33 个构建脚本、147 个 Kotlin 文件（98 commonMain、45 commonTest、
  jsMain/jvmMain/mingwX64Main/posixMain 各 1）。
  其中 spec **16 个**：context 6、runtime 4、session 1、state 2、storage 3。
  无跟踪的 resources、cinterop definition、独立生成源码目录。
- 所有 33 个项目均使用 `kodex.kmp-host`，不是 `kmp-shared`；
  JVM、JS Node、linuxX64、linuxArm64、macosArm64、mingwX64 覆盖相同。
  [Host 插件][host] L8–21、[目标与编译设置][compiler] L13–33。
  显式 API、JDK 25、warnings-as-errors 要保留；不增加 wasm/iOS 或裁剪 targets。
- 可优先评估 **7 个合并单元、减少 10 个叶项目**，详见下表。
  对 203 个本仓叶构建脚本的字面 `project(...)` main 依赖作内存中图投影，
  其他领域保持原图，候选方案未发现 main 项目环。
  这只是静态字面边检查，不等同 Gradle variant、metadata 或编译验证。
- 不能把 runtime/session/state 的中央 contract 机械塞进其实现：
  已有明确闭环证据（H1–H3）。
  clean-models 也不能因名字是 spec 就忽略其 sealed 层次、序列化与工具反向依赖。

### 映射规则与完整 spec 清单

- 下表旧路径以 `Kodex/` 为根；旧 Gradle ID 是 `:` 加路径各段用 `-` 连接。
  例：`agent-runtime/spec/decorator/compact` =
  `:agent-runtime-spec-decorator-compact`。
  真实注册与 ID 规则见 [settings][settings] L66–72、L360–429。
- **M**：候选合并。旧 spec `src/commonMain/kotlin/**` 原 package/KDoc
  移到目标 `src/specMain/kotlin/**`；所有列出的 impl
  `src/commonMain/kotlin/**` 移到目标同名层；
  旧 `commonTest` 合入目标 `commonTest`，platform/intermediate 原名保留。
  创建真实 `specMain`，`commonMain.dependsOn(specMain)`，不建 `implMain`。
- **K**：本轮建议保留现有项目、路径与源集；不是遗漏。
  单独改名或新造空层不减少项目数，不与本轮合并绑在一起。
- 新目录/ID 是**待用户确认的规划命名**，非已存在项目或 canonical 决定。

| # | 旧 spec 路径；跟踪源码 main/test 数 | 相关实现（不是所有依赖者都算实现） | 候选目标与处置 |
| --- | --- | --- | --- |
| C1 | `agent-context/spec/agents-md`；1/0 | `agent-context/impl/agents-md-filesystem`（加载快照）；prefix 使用它；renderer 消费模型 | **M** → `agent-context/agents-md` / `:agent-context-agents-md`，合 spec + filesystem |
| C2 | `agent-context/spec/available-skill`；1/0 | 无独立同名 impl；skill-filesystem 生产 AvailableSkill，prefix-render 渲染 | **K** → 原路径；是 prefix 与 skill 的共享模型，不能单独归入某一个 renderer |
| C3 | `agent-context/spec/contract`；1/0 | 无唯一 impl；AgentContextSettings 由应用/RPC 与测试提供，prefix/两类 filesystem 借用 | **K** → 原路径；保留设置契约及序列化数据 |
| C4 | `agent-context/spec/prefix`；1/0 | `agent-context/impl/prefix`（resolver）+ `agent-context/impl/prefix-render`（3 个 renderer 文件） | **M** → `agent-context/prefix` / `:agent-context-prefix`，一对多一起合并 |
| C5 | `agent-context/spec/prompt-dsl`；1/1 | 无独立 impl；PromptXmlBuilder 已包含构造/转义算法 | **K** → 原路径；已有自足 DSL，不为减少项目数之外的目标强拆接口 |
| C6 | `agent-context/spec/skill`；2/0 | `agent-context/impl/skill-filesystem`（SkillsResolver/ResolvedSkills）+ `agent-context/impl/skill-render` | **M** → `agent-context/skill` / `:agent-context-skill`，一对多一起合并 |
| R1 | `agent-runtime/spec/contract`；2/0 | `agent-runtime/impl/composition` 的私有 AgentRuntimeImpl；compact/tool 是 ResumableAgentLayer 实现 | **K** → 原路径；暂不并入 composition（H1） |
| R2 | `agent-runtime/spec/decorator/compact`；2/0 | `agent-runtime/impl/decorator/compact`（runtime + retry policy） | **M** → `agent-runtime/decorator/compact` / `:agent-runtime-decorator-compact` |
| R3 | `agent-runtime/spec/decorator/tool`；1/0 | `agent-runtime/impl/decorator/tool`（runtime + routing） | **M** → `agent-runtime/decorator/tool` / `:agent-runtime-decorator-tool` |
| R4 | `agent-runtime/spec/steer`；1/0 | `agent-runtime/impl/steer`（StateFlowSteerProvider） | **M** → `agent-runtime/steer` / `:agent-runtime-steer` |
| S1 | `agent-session/spec/contract`；2/0 | `agent-session/impl/filesystem` + `agent-session/impl/in-memory`；分别创建真实 session，且包含缓存/存储包装 | **K** → 原路径，两种 impl 均保留（H2）；不是只找到一个 impl |
| T1 | `agent-state/spec/context-window`；1/1 | 无同名 impl；文件直接实现 token budget 扩展函数；compact/get-context-remaining 调用 | **K** → 原路径；不并入 state 实现（H3） |
| T2 | `agent-state/spec/contract`；4/0 | `agent-state/impl/state` 主实现，另 runtime 各层委托实现此接口 | **K** → 原路径；不与 impl/state 机械合并（H3） |
| D1 | `agent-storage/spec/clean-models`；40/8 | 无同名 impl；模型文件本身带 projection/failure 逻辑，filesystem/in-memory 保存这些对象 | **K** → 原路径；完整 sealed/serializer 单元保留 |
| D2 | `agent-storage/spec/contract`；5/0 | `agent-storage/impl/filesystem` + `agent-storage/impl/in-memory`；另 session 缓存包装、RPC/client、app/rpc 只读实现 | **M** → `agent-storage/core` / `:agent-storage-core`，合 spec + 两种 storage impl，**不吞 session/RPC 包装** |
| D3 | `agent-storage/spec/contract-ext`；4/1 | 无同名 impl；四个文件实现初始化、projection、compaction/retention 扩展，测试用 in-memory | **K** → 原路径；其 main 依赖 D2 新项目，test 也改指 D2；可另评估合入 core，但本方案不做 |

### 每个 spec 的依赖、插件差异与消费方变化

本节 `A/I` 表示原 `api/implementation`；没有标注 test 的依赖均来自
commonMain。M 项的 spec 原依赖必须搬到 **specMain**，不能只保留在
commonMain；impl 自己的库与依赖仍在 commonMain。内部被合并的
`project(...)` 边删除，而不是改成 self dependency。原有 A/I 不借迁移改写。

#### Context（C1–C6）

- **C1**：[spec 构建][c1] L1–9：Host、A kotlinx-io。
  [filesystem][ci1] L1–16：额外 TestBalloon；A settings-contract、
  C1、utils-kotlinx-io-coroutines-impl，I coroutines；
  test I 同 IO impl。合并后这些 IO/协程实现依赖留 commonMain，
  specMain 仅需 kotlinx-io。
  C4 的 specMain 依赖 C1 新项目会合法取得 filesystem 实现及其传递依赖，
  但 C4 契约源码仍只引用模型。直接消费者原来只有 filesystem 自身、
  prefix spec；prefix resolver 通过 filesystem 使用 `loadAgentsMd`。
  所有这几个项目引用改为新 C1，load 函数 package 不改。
  [模型][agents-model] L29–62 的 warning sealed 层次整体留 specMain，
  不把三个子类分散到 commonMain。
- **C2**：[构建][c2] L1–9：Host、A kotlinx-io，无测试/序列化插件。
  `AvailableSkillModels.kt` 含 SkillScope、SkillSource、AvailableSkill、
  SkillWarning；C4/C6 spec 均 A 它，
  prefix-render A 它，skill-filesystem I 它且 test I 它，skill-render test I 它。
  保留此共享入口可避免把 metadata 契约错误绑定到文件系统或渲染实现。
  没有消费者 ID 变化；C4/C6 合并后上游引用保持原 ID。
- **C3**：[构建][c3] L1–12：Host + serialization；A serialization-core、
  utils-shell-client-spec、kotlinx-io。
  [设置模型][context-settings] L18–54 的接口与两个 Serializable 类在同文件；
  不按“纯接口”只迁一半，也不移动到 prefix/skill 模块造成共同配置耦合。
  两个 filesystem、prefix、session contract、state impl、state-test、
  app/component/context-source-settings/spec、rpc/spec/models 直接消费；
  所有 ID 保留。RPC 还有具体 [ContextSnapshot][backend-settings]
  L143–151 实现，不属于单个 filesystem 实现。
- **C4**：[spec][c4] L7–11：A C1、C2、utils-shell-client-spec、kotlinx-io。
  [resolver][ci4] L8–20：A C3、C4、openai-models、coroutines、kotlinx-io，
  I 两个 filesystem、utils-os-environment-spec；test I IO impl。
  [render][ci4r] L8–13：A C4、C2、openai-models，
  I prompt-dsl、datetime。合并保留 TestBalloon，
  **不因 resolver 的 settings 依赖给 specMain 倒灌 C3**。
  state impl 原来分别 I resolver/render；integration-test 引 renderer。
  两个入口一并换成 C4 新 ID，去重依赖；resolver/render 的 package 与
  `render` 扩展调用保持不变。
- **C5**：[构建][c5] L1–4：Host + TestBalloon，无额外库。
  prefix-render 与 skill-render I 它；合并以后仍由 C4/C6 commonMain I 它。
  [PromptXml.kt][prompt] L15–82 的 public Builder 使用 internal constructor
  和 internal build；同文件含私有 escape 算法 L84–124。
  把 public Builder 放 specMain、构造或 helper 单独留 commonMain
  将产生 spec 反向可见性要求。应保留整个文件；如未来重划职责另行设计，
  不能用“声明名字公开”作为机械拆文件依据。
- **C6**：[spec][c6] L7–9：A C2、kotlinx-io；
  [filesystem][ci6] L8–17：A C3、C6、IO impl、coroutines，I C2，
  test I C2/IO impl；[renderer][ci6r] L8–13：A C6，I C5，test I C2。
  合并两个 impl 保留 TestBalloon；
  对 C2 的 A 依赖移到 specMain，C3/IO/coroutines/prompt-dsl 依赖留 commonMain。
  直接消费者原为两个 impl；prefix resolver 的 I skill-filesystem
  换 C6 新项目，不把 Settings/ResolvedSkills 复制一份。
  [SkillModels][skill-model] L12–20 的 SkillResourceResult 子类型与根一起
  留 specMain，具体 FileSystemResolvedSkills 仍是 commonMain 私有实现。

#### Runtime（R1–R4）

- **R1**：[构建][r1] L7–11：Host；
  A state-contract、storage-clean-models、tool-unified-exec-spec、coroutines。
  [composition][ri1] L8–35 除 contract 还有 A session/state contract，
  I 三个 decorator/steer、state impl、storage contract、MCP/OpenAI、
  apply-patch/current-time/get-context-remaining/image-generation/plan/
  tool-search/unified-exec/view-image/web-run、kodex-home、logging、
  shell-client impl 等，并无 TestBalloon 插件/自有 test。
  不可将以上依赖放进 contract commonMain 后称“只是 source-set 改名”；
  H1 闭环需先决策。直接消费者 composition、两个 decorator 的 spec/impl、
  session contract 均保留 R1 ID。
  composition 中私有 AgentRuntimeImpl，和 ResumableAgentLayer 的
  decorator 委托实现都纳入本轮审查，不等于各自有重复的 AgentRuntime 类。
- **R2**：[spec][r2] L7–8：Host、A R1；
  [impl][ri2] L8–26：增加 TestBalloon；
  A 自身 spec、R1、steer spec、model-catalog、logging、coroutines；
  I state-context-window；test I tool decorator、steer impl、state impl、
  state-test、in-memory storage、openai-client-test、model-catalog impl、
  utils-coroutines-spec、coroutines-test。合并后 specMain 只 A R1；
  steer 引用按 R4 换 ID，commonTest 的 impl 引用也同步。
  composition、integration-test 的 compact impl 引用改新 R2。
  retry exception 仍属于 specMain，internal retry policy 留 commonMain。
- **R3**：[spec][r3] L7–8：Host、A R1；
  [impl][ri3] L8–30：增加 TestBalloon，
  A 自身 spec、R1、tool-contract、tool-search-impl、logging、coroutines，
  I utils-logging-impl；
  test I state impl/state-test/in-memory storage/MCP/openai-client-test/
  json-codec/apply-patch/plan/unified-exec/utils-coroutines/
  IO impl/shell-client-impl/kotlinx-io。
  specMain 仍只 A R1，不把 concrete ToolSearchEngine 当 contract 依赖。
  composition、compact tests、MCP composition tests 的 impl 引用改新 R3；
  其他合法实现组合保留。
  [ToolRuntime 实现][tool-runtime] L26–32、L152–164 的 internal constructor
  与 public factory 整体在 commonMain，不能为“合并后访问方便”放宽可见性。
- **R4**：[spec][r4] L7–9：Host、A clean-models/coroutines；
  [impl][ri4] L8–14：增加 TestBalloon，A 自身 spec/coroutines，
  test I utils-coroutines-spec/coroutines-test。
  specMain 显式 A clean-models/coroutines；StateFlowSteerProvider 留 commonMain。
  compact commonMain 原 A steer-spec；composition、compact tests、
  integration-test 原引用 steer-impl，全部改 R4 新 ID。
  不把 `waitForSteer` 与 `take` 合成 expect/actual；它们是同一队列的普通契约。

#### Session 与 State（S1、T1–T2）

- **S1**：[spec][s1] L7–15：Host；A C3、R1、storage-contract、MCP、
  openai-client、model-catalog、shell-client-spec、coroutines。
  [filesystem][si1] L8–25：TestBalloon；A S1/IO impl/coroutines，
  I composition/state impl/filesystem storage/utils-coroutines/
  filesystem-lease/read-write-mutex/cache4k；
  test I openai-client-test/storage-ext/另一 in-memory session/session-test/
  coroutines-test。
  [in-memory][si2] L8–21：TestBalloon；A S1/coroutines，
  I composition/state impl/in-memory storage/utils-coroutines，
  test I storage-ext/session-test/openai-client-test/json-codec/coroutines-test。
  两 impl 都创建真实 runtime，不能因为一个是 in-memory 就挪进测试源集。
  本轮保留三个 ID；仅它们的 storage、steer/decorator 引用随对应合并更新。
  S1 直接消费者另有 composition、session-test、RPC server。
  内含有行为的 [KodexAgentDependencies.close][session-dependencies] L10–45：
  逐项关闭借用链并保留 suppressed 异常；此文件是契约的一部分，
  不把 close 行为分发到两个 impl 制造副本。
- **T1**：[构建][t1] L8–20：Host + TestBalloon；
  A state-contract/openai-models/model-catalog，I storage-contract；
  test I state impl/state-test/in-memory storage/openai-client-test/
  model-catalog-impl/utils-coroutines-spec。
  [算法][budget] L20–32 从一个 snapshot index 读 settings/tokenCount；
  没有独立 impl，不能漏掉此 spec 下的可执行逻辑。
  main 消费者 compact、get-context-remaining；保留原 ID。
- **T2**：[spec][t2] L7–11：Host；A coroutines/storage-contract/
  clean-models/openai-models。
  [impl][ti2] L8–38：额外 TestBalloon；
  A T2/C3/storage-ext/MCP/openai-client/coroutines/tool-search-spec；
  I prefix/prefix-render，以及 tool-search/apply-patch/current-time/
  image-generation/view-image/get-context-remaining/plan/request-user-input/
  multi-agent/unified-exec/web-run impl、json-codec/utils-coroutines；
  test I prefix-render/两种 storage/openai-client-test/IO impl/coroutines-test。
  T2 不合并；composition、runtime contract、T1、history spec/viewmodel、
  app/rpc、RPC server、get-context-remaining、plan 等继续用 T2 原 ID。
  [历史 impl package 中的异常][state-exception] L5–14 实际属于 spec，
  不可按 package 名误移到实现；接口及扩展同文件有 KDoc/可观察行为。

#### Storage（D1–D3）

- **D1**：[构建][d1] L1–25：Host + serialization + TestBalloon；
  A serialization-json/schema-json/openai-models、
  tool-image-generation/request-user-input/multi-agent/tool-search/unified-exec/
  view-image 的 specs、utils-patch-spec。
  test I json-codec/patch-impl/IO impl。完整保留 40 main、8 test 文件，
  包括 CleanOpenAiEvent、PendingToolEventFailure、stable 与 unstable 两组；
  它不是“40 个只有字段的 DTO”。
  直接消费者 runtime/steer/state/storage contracts、tool-contract、
  RPC contract/models、app agent/composer/history-index/history/
  request-user-input/suggest-subagent-task specs、history view、view/components。
  这些消费者 ID 不变，工具领域合并却可能令它新增环（H4）。
- **D2**：[spec][d2] L1–12：Host + serialization；
  A coroutines/clean-models/openai-models。
  [filesystem][di2f] L5–31：Host + serialization + TestBalloon，
  A D2/IO impl/serialization-json，I json-codec；test I storage-ext；
  [in-memory][di2m] L1–16：Host + TestBalloon，
  A D2，I coroutines/read-write-mutex-impl；
  test I storage-ext/coroutines-test。
  新 core 取插件并集；specMain A 原三项，commonMain 保留 IO/json-codec/
  json/read-write-mutex 等实现依赖，两组 commonTest 都搬，不用它们
  的运行结果推断另一平台已通过。
  filesystem 的四个 actual 目录和 posix 模板也一起搬（H6）。
  所有原 D2、filesystem、in-memory 引用换 `:agent-storage-core` 并去重。
  包括 S1 的 spec 和两 impl、state/compact/tool tests、integration-test、
  history-index/history 的 spec/impl、app/view/app RPC test-support、
  RPC client/contract/server 等，不能只替换生产入口。
  跨领域实现 [RpcCachedIndexVersioned][rpc-cache] L106–116、
  [RpcSessionBinding][rpc-storage] L150–157 继续留各自项目；
  session 的 [CachedAgentStorage][session-cache] L55–65 与 L127–135
  也是 D2 契约实现，但有 Session 生命周期，不归入 core。
- **D3**：[构建][d3] L8–12：Host + TestBalloon，A D2，
  test I in-memory storage；改后两者均指 core，分别仍是 main/test 边。
  四个文件为 AgentStorageInitialization、AgentStorageProjection、
  CompactionStorage、RemoteCompactionRetention，无独立 impl。
  state impl A 它；RPC server I 它，app/test-support/rpc main I 它；
  session/两 storage/integration/app history/application/session tests 消费它。
  不搬成纯 test-support，不漏掉生产依赖，也不为了切断测试边把
  初始化与 checkpoint 的操作挪到各 storage 中复制。
  若未来将此项目也合进 core，需把扩展实现放 commonMain、重新检查
  spec 对它们的引用；不在本候选“7 单元”中暗含这一变更。

### 具体排雷：观察到的阻塞与待验证风险

#### H1 — Runtime contract 并 composition 会闭环（已观察到依赖证据）

- `composition --I--> decorator-compact.impl --A--> runtime-contract`；
  `composition --I--> decorator-tool.impl --A--> runtime-contract`。
  证据：[composition][ri1] L8–14、[compact][ri2] L9–11、
  [tool][ri3] L9–10 及两个 decorator spec 对 contract 的 A 边。
- 即使两个 decorator 各自 spec+impl 已折叠，仍是
  `RuntimeCombined -> CompactCombined/ToolCombined -> RuntimeCombined`。
  `implementation` 不是消除项目环的办法，改成 `api` 也不能解决。
- 另有 `composition --A--> session-contract --A--> runtime-contract`
  （[ri1] L9、[s1] L9），所以只“把所有 runtime 项目并成一个”
  也还会留下 `RuntimeCombined -> session-contract -> RuntimeCombined`。
- 决策选项：本方案保留 R1 作为基础契约；
  若坚持消除此项目，先设计拆出不依赖 runtime 的服务集合，
  或把多领域组合根统一重划。不能伪装成机械搬文件。

#### H2 — Session contract 与任一种仓库合并都会回到自己

- `session-filesystem --I--> runtime-composition --A--> session-contract`，
  `session-in-memory --I--> runtime-composition --A--> session-contract`。
  证据：[si1] L12、[si2] L11、[ri1] L9。
- 不需要两个 impl 一起合并就能触发环。
  concrete 代码确实需要该边：
  [buildMasterAgentRuntime][composition-code] L40–42/L67–95 取
  KodexAgentDependencies；[filesystem factory][session-factory] L45–57
  先建 state 再建 runtime，再把 lease 绑定 Session job。
- 建议保留 S1；若只抽出 KodexAgentDependencies 到独立下层，需另外审查
  `session-contract -> runtime-contract` 与 H1 的组合，不能声称已解决全图。

#### H3 — State contract 合 impl/state 的跨领域环

- `state-impl --I--> tool-plan-impl --A--> state-contract`：
  [ti2] L24、[plan-build] L10。
- `state-impl --I--> tool-get-context-remaining-impl --A--> state-contract`，
  且另一支是 `get-context-remaining -> context-window -> state-contract`：
  [ti2] L23、[budget-tool-build] L10/L15、[t1] L9。
- 把 context-window 也并入 state 不能切断前两条。
  state 实现中的 [KodexVisibleToolSpecs][visible-tools] L9–24/L33–43
  从具体工具取 specs，这是现有职责，不是无用 Gradle 依赖。
- 建议保留 T2/T1；若后续要合并，先把工具 description 与绑定 state 的
  handler 拆开或注入，属于 API/组合职责改动，需 tools 审查线共同决策。

#### H4 — clean-models 的跨领域工具合并反向环

- D1 A tool-unified-exec-spec（[d1] L17），而 unified-exec impl
  A tool-contract（[exec-build] L23–24），tool-contract A D1
  （[tool-contract-build] L10）。
  该工具若合并 spec/impl，会形成
  `clean-models -> unified-exec-combined -> tool-contract -> clean-models`。
- image-generation、view-image 同样有
  `D1 -> tool-spec -> tool-impl -> tool-contract -> D1`；
  [d1] L13/L18、[image-build] L9–12、[view-image-build] L9–10。
  tool-search 不照搬这个结论：其 impl 当前依赖 model/schema/search-index，
  不直接 A tool-contract（[search-build] L8–13）。
- 即使本领域不动 D1，外线候选也会影响图；请协调者对这些边统一投影。
  不在此报告替 tools 决定“全部合并”或修改其源码。

#### H5 — test-support 不能用无类型项目图判死环，也不能挪到 commonTest

- session-test 是 **发布 main fixture**，A S1、I state-test/
  openai-client-test/model-catalog/models/coroutines
  （[session-test-build] L7–13）。
  [fixture][session-fixture] L15–30 返回具体依赖集，
  app/test-support/rpc 的 main 也用它。
  state-test 是 TestAgentContextSettings/TestMcpService fixture，
  [state-test-build] L7–12 只依赖 C3/MCP/coroutines/shell/IO，
  **当前不依赖 state-contract/impl**，不能凭目录名断言它构成 state 环。
- 保留两个 fixture 项目主源集与消费者的 test 边；合并到某个项目
  commonTest 会使其他项目无法通过普通 project(main variant) 获取它们。
  若变更发布测试变体，需另行设计而非目录移动。
- D3 `commonTest -> in-memory.main` 与 in-memory
  `commonTest -> D3.main` 当前并存（[d3] L9/L12、[di2m] L14）。
  D2 core 候选后是 `core.test -> D3.main -> core.main` 及
  `D3.test -> core.main`；不存在 `main -> test` 反向边，
  不应把这种测试菱形误报成 H1–H3 那种生产 main 环。
  实际 task/metadata 测试接线仍需授权后验证。

#### H6 — expect/actual、internal、sealed 与插件/生成符号

- 唯一声明式 expect/actual 组为 [StorageUri][storage-uri] L5：
  `internal expect Path.toStorageUri()` 及 js/jvm/mingwX64/posix actual。
  它是 filesystem **实现细节**，放 core commonMain，不因含 expect
  就放 specMain。四份 actual 仍原源集，posix 覆盖 linux 双架构与 macosArm64；
  显式 [模板][di2f] L12–20 不可被新 dependsOn 弄丢。
  Windows formatter L7–16 及 [JS actual][storage-uri-js] L5–15 的
  Windows/非 Windows 路径分支要保留，不新增 native cinterop。
- Sealed 根与 direct subclasses 必须作为整体保留原源集/包关系：
  C1 AgentsMdWarning、C6 SkillResourceResult、T2 KodexAgentStateValue，
  D1 的 CleanIndexEntry/StableIndexEvent/StableWorkEvent/
  CompactionRetainedItem/UnstableCleanEvent/PendingToolEvent 以及工具子层次。
  D1 [StableToolHistory][stable-history] L12–98、
  [PendingToolHistory][pending-history] L10–62 的 internal helper
  被模型 projection 引用；不可把 helper 放 commonMain、模型留 specMain。
- 同项目 impl 可看到 spec internal 是已选设计带来的边界变化。
  不为了迁移把 `internal` 改 public；也不将 private AgentRuntimeImpl、
  private filesystem skill authority 或 cache 生命周期暴露为“公共契约”。
  编译器不能代替“spec 禁止引用 impl”的职责检查。
- Serialization 在 C3、D1、D2 spec，以及 filesystem impl 启用；
  候选 core 必须保留此插件及 specMain 的依赖。
  D1 模型已有 `serializer()` 调用/SerialName；D2
  [TokenCountSnapshot][token-model] L7–56 为持久化模型；
  [FileSystemAgentStorage][filesystem-storage] L26–65 实际调用这些 serializer。
  不可假定插件只为旧 commonMain 生成结果后，新 specMain 自动有符号；
  必须验证新编译/metadata 的生成符号、sealed 序列化完整性。
- 本领域 33 个构建脚本未配置 KSP、RPC 编译插件或自定义生成任务。
  TestBalloon 插件注入测试支持（[test-plugin] L1–8、
  [compiler] L50–55）；没有源码跟踪的 generated tree
  不表示可以删除插件生成的 test entry point。
  此处是源码观察，没有读取旧 build 输出当作迁移验证。

#### H7 — 持久化与生命周期回归点是具体行为，不是“移动应该没影响”

- core 两种存储测试全部保留；契约是六个 timeline。
  [FileSystemAgentStorage][filesystem-storage] L85–102 的目录常量与
  [FileSystemLayout][filesystem-layout] L140–166 的 record/latest 命名
  不能因改项目路径而改。filesystem-layout 当前只依赖 IO impl，
  **storage impl 并不直接依赖它**；它服务 app/migration 历史布局，
  保留 `agent-storage/impl/filesystem-layout`，
  不把迁移历史协议绑到新的 core。
- D2 原纯契约消费者 RPC client/contract、history spec 会随 core 新增
  filesystem、JSON codec 和内存锁实现的依赖闭包；S1 contract 也受影响。
  允许消费实现不等于依赖闭包不变。原 in-memory 的 coroutines 是 I，
  合并后继承 specMain 的 A coroutines；这是依赖并集的具体例子，
  需审查发布依赖与 JS/Native linker inputs，而非承诺 spec-only 隔离。
- D1 goldens 是测试 Kotlin 中的字符串而不是 resources 目录：
  [StableTimelineGoldenTest][golden] L7–59；
  filesystem [兼容测试][legacy-storage-test] L22–43 使用历史 serialized type，
  [LegacySettingsCompatibilityTest][legacy-settings-test] L11–35 覆盖旧 settings。
  不变 package、SerialName、默认字段、JSON codec 和 fixture 字节；
  “只改目录所以无需 serializer 测试”不成立。
- 生命周期保持
  Session scope → child State scope → decorator/工具借用链。
  [Session factory][session-factory] L33–67 管 lease 关闭及失败清理；
  [composition][composition-code] L75–95 管 fixed-tools 关闭，
  L144–183 管取消后 clearPending 与 runningTurn 槽；
  不能借一次合并改为全局单例缓存或把关闭责任转给 decorator。
- 针对行为的现有反例测试：[running-turn][turn-tests] L196–225/L353–398
  检验 shared services 关闭顺序、manual/automatic compaction 占槽；
  [state-hard-cutover][state-cutover-test] L40–79 检验初始化失败取消子 scope
  但不关闭借用资源；[session-cleanup][session-cleanup-test] L53–140
  检验 create/fork 异常及取消清理；这些仍在原测试层，不能只保留 happy path。

### 机械可迁移项、待决策项与建议批次

- **可机械规划**：M 行按上述真实源集移动；保留 package/KDoc/文件名；
  specMain 搬原 spec A/I 依赖，commonMain 保留 impl 依赖；
  并 TestBalloon/serialization 插件；精确改项目 ID/去 self 边；
  更新 settings 模块树及外域消费者；不改业务代码、序列化协议和协程责任。
- **批次建议**（尚未执行）：
  1. R4 steer：最小一对一，覆盖 coroutine contract 与测试依赖；
  2. C1 agents-md、C6 skill，再 C4 prefix：覆盖一对多、共享模型、
     renderer 与 filesystem 组合，C2/C3/C5 保留；
  3. R2 compact、R3 tool：保持 R1，覆盖 composition/MCP/integration 消费；
  4. D2 core：一对多、serialization、posix/expect/actual 与 fixture 菱形，
     独立观察更大的依赖传播范围；D1/D3 保留。
- **必须决策**：
  - 是否接受上述 7 个单元及新路径，或将 filesystem-heavy 合并延后；
    不能为“16 个 spec 全扫完”强求 16 个都消失。
  - R1/S1/T2 保留契约，还是另行授权重划依赖/组合根以消环；
    本文不选择后者、不做新接口或 service-container 设计。
  - C5/T1/D3 这类原 spec 内已有逻辑的模块是否另起职责清理；
    本轮保留并明确记录，避免把“职责完美分离”冒充搬迁结果。
  - 全局 convention 的接线/精确 internal 诊断抑制由协调者定；
    仅使用 `CommonMainOrTestWithDependsOnDiagnostic` 的选定例外，
    检查范围不得泛化到新的 commonTest 边，不在每模块复制构建逻辑。
  - 工具合并必须先处理 H3/H4 的跨领域环；本报告静态无环候选
    **只在其他领域保持原图时成立**。

### 验证清单

#### 本轮已做（只读/文档）

- [x] 加载项目指导、指定任务/设计、文档及工作流技能。
- [x] 清点 16 spec、全部 33 本领域构建脚本及 147 Kotlin 文件；
  扫描 imports/声明/测试用例/序列化与平台差异，无缺失 spec 行。
- [x] 同时扫描全仓直接 `project(...)` 消费边；区分 main/test；
  对 naive contract 合并给出闭环链，并投影 7 单元候选。
- [x] 记录只有源码支撑的观察，未把静态扫描当编译/性能结果。
- [x] 检查本文内部名词、方案与 canonical 方向一致，保留待决策事项。
- [x] 仅修改本子任务，未启动 Gradle、编译、IDE、远程操作或新 Session；
  未提交、未改产品、未改主任务/其他审查者文件。

#### 获授权后才执行

- [ ] 模型：逐目标核对 `platform/intermediate -> commonMain -> specMain`，
  commonTest 不新增倒边；posix/Main/Test 层次都仍存在。
- [ ] 诊断对照：无抑制出现选定错误、有精确抑制只解除该错误；
  不屏蔽其他错误；对 compiler warnings-as-errors 与 internal 属性警告
  单独确认，不改成宽泛 suppression。
- [ ] 图：在所有领域实际候选合并之后重算 main 图，
  对 H1–H4 给出消环证明；再分 main/test task 与 metadata edges 验证，
  不让 fixture main 反向依赖被测试的 test variant。
- [ ] API/依赖：逐个被替换的旧 ID 零残留（含 settings、integration 与测试）；
  不重复 DTO/alias/转发项目；核对所有 commonMain→specMain 继承的库；
  确认 spec 不能引用 commonMain，合法 impl 组合仍可用，
  及 internal 可见性变化没有被当作任意职责突破。
- [ ] 每个 Host target 的 metadata/编译与独立消费者；
  JS Node 不能因只有 Native/JVM 实验就漏验；不扩大 target 矩阵。
- [ ] 测试完整性：C1 1、C4 4、C6 2、R2 2、R3 1、R4 1、
  D2 6 个 Kotlin 测试文件仍被发现；保留 C5 1、T1 1、D1 8、D3 1、
  S1 两 impl 共 5、T2 impl 11、filesystem-layout 1 个测试文件
  及 session-test/state-test 的 main fixture 可消费性。
- [ ] 状态/取消：运行 existing steer/compaction/tool、runningTurn/CAS、
  StateFileSystemPublication/CachedAgentStorage、
  FileSystemSessionCleanup/FileSystemWriteCleanup 等反例；
  borrowed client/catalog/MCP 不被中间层关闭，lease 与工具不泄漏。
- [ ] 持久化：D1 全部 sealed unions、旧 type golden、旧 settings、
  Raw fork/copy、StorageUri 四平台 actual、六 timeline 常量与读写协议；
  specMain serializer 与 TestBalloon 生成入口实际可见。
- [ ] app/RPC 集成与独立 main fixture 消费；publication coordinates、
  runtime classpath/JS 包依赖变化由消费者验证，不要求 spec-only 产物。
- [ ] 获准隔离资源后再测真实 IDEA Sync/导航/索引/内存与增量编译；
  项目减少 10 个仅为图计数，不是性能收益数据。

### 结束核对

- 再次只读核对 Kodex HEAD：
  `4388871137c56bd1b73d5b739cc98f036a790f3c`，与开始一致；
  `git -C Kodex status --porcelain` 为空。
- 本文文件 URI/行号锚点已逐个检查存在；16 个 spec 清单与已跟踪构建脚本
  相符。文档复核修正了 RPC 具体类名和 JS actual 分支描述。
- 外层修改仅本子任务为本次审查所写；并行任务既有修改未覆盖。
  无临时文件、后台会话或持有资源需要释放。

### 证据索引（本地文件 URI；行号为审查基线）

[settings]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L360
[host]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-host.gradle.kts#L8
[compiler]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/KodexHostKmp.kt#L13
[test-plugin]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-tests.gradle.kts#L1
[c1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/agents-md/build.gradle.kts#L1
[c2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/available-skill/build.gradle.kts#L1
[c3]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/contract/build.gradle.kts#L1
[c4]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/prefix/build.gradle.kts#L7
[c5]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/prompt-dsl/build.gradle.kts#L1
[c6]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/skill/build.gradle.kts#L7
[ci1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/agents-md-filesystem/build.gradle.kts#L1
[ci4]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix/build.gradle.kts#L8
[ci4r]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix-render/build.gradle.kts#L8
[ci6]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/skill-filesystem/build.gradle.kts#L8
[ci6r]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/skill-render/build.gradle.kts#L8
[r1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/contract/build.gradle.kts#L7
[r2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/decorator/compact/build.gradle.kts#L7
[r3]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/decorator/tool/build.gradle.kts#L7
[r4]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/steer/build.gradle.kts#L7
[ri1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/build.gradle.kts#L8
[ri2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/compact/build.gradle.kts#L8
[ri3]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/build.gradle.kts#L8
[ri4]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/steer/build.gradle.kts#L8
[s1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/spec/contract/build.gradle.kts#L7
[si1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/build.gradle.kts#L8
[si2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/in-memory/build.gradle.kts#L8
[t1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/spec/context-window/build.gradle.kts#L8
[t2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/spec/contract/build.gradle.kts#L7
[ti2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/build.gradle.kts#L8
[d1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/build.gradle.kts#L1
[d2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract/build.gradle.kts#L1
[d3]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract-ext/build.gradle.kts#L8
[di2f]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/build.gradle.kts#L5
[di2m]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/in-memory/build.gradle.kts#L1
[session-test-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/test/build.gradle.kts#L7
[state-test-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/test/build.gradle.kts#L7
[plan-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/plan/build.gradle.kts#L10
[budget-tool-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/get-context-remaining/build.gradle.kts#L10
[exec-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/build.gradle.kts#L23
[tool-contract-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/contract/build.gradle.kts#L10
[image-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/build.gradle.kts#L9
[view-image-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/view-image/build.gradle.kts#L9
[search-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/tool-search/build.gradle.kts#L8
[agents-model]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/agents-md/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/agentsmd/contract/AgentsMdModels.kt#L29
[context-settings]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/contract/AgentContextSettings.kt#L18
[prompt]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/prompt-dsl/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/promptdsl/PromptXml.kt#L15
[skill-model]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/skill/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/skill/contract/SkillModels.kt#L12
[tool-runtime]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/KodexToolRuntimeImpl.kt#L26
[composition-code]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L40
[session-dependencies]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/contract/KodexAgentDependencies.kt#L10
[session-factory]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexAgentSession.kt#L33
[session-cache]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt#L55
[session-fixture]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/test/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/test/TestKodexAgentDependencies.kt#L15
[budget]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/spec/context-window/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contextwindow/ContextWindowTokenBudget.kt#L20
[state-exception]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateInvalidTransitionException.kt#L5
[visible-tools]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/tool/KodexVisibleToolSpecs.kt#L9
[storage-uri]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/StorageUri.kt#L5
[storage-uri-js]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/jsMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/StorageUri.js.kt#L5
[stable-history]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableToolHistory.kt#L12
[pending-history]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/unstable/PendingToolHistory.kt#L10
[token-model]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/TokenCountSnapshot.kt#L7
[filesystem-storage]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemAgentStorage.kt#L26
[filesystem-layout]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem-layout/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystemlayout/FileSystemLayout.kt#L140
[golden]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/src/commonTest/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableTimelineGoldenTest.kt#L7
[legacy-storage-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemAgentStorageTest.kt#L22
[legacy-settings-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentstorage/filesystem/LegacySettingsCompatibilityTest.kt#L11
[turn-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/in-memory/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/inmemory/AgentRuntimeRunningTurnTest.kt#L196
[state-cutover-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonTest/kotlin/io/github/stream29/kodex/agentstate/impl/AgentStateHardCutoverTest.kt#L40
[session-cleanup-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemSessionCleanupTest.kt#L53
[backend-settings]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalState.kt#L142
[rpc-cache]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/rpc/client/RpcCachedIndexVersioned.kt#L106
[rpc-storage]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionBinding.kt#L150
