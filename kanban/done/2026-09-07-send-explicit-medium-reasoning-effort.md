# Task Tree

- [done] 修正选择 Medium 时省略请求 effort
  - [done] 确认该省略行为非用户预期
  - [done] 确认共享 DTO 及 Responses/Search 统一明确发送 effort
  - [done] 审查最小修改范围、验证缺口及实施阻塞点
  - [done] 修正既有序列化
    - [done] 将 Reasoning.effort、ResponsesApiRequest.reasoning、SearchRequest.reasoning 改为 EncodeDefault.ALWAYS
    - [done] 更新对应 KDoc，保留 summary/context 默认省略及其他字段行为
  - [done] 补齐离线回归测试
    - [done] 验证默认构造与显式 Medium 的完整请求 JSON
    - [done] 验证所有现有 effort 档位及 Custom 原样编码
    - [done] 验证 summary/context 默认省略和非默认值保留
    - [done] 验证旧设置解码、新设置编码与往返语义
    - [done] 验证普通、压缩、标题及 Search 调用方的请求投影
  - [done] 验证真实客户端请求体
    - [done] 使用本地 HTTP fixture 捕获普通、压缩及 Search 的 Medium/Low 请求 JSON
    - [done] 确认压缩仍走 responses 路径并保留专用 header
  - [done] 完成授权后的验收
    - [done] 运行定向离线测试及本机 native 检查，分别记录结果
    - [done] 核对最终差异并保留并行修改

# Details

## 执行结果（已完成）

- 三处 ALWAYS 注解及 KDoc 已修改，保留其余字段语义；默认 Search JSON
  与旧的“reasoning 整体省略”断言已同步。
- 新增模型序列化矩阵、标题/WebRun 调用方测试；AgentState 同时覆盖
  Medium/Low 的普通、Retryable 重试、compaction 及压缩后请求。
- `ExplicitReasoningHttpTest` 使用 loopback HTTP server、假认证和真实客户端，
  验证六次请求的 JSON、responses 路径及 compaction header；客户端和服务器均关闭。
- JVM models、定向 client、AgentState、session-title、定向 WebRun 测试通过；
  models 与 AgentState linuxX64Test 通过，CLI Linux release 链接通过。
- 已更新 `checklist/openai-model-alignment.md`。未调用真实模型、未提交或发布。
- 以下保留推进前的决策和验证计划；原“暂不执行”限制已由后续授权解除。

- 执行授权更新：用户随后明确要求由当前会话执行全部四项任务；下述旧的
  “暂不执行”记录仅为历史，当前按统筹任务顺序实施，不创建提交。

- 2026-09-07 用户确认共享调用方都明确发送 effort，具体计划已完成；随后明确授权仅移至 executable，不执行，由用户未来安排执行人。未授权本次修改 Kodex 实现、运行实施验证或创建 Git commit。
- 目标：默认构造与显式选择 Medium 都发送 `"reasoning":{"effort":"medium"}`，不依赖后端默认值；不声称已证明实际降档。
- 保留 Medium 默认值；summary/context 的 Auto 继续省略。无需新增“使用模型默认”UI、档位、持久化状态、自定义 serializer 或全局编码配置。
- 用户接受标题请求、Search 和设置编码的共享影响；旧设置缺失 effort 仍解码为 Medium，不迁移或批量重写历史文件。
- 不修改压缩保留算法、上下文换窗、缓存键或 MCP 文本任务；保留工作区其他并行修改。

## 最小修改范围

- 生产代码仅计划修改 [ResponsesApiModels.kt:38、383–398](../../Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ResponsesApiModels.kt#L38) 和 [WebSearchModels.kt:16–33](../../Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/WebSearchModels.kt#L16) 的三处注解及相关 KDoc。
- [OpenAiJsonCodec.kt:5–8](../../Kodex/openai/json-codec/src/commonMain/kotlin/io/github/stream29/kodex/openai/jsoncodec/OpenAiJsonCodec.kt#L5) 已开启 encodeDefaults；无需修改。ALWAYS 将字段要求固定在 DTO 上，避免依赖调用方全局配置。
- 普通与压缩分别在 [KodexAgentStateImpl.kt:193、281](../../Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L193) 使用同一设置投影；[OpenAiClient.kt:216、242](../../Kodex/openai/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L216) 都 setBody(request)，压缩使用 responses 路径及专用 header。
- [标题生成:99](../../Kodex/app/shared/session-title/src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/OpenAiSessionTitleGenerator.kt#L99) 显式构造 Reasoning；[WebRunToolClient.kt:23](../../Kodex/tool/web-run/src/commonMain/kotlin/io/github/stream29/kodex/tool/webrun/WebRunToolClient.kt#L23) 使用 SearchRequest 默认 Reasoning。本任务不让 Search 继承 Session effort，只将既有 Medium 默认值明确发送。
- 设置复用 [KodexAgentSettings.reasoning:75](../../Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/CompactionModels.kt#L75)；后续序列化可增加 effort 字段，但对象状态与解码默认值不变。

## 验证分解

- `openai/models` commonTest：新增 Reasoning/request 序列化矩阵，以实际 OpenAiJsonCodec 编码完整 ResponsesApiRequest、SearchRequest，断言嵌套 JSON 字段；补充 encodeDefaults=false 的编码器验证三处 ALWAYS。
- 矩阵：省略构造参数、显式 Medium；None/Minimal/Low/High/XHigh/Max/Custom；summary 的 Auto/Concise/Detailed；context 的 Auto/CurrentTurn/AllTurns。至少覆盖 Medium 搭配非默认 summary/context，防止 effort 再次丢失。
- 保持 instructions、service_tier、text、client_metadata 等原有默认省略；更新 [OpenAiSubscriptionSerializationTest.kt:107–125](../../Kodex/openai/client/src/commonTest/kotlin/io/github/stream29/kodex/openai/client/OpenAiSubscriptionSerializationTest.kt#L107) 中要求整个 reasoning 缺失的旧断言。
- 设置测试：旧 JSON 缺失 reasoning 或缺失 effort、显式非默认 effort；验证解码值、新编码中 medium 字段和往返相等，不新增迁移。
- `agent-state/impl` commonTest：从 Medium/Low 设置分别触发普通与压缩路径，捕获 mock client 收到的 DTO 后使用生产 codec 编码，检查 effort，保持原有 input/metadata/压缩投影断言。
- `app/shared/session-title`、`tool/web-run` commonTest：复用已有 mock 捕获请求，检查标题 Medium/Low 与 WebRun 默认 Medium 的编码 JSON；不是只比较 Kotlin effort 对象。
- `openai/client` jvmTest：新增测试局部 loopback HTTP fixture，以 OpenAiClientConfig.baseUrl 指向本机、假认证、关闭重试，捕获真实客户端 body；返回最小合法 SSE/JSON，分别验证普通、压缩、Search 的 Medium/Low、默认 summary/context 缺失和已有协议字段。关闭 client/server，不写凭据或保留临时文件。
- 复用现有客户端配置入口，不新增生产 HttpClient 注入接口；host 测试基础设施或额外依赖按 openai-module-boundaries 放在 utils/host-test-support，只有确有必要才改构建文件。
- 执行顺序：定向 JVM 序列化测试 → 调用方离线测试 → loopback HTTP 测试 → 本机 linuxX64 相关 commonTest。测试任务名/筛选参数在执行前由本地 Gradle/TestBalloon 定义核实，不直接执行整个 client 测试集。

## 阻塞点审查

- 无待定产品设计；生产修改不依赖缓存键或 MCP 任务。
- 当前处于 executable 待安排状态；用户明确要求本次不执行，未来执行人须取得用户的实施安排后再开始，不能将目录状态视为执行授权。
- 验证准备项：当前 client 测试有真实后端且显式启用的用例，见 [OpenAiResponseStreamingTest.kt:23–40](../../Kodex/openai/client/src/commonTest/kotlin/io/github/stream29/kodex/openai/client/OpenAiResponseStreamingTest.kt#L23)。必须先确认定向筛选，不能把全量测试当作无副作用离线检查。
- HTTP 测试缺口：现有压缩测试主要验证事件收集/重试，mock DTO 不能替代实际 HTTP body。baseUrl 可配置，本地捕获方案无需真实凭据；fixture 尚未实现或运行，其可执行性须在实施中验证。
- 2026-09-07 只读检查发现 IDEA 进程及 Gradle 9.5.1 Daemon，Daemon JVM 为 `/home/stream/.jdks/openjdk-26.0.2`；项目 toolchain 为 25。执行前重新识别项目 IDE 并使用相关能力，重新确认 Daemon JVM、显式传给 gradlew；无法复用时报告阻塞，不擅自换设备。
- 当前候选生产修改文件未出现在 Kodex dirty 清单中；实施前重新检查。其他 UI 并行修改不清理、不回滚。
- 本轮仅做源码与测试内容检查，未编译、未运行单元测试、未进行本地 HTTP 或真实后端验证。真实后端是否接受各档位与实际效果未证明；本任务验收重点是忠实发送 JSON，不能将 loopback 验证表述为后端验证。
