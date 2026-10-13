# Task Tree

- `核对图片请求实现与已有超时证据`()
- `确认图片独立超时与保留重试的最小修复`()
- `实现两个图片端点的独立超时与回归测试`()
- `运行本地定向验证并检查普通请求与 SSE 回归`()
- `记录验证结果与未运行平台并归档任务`()

# Details

## 范围与状态

- 状态：最小源码修复与 JVM 验证完成；Native 验证未完成，未发布。
- 用户先要求排查近期图片生成超过 90 秒的失败并创建 discussion，随后在 Session 813 明确授权先修图片生成超时。
- 用户通过结构化选择确认：两个图片端点的单次请求与 socket 空闲超时改为独立的 10 分钟，保留现有自动重试。
- 本次仅修改图片客户端超时及定向测试；不修改全局设置、普通请求或 SSE 策略，不重新发起真实图片请求。
- 用户随后明确授权提交、推送本次修复及对应上下文；不包括其他讨论任务、版本发布或替换已安装 CLI。

## 已核实证据

- 场景：以三张真人头像生成「迹境 / JIJING」团队概念图，工具为 `image_gen.imagegen`，请求端点为 `/images/edits`。
- 以下时间均为 2026-10-13，UTC+8：
  - Session 807：21:13:18 开始，21:20:51 超时失败，工具总耗时约 453.241 秒；调用 ID 为 `call_095310a74c9c44e2acb3ab851aa5da28`。见[开始日志：38271 行](file:///home/stream/.kodex/log/Kodex.log#L38271)、[错误日志：38522–38523 行](file:///home/stream/.kodex/log/Kodex.log#L38522-L38523)。
  - Session 810：21:34:19 开始，21:41:52 再次超时失败，工具总耗时约 453.248 秒；调用 ID 为 `call_f34e7fd2c0e14ad9b0fe69053e52fde6`。见[开始日志：39116 行](file:///home/stream/.kodex/log/Kodex.log#L39116)、[错误日志：39380–39381 行](file:///home/stream/.kodex/log/Kodex.log#L39380-L39381)。
- 两次主日志均报告 `io.ktor.client.plugins.HttpRequestTimeoutException`，明确包含 `request_timeout=90000 ms`；随后出现 `Agent turn failed`。
- 修复前客户端默认单次 request timeout 为 90 秒、最多重试 4 次；全局 request/socket timeout 均使用该值，`editImage` 未单独覆盖超时。修复位置见[默认配置](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClientConfig.kt#L10-L16)、[图片请求](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L285-L314)。
- 453 秒与「5 次各 90 秒尝试＋重试间隔」高度吻合，但没有逐次 HTTP 尝试日志，实际次数仍是推断；不能把工具总耗时当成一次 HTTP 请求耗时。
- 已有 Session 809 手动对照：使用相同提示词和三张原图，取消整体／读取超时后，单次请求 HTTP 200，106.260 秒取得有效 PNG；上传在第 3.100 秒完成，响应头在第 103.803 秒到达。见[手动诊断报告：7–24 行](file:///home/stream/ACodeSpace/local/CtripContext/resources/ctrip/projects/hackathon-team-image-manual-request-2026-10-13.md#L7-L24)。
- 该成功请求证明 90 秒不足以覆盖本次生成；不同 transport、执行时间和服务状态使其不构成原失败的严格受控复现，不能据此认定唯一根因或服务器内部状态。
- 两次会话最终工具结果记录为 `user interrupt`，但主日志先报告超时，约 8 秒后才出现 host completion；只看会话结果会漏掉此前异常。见[807 工具记录：1 行](file:///home/stream/.kodex/sessions/807/work/119.json#L1)、[810 工具记录：1 行](file:///home/stream/.kodex/sessions/810/work/40.json#L1)、[host completion：38560 行](file:///home/stream/.kodex/log/Kodex.log#L38560)、[host completion：39454 行](file:///home/stream/.kodex/log/Kodex.log#L39454)。

## 已确认实施方案与验收

- 在 `OpenAiClientConfig` 末尾增加正数 `imageRequestTimeoutMillis`，默认 `600_000`；仅供客户端配置，不新增用户设置或迁移。
- `/images/generations` 与 `/images/edits` 都在请求级覆盖 `requestTimeoutMillis` 和 `socketTimeoutMillis`；仅延长总超时不足以消除旧的 90 秒 socket 空闲限制。
- 普通非流式请求默认 `90_000`、SSE socket 默认 `600_000` 与 SSE 无 request 总超时语义保持不变。
- 保留 `config.retry` 的状态码、transport、取消分类及最多四次重试；10 分钟是单次预算，包含重试的整次图片操作可能约 50 分钟，不新增总操作预算。
- 参数测试覆盖默认值、独立配置和非正值拒绝；本地 loopback 使用缩短计时验证两个端点越过普通超时后成功、最终超时、有限重试、取消及普通请求/SSE 不受污染。
- 验证使用虚构 token 和图片数据，不接触真实账户或头像；服务、线程与客户端由测试显式关闭。
- JVM 定向测试必须运行；Native 编译/测试视可用资源执行，未运行的平台明确记录，不用 JVM 通过代替全平台验收。
- 每次尝试的额外日志、最终 `user interrupt` 归因、幂等或查询能力研究不纳入本次最小修复。

## 相关历史决策

- [POST-SSE 超时语义任务](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-08-24-fix-post-sse-timeout-semantics.md#L1)曾明确保留普通非 SSE 请求的 90 秒总超时。本次只增加图片端点的例外，普通请求仍不放宽，不改写历史任务。

## 验证结果 — Session 813

- 显式使用 Temurin Java 25.0.3、Gradle 9.6.1，最多两个 worker；开始时本机没有运行中的 Gradle Daemon，未发现本项目的活动 IDE。
- `:openai-client:jvmTest` 通过：87 项，零失败；包含 9 项新增图片 loopback HTTP 测试和 4 项客户端配置测试。
- `:utils-ktor-client-ext-impl:jvmTest` 通过：7 项，零失败；覆盖 SSE 长连接、heartbeat、空闲超时与普通请求总超时。
- 新图片测试以普通超时 150 ms、图片超时 2,000 ms、慢响应 400 ms 验证两个端点成功；反向缩短图片超时验证失败，覆盖 503 后重试成功、transport 最多一次重试、取消不重试及后续请求可用。
- [新增测试源码：38–162 行](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/client/src/jvmTest/kotlin/io/github/stream29/kodex/openai/client/OpenAiImageTimeoutHttpTest.kt#L38-L162)；
  [图片测试 XML](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/client/build/test-results/jvmTest/TEST-suite_io.github.stream29.kodex.openai.client.openAiImageTimeoutHttpTest.xml#L2-L13)。
- 首次新增测试编译因误用 `OpenAiResponseResult` 别名的 `.result` 失败，修正为直接断言 `OpenAiResult` 后复跑通过；没有遗留编译错误。
- `:openai-client:linuxX64Test` 已尝试，但未缓存的 Kotlin/Native 2.4.20 工具链下载缓慢；在编译和测试开始前停止，不能声称 Native 验证通过。
- macOS、Windows 和 JS 未运行；未构建或替换 CLI，未发起真实生图，未验证真实服务端的 10 分钟阈值。
- 本轮 Gradle/编译进程已退出，未完成工具链下载的临时归档已删除；loopback 服务、线程与客户端均由测试关闭。
- BuildKodex 与 Kodex 的 `git diff --check` 通过；项目图、RPC、设置格式、图片 DTO、持久化和工具层均未修改。

## 提交与推送

- Kodex 修复提交 `3223a52d`：`fix: give image requests a dedicated ten-minute timeout`，已推送至 `origin/main`。
- BuildKodex 本次同步提交仅包含 Kodex 指针、图片超时 checklist 和本任务记录；其他讨论卡保留在工作区，不纳入提交。
- 本轮没有版本发布或安装替换；Native 验证限制仍按上文保留。
