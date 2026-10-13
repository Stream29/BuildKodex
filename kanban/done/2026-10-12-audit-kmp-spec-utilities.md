# Task Tree

- `Inventory and review every utility spec project`()
- `Propose consolidation mapping and identify concrete hazards`()
- `Record source evidence and validation requirements`()

# Details

- Parent: [KMP source-set migration plan](2026-10-12-discuss-kmp-spec-impl-source-sets.md#L1).
- Scope: all `utils` projects in `Kodex/`, implementations and their consumers.
- Baseline: `4388871137c56bd1b73d5b739cc98f036a790f3c`; verify HEAD before/after review.
- Read [confirmed design](../../checklist/kmp-spec-impl-source-sets.md).
  Target is platform/intermediate -> commonMain (impl) -> specMain.
- Static audit only. No build, product edits, commits, device/IDE control, or new Sessions.
- Only this report is writable by this review line.
- Deliver a complete per-spec inventory, concrete old->new project/source mapping,
  mechanical moves vs decisions, local and cross-domain edges, and file/line evidence.
- Review spec-only libraries, same-directory pairs, platform asymmetry, Native
  cinterop/expect-actual, pure helpers, tests/resources, and cyclic project collapse.
- Distinguish observed blockers, plausible unverified risks, and checks needed.
  Do not claim compilation or performance results from source inspection.

## Review result

### 状态、口径与结论

- 静态审查完成；未实施迁移。本子任务树已执行完毕，文件留在原位置供主任务汇总。
- 开始核对：Kodex HEAD 为 `4388871137c56bd1b73d5b739cc98f036a790f3c`，
  内层 `git status --short` 为空。结束复核见末节。
- 外层 BuildKodex HEAD 为 `c0a7d34c4d8a5a7e69538e68d80b56d5fd0bc671`；
  外层存在其他任务的已修改、删除及未跟踪文件，本线不处理它们。
- 范围：18 个 spec、11 个独立 impl，共 29 个 utility 构建项目；
  221 个跟踪文件，其中 189 个 Kotlin 文件。只以跟踪的源码、构建脚本为依据，
  不把工作目录里残留的旧 `build/` 产物或旧目录当成当前项目。
- 18 个 spec 全部登记在
  [settings:473–539](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L473)。
  注册器以目录段拼平项目 ID，只在根目录或有 build 脚本的目录注册项目：
  [settings:58–87](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L58)。
- 核心结论：
  - 5 对平台集合相同，可进入具体合并设计；并不表示源码职责已全部可机械搬迁。
  - 6 对平台集合不同，直接沿用任一侧 convention 都不能证明保留原平台覆盖。
  - 7 个没有独立 impl 的库都含函数体或平台实现，不能把“只有 spec 项目”误当成“只有契约”。
  - 确认 2 类源码静态阻塞：IO 同包同名 internal 常量、Shell 同名 JVM 文件门面。
  - utilities 候选折叠图有 18 节点、15 条不同的跨库边；未发现环。
    这是静态图结论，不是 Gradle 模型、编译、发布或运行验证。

### 目标结构与路径记号

- 固定采用
  [canonical 设计](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/kmp-spec-impl-source-sets.md#L3)：
  `platform/intermediate -> commonMain (impl) -> specMain`，
  `commonMain.dependsOn(specMain)`，不新建 `implMain`，不恢复空 commonMain 根的旧方案。
- 消费者可以依赖合并项目中的实现；不以“只能消费 spec artifact”为审查条件。
  同时，契约代码不能靠反向依赖获取 commonMain 的实现符号或依赖。
- 下表的 `M(name)` 是**候选**：
  `:utils-name-spec` + `:utils-name-impl` → `:utils-name`，
  项目目录 `Kodex/utils/name/`，保留原 Kotlin package。
  不是将 11 个库进一步合成一个大 utils 项目。
- 对获准的 M：
  - 原 `spec/src/commonMain/kotlin/**` 中确认属于契约的内容 →
    `src/specMain/kotlin/**`；原有默认实现/算法先按各项的职责决策处理。
  - 原 `impl/src/commonMain/kotlin/**` → `src/commonMain/kotlin/**`。
  - 原 `impl/src/<平台或中间>Main/**` → `src/<相同源集名>/**`；
    `.def` → `src/nativeInterop/cinterop/`，同时修正项目相对路径。
  - 原两侧 `src/*Test/**` → 同名 test 源集；
    不新增 specTest、不额外设置 commonTest.dependsOn(specMain)。
  - 原 spec 依赖配置迁到 specMain，impl 配置迁到 commonMain/原中间层；
    删除旧 impl→本库 spec 的项目边，改依靠源集 dependsOn。
  - 两个旧项目 ID 的消费者边均重定向到新 ID，保留其原声明所在源集和
    api/implementation 意图；同配置重复边再去重，不能顺便把测试依赖提升到 main。
- `P(name)` 表示本轮建议保留 `:utils-name-spec` 及
  `Kodex/utils/name/spec/` 原路径：没有独立 impl 可减少，先不为凑层次制造空接口。
  若另行决定重命名到 `:utils-name`，属于独立路径/发布决定，不是本次已选合并。
- 精确 KGP 诊断抑制已由主任务确认：
  `kotlin.internal.suppressGradlePluginErrors=CommonMainOrTestWithDependsOnDiagnostic`。
  本线不重复实验、不将它登记为“无解阻塞”；它也不解决下面的平台、符号或门面问题。

### 平台基线

| 记号 | 当前声明的 targets / 环境 | 构建证据 |
| --- | --- | --- |
| S/shared | JVM；JS browser+Node；wasmJs browser+Node；linuxX64、linuxArm64、macosArm64、mingwX64、iosArm64 | [shared:19–38](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-shared.gradle.kts#L19) |
| H/host | JVM；JS Node；linuxX64、linuxArm64、macosArm64、mingwX64；无 wasmJs/iosArm64 | [host:8–21](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-host.gradle.kts#L8)、[host targets:22–32](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/KodexHostKmp.kt#L22) |
| C/cli | JVM 与上述四个桌面 Native；无 JS/wasmJs/iosArm64 | [cli:8–10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-cli.gradle.kts#L8) |

- H 的 mingw binaries 注入 `-lole32`；切到 S 不是仅改 targets 名单，
  还会丢失这类行为。JS target 相同也不意味着 browser/Node 环境等价。
- utilities 本身没有 Mosaic 层；消费方有，例如
  [OpenAI login view:11–19](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/impl/view/build.gradle.kts#L11)。
  合并不能把其 mosaicMain 的 utility 依赖粗暴提升到其他层。

### 全部 spec 清单与候选映射

“impl 数”是独立 Gradle impl 项目数；后文另列同项目中的平台后端/具体实现数量。
测试数字是跟踪的 Kotlin 文件数，含 fixture，不是测试 case 数。

| spec（链接为该项目 build 脚本） | impl 数；spec→impl 平台 | 候选、新路径或保留方式 | 源/测试概况 |
| --- | --- | --- | --- |
| [coroutines](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/coroutines/spec/build.gradle.kts#L1) | 0；S | P(coroutines)，纯辅助函数，不制造第二项目 | spec commonMain 2、commonTest 2 |
| [external-url](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/spec/build.gradle.kts#L1) | 1；S→C | M(external-url)，平台方案未决前保留原两项目 | spec main 1；impl commonMain 2、JVM/Native 各 1、commonTest 1 |
| [filesystem-lease](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/spec/build.gradle.kts#L1) | 1；H→H | M(filesystem-lease) | spec main 1；impl commonMain 4、commonTest 3、jvmTest 1 |
| [images-codec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/spec/build.gradle.kts#L1) | 1；H→H | M(images-codec)，与 images 保持两个库 | spec main 2；impl commonMain 2、平台 5、commonTest 3 |
| [images](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/build.gradle.kts#L1) | 0；S | P(images)，已有契约与算法混合 | spec commonMain 6、commonTest 6 |
| [kodex-home](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kodex-home/spec/build.gradle.kts#L1) | 0；H | P(kodex-home)，host 查询辅助库 | spec commonMain 1、commonTest 1 |
| [kotlinx-io-coroutines](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/spec/build.gradle.kts#L1) | 1；S→H | M(kotlinx-io-coroutines)，平台决策及常量冲突解除前保留 | spec main 2；impl commonMain 1、平台/中间层 13、测试 4 |
| [kotlinx-io-serialization](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-serialization/spec/build.gradle.kts#L1) | 0；S | P(kotlinx-io-serialization)，已有具体 serializer | spec commonMain 1；无自有测试 |
| [ktor-client-ext](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/spec/build.gradle.kts#L1) | 1；H→H | M(ktor-client-ext)，需裁定辅助函数/引擎职责 | spec commonMain 4、commonTest 2；impl linuxMain 14、jvmTest 1、linuxX64Test 5 |
| [logging](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/logging/spec/build.gradle.kts#L1) | 1；S→H | M(logging)，须保留 scoped API 宽平台及窄 fileLogging 层 | spec commonMain/commonTest 各 1；impl fileLoggingMain/Test 各 1 |
| [os-environment](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/spec/build.gradle.kts#L1) | 0；H | P(os-environment)，不等于没有实现 | spec commonMain 1、JVM/JS/Native 各 1、commonTest 1 |
| [patch](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/spec/build.gradle.kts#L1) | 1；H→H | M(patch)，需裁定算法和 filesystem 操作归属 | spec commonMain 4、commonTest 4；impl commonMain 1、commonTest 2 |
| [process-client](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/spec/build.gradle.kts#L1) | 1；S→H | M(process-client)，平台方案未决前保留 | spec main 1；impl commonMain 2、后端 4、`.def` 1、测试 8 |
| [read-write-mutex](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/spec/build.gradle.kts#L1) | 1；S→S | M(read-write-mutex)，最直接的首批候选 | spec main 1；impl commonMain 2、commonTest 2 |
| [rpc-exception](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/rpc-exception/spec/build.gradle.kts#L1) | 0；C | P(rpc-exception)，保留序列化契约与恢复辅助逻辑 | spec commonMain 7、commonTest 1 |
| [search-index](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/spec/build.gradle.kts#L1) | 1；S→H | M(search-index)，平台方案未决前保留 | spec main 1；impl commonMain/JS/luceneMain 各 1、commonTest 1 |
| [shell-client](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/spec/build.gradle.kts#L1) | 1；S→H | M(shell-client)，平台、门面及中间层问题解除前保留 | spec main 5、commonTest 1；impl commonMain 6、平台/中间层 18、`.def` 2、测试 12 |
| [terminal-text](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/terminal-text/spec/build.gradle.kts#L1) | 0；S | P(terminal-text)，Unicode 算法辅助库 | spec commonMain/commonTest 各 1 |

### 逐库源码、实现和职责审查

1. **coroutines（P，0 独立 impl）**
   - `childScope`、`supervisorChildScope`、`cancelAndJoin` 与
     `runCatchingCancellable` 已经是完整实现，不是未实现声明：
     [ChildScope:22–53](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/coroutines/ChildScope.kt#L22)、
     [RunCatchingCancellable:9–12](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/coroutines/RunCatchingCancellable.kt#L9)。
   - 只依赖 coroutines，无 platform/expect/actual。可保留这种有规范 KDoc 的辅助 API；
     不为获得 specMain 人造接口。两份 commonTest 继续随库保留。
2. **external-url（M，1 项目、JVM/Native 两条 actual 路径）**
   - 真正契约是
     [OpenExternalUrlResult:4–12](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/externalurl/ExternalUrl.kt#L4)，
     候选放 specMain；两个 impl common 文件放 commonMain，JVM/nativeMain 保持。
   - [openExternalUrl:15–20](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/externalurl/ExternalUrl.kt#L15)
     调用 internal expect；[impl:2、9–11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/impl/build.gradle.kts#L2)
     是 C，依赖 process-client impl。沿用 S 会缺 JS/wasm/iOS 实现，沿用 C 会删除原 spec targets。
   - 两侧同名 `ExternalUrl.kt` 不可覆盖；分到不同源集可保留。
     spec 文件只有类型，不据此宣称有 JVM 文件门面冲突。
   - login view 同时消费 result 与函数：
     [OpenAiLoginPopup:25–26](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/OpenAiLoginPopup.kt#L25)；
     合并后合法，不应把实现函数从调用方隐藏。
3. **filesystem-lease（M，1 项目、三种工厂）**
   - [FileSystemLease:14–34](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLease.kt#L14)
     是契约/异常；四个 impl common 文件实现普通、共享读、写 lease 和支持逻辑。
     都是 H，无新增平台 actual，路径移动较直接。
   - [impl:3、10–18](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/build.gradle.kts#L3)
     的 serialization 插件及内部 heartbeat JSON 留在实现侧；
     契约 coroutines API 依赖放 specMain，IO impl/coroutines helper/os-environment 留 commonMain。
   - [跨进程测试:23–28](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/jvmTest/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseProcessTest.kt#L23)
     硬编码 `FileSystemLeaseProcessTestKt`，只移动目录不改 package/文件名；
     合并后的测试 classpath/子进程入口待运行核验。
4. **images-codec（M，1 项目、6 个具体 transformer 对象）**
   - spec 的异常类型可入 specMain；但
     [PromptImageCodecIo:23–66](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/codec/PromptImageCodecIo.kt#L23)
     是 read/write 辅助实现。需决定保留为有规范的契约辅助函数，还是整体放 commonMain；
     不可拆走调用体后留下 spec→impl 引用。
   - Korim、JVM ImageIO、Sharp、Skiko、GdiPlus、WindowsNative 都在同一 impl 项目；
     不是六个 impl Gradle 项目。WindowsNative 还有
     [GdiPlus→Korim 回退:131–150](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/mingwX64Main/kotlin/io/github/stream29/kodex/utils/images/codec/GdiPlusPromptImageTransformer.kt#L131)。
   - [HostPromptImageTransformer:8](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/codec/HostPromptImageTransformer.kt#L8)
     的四条 actual 路径为 JVM、JS、skikoNative、mingw。
     [impl:11–49](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/build.gradle.kts#L11)
     的 headless JVM、skikoNative（Linux 双架构+macOS）、Node sharp、
     TwelveMonkeys、Korim 依赖不能压平到 specMain。
   - images 的 `PromptImageTransformer` 才是基础契约；
     不把 S 的 images 并入 H 的 codec，否则同时扩大平台/算法决策范围。
5. **images（P，0 独立 impl，跨库有 codec 实现）**
   - “0”不是无实现：`PromptImageTransformer` 的具体实现来自上一项。
     [PromptImageProcessing:60–74](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/PromptImageProcessing.kt#L60)
     声明契约；同文件还含对象拷贝/规划/执行辅助逻辑，
     [EncodedImage:23–34](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/PromptImageProcessing.kt#L23)
     依赖同库 data URL 函数。
   - 六个 main 文件（byte readers、data URL、info、MIME、dimensions、processing）
     相互使用纯算法；仅抽出某个数据类到 specMain 可能留下向下调用。
     保留 S 独立库，若重新分层须按符号/函数体处理，不整目录归类。
6. **kodex-home（P，0 独立 impl）**
   - [KodexHome:3–14](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kodex-home/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kodexhome/KodexHome.kt#L3)
     是启动时求值的 host 路径，不是资源所有者契约；依赖 os-environment 辅助函数。
     保留 H 与原目录；没有把它变成纯 S 模型库的依据。
7. **kotlinx-io-coroutines（M，1 项目、4 条 filesystem 后端路径）**
   - `CoroutineFileSystem`、`FileFingerprint`、raw source/sink/closeable 是契约；
     [CoroutineFileSystem:75–125](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineFileSystem.kt#L75)
     同时含默认方法；`CoroutineRawIo.kt` 含 use/copy/read/write 的具体函数体。
     这些默认方法不能依赖移到 commonMain 才能看见的辅助符号。
   - [impl 层次:11–22](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/build.gradle.kts#L11)
     为 blocking(JVM+posix)、posix(Linux 双架构+macOS)；
     Native IoDispatcher 另在 nativeMain。保留这些交叉层次，不能只留下 nativeMain。
   - SystemCoroutineFileSystem/IoDispatcher expect 及 JVM、Node、POSIX、mingw
     actual 留实现侧；stat 时间戳/权限 actual 继续各置 linuxX64、linuxArm64、macosArm64。
     [posix:26、48–50](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/posixMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/SystemCoroutineFileSystem.posix.kt#L26)。
   - 有确定的同名 internal 常量阻塞，见下节 B1；S→H 另有平台决策。
     不补 wasm/iOS 实现，也不默认丢掉原契约的 wasm/iOS 消费能力。
8. **kotlinx-io-serialization（P，0 独立 impl）**
   - [PathAsStringSerializer:12–21](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-serialization/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxioserialization/PathAsStringSerializer.kt#L12)
     是具体 `KSerializer<Path>`，无 expect/actual、无自有测试。
   - 对外 wire contract 不能随路径迁移改变；
     [MCP:58](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpManager.kt#L58)
     及 [OpenAI:71](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/CompactionModels.kt#L71)
     直接在 `@Serializable(with=...)` 中使用它。保持独立 S utility，不创造委托层。
9. **ktor-client-ext（M，1 项目、自定义 Linux engine）**
   - 原 spec 四文件含 headers 扩展、SSE 请求流和插件；
     [SseCompatibility:25–64](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/SseCompatibility.kt#L25)
     访问 Ktor InternalAPI，`SseRequests` 通过同库 internal attribute 配合它。
     这不是“接口在 spec、唯一实现恰好在 impl”的简单配对。
   - 候选保持依赖一致的一组 SSE helpers；可整体入 commonMain，
     是否还有独立且有意义的 specMain 内容需决定，不能创建空标记契约。
   - [impl:11–25](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/build.gradle.kts#L11)
     分平台 api engines；Linux 14 文件使用 libcurl/Ktor Native，
     [KodexLibcurl:9–22](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexLibcurl.kt#L9)
     的绑定由依赖提供，不是本库缺了 `.def`。不能移入 commonMain。
   - JVM SSE 集成测试与 Linux loopback/cleanup 测试分别保留；
     JS 仍是 H/Node，不能因没有 JS impl 文件就推断整库没有 JS target。
10. **logging（M，1 项目、一个 file logger 集成）**
    - [ScopedLogging:12–102](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/logging/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/logging/ScopedLogging.kt#L12)
      实现 KLogger 包装器；不是抽象日志接口。
    - [fileLogging 模板:11–30](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/logging/impl/build.gradle.kts#L11)
      仅供 JVM/Native，依赖 IO impl 与 Kermit；
      [ApplicationLogging:12–26](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/logging/impl/src/fileLoggingMain/kotlin/io/github/stream29/kodex/utils/logging/ApplicationLogging.kt#L12)
      确有 host 文件写入/全局配置行为。
    - S scoped API 与 H file logging 不能选择其一覆盖另一者。
      若设计保留 S 项目且限制 fileLogging 为旧四个 Native+JVM，
      不能原样 `withNative()`，因为新增到该项目的 iosArm64 也会进入组。
      scoped helper 是否保留 specMain 同样是职责决定。
11. **os-environment（P，0 独立 impl、3 条 actual 路径）**
    - [common:9–35](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironment.kt#L9)
      有三个 public expect，以及依赖它们的辅助实现。
    - [JVM:5–15](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/spec/src/jvmMain/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironment.jvm.kt#L5)
      使用 System/ProcessHandle；
      [JS:5–12](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/spec/src/jsMain/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironment.js.kt#L5)
      访问 `process`；
      [Native:6–17](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/spec/src/nativeMain/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironment.native.kt#L6)
      使用 posix。保留 H，不能当作纯 spec 全移到 specMain 或 S。
    - 若未来在同项目内分层，expect 可作契约入 specMain；
      环境拼装算法与 actual 留下游，但必须保持 common helper 的可见方向。
12. **patch（M，1 项目、一个 host 默认入口）**
    - [PatchModels:16](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/PatchModels.kt#L16)
      等 wire/models 可入 specMain；解析器、行匹配和
      [显式 filesystem apply:19–28](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/PatchFileSystem.kt#L19)
      都已经是算法实现，不得只因来自 spec/ 就宣布其职责全是契约。
    - [HostPatchFileSystem:7](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/HostPatchFileSystem.kt#L7)
      是默认 SystemCoroutineFileSystem 的重载，入 commonMain。
      两个 applyToFileSystem 参数不同，不是重复符号；不得误删一个。
    - 若算法放 commonMain，连带 helper 一起下移；
      契约模型及 serialization 插件保留，KDoc 跟声明。
      原 spec 4 份/parser 测试与 impl 2 份/IO 测试全部保留。
13. **process-client（M，1 项目、JVM/Node/POSIX/Windows 四后端）**
    - spec 的 ProcessCommand/Session/Client/Exception 可入 specMain；
      common impl 的 owner factory、acquisition guard、raw IO 留 commonMain。
      同名 `ProcessClient.kt` 两侧要保留；spec 只有类型，不直接构成门面冲突。
    - [impl:13–24、43–51](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/build.gradle.kts#L13)
      的 `-Xexpect-actual-classes`、posix 模板和 Linux `_GNU_SOURCE` cinterop 均不可丢。
    - [process_client_spawn.def:1–19](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/nativeInterop/cinterop/process_client_spawn.def#L1)
      独立 cinterop package，含 Apple 特殊环境访问；Native 系统头不是可搬到 specMain 的资源。
    - S→H 决策与 IO S→H 相连；仅修本库 actual 不足以让其 commonMain 的 IO impl 依赖支持更宽平台。
14. **read-write-mutex（M，1 项目、mutex 实现和 SafeRw helper）**
    - [ReadWriteMutex:13–38](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/ReadWriteMutex.kt#L13)
      是真实 interface+states，可直接进 specMain。
    - [impl factory:11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/ReadWriteMutex.kt#L11)
      与 SafeRw 进 commonMain；S→S、没有 actual/cinterop。
      interface 与同名工厂本来就是合法 API，不是声明冲突；
      同文件名在两个源集中保留，spec 无顶层成员，不必为门面强行改名。
15. **rpc-exception（P，0 独立 impl）**
    - 六个异常类型（含 sealed base）和 RestoreRemoteException helper；
      [RemoteException:13–16](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/rpc-exception/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/rpcexception/RemoteException.kt#L13)
      的 message 自身执行序列化，
      [RestoreRemoteException:15–28](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/rpc-exception/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/rpcexception/RestoreRemoteException.kt#L15)
      是 public inline 算法。不能认为类文件没有实现逻辑。
    - 保留 C 与 JSON 插件/API 依赖；不为看起来像通用异常就拓展 targets。
      未来拆分要保留 sealed hierarchy 同源集归属与 wire discriminator；
      现有恢复/取消语义测试继续保留。
16. **search-index（M，1 项目、2 个算法后端）**
    - spec 的 SearchDocument/SearchIndex 放 specMain，factory 放 commonMain；
      [JS fallback:3–8](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/searchindex/PlatformSearchIndex.kt#L3)
      与 [Lucene:3–20](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/impl/src/luceneMain/kotlin/io/github/stream29/kodex/utils/searchindex/PlatformSearchIndex.kt#L3)
      的行为不是同一种排序实现，保留分支。
    - [impl:11–26](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/impl/build.gradle.kts#L11)
      的 luceneMain(JVM+Native) 与其依赖保留。
      若改 S，`withNative()` 会把 iosArm64 纳入；wasmJs 仍无 actual。
      两个同名 PlatformSearchIndex.kt 位于互斥平台组，不是当前 JVM 重复。
17. **shell-client（M，1 项目、JVM/Node/POSIX/mingw 四后端）**
    - spec 中 Client/Session/Settings/Shell/StdoutBuffer 与 DTO/serializer 是契约候选；
      Shell 类型识别、snapshot 渲染又含真实实现。
      声明不能在 specMain 引用只放 commonMain 的 helper；
      不因函数体存在就改 wire format 或换成 expect/actual 契约。
    - [impl:18–31、42–59](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/build.gradle.kts#L18)
      的 processClientMain 覆盖 JVM/JS/mingw/posix，承载 pipe IO；
      process-client impl 依赖在这层，不能误搬到 specMain。
      PTY4J、Node node-pty 分别留 JVM/JS；common 模板不要丢 posix。
    - [impl:62–80](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/build.gradle.kts#L62)
      的 shellClientSpawn/shellClientConPty 两个 cinterop、
      Linux `-lutil`、expect/actual-class flag 必须保持；
      三个 utility `.def` 的 cinterop package 不同，无观察到包碰撞。
    - 另有 B2 门面冲突；S→H 平台决策还牵连 process-client、IO、os-environment。
18. **terminal-text（P，0 独立 impl）**
    - [TerminalText:3–64](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/terminal-text/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/terminaltext/TerminalText.kt#L3)
      含 terminal cell DTO、grapheme/width/prefix/suffix 算法；
      [build:9–17](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/terminal-text/spec/build.gradle.kts#L9)
      两个 Unicode 依赖特意 `isTransitive=false`。
    - 保留 S 独立 helper；如果拆 DTO 到 specMain，要把 private boundary 和 Unicode
      算法作为一致实现组处理。测试覆盖以原文件为准，不声称 Unicode targets 已运行验证。

### 静态阻塞、文件与资源碰撞

- **B1／确定的同包符号冲突（IO）**：
  [spec CoroutineRawIo.kt:9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineRawIo.kt#L9)
  和 [impl SystemCoroutineFileSystem.kt:8](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/SystemCoroutineFileSystem.kt#L8)
  各声明 `internal const val CoroutineIoSegmentByteCount`。
  旧 module internal 隔离消失，合并后不能保留两份。
  若 raw helpers 仍在 specMain，保留上游一份、删除 impl 副本是候选机械修复；
  两侧当前值同为 `64 * 1024`。不能反过来只留 commonMain，
  否则 spec 的 read/write/copy 函数无权引用它。
- **B2／确定的同名 JVM 门面条件（Shell）**：
  spec 与 impl 的 `StdoutBuffer.kt` 同包，均有顶层函数；
  [spec:83–84](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/StdoutBuffer.kt#L83)
  与 [impl:237–238](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/StdoutBuffer.kt#L237)
  甚至各有 private `Long.saturatingPlus`。private 同名本身不是跨文件可见性冲突，
  但两文件会争用同一个 `StdoutBufferKt` JVM 门面；换源集目录不解决。
  候选只将 impl 文件改名为 `MutableStdoutBuffer.kt`，不要为了省文件删除契约模型方法。
  此判断由源码条件与
  [Kotlin 官方文件门面规则](https://kotlinlang.org/docs/coding-conventions.html#source-file-names)
  推得，本轮没有实际跑编译器诊断。
- **B3／确定的平台不等价（六对）**：
  external-url、IO、logging、process-client、search-index、shell-client。
  “保留窄侧 convention”删除旧 spec 平台，“直接使用宽侧 convention”又使 host
  expect/actual、Node API、窄平台依赖或 withNative 组改变覆盖；均不属无条件机械迁移。
  可选择暂保留独立 spec，或另行设计保留宽项目并限定 host 实现分支；
  不能未经授权裁剪目标或实现新平台。
- **搬迁覆盖与符号冲突分开检查**：
  两侧相同相对 main 路径共五组：external-url/ExternalUrl.kt、
  process-client/ProcessClient.kt、read-write-mutex/ReadWriteMutex.kt、
  shell-client/ShellClient.kt、shell-client/StdoutBuffer.kt。
  分别放 specMain/commonMain 避免文件覆盖；只有最后一组确认同时产生门面条件。
- 所有 29 个 utility 项目未发现跟踪的 `resources/` 文件、自定义资源源目录或
  自有代码生成源目录。无 spec/impl 同路径测试文件碰撞。
  `.def` 是 cinterop 输入而非普通 resources；
  不能据“无 resources 冲突”推断 Native 无迁移风险。
- tests 中同名 fixture 有意跨平台互斥：
  IO 的 CoroutineFileDescriptorTest、process 的 ProcessClientTestCommands、
  shell 的 ProcessSessionTestShell。不合并到 commonTest；
  同平台合并测试时也不导入旧 build 下的 TestBalloon generated entrypoint。

### 消费者与跨库边

#### 全部直接项目消费者

表中 ID 省略前导 `:`；`U/` 代表 `utils-` 前缀。按消费者项目去重，
**含 main 与 test 边**，不代表表中每个消费者都在 main 引用实现。
旧 impl→本库 spec 的 11 条内部边单列为本库 impl，不会保留为合并后的 self dependency。

| utility | 原 spec 的直接消费者（除本库 impl） | 原 impl 的直接消费者 |
| --- | --- | --- |
| coroutines | agent-runtime-impl-decorator-compact、agent-runtime-impl-decorator-tool、agent-runtime-impl-steer；agent-session-impl-filesystem、agent-session-impl-in-memory；agent-state-impl-state、agent-state-spec-context-window；app-component-history-index-impl-viewmodel、app-component-history-impl-view、app-component-history-impl-viewmodel、app-component-new-session-impl-viewmodel、app-component-openai-login-impl-viewmodel、app-component-path-picker-impl-viewmodel；app-impl-application、app-impl-session、app-impl-view、app-test-support-rpc；hook-impl-notification、integration-test、mcp-impl-composition、rpc-impl-server、tool-impl-get-context-remaining、tool-impl-plan；U/filesystem-lease-impl、U/process-client-impl、U/shell-client-impl | — |
| external-url | app-component-openai-login-impl-view | app-component-openai-login-impl-view、app-impl-view |
| filesystem-lease | 无 | agent-session-impl-filesystem、app-migration-impl |
| images-codec | 无 | tool-impl-image-generation、tool-impl-view-image |
| images | tool-impl-image-generation、tool-impl-view-image、U/images-codec-spec、U/images-codec-impl | — |
| kodex-home | agent-runtime-impl-composition、app-cli、app-impl-application | — |
| kotlinx-io-coroutines | app-migration-spec、U/images-codec-spec、U/patch-spec、U/process-client-spec | agent-context-impl-agents-md-filesystem、agent-context-impl-prefix、agent-context-impl-skill-filesystem；agent-runtime-impl-decorator-tool、agent-session-impl-filesystem、agent-state-impl-state；agent-storage-impl-filesystem-layout、agent-storage-impl-filesystem、agent-storage-spec-clean-models；app-component-path-picker-impl-view、app-component-path-picker-impl-viewmodel；app-impl-application、app-impl-rpc、app-impl-session、app-migration-impl、app-settings-impl-filesystem、app-test-support-rpc；hook-impl-notification、integration-test、openai-impl-codex-cli-storage、rpc-impl-server；tool-impl-apply-patch、tool-impl-image-generation、tool-impl-view-image；U/filesystem-lease-impl、U/images-codec-impl、U/logging-impl、U/patch-impl、U/process-client-impl、U/shell-client-impl |
| kotlinx-io-serialization | mcp-spec-contract、openai-spec-models | — |
| ktor-client-ext | 无 | openai-impl-client |
| logging | 无 | agent-runtime-impl-composition、agent-runtime-impl-decorator-tool、app-cli、app-component-session-settings-impl-viewmodel、app-impl-application、app-impl-rpc、mcp-impl-composition、rpc-impl-server |
| os-environment | agent-context-impl-prefix、app-cli、app-component-path-picker-impl-viewmodel、app-impl-application、app-impl-rpc、integration-test、openai-impl-client；U/filesystem-lease-impl、U/kodex-home-spec、U/shell-client-impl | — |
| patch | agent-storage-spec-clean-models、app-component-history-impl-view、integration-test | agent-storage-spec-clean-models、tool-impl-apply-patch |
| process-client | mcp-spec-stdio、U/shell-client-spec | mcp-impl-composition、mcp-spec-stdio、U/external-url-impl、U/shell-client-impl |
| read-write-mutex | 无 | agent-session-impl-filesystem、agent-storage-impl-in-memory、mcp-impl-composition |
| rpc-exception | app-component-agent-spec、app-component-history-index-impl-viewmodel、app-component-history-impl-viewmodel、integration-test、rpc-impl-client、rpc-impl-server、rpc-spec-contract | — |
| search-index | 无 | tool-impl-tool-search |
| shell-client | agent-context-spec-contract、agent-context-spec-prefix、agent-session-spec-contract、agent-state-test、rpc-spec-models、tool-spec-unified-exec | agent-runtime-impl-composition、agent-runtime-impl-decorator-tool、hook-impl-notification、integration-test、rpc-impl-server、tool-impl-unified-exec |
| terminal-text | app-component-session-catalog-impl-view、app-component-session-tab-bar-impl-view、app-component-suggest-subagent-task-impl-view、app-impl-view、app-view-components、integration-test | — |

#### 必须保留的消费差异和传播

- 同库 spec+impl 的消费者不等于冗余职责：
  [MCP stdio:8–19](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/build.gradle.kts#L8)
  main api process spec、commonTest implementation process impl；
  [clean models:19–24](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/build.gradle.kts#L19)
  main api patch spec、test implementation patch/IO impl。
  合并后保留源集归属，不能“去重”成 main implementation。
- [agent-context contract:10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/contract/build.gradle.kts#L10)、
  [session contract:14](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/spec/contract/build.gradle.kts#L14)、
  [RPC models:18](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/models/build.gradle.kts#L18)、
  [unified-exec spec:9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/unified-exec/build.gradle.kts#L9)
  目前 api 传播 shell spec。重定向后会传播合并库及其实现依赖闭包，
  这在设计上允许，但须验证 Node/Native cinterop/PTY 依赖不污染不适用的源集；
  不是靠把 api 改成 implementation 就能隐藏公开实现。
- 源码消费者不只上表的显式边。对全部跟踪 Kotlin import 的扫描找到
  65 个非 utility 消费项目、516 行 utility import；
  在包含测试边的保守图中均可找到依赖路径，尚不能据此证明逐源集可见性。
  部分通过其他项目 api 获取 utility：
  如 [McpStdioTransport:4–8](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/commonMain/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransport.kt#L4)
  间接使用 raw IO；frontend 的 terminal-text 经 app-view-components 传播。
  因此验证不能只挑“显式依赖 impl”的项目。
- [tool search:4–11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/tool-search/src/commonMain/kotlin/io/github/stream29/kodex/tool/toolsearch/ToolSearchEngine.kt#L4)
  同时使用 SearchDocument/SearchIndex/createSearchIndex，是合法契约+实现组合。
  [view-image:5–8](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/view-image/src/commonMain/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageToolClient.kt#L5)
  与 [image-generation:14–17](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationToolClient.kt#L14)
  同时使用 Host transformer、codec helpers、IO API/默认实现，不能误判为违规。

下列为**有源码 import、没有直接声明该 utility 项目依赖**的全部 18 个“库—消费项目”组合。
表内 main/test 为观察到的代表引用位置，不是把整个项目归类为该单一源集；
其余源码 import 消费者均已在直接消费者表中。

| utility | 间接源码消费者 | 代表源码证据 |
| --- | --- | --- |
| kotlinx-io-coroutines | app-component-settings-impl-viewmodel；mcp-spec-stdio | [settings test:10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/SettingsViewModelTest.kt#L10)、[stdio main:4](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/commonMain/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransport.kt#L4) |
| patch | agent-state-impl-state；app-component-history-impl-viewmodel；app-impl-view | [state main:39](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/tool/PendingToolEventProjection.kt#L39)、[history test:29](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/history/HistoryItemHeaderFactoryTest.kt#L29)、[view JVM test:38](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsRecordingTest.kt#L38) |
| rpc-exception | app-component-agent-impl-view；app-impl-rpc；app-impl-session | [agent mosaic test:45](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/ComposerHostRepairTest.kt#L45)、[rpc main:42](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentViewModel.kt#L42)、[session test:7](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/session/src/commonTest/kotlin/io/github/stream29/kodex/cli/session/AgentHistoryActionTest.kt#L7) |
| shell-client | agent-context-impl-prefix；agent-context-impl-prefix-render；agent-context-impl-skill-filesystem；agent-state-impl-state；app-migration-impl；app-settings-impl-filesystem | [prefix test:10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix/src/commonTest/kotlin/io/github/stream29/kodex/agentcontext/prefix/impl/AgentContextPrefixResolverTest.kt#L10)、[render main:10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix-render/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/render/AgentContextPrefixRenderer.kt#L10)、[skill test:12](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/skill-filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentcontext/skill/filesystem/FileSystemSkillsResolverTest.kt#L12)、[state test:41](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonTest/kotlin/io/github/stream29/kodex/agentstate/impl/AgentContextProjectionTest.kt#L41)、[migration test:16](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/MigrateToV047CurrentStoresTest.kt#L16)、[settings main:20](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/SplitSettingsFiles.kt#L20) |
| terminal-text | app-component-agent-impl-view；app-component-history-index-impl-view；app-component-history-impl-view；app-component-session-sidebar-impl-view | [agent mosaic:28](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeStatusBar.kt#L28)、[index mosaic:59](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/historyindex/HistoryIndexComponent.kt#L59)、[history mosaic:31](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/patch/PatchToolEventView.kt#L31)、[sidebar mosaic:68](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-sidebar/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt#L68) |

#### 折叠图与环

- 方法：只读枚举 203 个非 root 项目脚本（排除 build-logic/外部仓库），
  收集 879 处字面量 `project(":…")` 引用，所有目标可对应到这些项目。
  计入测试边，故比仅 production 图更保守；未执行 Gradle 或解析外部 artifact 的依赖图。
- 对 29 个 utility 节点执行 M/P 的身份投影：11 对合并、7 个单项目可保持旧 ID；
  下表为方便阅读统一省略 `-spec`/`-impl`，**不表示 P 已获重命名批准**。
  本库内部 spec 边删除，其他边保留；得到 18 节点、15 条不同的跨库边。

| 合并后的库 | 指向其他 utility 的边（含测试） |
| --- | --- |
| external-url | process-client |
| filesystem-lease | coroutines、kotlinx-io-coroutines、os-environment |
| images-codec | images、kotlinx-io-coroutines |
| kodex-home | os-environment |
| logging | kotlinx-io-coroutines（fileLoggingMain） |
| patch | kotlinx-io-coroutines |
| process-client | coroutines、kotlinx-io-coroutines |
| shell-client | coroutines、os-environment、process-client（processClientMain/JVM test）、kotlinx-io-coroutines（commonTest） |
| 其余十库 | 无项目出边 |

- utility 构建没有指向非 utility 项目的字面量 project 出边；源码 import 检查也未发现
  utility 向 app/agent/tool/rpc 等域的反向引用。
  因而这组候选不出现 utilities 内环，也不出现经外域返回 utilities 的环。
  例如 `shell -> process -> IO`，`codec -> images` 都没有反向边。
- 原边证据：
  [filesystem:10–15](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/build.gradle.kts#L10)、
  [codec:34–38](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/build.gradle.kts#L34)、
  [process:29–33](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/build.gradle.kts#L29)、
  [shell:35–58](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/build.gradle.kts#L35)。
  上表只报告这组明确映射；其他领域内部的合并环仍由主任务协调者汇总，不替其下结论。

### 机械改动、需决策项与待验证项

**机械改动候选（先取得具体迁移授权）**

- 以 M 映射创建根 build 脚本、合并插件配置、建立真正 specMain，
  原 package 与平台/测试目录保持；更新 settings 注册和全部消费者的项目 ID。
- 迁移契约依赖至 specMain，去掉 impl→本库 spec 项目边；
  保留 explicit API、warnings-as-errors、JDK 25 等原约束。
- 在职责不变的前提下修复 B1，重命名 B2 的 impl 文件；
  不扩大为 API/生命周期/异常/序列化协议重写。
- 复制 `.def` 时修复相对路径，并保留 cinterop 名、package、编译参数与 linkerOpts；
  保留插件输入，不复制旧构建产物。

**需用户/主任务决策，不能机械处理**

- 六对平台差异的保留方案；尤其 S 契约+H/C 实现是否暂不合并。
  不把当前消费者大多是 H/C 当作删除 S targets 的授权。
- mixed spec 库的契约/默认方法/纯 helper 边界：
  images、codec IO、raw IO、Ktor SSE、scoped logging、patch、terminal-text、
  coroutine helpers、rpc helper。保留有意义的函数契约可以是选择，
  但不得把 helper 依赖移到下游后仍称 specMain 自洽。
- P 的路径/发布坐标是否另行调整；“只有 spec”不是必须合并或一定保留空 specMain 的理由。
- 新 convention 的接入及精确诊断抑制作用域由主任务统一决定，
  不在 18 个库逐个复制一套规则。

**未来获准后验证（本轮全部未执行）**

- 源集图：commonMain 仅有预期向 specMain 的边，无反向边；
  原 blocking/posix/processClient/skikoNative/lucene/fileLogging 层都在；
  诊断抑制没有掩盖错误的 commonTest 边。
- 平台与 metadata：逐个保留原 S/H/C 的 target/环境能力；
  JVM、JS Node、四个桌面 Native 均验证，原 S 契约还必须核对 wasmJs/iosArm64/browser。
  不把“没有当前 consumer”当作平台兼容证明。
- 依赖/插件：specMain 的 API 和实现辅助依赖能自洽；
  serialization 生成符号、sealed hierarchy、public inline 的 ABI、
  Native cinterop commonization、linker、传递 engine/npm/native 依赖正常。
- 测试接线：
  [tests convention:1–7](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-tests.gradle.kts#L1)
  及 [common test deps:50–55](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/KodexHostKmp.kt#L50)
  继续覆盖原 testBalloon/kotlin-test；两侧测试合并后 main internal/friend 可见性、
  suite 发现与子进程 classpath 正常。
- 代表用例：mutex/SafeRw；lease 跨进程及 cleanup；
  IO JVM/JS/linuxX64 descriptor；codec common contract + JVM headless + Node sharp + Windows fallback；
  Ktor JVM SSE + Linux curl cleanup；logging fileLogging；
  patch parser/IO；process 与 shell owner/pipe/PTY/Unicode/取消测试；
  search 双后端；RPC 恢复和 Shell/Path wire format。
- 消费者验收至少覆盖 api 传播的契约项目、main+test 不同侧的 MCP/clean models、
  OpenAI client、两个图像 tool、tool search 与 frontend；另做 metadata/发布和 IDE 导航/索引验证。
  本报告不声称构建性能或 IDEA 内存已改善。

### 交付复核

- 文档仅修改本报告；未改产品、主任务、checklist 或其他审查报告；
  未移动 kanban 文件，未提交，未创建 Session。
- 没有运行 Gradle、编译、测试、IDE 或远程/设备操作。
  静态扫描用 Git、文本读取和只读 Python 内存计算；未创建临时文件或后台资源。
- 已复读并核对职责决定、静态事实、推断和未执行验证的区分。
- 结束复核：Kodex HEAD 仍为 `4388871137c56bd1b73d5b739cc98f036a790f3c`，
  `git -C Kodex status --short` 仍为空；外层原有修改保留。
- 文件 URI 已逐条检查存在性及行号范围；源/测试文件计数与项目清单已重新对照。
