# Task Tree

- [done] 修复 OpenAI POST-SSE 超时语义
  - [done] 修正 POST-SSE 发送侧标记
    - [done] 在 render 后以 `SSEClientContent` 包装原始请求体
    - [done] 让 response adapter 复用同一份包装内容
    - [done] 保留 POST body、headers、错误与取消语义
  - [done] 改用传输空闲超时
    - [done] 保留非流式请求的 90 秒总超时
    - [done] 为 SSE 覆写 300 秒 socket timeout
    - [done] 让任意网络数据包滑动重置计时
    - [done] 让 comment heartbeat 继续为连接续命
  - [done] 补齐 CLI engine 行为
    - [done] 让 Linux KodexCurl 实现 socket timeout
    - [done] 保留 macOS Darwin 原生 socket timeout
    - [done] 接受 Windows WinHttp 原生阈值差异
    - [done] 不增加 common data-event watchdog
  - [done] 收敛 remote compaction 边界
    - [done] 增加 480 秒逻辑操作总预算
    - [done] 将 compaction 限制为最多两次重试
    - [done] 保留 completed、协议错误与取消语义
  - [done] 覆盖确定性回归场景
    - [done] 用本地 CIO 服务覆盖长 POST-SSE
    - [done] 用本地流服务覆盖 KodexCurl 空闲计时
    - [done] 在 Windows 与 macOS 验证 packet-idle 语义
    - [done] 覆盖非流式 90 秒超时不变
    - [done] 覆盖 sampling 与 compaction 终态映射
  - [done] 更新流式传输维护决策
  - [done] 运行 JVM、JS 与三类 CLI 验证
    - [done] 运行 JVM 与 JS 验证
    - [done] 运行 Linux KodexCurl 编译、测试与本地流验证
    - [done] 在 Windows WinHttp 上验证 packet-idle 语义
    - [done] 在 macOS Darwin 上验证 packet-idle 语义

# Details

## Problem

- 普通 Responses 与 remote compaction 共用 `postSseEvents` 和全局 90 秒 request timeout。
- `SseCompatibility` 当前只在响应适配阶段构造 `SSEClientContent`；发送阶段仍返回原始 JSON content，Ktor 因而把 POST-SSE 当作普通请求应用绝对总超时。
- 当前实现窗口内，普通 Responses 的 113 次 `Retryable` 中有 90 次在 89.5–90.5 秒结束；session 100 已有六个 turn 因连续重试耗尽而停止。
- Ktor 的 socket timeout 是网络包空闲时限，不是 SSE data-event 时限；comment、heartbeat 和任意响应字节都会续期。
- CIO 与 Darwin 支持 Ktor socket timeout；Ktor Curl、JavaScript 和当前 WinHttp 路径不支持该配置。
- Linux 使用项目内 KodexCurl，可以在本地 engine 中补齐 socket timeout。
- Windows 继续使用 WinHttp。Native CIO 的 TLS session 支持尚未完成，且 CIO 尚不支持 TLS 1.3，不能用于 OpenAI HTTPS。

## Decisions

- `OpenAiClientConfig.requestTimeoutMillis` 保持 90,000，只约束普通非 SSE 请求。
- POST-SSE 在发送阶段完成 Ktor SSE 标记，不再应用 request 总超时。
- SSE socket timeout 默认 300,000 毫秒，通过每个 POST-SSE 请求的 Ktor timeout capability 覆写全局 90 秒 socket timeout。
- 不增加 application data-event watchdog。任意网络数据都会续期；heartbeat-only 或持续发送残缺数据的普通 sampling 可能长期存活，这是方案 A 的已接受剩余风险。
- Linux KodexCurl 按请求读取 `socketTimeoutMillis`，依据请求与响应字节活动实现 engine-side packet-idle 计时，不使用基于平均速率的 libcurl low-speed timeout。
- KodexCurl socket timeout 必须以 `SocketTimeoutException` 关闭已暴露的 response body，不能误报 connect timeout 或转换成无 cause EOF。
- macOS 继续使用 Darwin 原生 socket timeout。
- Windows 继续使用 WinHttp。活动流必须能超过 90 秒 request timeout；空闲阈值使用 WinHTTP native receive timeout，不承诺与其他平台同为 300 秒。
- JavaScript 不属于原生 CLI 发布契约；保持编译兼容，不承诺 runtime socket idle timeout。
- 在收到协议终态前，clean EOF、连接 reset、transport `IOException` 和 socket timeout 都视为可恢复连接终止；不要求 transport 层精确区分 FIN、RST 与无终态 EOF。
- 普通 sampling 使用既有 resumable continuation 路径重试，保留已接收的 durable output；remote compaction 在成功前不写 checkpoint，重试同一份 snapshot。
- 普通 sampling 将 socket timeout 投影为现有无协议终态 `Retryable`；remote compaction 将其纳入现有 streaming 重试与结构化日志。
- 普通 sampling 保留当前初次请求后最多四次协议重试，不增加健康流的绝对总时限。
- Remote compaction 使用 480,000 毫秒绝对总预算和最多两次重试；HTTP 与 streaming 失败共享该预算。
- 外部取消立即传播，不转换为 idle、retry 或 deadline failure。
- 已收到 `response.completed` 后的物理断线不改变成功结果，也不触发重试。
- 不修改 `response.completed`、`response.failed`、`response.incomplete`、`[DONE]`、EOF 和 compaction output 数量校验。

## Implementation

- `Kodex/utils/ktor-client-ext/.../SseCompatibility.kt`
  - 对标 Ktor SSE 插件，在 `AfterRender` 设置 `SSECapability`、response adapter 和 reconnection client attribute。
  - 返回包装原始 rendered content 的 `SSEClientContent`。
  - Response adapter 校验并复用 outgoing `SSEClientContent` 的 delegate，并将 session 生命周期绑定到 response call context。
- `Kodex/utils/ktor-client-ext/.../SseRequests.kt`
  - 接受显式 socket timeout。
  - 通过 request-level `timeout` 覆写 SSE 的 `socketTimeoutMillis`。
  - 不拦截正常 EOF、上游异常或外部取消。
- `Kodex/utils/ktor-client-ext/.../kodexcurl/KodexCurlRaw.kt`
  - 从 `HttpTimeoutCapability` 读取 socket timeout。
  - 将 socket timeout 和请求信息传入 native handler。
- `Kodex/utils/ktor-client-ext/.../kodexcurl/KodexCurlCallbacks.kt`
  - 在请求与响应字节活动时更新当前 request 的 activity mark。
- `Kodex/utils/ktor-client-ext/.../kodexcurl/KodexCurlMultiApiHandler.kt`
  - 在现有 multi poll 循环中检查 packet-idle。
  - 超时后使用 `SocketTimeoutException` 关闭 response body 并清理 easy handle。
  - 保留 request identity、取消队列、失败连接禁用复用及资源释放顺序。
- `Kodex/openai/client/.../OpenAiClientConfig.kt`
  - 增加 SSE socket timeout、compaction deadline 和 compaction retry 上限配置及参数校验。
  - 保持 `requestTimeoutMillis` 默认值不变。
- `Kodex/openai/client/.../OpenAiClient.kt`
  - 将 300 秒 socket timeout 应用于普通与 compaction POST-SSE。
  - 仅解包并分类 Ktor SSE 包装的 transport timeout，不吞掉解析与协议错误。
  - 让 sampling socket timeout 保持现有 `Retryable` 投影。
  - 让 compaction socket timeout、HTTP 和 transport failures 共享最多两次重试。
  - 使用可区分外部取消的方式在整个 compaction 重试循环外应用 480 秒 deadline。

## Deterministic Tests

- JVM 本地 CIO 服务测试，不使用 mock：
  - 将 request timeout 缩短后，持续有响应数据的 POST-SSE 可运行超过该总时限并正常结束。
  - 服务端收到的 POST body 和 content headers 与包装前一致。
  - 响应数据间隔小于 socket timeout 时持续续期。
  - 纯 comment heartbeat 也会重置 socket timeout。
  - 无响应数据达到 socket timeout 时抛出 transport timeout。
  - 非 SSE 慢请求仍命中 request timeout。
  - 非 2xx SSE 响应继续保留原始 HTTP 异常。
- Linux KodexCurl 测试：
  - activity tracker 在请求或响应字节到达时滑动更新。
  - 无字节达到 socket timeout 时只终止对应 request。
  - response headers 已暴露后超时仍以 `SocketTimeoutException` 关闭 body。
  - 超时与外部取消均保留现有 request identity、资源释放和禁止失败连接复用语义。
  - 使用本地真实流服务验证有包长流和无包超时，不使用业务对象 mock。
- CLI 平台验证：
  - macOS Darwin 的有包流超过缩短后的 request timeout 并正常结束。
  - Windows WinHttp 的有包流超过缩短后的 request timeout 并正常结束。
  - Windows 空闲结束只验证可恢复终态，不断言 300 秒阈值。
- OpenAI client 与 runtime 测试：
  - clean EOF、连接 reset 与 transport `IOException` 在协议终态前进入有限重试。
  - sampling socket timeout 仍进入既有 `Retryable` 计数。
  - compaction socket timeout 可重试且总实际请求不超过三次。
  - partial sampling output 通过 resumable continuation 保留，不盲目重放已写历史。
  - partial compaction 不写 checkpoint，下一 attempt 使用同一 snapshot。
  - `response.completed` 后断线不产生额外请求。
  - compaction deadline 在 480 秒逻辑预算处终止，外部取消不被转换。
  - `response.completed` 仍立即成功并停止读取。

## Validation

- 使用 `/home/stream/.jdks/openjdk-26.0.2`。
- 当前 Linux 主机已通过：
  - `:utils-ktor-client-ext:jvmTest`（9 tests）
  - `:utils-ktor-client-ext:jsNodeTest`
  - `:utils-ktor-client-ext:compileKotlinLinuxX64`
  - `:utils-ktor-client-ext:linuxX64Test`
  - `:openai-client:compileKotlinJvm`
  - `:openai-client:compileTestKotlinJvm`
  - `:openai-client:compileKotlinLinuxX64`
  - `:openai-client:jvmTest --tests '*openAiRemoteCompactionV2Test*'`
  - `:openai-client:jvmTest --tests '*openAiClientConfigTest*'`
  - `git diff --check`
- 使用临时本地 Python chunked SSE 服务验证真实 KodexCurl：
  - 有包间隔小于 socket timeout 的流收到两个事件并正常结束。
  - 无包超过 socket timeout 的流以 `SocketTimeoutException` 终止。
  - 手工验证完成后已删除临时测试代码和停止服务。
- Windows VM 已通过 `:utils-ktor-client-ext:mingwX64Test`：
  - 真实 `WinHttp` client 在 50 ms request timeout 下收到跨越该时限的两个 SSE 事件。
  - 临时 JDK 25、测试工作区和测试文件均已删除，`win11` 已关机。
- Remote MacBook 已通过 `:utils-ktor-client-ext:macosArm64Test`：
  - 真实 Darwin client 在 50 ms request timeout 下收到跨越该时限的两个 SSE 事件。
  - 临时测试工作区、服务器和测试文件均已删除。
- `openai-client` 的既有 JS runner 当前在生成的 `Kodex-openai-client-test.js` 中失败；本任务不把该既有问题误报为 SSE 回归。
- 运行 `git diff --check`。
- 只有在执行阶段获得明确真实网络授权时，才重放超过 90 秒的普通 Responses 流和已选历史 compaction 快照。

## Durable Records

- 更新 `checklist/openai-module-boundaries.md`，记录 POST-SSE 不使用普通 request 总超时、使用 packet-idle socket timeout，以及 Windows 阈值差异。
- 更新 `checklist/kodex-curl-engine.md`，记录 KodexCurl 对 Ktor socket timeout capability 的实现和 timeout cause 传播要求。
- 不改写既有 done 任务；本任务完成后保留新的最终决策和验证结果。

## Excluded

- libcurl 请求体 rewind、TLS error buffer、失败连接复用与并发 retry storm。
- application data-event watchdog 和普通 sampling semantic-progress timeout。
- fork Ktor WinHttp、把 Windows 切换到 Native CIO，以及 JavaScript runtime socket timeout。
- UI 重试进度、自动压缩阈值和 session 数据格式。
