# Task Tree

- `Load authorized scope, frozen design and baseline with existing pilots`()
- `Project frontend consolidation graph and check move destinations`()
- `Implement all frontend spec units without changing declaration contents`()
  - `Merge S into VM/specMain`("account-usage")
  - `Merge S into VM/specMain`("application-preferences")
  - `Merge S into VM/specMain`("authentication-settings")
  - `Merge S into VM/specMain`("composer")
  - `Merge S into VM/specMain`("context-source-settings")
  - `Merge S into VM/specMain`("history")
  - `Merge S into VM/specMain`("history-index")
  - `Merge S into VM/specMain`("hook-settings")
  - `Merge S into VM/specMain`("mcp-settings")
  - `Merge S into VM/specMain`("new-session")
  - `Merge S into VM/specMain`("new-session-defaults")
  - `Merge S into VM/specMain`("openai-login")
  - `Merge S into VM/specMain`("path-picker")
  - `Merge S into VM/specMain`("request-user-input")
  - `Merge S into VM/specMain`("runtime-configuration")
  - `Merge S into VM/specMain`("session-catalog")
  - `Preserve S to VM/specMain pilot`("session-delete")
  - `Merge S into VM/specMain`("session-rename")
  - `Merge S into VM/specMain`("session-settings")
  - `Merge S into VM/specMain`("session-title-settings")
  - `Merge S into VM/specMain`("settings")
  - `Merge S into VM/specMain`("suggest-subagent-task")
  - `Merge S into VM/specMain`("usage-reset")
  - `Merge S into VM/specMain`("working-directory")
  - `Merge S into V/specMain`("session-tab-bar")
  - `Atomically merge app/spec/application and S(session-sidebar) into app/impl/application/specMain`()
  - `Merge app/settings/spec/persistence into app/settings/impl/filesystem/specMain`()
  - `Migrate retained S(agent) commonMain into its own specMain`()
  - `Migrate retained app/spec/session commonMain into its own specMain`()
  - `Migrate retained app/migration/spec commonMain into its own specMain`()
  - `Rewrite app-local old IDs and remove consolidation self-edges`()
- `Check byte identity, dependencies, plugins, scope and static graph`()
- `Record coordinator handoff and unexecuted acceptance gates`()
- `Accept scoped repository validation and archive the implementation task`()

# Details

## 收尾验收（当前）— Session 787 / 2026-10-13

- 用户授权“收尾，没有问题则标记完成”；协调者已完成冻结 `bbbc9525` 的
  全仓迁移检查，以及 remote MacBook 的真实 IDEA/导航、同预算暖 Sync、
  增量对照、metadata、完整 Release CLI 与 112 次原生测试；本线纳入主任务 scoped 验收，
  已移入 done。证据已保留并核对，私有验证副本及拥有资源已清理。
- 原四个运行时失败与两个测试编译缺口已在干净基线复现，不算通过；
  live/原禁用 IO、不可用宿主及外部已发布二进制兼容不在完成声明内。
- 详细结果见 [主任务](2026-10-12-discuss-kmp-spec-impl-source-sets.md#L1)；
  下文是实施与交接历史，旧“未验收/留 executable”不再代表当前状态。

## 协调者整合回填（历史）

- settings/全仓消费者已切换，完整模型、生产图和实际 JVM/JS/Linux main 编译
  已验证。测试与干净基线缺口见 [主任务](2026-10-12-discuss-kmp-spec-impl-source-sets.md#L1)。
- 本线交接后，Composer 测试因合并后的合法 smart cast 去掉一个 redundant cast；
  同一 assertIs/相等断言保留。下文字节一致统计是交接时状态，最终 432/433
  个测试文件字节不变，另一个仅此等价改动。
- 下文待协调者/待运行清单是交接历史，不代替主任务的实际结果和剩余门槛；
  本线没有自行运行重型验证，保持 executable 等待整仓人工验收。

## 授权、范围与基线

- 用户授权“全仓迁移执行”；本 Session 独占此报告及 `Kodex/app/` 产品变更。
- 主任务：[全仓源集迁移](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-12-discuss-kmp-spec-impl-source-sets.md#L1)。
  冻结规则：[KMP spec/impl 源集](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/kmp-spec-impl-source-sets.md#L1)。
  历史审查：[前端审查](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-12-audit-kmp-spec-frontend.md#L1)。
- 产品基线 `450e3bd7eeef460c7293d6f6fedb55cc1a4242c5`，加主 Session 当前试点。
  不复原已完成的 session-delete，不触碰其他子线的未提交变更。
- 不改 `settings.gradle.kts`、根、build-logic、其他产品域、主任务或全局 checklist。
  不提交、不另建 Session、不运行 Gradle、编译、测试、IDE 或远程重型操作。
- 已只读识别 IDEA 打开 BuildKodex；不操作用户窗口或触发 Sync。
- 源码完成不代表构建验收；报告留在 executable，等待协调者验收，不进 done。

## 完整迁移清单（31 个旧 spec）

- 路径相对 `Kodex/`；项目 ID 为 `:` 加路径以 `-` 连接。
- `S(c)=app/component/<c>/spec`，`VM(c)=app/component/<c>/impl/viewmodel`，
  `V(c)=app/component/<c>/impl/view`。映射不改目标项目 ID、API 或 package。
- 每行契约 `src/commonMain/kotlin/**` → 唯一目标 `src/specMain/kotlin/**`；
  原 spec `commonTest` → 目标 `commonTest`。原实现、平台、renderer、资源均原位。
- “文件/测试”为基线数量；session-delete 的 1 个文件已在目标 specMain。
  implementation 的测试不计入此列，但同样必须逐字节保留。

| # | 旧 spec | 唯一目标 | 契约文件 / 契约测试 | 插件职责 |
| --- | --- | --- | ---: | --- |
| 1 | S(account-usage) | VM(account-usage) | 2 / 0 | VM + serialization |
| 2 | S(agent) | 原项目保留，内部 specMain | 2 / 0 | CLI |
| 3 | S(application-preferences) | VM(application-preferences) | 2 / 1 | VM + serialization |
| 4 | S(authentication-settings) | VM(authentication-settings) | 3 / 1 | VM + serialization |
| 5 | S(composer) | VM(composer) | 1 / 0 | VM |
| 6 | S(context-source-settings) | VM(context-source-settings) | 2 / 0 | VM |
| 7 | S(history) | VM(history)，不合并 View | 12 / 2 | VM |
| 8 | S(history-index) | VM(history-index) | 1 / 0 | VM |
| 9 | S(hook-settings) | VM(hook-settings) | 1 / 0 | VM |
| 10 | S(mcp-settings) | VM(mcp-settings) | 2 / 0 | VM |
| 11 | S(new-session) | VM(new-session) | 1 / 0 | VM + kRPC patch |
| 12 | S(new-session-defaults) | VM(new-session-defaults) | 2 / 2 | VM + serialization |
| 13 | S(openai-login) | VM(openai-login) | 3 / 1 | VM |
| 14 | S(path-picker) | VM(path-picker) | 4 / 0 | VM，Koin 注解留 commonMain |
| 15 | S(request-user-input) | VM(request-user-input) | 1 / 0 | VM |
| 16 | S(runtime-configuration) | VM(runtime-configuration) | 1 / 0 | VM |
| 17 | S(session-catalog) | VM(session-catalog) | 4 / 1 | VM + serialization |
| 18 | S(session-delete) | VM(session-delete)，既有试点不回退 | 1 / 0 | VM |
| 19 | S(session-rename) | VM(session-rename) | 1 / 0 | VM |
| 20 | S(session-settings) | VM(session-settings) | 1 / 1 | VM |
| 21 | S(session-sidebar) | app/impl/application，与 application 原子 | 1 / 0 | VM + kRPC patch |
| 22 | S(session-tab-bar) | V(session-tab-bar) | 1 / 1 | View + Compose |
| 23 | S(session-title-settings) | VM(session-title-settings) | 3 / 1 | VM + serialization |
| 24 | S(settings) | VM(settings) | 2 / 0 | VM + serialization + kRPC patch |
| 25 | S(suggest-subagent-task) | VM(suggest-subagent-task) | 1 / 0 | VM |
| 26 | S(usage-reset) | VM(usage-reset) | 2 / 0 | VM |
| 27 | S(working-directory) | VM(working-directory) | 1 / 0 | VM |
| 28 | app/spec/application | app/impl/application，与 sidebar 原子 | 3 / 0 | VM + kRPC patch |
| 29 | app/spec/session | 原项目保留，内部 specMain | 4 / 0 | CLI |
| 30 | app/migration/spec | 原项目保留，内部 specMain | 5 / 1 | Host + tests，保留 JS/Node |
| 31 | app/settings/spec/persistence | app/settings/impl/filesystem | 2 / 0 | CLI + serialization + tests |

## 源集、依赖与插件落点

- 30 个唯一迁移目标均先应用 `kodex.kmp-spec-impl`，再应用原目标/renderer convention；
  实际 `specMain` 及唯一 `commonMain → specMain` 由现有 convention 创建。
  不用 commonMain 目录别名，不新建 implMain/specTest，不复制 convention。
- 原 spec 的 `commonMain.dependencies` 上移 `named("specMain").dependencies`。
  implementation、测试、平台库及 View 框架依赖不随契约上移。
- VM 表示保留 `kodex.kmp-viewmodel` 的五个 CLI targets 和 Koin compiler；
  Koin core/annotations 注入仍在 commonMain。所有原 tests 插件保留。
- 保留 7 个 serialization 使用者：account-usage、application-preferences、
  authentication-settings、new-session-defaults、session-catalog、
  session-title-settings、settings；原 serialization-json 测试库留 commonTest。
- new-session、settings、application 原 kRPC patch 不删除、不改变 substitution。
- Agent/root Session 保留共享契约边界；migration 保留 Host 的 JS/Node，
  impl 仍是原 CLI 项目，不强行扩大实现平台或裁契约平台。
- migration impl 的 commonMain 版本生成、历史测试排除/filter 不改；
  root View 的 jvmTest 文档资源、全部 Mosaic/Native/test 层保持。
- application/sidebar 一次合入同一 owner；删除内部 sidebar 依赖与目标自边，
  specMain 只汇总两个原契约块，不移入 RPC/server/DI/文件系统实例依赖。
- persistence specMain 保留 RPC models/coroutines/io 的 API；
  filesystem impl、kaml、serialization 实现库继续 commonMain。
  见 [真实依赖块](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/impl/filesystem/build.gradle.kts#L10)。

### 契约依赖核对矩阵

- 本 Session 将原 spec API 配置逐项移入 specMain，不改变 api/implementation。
  以下为前端落地时的逻辑依赖；外部项目 ID 的最终拼写由协调者跨域重写，
  不以这张逻辑表覆盖并行更新。`C=coroutines.core`、`I=kotlinx.io.core`、
  `Ser=serialization.core`；VM/S 缩写沿用迁移表，均是项目 API。

| 契约 owner | specMain 直接 API 输入 |
| --- | --- |
| VM(account-usage) | OpenAI account-usage、C、Ser |
| S(agent) | VM(composer/runtime-configuration/history-index/request-user-input/suggest-subagent-task/history)、storage clean-models、OpenAI models、RPC models、tool request-user-input/multi-agent/unified-exec、utils rpc-exception、C、I |
| VM(application-preferences) | Ser、C |
| VM(authentication-settings) | OpenAI models、C、Ser |
| VM(composer) | storage clean-models、OpenAI models、C |
| VM(context-source-settings) | agent-context contract、C |
| VM(history) | storage clean-models/contract、state contract、C |
| VM(history-index) | storage contract/clean-models、C |
| VM(hook-settings) | Hook notification contract、C |
| VM(mcp-settings) | MCP contract、C、I |
| VM(new-session) | app/spec/session、VM(composer/runtime-configuration) |
| VM(new-session-defaults) | Ser、OpenAI models、C |
| VM(openai-login) | C |
| VM(path-picker) | C、I |
| VM(request-user-input) | storage clean-models、tool request-user-input、C |
| VM(runtime-configuration) | OpenAI models、C |
| VM(session-catalog) | VM(session-delete)、C、Ser |
| VM(session-delete) | 原契约无显式依赖，沿用现有试点 |
| VM(session-rename) | C |
| VM(session-settings) | OpenAI models、VM(path-picker/working-directory/session-rename)、C、I |
| V(session-tab-bar) | C；View 框架只在下游 |
| VM(session-title-settings) | Ser、OpenAI models、C |
| VM(settings) | VM(session-settings/new-session-defaults/context-source-settings/session-title-settings/application-preferences/authentication-settings/account-usage/usage-reset/hook-settings/mcp-settings/working-directory/path-picker)、app/spec/session、RPC models、MCP contract、OpenAI account-usage/models、C、I、Ser |
| VM(suggest-subagent-task) | storage clean-models、tool multi-agent、OpenAI models、C、I |
| VM(usage-reset) | VM(account-usage)、OpenAI account-usage、C |
| VM(working-directory) | VM(path-picker) |
| app/impl/application（含 sidebar） | VM(working-directory/session-rename/session-delete/openai-login/path-picker/session-catalog/new-session/settings/application-preferences)、S(agent)、tool multi-agent、app/spec/session、C、I |
| app/spec/session | S(agent)、OpenAI models、C |
| app/migration/spec | utils kotlinx-io-coroutines spec |
| app/settings/impl/filesystem | RPC models、C、I |

- [Agent 真实 specMain](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/spec/build.gradle.kts#L8)、
  [root Session 真实 specMain](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/spec/session/build.gradle.kts#L8)、
  [migration 真实 specMain](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/spec/build.gradle.kts#L9)
  均保留原 API 与项目边界，commonMain 不填造实现。
- [Path Picker](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/impl/viewmodel/build.gradle.kts#L9)
  展示真实的 C/I 上游与 coroutines/filesystem/os 下游分层；
  Koin 注解源码未移动，linuxX64Test 原位保留。

### Tab-bar Compose 静态核查

- 纯契约源码没有 Compose/Mosaic import，但合并到 View 后会参与 Compose
  编译插件的目标编译。不能以“没有 @Composable”推断无需生成依赖。
- 官方 2.4.20 插件会执行 runtime 检查及 class stability lowering：
  [IR 接线](https://raw.githubusercontent.com/JetBrains/kotlin/v2.4.20/plugins/compose/compiler-hosted/src/main/java/androidx/compose/compiler/plugins/kotlin/ComposeIrGenerationExtension.kt)，
  [stability lowering](https://raw.githubusercontent.com/JetBrains/kotlin/v2.4.20/plugins/compose/compiler-hosted/src/main/java/androidx/compose/compiler/plugins/kotlin/lower/ClassStabilityTransformer.kt)。
- 本仓 Mosaic runtime 的 commonMain API 导出 Compose runtime；
  tab-bar 已在 commonMain 增加 `implementation(libs.mosaic.runtime)`，
  供下游共享/目标编译的插件生成代码使用。原 mosaicMain 的 runtime/UI
  依赖不搬动；specMain 仅保留原 coroutines API，不注入 View 框架。
- 这属于静态依赖接线推断，不是编译结果；尤其 specMain 独立 metadata
  compilation 与 Compose 插件生成的实际行为仍须协调者验证。若与冻结边界
  冲突，记录并停止该单元，不以向 specMain 注入框架库或关闭插件绕过。
  见 [tab-bar specMain/commonMain/mosaicMain 接线](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/impl/view/build.gradle.kts#L9)。

## 静态检查与实施进度

- [x] 基线和试点 diff 已读；31 个 spec 完整覆盖。
- [x] 投影当前前端映射，分离 Main/Test，并加入显式 kRPC patch 候选边：
  未发现生产环；源文件/测试目标路径无碰撞。不是 Gradle resolved graph。
- [x] 产品源码移动及构建脚本完成，session-delete 原样保留。
- [x] 基线逐字节对照：全部 **440 个 app src 文件**按映射完全一致，无漏项、
  内容变化或额外源文件。含 **72 个契约、191 个 Kotlin 测试文件**
  （其中 12 个原 spec 测试）及唯一 jvmTest JSON 资源；这是文件数，不是测试用例数。
- [x] 相对 450e3bd7 共 83 个 src 路径变化：72 个契约移动 + 11 个契约测试移动。
  包含既有 delete 的 1 个移动；本 Session 新增移动 82 个，migration 测试原位。
- [x] 30 个唯一项目 opt in，72 个 Kotlin 契约文件均实际位于 specMain。
  独立 Agent/root Session/migration 的 commonMain 无业务源码。
- [x] 600 个显式 Main/Test 依赖配置按基线、迁移映射、去自边及去同配置重复边
  集合对照一致；唯一新增库输入为 tab-bar commonMain 的 Mosaic runtime。
  原插件并集/目标 convention 保留；跨 api/implementation 的原组合依赖不擅自删。
- [x] 7 个原 spec serialization 插件并入目标；filesystem 原 serialization 另保留，
  共 8 个迁移目标应用该插件。原 Koin/Compose/kRPC/test 插件不删除。
- [x] 没有 app-local 已合并旧 ID、合并自边、specMain UI/DI import 或框架依赖；
  不新增项目级 source-set dependsOn，测试边由原 convention 保持。
- [x] 合并输入无路径碰撞；文本级顶层类型重名与同 package/filename 的
  JVM facade 候选检查均无命中。不是完整符号/ABI/编译器检查。
- [x] 固定 450e3bd7 + 主试点 + 前端映射重新投影：174 个有 build 脚本的项目、
  601 条生产边（含 kRPC patch 候选），无环；Test 单独处理。
  初次行式扫描得到 599 条，最终平衡括号扫描修正为 601 条，采用后者。
  此结果仅证明该固定投影；其他域正在并行迁移，联合最终图仍归协调者验收。
- [x] app 构建脚本基线 91 → 当前 63：28 个旧 spec 去除（含已迁 delete），
  本轮新删 27 个。**不是 settings 注册项目数或 Gradle 实际模型测量。**
- [x] app 与本报告 diff whitespace 检查通过；未写临时脚本/输出文件，
  未占用构建/远程/IDE 资源，无需清理额外资源。
- [x] 已保留协调者跨域 ID 并行重写；静态检查在产品改动后执行，
  不覆盖并行更新、不声称最后一次全仓快照已稳定。
  例如收尾 migration impl 仅出现 filesystem-lease 旧 ID 的协调者改写；
  版本生成、源目录、测试 exclusion/filter 均与基线一致。

## 协调者交接与待验收

- 协调者负责删除已合并的 28 个旧 spec 注册项（含已迁 delete）；
  本轮新增删除 27 个旧 spec build 脚本，不留下转发项目。
- 协调者统一重写跨域旧 ID（包括 app 对其他域旧 ID 的依赖）；
  本 Session 仅负责 app 对已合并 app ID 的引用。
  核验曾发现下表引用；它们是交接索引，不代表当前仍残留，
  协调者可能已在并行消除，必须以最终搜索为准。

| 协调者持有文件 | 前端旧 ID → 新 ID |
| --- | --- |
| integration-test/build.gradle.kts（jvmTest） | app-component-session-catalog-spec → app-component-session-catalog-impl-viewmodel；app-component-settings-spec → app-component-settings-impl-viewmodel；app-spec-application → app-impl-application |
| rpc/impl/server/build.gradle.kts | app-settings-spec-persistence → app-settings-impl-filesystem |
| rpc/spec/contract/build.gradle.kts | app-component-account-usage-spec → app-component-account-usage-impl-viewmodel；app-component-authentication-settings-spec → app-component-authentication-settings-impl-viewmodel；app-component-session-catalog-spec → app-component-session-catalog-impl-viewmodel |
| rpc/spec/models/build.gradle.kts | app-component-application-preferences-spec → app-component-application-preferences-impl-viewmodel；app-component-authentication-settings-spec → app-component-authentication-settings-impl-viewmodel；app-component-new-session-defaults-spec → app-component-new-session-defaults-impl-viewmodel；app-component-session-title-settings-spec → app-component-session-title-settings-impl-viewmodel |

- 前端全部映射见 31 行清单；不额外美化项目 ID。删旧注册及全仓引用后
  再检查联合生产 SCC、源集 variants/metadata 和测试任务 owner。
- 全局 checklist 路径同步也由协调者处理，本 Session 未改这些文件；
  frontend-application-boundary、path-picker 等旧路径不能被当作最终落地路径。
- 待运行：真实 source-set/compilation/metadata/variant/任务图，
  diagnostics 精确抑制控制、contract 反向可见性负例、所有原 target/test。
- 待运行：serialization/Koin/Compose/kRPC 在新源集上的生成、
  View/VM/root Application/RPC/CLI/fixtures/integration 消费者编译与回归。
- 待运行：IDE Sync/导航/索引、外部消费、其他宿主平台和性能；
  本轮不外推 session-delete 首批构建结果到其余 30 个 spec。
