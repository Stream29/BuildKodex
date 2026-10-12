# Task Tree

- `Read authorized scope, baseline, design and tool audit`()
- `Classify all thirteen tool specs and associated implementation sources`()
- `Move contracts and consolidate the nine authorized tool pairs`()
  - `Apply spec convention before host convention and lift contract dependencies`()
  - `Retain four contract projects and three independent implementations`()
  - `Preserve source bytes, targets, platform hierarchy and tests`()
- `Run scoped static checks and record evidence`()
- **`Await repository acceptance of the migration checkpoint`()**

# Details

## 协调者整合回填

- settings/全仓消费者已切换，完整模型、生产图和实际 JVM/JS/Linux main 编译
  已验证。测试与干净基线缺口见 [主任务](2026-10-12-discuss-kmp-spec-impl-source-sets.md#L87)。
- 下文静态计数及待协调者/待动态检查是本线交接时记录，非当前全仓进度；
  重型验证由协调者执行，原禁用 IO 未启用。本线仍等待整仓人工验收。

## 授权、范围与基线

- 用户本轮明确授权全仓迁移中的 tool 子线；本记录由 Session 782 独占。
- 产品写入仅限 `Kodex/tool/`；另外只维护本任务。
  根 settings、build-logic、其他域、全局 checklist 和主任务由协调者处理。
- 基线 `450e3bd7eeef460c7293d6f6fedb55cc1a4242c5` 加当前试点。
  开始时 tool 域无工作区改动；其他域已有修改不回退、不接管。
- [主任务](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-12-discuss-kmp-spec-impl-source-sets.md#L1)、
  [冻结设计](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/kmp-spec-impl-source-sets.md#L1)、
  [历史静态审查](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-12-audit-kmp-spec-tools.md#L1)
  已读取；历史审查的“未获实施授权”不覆盖本轮明确授权。
- 已加载 AGENTS、buildkodex-change、kanban/checklist/programmatic-planning、
  shared-context、ask-user、acodespace、ide-collaboration、better-code-abstract、
  better-document-workflow、gradle 和 toolchain-preference 技能。
- 已只读识别本地 IDEA 打开 BuildKodex；遵守本轮禁止操作 IDE 的边界。
  不运行 Gradle、编译、远程重型，不提交、不新建 Session；编辑使用 apply_patch。

## 全部十三个 spec 的去向

路径相对 `Kodex/`；每个合并条目同时吸收原 `tool/impl/<名字>`。
两个旧 ID 均映射为新 ID；保留项目的 ID 不变。

所有契约源码层为 `src/specMain`。文件数为新契约项目的
specMain/commonMain/commonTest；独立实现的文件数不计入该列。

| 原 spec 名字 | 新项目构建脚本 / Gradle ID | 文件数 S/M/T |
| --- | --- | --- |
| apply-patch | [tool/apply-patch](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/apply-patch/build.gradle.kts#L1-L19) / `:tool-apply-patch` | 1/2/1 |
| builder | [tool/builder](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/builder/build.gradle.kts#L1-L24) / `:tool-builder` | 1/1/1 |
| contract | 保留 [tool/spec/contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/contract/build.gradle.kts#L1-L18) / `:tool-spec-contract` | 2/1/1 |
| current-time | [tool/current-time](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/current-time/build.gradle.kts#L1-L25) / `:tool-current-time` | 1/2/1 |
| get-context-remaining | [tool/get-context-remaining](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/get-context-remaining/build.gradle.kts#L1-L33) / `:tool-get-context-remaining` | 2/1/2 |
| image-generation | 保留 [tool/spec/image-generation](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/image-generation/build.gradle.kts#L1-L16) / `:tool-spec-image-generation` | 2/0/0 |
| multi-agent | [tool/multi-agent](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/multi-agent/build.gradle.kts#L1-L21) / `:tool-multi-agent` | 3/0/1 |
| plan | [tool/plan](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/plan/build.gradle.kts#L1-L30) / `:tool-plan` | 2/1/2 |
| request-user-input | [tool/request-user-input](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/request-user-input/build.gradle.kts#L1-L21) / `:tool-request-user-input` | 3/0/1 |
| tool-search | [tool/tool-search](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/tool-search/build.gradle.kts#L1-L22) / `:tool-tool-search` | 4/2/1 |
| unified-exec | 保留 [tool/spec/unified-exec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/unified-exec/build.gradle.kts#L1-L17) / `:tool-spec-unified-exec` | 3/0/0 |
| view-image | 保留 [tool/spec/view-image](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/view-image/build.gradle.kts#L1-L16) / `:tool-spec-view-image` | 2/0/0 |
| web-run | [tool/web-run](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/web-run/build.gradle.kts#L1-L25) / `:tool-web-run` | 1/2/2 |

## 实际职责分类

- 原 spec 的 16 个生产文件所含契约全部进入真实 specMain；
  唯一混合文件 `Tool.kt` 按声明拆分。
  `JsonToolBuilderSpec` 的结果构造 helper、`ToolName` 的校验/匹配保留为
  契约默认行为，仅引用契约类型，不引用实现层。
- [Tool 接口](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/contract/src/specMain/kotlin/io/github/stream29/kodex/tool/contract/Tool.kt#L7-L24)
  进入 specMain；[typedTool 适配实现](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/tool/contract/Tool.kt#L7-L28)
  留 commonMain。typed pending/downcast/resource-free close 行为、
  声明签名及各自 KDoc 不重写；独立项目边界不变。
- 从原 impl 上移纯契约文件：
  - get-context-remaining 的 `GetContextRemainingTools.kt`；
    plan 的 `PlanTools.kt`。
  - multi-agent 的 `SuggestSubagentTaskSchemas.kt`、`SuggestSubagentTaskTools.kt`；
    request-user-input 的 `RequestUserInputSchemas.kt`、`RequestUserInputTools.kt`。
  - tool-search 的 `ToolSearchConstants.kt`、`ToolSearchSchemas.kt`、
    `ToolSearchTools.kt`：模型可见定义/默认值和基于 sourceInfo 的纯描述构造。
  - image-generation 的 `ImageGenerationSchemas.kt`、
    view-image 的 `ViewImageSchemas.kt`（包含纯 options 模型）进入各自独立契约。
- builder 的 JSON decoding/event adapters、tool-search 的 document conversion/
  engine、各客户端、状态绑定 handler、artifact persistence 保持 commonMain。
- apply-patch/current-time/image-generation/view-image/web-run 的混合 Tools 文件
  保持 commonMain：对象同时暴露规范与具体客户端构造，不把客户端依赖上移，
  不为此次目录迁移重写对象 API。
- unified-exec 的 schema/描述渲染继续 commonMain：
  `ExecCommandParametersSchema` 读取 `execCommandHostPlatform`，
  renderer 引用 `UnifiedExecTools`；不整体搬入独立契约造成反向引用。
  同文件的纯 write/output schema 本轮随混合文件保留；
  拆成两个独立项目的同名文件会产生 `UnifiedExecSchemasKt` 门面冲突，
  换文件名则改变门面归属，因此不混入未设计的 API 重组。
  两工具共享 client、close、平台 actual 和禁用 real IO 测试政策不变。
- serialization 插件取原两项目并集；契约 serialization/schema/OpenAI/libs
  放 specMain，原实现独占依赖与测试依赖保留原层。原源码中的 opt-in
  注解和项目级编译政策随源码/约定保留。
- contract 显式补充 specMain 的 serialization core（ToolName 原来借助传递
  依赖获取它）。image/view 的 schema 库随纯 schema 进入独立 specMain，
  从原 impl 删除重复直接声明，仍由契约 api 传递可见。
- multi-agent/request-user-input 原“impl”只有纯模型可见规范；合并后
  commonMain 无业务文件是实际分类结果，不虚造实现。

## 协调者集成边界

- 九组映射公式：
  `:tool-spec-X`、`:tool-impl-X` → `:tool-X`，X 为表内九个合并条目。
- 本线删除合并自边；其他旧 ID 引用可以暂留，协调者统一替换跨域引用、
  tool 内非自边以及根 settings 注册。没有 alias/转发项目或 DTO 复制。
- tool 内仍有四处 `implementation(project(":tool-impl-builder"))`：
  [current-time:18](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/current-time/build.gradle.kts#L18)、
  [get-context-remaining:20](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/get-context-remaining/build.gradle.kts#L20)、
  [plan:18](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/plan/build.gradle.kts#L18)、
  [unified-exec commonTest:32](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/build.gradle.kts#L32)；
  统一改为 `:tool-builder`，前三处 main，末处 Test。
  当前源码交付是协调者集成前状态，不能直接用于完整构建。
- 保留 image-generation/unified-exec/view-image 两项目，避免历史 B1：
  `clean-models -> tool DTO -> tool contract -> clean-models`。
- plan/context 的 B2 仍需全仓核查：若折叠 state contract/state，
  state 与两个工具及 context-window 会构成生产回路；不能因目录迁移忽略。
  main 与 commonTest 反向边分别检查。

## 静态检查结果

- 本轮源码及静态检查交付完成；任务树停在协调者集成/动态验证。
  检查通过仅指下面列出的静态断言，不指 Gradle 模型或构建通过。
- 以 `git show 450e3bd7:<原路径>` 逐文件对照：原 75 个 Kotlin 文件覆盖
  完整，当前 76 个（Tool.kt 拆为两个）。
  74 个整文件逐字节一致，其中 49 个移动、25 个原位保持；
  两个 Tool.kt 删除重复 import/header 后重组，逐字节等于原文件。
  API、KDoc、schema 文本、SerialName、默认值和业务代码均未改写。
- 源码数：specMain 27、commonMain 21、commonTest 20，
  四个原平台 Main actual 和四个 Test actual 全部保持原位/字节。
  commonTest 不增 dependsOn、不新增 specTest。
- 构建脚本静态盘点 **25 → 16**（九合并 + 四独立契约 + 三独立实现）。
  13/13 契约项目先应用 `kodex.kmp-spec-impl` 再应用 `kodex.kmp-host`，
  通过 `sourceSets.named("specMain")` 配置真实源集；源集边由既有试点
  convention 提供。本线未改 convention 或精确诊断抑制属性。
- 对每组合并脚本断言：依赖 api/implementation 并集等于原两项目减去自边，
  commonTest 块逐字一致、serialization/tests 插件保留；重复的上游 api
  已由 commonMain 继承。四个独立契约上移依赖，仅新增明确的 core/schema
  库声明；原平台 convention 与目标声明没有新增/删减。
- unified-exec impl 构建脚本逐字节未变，原 posix hierarchy、jsMain、
  测试依赖及八个 actual 文件保持。tool 本地没有直接 cinterop 配置；
  不改传递 shell/images/search-index 的 Native/JS/interop 配置。
  `realIoTestConfig` 的 `isEnabled = false` 保持，不把默认测试等同真实 IO。
- own-module specMain→commonMain 静态声明名扫描未发现反向引用；
  已排除扩展函数 receiver 名误报，屏蔽字符串/注释。这不是编译器证明。
- 合并层检查无新同包同名文件门面候选。独立 contract 的两份 Tool.kt
  是有意拆分：spec 文件只有 Tool 接口，没有顶层函数/属性门面；
  helper 在原 commonMain 文件名中保留原 `ToolKt` 门面归属。
  JVM 实际输出仍待编译核验。
- 基线字面 production project 边的九工具合并投影无非平凡 SCC；
  未求值 Gradle variant/插件生成边，也不代替协调者最终全仓图检查。
  反例复核仍得到：
  - B1 五节点 SCC：clean-models、tool contract、image-generation、
    unified-exec、view-image。
  - B2 四节点 SCC：折叠后的 state core、context-window、
    get-context-remaining、plan。
  因而保留边界的理由没有被迁移抹掉；测试反向边未混入 main。
- `git diff --check -- tool` 通过；另检查未跟踪的新 kt/kts：
  无尾随空格/CR、均有结尾换行。静态检查脚本通过 stdin 运行，
  不创建产品探针、临时文件或后台进程。
- 本任务完成删减、变更含义及全文一致性复核；所有文件 URI 存在，
  所列行号有效，唯一粗体执行位置是等待集成/验证。

## 待验证与交接

- 待协调者集成：注册新项目、清零被移除 ID 的消费者、最终跨域生产 SCC。
- 全局 tool-handler-decisions/clean-model-rust-alignment 的旧目录假设、
  主任务进度和其他域引用由协调者更新，本线不修改全局 checklist。
- 待另行授权的动态验证：Gradle 模型、specMain serializer/metadata、
  JVM/JS Node/四 Native compilation/link/interop、TestBalloon/消费者/wire golden、
  IDE 导航与性能。真实 IO 测试默认禁用，不擅自启用。
- 任务保持 executable；源码/静态检查完成不表示构建通过或验收完成。
