# Task Tree

- `Review the previous specMain migration and representative external calls`()
- `Record the confirmed exported API boundary for every business module`()
- `Evaluate lint integration against the cleaned product baseline`()
- `Configure and benchmark the authorized detekt experiment`()
- **`Commit and push the configured lint and its validation record`()**
- `Plan dependency-safe migration batches and acceptance checks`()
- `Migrate exported declarations and validate consumers`()

# Details

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
- 旧迁移结项说明源集和布局的历史完成状态，不证明已经满足本轮更严格的
  导出 API 覆盖要求；不改写旧验收证据。

## 只读抽查发现

- `agent-state/context-window` 的 `tokensUntilCompaction` 在 commonMain，
  被 `tool/get-context-remaining` 和 `agent-runtime/decorator/compact` 调用。
- `agent-context/prefix` 的 `AgentContextPrefixResolver` 在 commonMain，
  被 `agent-state/core` 直接构造；spec 中的结果模型不替代服务契约。
- `agent-runtime/steer` 已有 `SteerProvider` spec，但 composition 模块仍直接
  构造 commonMain 中的 `StateFlowSteerProvider`，创建入口需分离。
- Composer 已有 factory spec，但 app/rpc 仍直接调用 commonMain 中的
  `createComposerViewModel`；需要审查实际调用入口，而不只检查接口是否存在。
- 以上是当前工作树的生产调用抽查，不是完整符号解析或全仓缺口清单。

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

## Lint 配置调研 — 2026-10-20 / Session998

### 基线与范围

- 用户清理后的产品子模块工作树干净，HEAD 为
  `fc36bce2b1d20f1ea1bd5b18db6053714c4740b6`；外层仍有既有文档改动，
  不视为用户授权清理或重写。
- 当前业务 convention 已启用 explicit API，但没有接入产品 Kotlin lint。
  四份现有 GitHub workflow 都是 fork package 工作流，不把本门禁塞进发布流程。
- Git 跟踪输入中有 668 份生产 Kotlin 文件，分布在 20 种 Main 目录，
  其中 specMain 为 250 份。这是静态目录盘点，不是已求值的 Gradle 源集模型。

### 调整后的推荐方案（待兼容性与性能验证）

- 已用 detekt 自定义规则承载 `checkSpecApi`，复用其规则加载、测试、报告和
  Gradle 任务设施，不自建 parser runner。实际接线集中在根 build.gradle.kts，
  读取真实 KMP main 模型，不给每个业务 convention 注入一套 detekt 插件。
- 根入口使用一个聚合 Detekt 任务，生产文件去重后批量分析；
  不在每个业务模块机械套用整套插件、为每个 source set/target 生成重复扫描。
- 结构规则先使用 light/no-type-resolution 模式，只启用自定义边界规则：
  `disableDefaultRuleSets=true`、`allRules=false`、`ignoreFailures=false`。
  不提供产品 analysis classpath；detekt 工具及自定义规则依赖仍需正常解析，
  不能误把它们清空。初始 `parallel=false` 仅关闭并行 AST 构建，不代表框架
  完全串行；对照实测后再决定是否开启。
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
- 独立任务与正反例已完成，测量和缓存检查见上述子任务；全仓报告为 575 个结构问题，
  不是完整 API 审计结论。按后续获批的迁移顺序
  接入 check/独立 CI job；不自动添加 suppress 或生成掩盖现状的 baseline。
  已配置产品 lint 并在本机运行 Gradle；未迁移业务 API 或接入 CI。
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
- 本机已有用户 IDE/Gradle/Kotlin 进程，本轮不运行全仓构建、不停止这些进程。
  使用已缓存的 Kotlin 2.4.20，在独立 Java 进程、`-Xmx256m` 下解析 6 组样例：
  显式/隐式 public、内部类 override、actual 函数、actual 类的额外公开成员、
  构造属性与 companion。全部完成 PSI 解析；未执行规则或语义匹配。
- 探针前两次启动分别误取 stdlib 的 metadata variant、缺少新版 extensions storage
  初始化；改用 JVM runtime variant 并按 2.4.20 API 初始化后成功。
  无仓库文件或临时文件写入，进程已退出，环境已 dispose；这些结果只证明解析入口，
  不证明实际 Gradle 接线、规则准确性、配置缓存或任何目标编译已经通过。
