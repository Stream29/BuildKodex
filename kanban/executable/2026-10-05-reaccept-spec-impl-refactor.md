# Task Tree

- `Freeze new migration and pin the review baseline`()
- `coroutineScope {`
  - `async { Review platform and process utilities() }`
  - `async { Review remaining utilities() }`
  - `async { Review Tool and MCP domains() }`
  - `async { Review Runtime and OpenAI domains() }`
  - `async { Review Context, Storage, Session, Hook and RPC roots() }`
  - `async { Review settings and account components() }`
  - `async { Review Agent and navigation components() }`
  - `Review whole-repository dependencies and migration history`()
  - `awaitAll(reviews)`
- `}`
- `Recheck reported blockers against actual production call chains`()
- `Publish coverage, findings and ordered repair gates`()
- `Record authorized repair scope and isolate work ownership`()
- `Repair confirmed defects with centralized builds`()
- `Record acceptance and scoped repair commits`()
- **`Track remaining policy and runtime gates independently`()**

# Details

## Authorization and baseline

- 用户要求对之前的 spec/impl 重构重新验收，并要求一组多个 Session 并行。
- 暂停 app/shared 新迁移；第一轮只读审查已完成，用户随后授权本任务中的有界修复。
  用户随后接受当前修复并要求分批提交；本轮提交，不推送。
- 内层基线：`refactor/spec`，`821b66275aae2f0a9541fa2e9e03023d875c30b5`。
- 内层工作树在启动时干净；外层其他用户文件、子模块与共享 IDE 不动。
- 覆盖从 `f1a90006`、`5d43da38` 到上述基线的全部迁移批次，包括已归档任务；
  不将未迁移根模块本身算作本次重构失败。
- 独立 Session 必须经用户确认后启动；接受创建不等于审查已经完成。
- 审查报告及下方原始证据引用固定基线，修复后部分文件会移动或删除；
  不把历史 URI 当作当前源码位置。新结果记录在修复任务及集中验证 checkpoint。

## Common acceptance rules

- 每个已迁移领域追踪：原声明 → 当前 spec → 真正实现/装配 → 生产调用 → 测试。
- 记录实际代码位置、FQCN、实现关系或模型使用、生产入口和历史提交。
- 实现必须使用真正的契约；仅添加 Gradle 依赖、不接入的占位接口不算迁移。
- DTO、协议模型、纯函数和测试依赖可以构成真实 spec，不强迫每个项目都有
  `implements`；判定未使用需核对序列化、工厂、生成代码和真实装配。
- 检查新旧平行模型、旧生产声明、compatibility bridge、无语义 wrapper 和重复状态。
- 保留真实 RPC/storage 边界、derived presentation 和 renderer-local state；
  不能仅凭 `Adapter`、`Handle` 等名称认定代码无用。
- 对照用户确认的 KDoc、异常、所有权、取消、状态与交互、渲染及持久化/wire 保证。
- 原有正确实现的测试通过不能证明新 spec 已被接管；检查测试是否验证实际调用链，
  是否绕过 UI callback、是否只覆盖孤立假实现。
- 明确区分已知混合依赖、未经授权的 API/行为变化和本次新增旁路；记录既有例外，
  不把历史文档中的“通过”直接当作结论。
- 第一轮未启动 Gradle、IDE sync、真实浏览器/网络/进程测试，不声称新运行时验证。
  修复阶段由协调者集中构建；并行线仍不运行构建或操作共享 IDE/设备。

## Finding levels and reporting

- `B1 confirmed blocker`：真实契约未接入、平行权威、确认的行为/wire/所有权错误。
- `B2 suspected blocker`：有具体风险路径，但证据尚不足；列出需要的复现实验。
- `D documented debt`：有效实现仍有边界债务，明确批准/记录来源；不是无条件通过。
- `R reviewed path`：已追踪关键路径，无上述阻塞发现；静态审查不等于全平台验收。
- `U unreviewed`：尚未完成或无法判断；不得默认计入通过。
- 每份报告列出完整范围表、每项证据、测试局限、未审查部分和建议最小消融。
- Shell 已知阻塞只是起点；不预设其他领域有错，也不预设它们都正确。
- 审查者只写所属子任务；协调者整合总报告并独立复核关键发现。

## Parallel review ownership

- [Platform/process utilities](../done/2026-10-05-audit-platform-process-utils.md)
- [Other utilities](../done/2026-10-05-audit-other-utils.md)
- [Tool/MCP](../done/2026-10-05-audit-tool-mcp.md)
- [Runtime/OpenAI](../done/2026-10-05-audit-runtime-openai.md)
- [Backend roots](../done/2026-10-05-audit-backend-roots.md)
- [Settings/account components](../done/2026-10-05-audit-settings-account-components.md)
- [Agent/navigation components](../done/2026-10-05-audit-agent-navigation-components.md)

## Current checkpoint

- 七条只读 Session `496–502` 均已交接 `REVIEW READY` 报告。
- 协调者已复核下列关键发现、生产调用链与引入历史，并区分重构问题和原有缺陷。
- **重验不通过，不能继续把此前全部批次称为迁移完成。** 新迁移保持暂停。
- 审查阶段仅修改任务文档，未修代码或运行新验证；七份报告保持为该基线证据。
- 用户已授权“去修”，开始修复；新领域迁移仍暂停。修复不包含提交或推送。

## Coordinator inventory and independently checked evidence

- 基于固定基线的 Git tracked 文件清点，共 186 个范围内 Gradle 项目；
  不包括 buildSrc、vendor、生成目录或尚未迁移的根模块。

| Root | Projects |
| --- | ---: |
| utils | 33 |
| tool | 25 |
| mcp | 6 |
| agent-runtime | 10 |
| openai | 12 |
| agent-context | 11 |
| agent-storage | 6 |
| agent-session | 4 |
| hook | 2 |
| rpc | 6 |
| app/component | 71 |

- 直接主源码 spec → impl 依赖有四条，均指向 `utils-shell-client-impl`：
  [context contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/contract/build.gradle.kts#L10)、
  [context prefix](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/prefix/build.gradle.kts#L10)、
  [session contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/spec/contract/build.gradle.kts#L15)、
  [unified exec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/unified-exec/build.gradle.kts#L9)。
  这是边界债务清单，不据此认定四个领域都有独立新增缺陷。
- `agent-storage` 的另外三条 spec → impl 依赖位于 `commonTest`，不计为主源码
  依赖反转；该名称扫描也不能覆盖所有未带 `impl` 后缀的混合旧项目。
- 已独立确认一个 Shell 之外的同类未接管问题：`6606efe9` 新增
  `McpStdioTransportFactory` 路径，但当前生产
  [McpServiceImpl](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt#L213)
  仍调用具体 `ProcessClient` 的旧扩展。新
  [factory](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/stdio/src/commonMain/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransport.kt#L49)
  使用 `ProcessClientSpec`，并复制了原有启动/transport 装配算法；全 tracked Kotlin
  引用和引入提交均已核对，当前没有生产 factory 调用。
- 该发现属于 `B1` 契约接管阻塞，不代表现有 MCP 进程启动行为已经坏掉；
  修复须消除双路径、接管真实进程契约，而非新增第三个 adapter。
  `ProcessCommand`、`ProcessSession`、`ProcessException` 确实已由实际实现使用，
  不能因此把 process-client 的整个 spec 判为未使用。
  Process launching spec 未实现与 MCP 双入口分别需要修正，MCP stdio 的影响只计一次。

## Consolidated reacceptance verdict

| Review line | Evidence-backed result | Limits |
| --- | --- | --- |
| Platform/process utils | Shell、Process launching、OS Environment、图片 codec 存在新增契约未接管或重复接口；Kodex Home provider 实际被调用，不判为空契约 | 平台取得/清理/取消的疑点未运行验证 |
| Other utils | URL opener 违反现有全局 API 决策；Patch 新增纯转发层；其余值模型、纯函数、实际 IO/search/serialization 契约已追踪 | SSE 测试依赖迁移疑点及 lease/owner/Node 权限需实验 |
| Tool/MCP | Tool schemas 真正用于注册和解析；MCP core 契约真实实现，两个新 transport factory 未接管；另发现原有工具行为与协议承诺不符 | 不宣称 live MCP 或工具集成已重验通过 |
| Runtime/OpenAI | 生产核心/装饰器和 OpenAI 协议/客户端契约已实际接入；mock builder 的独立性承诺不成立 | 清理失败、retry 决策冲突和外部服务未验证 |
| Backend roots | 真实契约硬迁移，无平行替代模型；原有文件 create/cleanup 与 Home 规则存在缺陷或冲突 | Native/KRPC、文件中断恢复与双重失败场景未运行 |
| Settings/account components | 所有组件都有真实生产链；MCP effects 的宿主生命周期不匹配；Hook reporting KDoc 和 Login 文案不准确 | burst input、真实根弹窗转换及 OAuth 平台行为未运行 |
| Agent/navigation components | 原 AgentHistory/NewSession 身份已直接接管；Composer 两处生产接线回归，仍有少量 alias/转发壳 | root registry/queued index、loader 释放和压力 timeout 风险未闭合 |

- 后端复核覆盖三次提交的 **220 个 Kotlin R100 迁移**：
  旧路径全部不存在，当前仅两个 RPC wire 测试随后增补；移动的主源码仍与原始
  blob 相同。可确认这些领域不是 Shell 式“另建接口保留旧真源”，
  但保留原始实现也同时保留了原始缺陷。
- 186 是项目清点数量，**不是 186 个项目通过验收**；`R` 表示静态生产链追踪，
  各报告的 `U` 仍含运行时、平台、并发及失败覆盖。
- 不合并各线的 B1 数量作总数：MCP stdio/Process 存在跨线影响，同一组件还有
  行为回归、结构残留、文档冲突等不同严重度。

## Migration defects and minimal repair gates

### Gate A — restore real production behavior

| Finding | Independently checked evidence | Minimum repair/acceptance requirement |
| --- | --- | --- |
| Composer 错误和运行提示被宿主隐藏 | [AgentRuntimeScreen:184](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L184) 设置 `showAuxiliary=false`，同时关闭状态/错误展示；[RpcComposer:79](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcComposer.kt#L79) reporter 空实现。源自 `821b6627` | 真正宿主 Enter → 真实 Composer → fake failing binding 的测试；错误有可见出口，恢复 steer 提示，不重复 preview |
| Composer resume 绕过原异常处理 | [RpcComposer:67–73](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcComposer.kt#L67) 在 binding owner 下直接 launch；[RpcSessionViews:147](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionViews.kt#L147) 是普通 Job。原 `operate` 捕获普通失败，新路径可使整个 binding 失效 | 恢复原普通失败通知边界，不能恢复第二套 submit 算法；验证普通 resume 失败后 Session 仍 Ready，取消/SessionNotActive 保持区别 |
| MCP effects 生命周期短于 Settings owner | [SettingsPopup:43](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt#L43) 挂载唯一 consumer；Settings → Login 后 Settings VM 在 `returnTo` 保留，但该 composition 移除；[McpSettingsEffects:123](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/McpSettingsComponent.kt#L123) 取消正在处理的 MCP attempt | 将唯一 effect handler 与真实 Settings owner 对齐；用 gated preparation/opener 加真实 root popup 转换验证，不新增全局 URL 服务或重复 consumer |

- Composer 的实际 UI Stop 仍走旧的 caught command 边界；不能把潜在 cancel port
  问题说成已确认的 Stop 按钮回归。
- 以上为代码可确定的接线/生命周期差异；实际用户发生率未测，不使用“已运行复现”措辞。

### Gate B — delete fabricated boundaries and finish real cutovers

| Finding | Source/report evidence | Ablation / hard-cutover requirement |
| --- | --- | --- |
| Shell 三种独立 `*Spec` 无实际实现或消费者 | [Shell spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ShellClientSpec.kt#L8)；平台报告 PU-01；`6606efe9` | 删除替代 command/session/client，迁移真实 Shell、command、session/output 契约；默认解析作为明确平台装配，不造 resolver/DTO bridge |
| ProcessClientSpec 未由任何 actual client 实现 | [Process launching spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClientSpec.kt#L38)；平台报告 PU-02 | 保留实际 command/session/exception，接管唯一真实 launching API 和工厂/消费者，补回被删减的 stream/close/环境语义 |
| OS Environment 新接口被实际查询旁路 | [OS interface + delegating object](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironment.kt#L20)；平台报告 PU-03 | 选一个真实调用边界；不同时暴露旧查询和从未使用的新查询 authority |
| PromptImageCodec 是不用的第二个 transformer | [duplicate codec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/codec/ImageCodecSpec.kt#L11)；平台报告 PU-04 | 删除它，保留实际 `PromptImageTransformer`、模型、异常和 IO，规范文档附于真实操作 |
| MCP stdio/HTTP factory 未接管 | Tool/MCP 报告 M1，前文已有独立链路证据 | 删除重复 stdio 算法，只有一条真实装配路径；先确定有必要的契约，不为维持已造接口再造 adapter |
| ExternalUrlOpener 多余且违反已确认 API 规则 | [URL checklist:5](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/external-url.md#L5)；其他 utils 报告 URL | 删除多余 interface/singleton forwarding，保留全局函数和实际 result DTO |
| PatchApplier 只是旧扩展到新 class 的纯转发 | [PatchFileSystem:7–26](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/PatchFileSystem.kt#L7)；其他 utils 报告 PATCH | 消融无语义 layer，保留一个实际操作；函数 API 本身不违规，不能仅为证明 interface 使用而强迫 Tool 新增 DI/工厂 |
| Delete alias / 测试用 Catalog wrapper 残留 | [delete alias:65](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/contract/application/src/commonMain/kotlin/io/github/stream29/kodex/app/application/contract/ApplicationPopupState.kt#L65)、[Catalog wrapper:11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionCatalog.kt#L11)；Agent/nav F3/F4 | 直接用实际 spec/impl，删两个无行为 spelling；保留真正承担 RPC 读写的 `RpcSessionCatalogDependencies` |

- 本表的结构问题不是八个运行时故障。尤其 URL、Patch 的实际算法被调用；
  问题是为了配 spec/impl 而添加了无必要层，而不是算法已经失效。
- 不能机械采用子报告的所有“让消费者注入新接口”建议；先消融，只有真实职责
  需要的边界才保留。目标是唯一事实来源，不是让每个 interface 获得引用。
- Kodex Home provider 真实被 getter 调用，不计作 disconnected spec；
  Session Settings effect/getter/旧 overload 有历史明确保留记录，单列消融候选，
  不是新发现的第二套生产状态。

### Gate C — correct contracts and test-double independence

- **Mock builder 独立性不成立**：`build()` 复制的 full-handler closure 仍读 builder 的
  mutable simple handler；修改 builder 可改变已构建 client。实际响应总走这个 full
  handler，constructor 的 simple-handler 字段已无行为用途。
  证据：[MockOpenAiClient:59](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client-test/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/test/MockOpenAiClient.kt#L59)、
  [build:149](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client-test/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/test/MockOpenAiClient.kt#L149)。
  修复门槛是 snapshot 后构建 A，修改 builder，A 不变、新 B 使用新 handler；
  不引入第三个 mock adapter。这是测试支持问题，不是 HTTP 客户端未接管。
- **Hook 契约与 adapter 不符**：新 KDoc 承诺 eventual conflict reporting，
  实际旧规则是 stale/same-name conflict 返回 unchanged settings，queue 随后清 failure。
  证据：[Hook spec:8](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/hooksettings/HookSettingsViewModel.kt#L8)、
  [adapter:27](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcHookSettingsDependencies.kt#L27)。
  重构新增的是不实保证，不能以修改持久化策略来偷偷满足它；先确定保留行为还是
  改冲突处理，再用实际 adapter 验证。

## Inherited defects and policy discrepancies — separate backlog

- **Session 创建中途失败留下保留目录**：真实 file create 在 `ofEmpty` 外没有清理；
  RPC 拿不到尚未返回的 index，因此上层 cleanup 不覆盖它。
  [repository:142](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt#L142)。
- **文件写入/fork cleanup 可掩盖原始失败或取消**：finally/catch 中直接执行可能抛错的
  delete；与 suppressed-failure 规则不符。
  [storage:242](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemIndexVersioned.kt#L242)、
  [fork:168](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt#L168)。
  Runtime pending-tool cancellation cleanup 的同类错误路径待注入失败验证。
- **BackendReady 提前创建/扫描 `sessions/`**：实际 host eager factory 与 Home 按需规则
  不符；这是原有装配策略问题，需确认规范是否仍有效，再决定单一 owner 的 lazy admission。
- **工具承诺未实现**：`num_last_images_to_include` 路径无条件失败；
  `autoResolutionMs` 只保存/展示，无计时完成消费者。两个字段/其余 DTO 真被使用，
  不能说整个 spec 不存在；修复或改变公开能力都需独立行为/wire 评审。
- **MCP 名称投影冲突**：合法的 `a-b` / `a_b` server/tool 名被相同归一化路由替换，
  真实 ToolRouting 随后拒绝重复。应补纯 catalog/routing fixture，不需 live server。
- **MCP 固定版本与实际 SDK 接受旧版本冲突**、**remote compaction retry 规范 2 与
  当前默认 20 冲突**：差异已确认，行为取舍待确认；不自动改协议 gate/retry。
- **Login 成功文案不准确**：固定 target 可为 Codex，完成并不切换 source，
  却显示“正在使用私有凭据”。原 renderer 已有该文案；最小修改是中性完成提示，
  不是新增认证 source authority。
- 本节均有前移历史证据；不是目录迁移引入的回归，也不因原测试通过就关闭。
  优先级：保留目录/原始失败 > 明确失效的工具能力/路由 > 政策冲突/文案。

## Unclosed experiments and repair order

- **先验证迁移覆盖缺口**：`ktor-client-ext` JVM SSE tests 已移到 impl，但 server test
  dependencies 仍在 spec 的 `jvmTest`；静态依赖不传递该测试配置。
  必须查实际 test classpath/编译这一目标后才声称 unresolved symbols；本轮未运行。
- **组件行为实验**：真实宿主 Composer Enter + injected failures，Settings → Login
  retained-owner effects，Hook/MCP 两字段 callback 在 recomposition 前触发。
- **资源实验**：进程取得交接取消、Node cleanup ownership、lease heartbeat loss、
  wrong-owner mutex unlock、Node private-file 权限、storage residue/noncanonical 文件。
- **窗口/根所有权实验**：Tab identity map 对已关闭 handle 的强引用保留；
  History evicted lazy loading jobs；root queued select/materialize by index、
  registry release/open 竞态。后两者为未迁移根的既有风险，不冒充新组件回归。
- 已知 History 1000-item timeout 继续保持未闭合，不能把 loader 风险当作已知根因，
  也不能用几次重跑通过代替解释。

建议后续按三条有界修复线并行，代码所有权分开、最终集中验收：

1. **真实契约硬迁移**：平台 utils + Process/Shell + MCP transport 的依赖闭包，
   附带图片/URL/Patch 消融；此线允许内部分批，不要求新增空 spec。
2. **已迁移组件接线修复**：Composer、MCP effects、Hook 契约及 alias/转发壳，
   加实际宿主 UI/异常测试；不重写 Application registry。
3. **测试/验证可靠性**：mock snapshot、SSE classpath 和定向生命周期复现实验；
   只为报告确认实际行为，不把 B2 直接当作修复授权。

- 用户现已授权修复上述确认问题，包含明确的 Session create/cleanup 语义；
  未复现的 B2 先做定向实验。工具能力/wire、协议严格版本和 retry 政策取舍仍需
  独立评审，不把猜测当作修改授权。app/shared 新迁移继续冻结。
- 修复后验收必须同时满足：旧声明/旁路删除、真实装配使用唯一契约、原语义/序列化保留、
  真实宿主回调与失败测试、被影响目标的构建及平台未验证项准确记录。

## Authorized repair ownership

- [协调者](../done/2026-10-05-repair-verification-and-ablation.md)负责小型 utils 消融、
  mock/SSE 验证可靠性、任务/清单、统一构建和最终复核。
- [平台线](../done/2026-10-05-repair-platform-contracts.md)负责真实 Shell/Process/OS 契约
  以及 MCP transport 唯一路径。
- [Composer 线](../done/2026-10-05-repair-composer-host.md)负责真实 Agent 宿主状态/异常接线。
- [Settings 线](../done/2026-10-05-repair-settings-lifetimes.md)负责 MCP effects 所有权、
  Hook 契约、Login 文案及残余 compatibility。
- [Storage 线](../done/2026-10-05-repair-storage-cleanup.md)负责 reserved-directory
  和 primary/suppressed cleanup；不修改冻结 migration。
- 独立 Session 创建须经用户确认；并行线不运行 Gradle、不提交、不推送，
  不操作 IDE/其他共享资源，只改分配路径并在独立修复任务交接。

## Central repair validation — current checkpoint

- 四条修复线 `503–506` 已交接并集中验证；用户接受当前修复范围。
- 内层分批提交：`eff7b10e` 平台/MCP、`a1adf2f2` 组件宿主、
  `88d26812` 存储清理、`e89c5094` 消融/验证；未推送。
- 七份审查报告和五份完成的修复任务归档 done，基线审查证据保留；
  本主任务继续承载未闭合问题，不把局部修复验收等同全部问题解决。
- 真实 Shell/Process 原 FQCN 现在是 spec 契约，实际平台 owner 实现它们，
  scope 工厂返回该契约，生产消费者不再经过未使用的 `*Spec` 替代类型。
  Shell 纯模型/serializer/settings 入 spec，host discovery 留在 impl。
- OS 保留原 functional API 和真实 actuals；MCP 删除未使用 factory 和重复 startup，
  合并为被生产装配直接使用的 stdio/HTTP 函数式契约，没有新增 resolver/adapter。
- 删除无用途的图片 codec、URL opener、Patch applier interface/impl 层；
  保留真实 transformer、URL result、Patch 模型/解析器和原文件系统操作。
- Composer 使用唯一 submit 算法和原 Agent caught-command/failure 边界；
  真实 Enter 测试覆盖失败、取消、恢复及 pending preview，不再只测孤立假 VM。
- Settings 根保留 exact-owner MCP consumer，Login 转换不取消；整棵 renderer
  卸载仍保持原 captured-effect cancellation。删除 alias/转发构造器，修正 Hook
  admission 文档与 Login 完成文案，清除 departed tab identity 引用。
- Hook/MCP 的真实同帧两字段输入均复现名称被清空。唯一 `updateDraft` 改为
  对最新 exact-token draft 执行同步字段更新；删除旧 whole-draft API，不保存 View 副本。
- Storage 清理只作用于 owned reserved/temp 目标，NonCancellable 清理保留原始
  异常/取消并附加 suppressed cleanup；成功操作的 cleanup failure 仍可见。
- Mock handler build snapshot、SSE test classpath 已修复；wrong-owner Mutex 和
  四类 History dormant-loader release 均真实复现后最小修正。

### Executed checks

- 使用现有 Daemon JVM、`--max-workers=1`，主矩阵 37 个 JVM test 目标通过；
  追加六个消费者 test 目标也通过。43 个目标中两个无测试源码，
  其余项目 XML 合计 **869 项、0 失败/错误/跳过**。
  汇总包含本轮先前通过的 up-to-date 结果，不声称 869 项最后一次全部重新执行。
- 追加消费者：通知 Hook **10**、Runtime Tool **12**、Context Prefix **3**、
  Path Picker VM **10**、filesystem lease **7** 项通过；Kodex Home impl 无 test source。
  CLI JVM 主源码编译通过，未运行交互式 CLI smoke。
- Storage 新故障矩阵实际执行 **20 + 66 = 86** 项，全通过；原测试亦通过。
- 真实 Composer host **6** 项、Settings/Login root **5** 项全通过；Hook/MCP
  genuine-renderer burst probes、invalid/stale/closed draft update、RPC queue 同时通过。
- History **37** 项 ViewModel、**49** 项 renderer 回归通过；本轮新增四项释放用例
  先红后绿。不将这些结果解释为已解决原 1000-item stress timeout 根因。
- Integration 测试源码 JVM 编译通过，未运行依赖外部服务的 Integration suite。
- Shell/Process/OS 的 JS、Linux x64 主源码编译（六个目标及其 spec 依赖）通过；
  JVM 真实 pipe/PTY 和 MCP 本地 I/O fixture 通过，不声称 JS/Native runtime 测试通过。
- 本轮未做 macOS/Windows/Linux arm64 运行、CLI smoke、live OAuth/browser 或 kernel
  故障测试。库编译不等于 Native binary/link/进程测试。

### Not closed by this repair

- 用户要求提交当前修复后继续讨论下一批；可恢复此前选定的完整 app/shared
  范围梳理，尚未授权其新实现。真实 Shell 修正已完成，不再以平行接口接入作为前置项。
- 工具失效字段/自动完成能力、MCP normalized-name collision、严格协议版本与
  remote-compaction retry 政策、eager repository admission 保持独立评审，不偷偷改 wire。
- 进程取得交接取消、Node 权限/cleanup、lease-loss、root queued-index/registry 竞态
  等实验尚未执行。原 History stress timeout 也仍是未闭合风险。
- 本 checkpoint 只关闭上述有证据和测试的修复项，不将静态审查的所有 R/D/B2
  或全部平台默认升级为通过。历史报告保留固定基线证据。
- 修复文档当前链接/行号和文本检查通过；旧三个项目根及其生成目录已清理。
  临时复现/构建日志清理，Daemon 留作复用且空闲；其他用户文件与 IDE 未操作。
