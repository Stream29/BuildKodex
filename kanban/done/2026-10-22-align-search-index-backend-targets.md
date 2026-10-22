# Task Tree

- `Recheck search index ownership after the ToolSearch split`()
- `Identify the remaining factory and target coverage mismatch`()
- `Distinguish library JS configuration from backend application targets`()
- `Confirm the four Native product targets without JVM`()
- `Freeze scoped target edits and protect parallel API work`()
- `Implement Native target convergence and search factory separation with subagent`()
- `Review the concrete before after project and source manifest`()
- `Run centralized serial Native metadata and lint acceptance`()
- `Record validation and remaining inactive source cleanup`()

# Details

- 用户要求并行派发 SafeRw 并继续看下一项，随后明确“JVM也移出去，我们已经不再支持JVM”。
  产品目标按[四个 Native 目标](../../checklist/kmp-spec-impl-source-sets.md#current-native-product-targets--2026-10-22)
  冻结，不再等待五/六目标选择。用户现已明确要求 subagent 执行，父 Session 同时讨论下一项。
  [Session1040](file:///home/stream/.kodex/sessions/1040) 已返回编辑回执，
  父 Session 已完成集中修正和验收。
- ToolSearch 拆分后，源码及构建脚本中的唯一直接生产消费者为
  tool/tool-search/backend，依赖为 commonMain implementation。
  前端共享 DTO 只依赖 ToolSearch contract；前序已验收的12项目24个生产配置中
  不含搜索 backend、search-index 或 Lucene。模型名含有 Search 不代表依赖此库。
- 用户质疑“当前没有 JS target，哪里来的六目标”。复核后需区分库的构建配置和
  后端应用目标；此前将两者等同并推荐“后端 Host 六目标”的方案已撤回。
- 迁移前 utils/search-index 的构建声明按 Shared 配置8目标，但工厂仅有库 Host 六目标：
  JVM、JS、linuxX64、linuxArm64、macosArm64、mingwX64。
  当时 JS 同时配置 browser/Node，二者不是独立的 Kotlin target。
  Wasm/iOS 只获得 SearchDocument/SearchIndex 等类型，没有创建能力。
  JVM/桌面 Native 使用 Lucene，JS 使用现有纯 Kotlin fallback。
- 实际 app/backend-host、app/backend-daemon 的遗留配置使用 kodex.kmp-cli，
  只调用 configureDesktopTargets：JVM、linuxX64、linuxArm64、
  macosArm64、mingwX64，共5目标，没有 JS。
  kodex.kmp-host 则额外无条件注册 JS Node；库能做 Node 测试不等于产品有 Node 后端。
- 已确认的新产品目标是 linuxX64/linuxArm64/macosArm64/mingwX64，JVM排除，
  不是沿用源码现有的5目标。ToolSearch/backend、agent-state/core、
  agent-runtime/composition 等原有含 JS 的 Host 配置，已连同依赖链收敛。
- 执行范围：主产品 build-logic 的目标/convention 配置、主产品业务项目
  build.gradle.kts 中必要的 JVM/JS 目标与依赖/任务接线，以及 utils/search-index
  的工厂 spec/actual 分离。先列实际改动清单，再在此范围实施；
  不修改嵌套上游子模块、根 lint 实现或 JVM 构建工具工程。
  configureDesktopTargets 去掉 JVM，Host 不再添加 Node，Shared 去掉 JVM但保留
  独立 Web 契约所需非 JVM 目标；搜索索引按 Native 四目标收敛。
- 不修改 SafeRw 所在 utils/read-write-mutex 的代码/测试或其新消费端用例；
  app/migration 整个模块留给下一项 API 讨论，不修改其构建脚本或生成器。
  父 Session 稍后负责集中构建；子任务不运行 Gradle/编译器，不提交推送。
- 旧 JVM/JS 源码若因目标移除变成非输入，可保留并列出清理清单，不机械改写成 Native，
  不悄悄删测试断言或宣称失去的 JVM 测试覆盖已由 Native 替代。
  不能只删搜索索引的变体导致上游继续索取它。
  不补 Wasm/iOS stub，不把原来的 JVM/Node 测试作为新产品支持范围的依据。
  Gradle/detekt 的 JVM 运行不在移除范围；Web/Wasm 契约按独立前端规划处理。
- 验收以 metadata、Native 消费者和可运行的 Native 测试为准，
  非本机目标分清 klib/可执行文件编译与真实运行。
  验收结果见下；遗留非输入源码不作为当前产品支持或测试覆盖。
- 修改前1525份源码哈希、业务构建脚本/convention和搜索索引快照：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/native-targets-before/`。
- 实测修正：4个 Native-only 值类只去除 JvmInline 注解/import，保留 value class、
  序列化与全部行为；将仍由 POSIX 使用的 BlockingCoroutineFileSystem 原文件
  逐字节移入 posixMain，避免删旧 blocking 层后漏接真正的 Native 实现。
- Lucene 依赖及共享源码目录接到4个实际 Native leaf，而非新的公共 metadata 层；
  工厂/测试分别逐字节迁到 commonMain/commonTest。
  这消除了 Lucene 与 kRPC 的 core_commonMain 元数据名称冲突，
  未改上游/依赖缓存或关闭 Werror；API/搜索算法不变，配置少于原层级。
- 最终解析已验证143个KMP项目无JVM产品目标，135个Host/CLI/Client项目均为4个Native，
  其余8个Shared项目保留独立非JVM portable覆盖，JVM构建工具不变。
  12个前端Native依赖图无MCP执行SDK、搜索执行/索引/Lucene或后端日志。
  CLI全metadata、Linux Native编译/调试链接及集成/RPC测试源码编译通过。
- Native测试按最终XML汇总为369项，367通过、2项存储失败；
  先前口头合计469属算术错误，已按XML更正。在1525份哈希完全匹配的本批迁移前工作区
  快照中以同名/同类型/同消息复现，两次测试源也相同。
  这是本批前快照对照，不称作干净HEAD复现；临时基线已清理。
  本次搜索与 Home provider 的4个Native编译均实际执行通过，其他平台未运行测试。
- 新 Linux CLI 调试可执行文件链接成功，隔离 HOME 执行 --help 返回0、
  无 stderr/文件写入，未重建或发布 release 二进制。
  严格/报告29→18项/4模块，恰好移除搜索1、Home2、MCP8，无新增发现。
  393个源码输入、空 analysis classpath，53个规则样例通过。
  旧 JVM/JS 测试并未全部转 Native；保留未激活目录与覆盖债务，不宣称全量替代。
  未改上游 fork、全局 JVM 配置或 lint 框架；未提交推送。证据：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/native4-validation-summary.json`。
- 依据：
  - [实际依赖](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/tool-search/backend/build.gradle.kts#L18)。
  - [模块目标及分层](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/build.gradle.kts#L5)。
  - [已收敛的 Host convention](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-host.gradle.kts#L1)。
  - [后端使用 CLI convention](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/backend-host/build.gradle.kts#L4)。
  - [现有四 Native 配置](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/KodexHostKmp.kt#L22)。
  - [工厂实际实现](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/src/commonMain/kotlin/io/github/stream29/kodex/utils/searchindex/SearchIndexFactory.kt#L3)。
  - [已验收的依赖隔离](../done/2026-10-21-clarify-search-index-ownership.md#L54)。
