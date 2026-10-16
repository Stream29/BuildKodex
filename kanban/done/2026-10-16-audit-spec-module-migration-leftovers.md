# Task Tree

- `Read the source-set design and previous migration manifests`()
- `Inventory current projects and retained spec source roots`()
- `Check production dependencies and project-consolidation counterexamples`()
- `Report merge candidates, misleading names and necessary boundaries`()
- `Record the confirmed uniform specMain and module naming rules`()

# Details

## 扫描后的用户确认 — 2026-10-16

- 统一使用 `specMain` 表达契约层；Gradle 业务模块名与模块目录不再使用
  spec 命名段，不再把 spec 当作独立项目类别。
- 默认在同一业务模块内组织 specMain/commonMain。确实需要独立契约来破环时，
  该模块叫 contract，内部依然使用 specMain；逐项保留破环证据。
- models、codec、transport、helper 使用实际业务名，不把所有旧 spec
  机械替换成 contract。“曾经独立”“多方消费”都不是自动保留 split 的理由。
- 纯实现/helper 统一保留源集层次，specMain 可以没有业务声明；
  撤回初轮关于可能取消空 specMain 的建议，不为填充该层制造接口。
- 已同步现行源集与模块边界 checklist。以下 40 项盘点仍是扫描时的实际结构，
  具体去向与合并范围仍为候选；规则确认不代表全仓搬迁或构建验收已经完成。
- 用户随后选择八项归并、深入跨平台同模块设计，并同时去除 impl 分类目录。
  最新候选映射和阻塞门禁由
  [后续实施任务](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-16-plan-uniform-spec-main-module-layout.md#L1)
  持有；下文五项/150 项仅是初轮扫描建议，不再作为最新实施范围。
- 后续迁移已切换源码路径；下文旧路径与行号保留扫描时含义。
  原工作树输入按旧路径保存在
  [冻结输入归档](file:///home/stream/ACodeSpace/demo/kodex-spec-main-878/baseline.tar)，
  对应 SHA-256 清单为
  [baseline inputs](file:///home/stream/ACodeSpace/demo/kodex-spec-main-878/baseline-inputs.json#L1)；
  当前源码与验收状态以上述实施任务为准。

## 范围与结论

- 用户要求扫描 spec 独立模块迁移到 specMain 后的遗留；本任务只完成静态审查。
  下列新路径、合并范围均是建议，不是已实施或已获批准的 manifest。
- 2026-10-16，Kodex HEAD 为 `03212868100c005850d4c5583fe9278065040cc6`；
  工作区另有 model-catalog、RPC server 的并行修改，本轮未改动。
- 扫描 Kodex 自有构建脚本、settings、conventions、跟踪源码和当前文档；
  排除 Mosaic、Ktor、KotlinMcpSdk、LuceneKmp、build-logic 自身作为业务项目的计数，
  以及 build/out/.gradle 中的历史产物。
- **源集迁移基本落地，项目合并与目录清理仍有后续空间。**
  不能把“已经有 specMain”理解为“原 spec 独立项目都已消除”，
  也不能把“路径还有 spec”理解为“没有迁移源集”。

| 当前静态盘点 | 数量 |
| --- | ---: |
| 非 root、有 build.gradle.kts 的业务项目 | 155 |
| settings 注册项目，含 root 与 12 个无脚本命名空间 | 168 |
| 显式应用 kodex.kmp-spec-impl 的项目 | 94 |
| 有跟踪 Kotlin 业务源码的 specMain owner | 85 |
| 路径含独立 spec 段的项目 | 40 |
| 上述 40 项中，specMain 没有业务 Kotlin 源码 | 9 |

- 40 项均已应用源集 convention；85 个有业务 specMain 的 owner 也均已接入。
  未发现“保留的 spec 项目忘记接入 specMain”的配置遗漏。
- 最明确的追加合并候选是 **images-codec、ktor-client-ext、patch 三对**。
  另有 **contract-ext → storage/core、available-skill → skill** 两项较小候选。
  五项联合生产图投影无环；若全部实施，业务项目数静态上可从 155 降至 150。
- 单纯移除目录/ID 中的 spec 段不会减少项目数。模型库、通用 helper、
  transport 应按业务命名，不能为了让 specMain 非空而虚造接口。

## 为什么上一轮“完成”仍会留下这些项目

- 上一轮明确选择了 40 个保留项目，不是本次发现了 40 个漏注册项目。
- [历史暂缓原因](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-12-discuss-kmp-spec-impl-source-sets.md#L627)：
  三对 utility 当时虽平台一致、合并图无环，但需要先分类算法、SSE、IO helper。
  当前已在各自项目中完成 specMain/commonMain 分类，该理由不再足以长期阻止合并。
- [现行约定](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/kmp-spec-impl-source-sets.md#L14)
  仍允许必要的独立契约；判断必要性应复核当前依赖角色，不能只引用历史目录。
- “共享模型”“有多个实现”“历史上叫 contract”是边界证据，不各自构成绝对禁止合并。
  无环也不是充分条件：还须检查职责归属、依赖闭包、平台及生成代码。

## 优先合并候选

| 候选 | 建议落点 | 当前证据与实施检查 |
| --- | --- | --- |
| utils/images-codec/{spec,impl} | utils/images-codec，异常契约在 specMain，codec/IO 在 commonMain 与平台层 | 两侧均 Host；旧 spec 只有 impl 直接消费。保留 skikoNative、Sharp、JVM headless 测试和平台依赖 |
| utils/ktor-client-ext/{spec,impl} | utils/ktor-client-ext，header 契约在 specMain，SSE/engine 在实现层 | 两侧均 Host；旧 spec 只有 impl 直接消费。保留 Linux curl 源码、平台 engine 与各测试 scope |
| utils/patch/{spec,impl} | utils/patch，PatchModels 在 specMain，parser/apply 在 commonMain | 两侧均 Host；多个消费者不构成环。合并 serialization 插件及全部测试，保留显式/default filesystem 重载 |
| agent-storage/spec/contract-ext | agent-storage/core/src/commonMain | 四个文件全是初始化、projection、compaction/retention 算法；没有自有契约；main 只依赖 core。core 对 ext 的反向引用仅在 commonTest，合并时删除该自依赖 |
| agent-context/spec/available-skill | agent-context/skill/src/specMain | 一个模型文件，仅 skill、prefix 两个项目消费；prefix 已依赖 skill，合并无需新增 prefix→skill 的项目方向。模型不应并入 prefix，否则可能拉出反向依赖 |

- 构建证据：
  [images spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/spec/build.gradle.kts#L8) /
  [impl](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/build.gradle.kts#L34)；
  [Ktor spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/spec/build.gradle.kts#L9) /
  [impl](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/build.gradle.kts#L8)；
  [patch spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/spec/build.gradle.kts#L10) /
  [impl](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/impl/build.gradle.kts#L8)。
- 扩展候选证据：
  [contract-ext](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract-ext/build.gradle.kts#L9) /
  [core 测试边](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/core/build.gradle.kts#L38)；
  [skill 上游模型](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/skill/build.gradle.kts#L9) /
  [prefix 已有 skill 依赖](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/prefix/build.gradle.kts#L22)。
- 可另复审 prompt-dsl 并入 skill、RPC contract 并入 client：
  两者分别投影无生产环，前者加入上述五项的联合投影也无环。
  但它们分别承担通用 DSL、多服务 RPC 协议职责，不能仅按文件少来认定应并入该实现。
  prompt-dsl 不应直接并入 prefix：skill→prompt-dsl、prefix→skill 会形成环。
- 五项候选的 Main/Test 分组内未发现同包同 basename 文件碰撞；
  这是文本候选检查，不代替 Kotlin 符号、JVM 签名与生成代码编译。

## 9 个 specMain 无业务源码的旧 spec 项目

| 当前项目 | 实际内容 | 建议 |
| --- | --- | --- |
| agent-state/spec/context-window | token budget 扩展算法 | 改名 agent-state/context-window；不能直接并入 state 实现，见后文环 |
| agent-storage/spec/contract-ext | storage 扩展算法 | 优先并入 core/commonMain |
| mcp/spec/stdio | 具体进程 transport | 改名 mcp/transport/stdio；是否合 composition 另查平台差异 |
| mcp/spec/streamable-http | 具体 HTTP/SSE transport | 改名 mcp/transport/streamable-http；不是独立的本地 spec port |
| openai/spec/json-codec | JSON 配置策略实现 | 改名 openai/json-codec；保留独立工具库，不强塞进 HTTP client |
| utils/coroutines/spec | childScope、runCatchingCancellable 等 | 改名 utils/coroutines |
| utils/kodex-home/spec | Home 路径求值 | 改名 utils/kodex-home |
| utils/kotlinx-io-serialization/spec | PathAsStringSerializer | 改名 utils/kotlinx-io-serialization |
| utils/logging/spec | scoped logger 包装 | 去掉 spec 名称；与 file logging 合并仍受 Shared/Host 差异约束 |

- 这些项目中的具体实现留在 commonMain 是正确分类，不建议移到 specMain。
- 它们当前的空 specMain 来自上一轮统一 opt-in，不是九份缺失的契约代码。
  用户随后确认统一使用该层次，因此保留空 specMain，不以此作为缺陷或移除理由。

## 完整 40 项复核表

- S/M 表示跟踪 Kotlin 文件数 specMain/commonMain，不含测试和其他平台源码。
- “保留”是本轮未找到适合机械合并的唯一 owner，不表示永远禁止重新划分。
  所列去旧名均是候选路径，尚未改动。

| 当前项目与构建入口 | S/M | 本轮处置判断 |
| --- | ---: | --- |
| [agent-context/spec/available-skill](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/available-skill/build.gradle.kts#L1) | 1/0 | 候选并入 skill/specMain |
| [agent-context/spec/contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/contract/build.gradle.kts#L1) | 1/0 | 共享设置契约；可按 settings 命名或重划 owner，不能仅因共享就沿用 contract |
| [agent-context/spec/prompt-dsl](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/prompt-dsl/build.gradle.kts#L1) | 1/1 | 独立 DSL 改名 prompt-dsl；并 skill 可选，并 prefix 成环 |
| [agent-runtime/spec/contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/contract/build.gradle.kts#L1) | 2/0 | 不直接并 composition：decorator/session 回路 |
| [agent-session/spec/contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/spec/contract/build.gradle.kts#L1) | 2/0 | 不直接并 filesystem/in-memory：runtime 回路 |
| [agent-state/spec/context-window](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/spec/context-window/build.gradle.kts#L1) | 0/1 | 实现 helper 去旧名；不直接并 state |
| [agent-state/spec/contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/spec/contract/build.gradle.kts#L1) | 3/1 | 不直接并 state：plan/context 工具回路 |
| [agent-storage/spec/clean-models](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/build.gradle.kts#L1) | 39/1 | 共享模型库去旧名；不与工具 handler 粗并 |
| [agent-storage/spec/contract-ext](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract-ext/build.gradle.kts#L1) | 0/4 | 候选并入 core/commonMain |
| [app/component/agent/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/spec/build.gradle.kts#L1) | 2/0 | 不直接并 Agent View：History View 回路 |
| [app/migration/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/spec/build.gradle.kts#L1) | 5/0 | Host 契约 / CLI 实现，先解决 JS 覆盖差异 |
| [app/spec/session](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/spec/session/build.gradle.kts#L1) | 4/0 | 不直接并 app session：RPC/settings 回路 |
| [hook/spec/notification](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/spec/notification/build.gradle.kts#L1) | 1/0 | 不并 executor；可另评估归 rpc models 的模型所有权 |
| [mcp/spec/contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/build.gradle.kts#L1) | 5/2 | 不单独并 composition：transport 回路，且 Host/CLI 不同 |
| [mcp/spec/stdio](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/build.gradle.kts#L1) | 0/1 | 具体 transport 去旧名 |
| [mcp/spec/streamable-http](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/streamable-http/build.gradle.kts#L1) | 0/1 | 具体 transport 去旧名 |
| [openai/spec/json-codec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/json-codec/build.gradle.kts#L1) | 0/1 | 共享 codec 去旧名 |
| [openai/spec/models](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/build.gradle.kts#L1) | 20/1 | 共享协议库改名 models；不能仅因无同名 impl 就认定无必要 |
| [rpc/spec/contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/build.gradle.kts#L1) | 11/0 | 可另评估并 client/specMain；静态无环，非一对一业务实现 |
| [rpc/spec/models](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/models/build.gradle.kts#L1) | 9/0 | 共享 wire/持久化模型改名 models；不直接并 client |
| [tool/spec/contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/contract/build.gradle.kts#L1) | 2/1 | 共享工具根契约保留业务边界，去 spec 分组名可另做 |
| [tool/spec/image-generation](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/image-generation/build.gradle.kts#L1) | 2/0 | 不直接并 handler：clean-models 回路 |
| [tool/spec/unified-exec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/unified-exec/build.gradle.kts#L1) | 3/0 | 不直接并 handler：clean-models 回路 |
| [tool/spec/view-image](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/view-image/build.gradle.kts#L1) | 2/0 | 不直接并 handler：clean-models 回路 |
| [utils/coroutines/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/coroutines/spec/build.gradle.kts#L1) | 0/2 | 独立 helper 去旧名 |
| [utils/external-url/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/external-url/spec/build.gradle.kts#L1) | 1/0 | Shared / CLI 不同，非直接合并项 |
| [utils/images-codec/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/spec/build.gradle.kts#L1) | 1/1 | 优先合并对应 impl |
| [utils/images/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/build.gradle.kts#L1) | 5/6 | 自足 images 库去旧名；不是 images-codec 的空 spec |
| [utils/kodex-home/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kodex-home/spec/build.gradle.kts#L1) | 0/1 | 独立 helper 去旧名 |
| [utils/kotlinx-io-coroutines/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/spec/build.gradle.kts#L1) | 2/1 | Shared / Host 不同，另有同包 internal 常量冲突 |
| [utils/kotlinx-io-serialization/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-serialization/spec/build.gradle.kts#L1) | 0/1 | 独立 serializer helper 去旧名 |
| [utils/ktor-client-ext/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/spec/build.gradle.kts#L1) | 1/3 | 优先合并对应 impl |
| [utils/logging/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/logging/spec/build.gradle.kts#L1) | 0/1 | 去旧名；与 impl 合并先处理 Shared / Host 与 fileLogging 层次 |
| [utils/os-environment/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/spec/build.gradle.kts#L1) | 1/1 | 已含 expect/actual 的自足库去旧名 |
| [utils/patch/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/spec/build.gradle.kts#L1) | 1/3 | 优先合并对应 impl |
| [utils/process-client/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/spec/build.gradle.kts#L1) | 1/0 | Shared / Host 不同；保留 process interop/平台实现覆盖 |
| [utils/rpc-exception/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/rpc-exception/spec/build.gradle.kts#L1) | 6/1 | 自足异常/恢复库去旧名 |
| [utils/search-index/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/spec/build.gradle.kts#L1) | 1/0 | Shared / Host 不同；不能把 iOS 等自动纳入 Lucene 实现层 |
| [utils/shell-client/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/spec/build.gradle.kts#L1) | 5/0 | Shared / Host 不同，另有 StdoutBuffer.kt 文件门面碰撞 |
| [utils/terminal-text/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/terminal-text/spec/build.gradle.kts#L1) | 1/1 | 自足文本算法库去旧名 |

## 不能被 specMain 自动消除的约束

- 生产图按 project 字面引用提取，区分 Main/Test，并纳入已知 kRPC patch
  convention 边；当前图无非平凡 SCC。以下是替换项目顶点后的反例，
  不是建议删除这些真实依赖。
- runtime contract 并 composition：
  composition → decorator-compact/tool → runtime contract；
  composition → session contract → runtime contract。
- session contract 并 filesystem/in-memory：
  session implementation → runtime composition → session contract。
- state contract 或 context-window 并 state：
  state → tool-get-context-remaining → context-window → state contract；
  state → tool-plan → state contract。
- 三个工具 DTO spec 各自并 handler：
  clean-models → tool DTO → tool contract → clean-models。
- hook notification model 并 executor：
  executor → rpc models → hook model。
- Agent spec 并 View、app Session spec 并持久化实现：
  分别经 History View、app RPC/settings VM 回到自身。
- MCP contract 单独并 composition：
  composition → stdio/streamable-http → contract。
  整体吞并 transport 可改变该局部回路，但仍需解决 Host/CLI 差异。
- Shared convention 保留 browser/Node JS、Wasm、iOS；
  [Host](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-host.gradle.kts#L12)
  是 JVM、Node JS、四桌面 Native；
  [CLI 目标](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/KodexHostKmp.kt#L22)
  没有 JS。不能通过只改插件或目录来无损合并不同覆盖的项目。
- 测试反向依赖不是生产环：例如 core.commonTest→contract-ext.main→core.main。
  若合并，应合入测试并删除自依赖，而不是据此错误保留一个生产项目。

## 后续清理边界与验收

- 优先顺序建议：三对同平台 utility；core/ext 和 skill/models；
  再统一无必要的 spec/impl 目录命名；最后单独讨论跨平台及组合根重划。
- 目录/项目 ID 迁移须同时修改 settings、所有消费者、脚本和现行文档，
  不保留 alias/转发项目。当前文档中的
  [mcp-spec-stdio 示例](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/docs/gradle-development.md#L51)
  现在仍有效，若改名须同步，不应提前当成坏链接。
- 统一源集与模块命名规则已确认并更新现行 checklist；逐项实施范围仍需定稿。
  历史 done 任务保持历史语境，不重写成当时已经清理了所有项目。
- 实施验收至少包括：最终生产图、原 targets/测试输入保留、metadata、
  serializer/kRPC 生成、受影响消费者编译与测试；
  images-codec/Ktor 的平台层次及 IO/Shell 的符号冲突要单列。
- 本轮只运行只读 Git、文本和 stdin Python 静态检查，新增本文作为审查交付；
  没有运行 Gradle/IDE/远程验证，没有创建产品临时文件、进程或设备资源。
  静态无环不代表构建、运行时、发布兼容或性能已验收。
- 初轮已校验 40 项表与实际项目集合完全一致、逐项 S/M 计数相符、
  当时全文 55 个文件 URI/行号有效，新文档空白检查通过。
- 扫描任务完成；后续实现需要另定 scope，不借旧任务 done 状态自动推进。
