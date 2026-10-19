# Task Tree

- `检查刷新调用链与凭据来源`()
- `使用现有凭据分段计时并执行对照实验`()
- `汇总瓶颈证据与实验边界`()
- `启动真实前端并测量界面刷新延迟`()
- `修正结论并清理独立前端`()
- `按一秒预算分解建连与接口等待并验证关键路径`()
- `更新根因证据与优化边界`()
- `隔离当前节点与其他节点到目标站点的延迟`()
- `给出原因和按收益排序的建议`()
- `核对用户新路由并复测真实前端和新连接`()
- `记录改动前后数据并清理复测资源`()

# Details

- 用户已自行改路由并要求复测。当前 ChatGPT 组为 FlowerCloud 日本高级 IEPL 专线 1，
  手动组仍为 iKuuu 日本 W03；现有 chatgpt.com 连接均已走 FlowerCloud。
  本轮不再调整任何节点或配置。

- 用户要求继续排查，至少交付原因和建议；本轮只用指定叶子节点的独立探测，
  不调用组测速、切换节点、配置更新或断开共享连接接口。

- 用户进一步要求解释 4 秒等待，并认为超过 1 秒值得追查；
  继续做有限只读实验，不将该性能目标视为已有事实或修改授权。

- 用户要求补充真实前端实验；上一轮直测和 RPC 实验不能代替端到端界面验证。
  本记录区分两轮证据，不将接口直测根因直接外推为用户界面慢的根因。

## 第一轮：接口与 RPC 的范围与方法

- 用户授权调查当前凭据刷新 usage 缓慢，要求实际实验；不包含业务修复或发布。
- 2026-10-17 15:03–15:06（Asia/Singapore），在本机实测。
- 当前来源为 Kodex；读取现有 access token，经 curl stdin config 提交认证头，
  凭据不进入命令参数、日志或报告。token 尚未过期。
- 对 `https://chatgpt.com/backend-api/` 下的 `wham/usage`、
  `wham/rate-limit-reset-credits`、`wham/profiles/me` 做分段计时。
  响应正文丢弃，仅记录状态码、时长、连接复用等元数据。
- 通过本机已运行 backend 的认证 WebSocket，调用
  `getAccountUsage`、`refreshAccountUsage`，不重启或更改 backend 配置。
  返回值只保留状态类型和快照时间，不输出账号或用量正文。
- 未刷新 OAuth token、消费 reset credit、调用模型或控制用户窗口。

## 实测结果

| 实验 | usage | reset credits | token profile | 说明 |
| --- | --- | --- | --- | --- |
| HTTP/2 三接口独立冷连接并行，第 1 轮 | 1.316s | 1.359s | 1.482s | 整轮 1.492s |
| HTTP/2 三接口独立冷连接并行，第 2 轮 | 3.564s | 4.124s | 3.845s | 整轮 4.134s |
| HTTP/2 三接口独立冷连接并行，第 3 轮 | 0.896s | 0.924s | 1.144s | 整轮 1.153s |
| 单进程顺序复用连接 | 首次 4.246s；末次 0.579s | 0.724s | 1.014s | 后三次 num_connects=0；非生产并行流程的耗时 |
| HTTP/1.1 三接口独立冷连接并行 | 0.782s | 1.031s | 1.225s | 单轮样本，不能据此认定 HTTP/1.1 更快 |

- 以上 16 个供应商 GET 均为 HTTP 200；curl 未配置重试，未出现传输错误。
- 第 2 轮的三条连接从启动到 TLS 完成均约 2.80s；
  顺序复用实验首次请求到 TLS 完成 3.335s，总时长 4.246s。
  其余冷连接样本到 TLS 完成约 0.19–0.26s。
- DNS/TCP 指标约毫秒级，但目标是 fake IP `198.19.0.43`，
  `ip route get` 确认经过 Mihomo TUN；不能把本地 TCP 完成视为上游连接已完成。
- 复用连接时，主 usage 请求 0.579s；附加统计仍有约 0.7–1.0s 的网络/服务等待。
- 运行中 backend 两次单独完整刷新耗时 **1.263s、1.029s**；
  后续 Get 均为 Available，fetchedAt 更新。Get 本身约 0.7–2.5ms，
  本机 WebSocket 握手约 2ms。
- 同时提交两次 backend 刷新，分别在 **0.968s、1.771s** 返回，
  最终快照更新；与互斥串行且不合并请求的实现一致。
- 四次 backend 刷新通常各请求三个接口，其内部每接口状态/重试没有单独计时，
  不将 curl 的“全部 200、无重试”结论扩展到 backend 内部。
- 原始协议探测第一次空参数编码使用 `[]`，Get 在约 1.6ms 内被 JSON 解码拒绝；
  改为 `{}` 后成功。该失败未提交 refresh，不计入刷新性能样本。

## 第一轮结论与证据

- 本次可定位的慢样本主要增量发生在 **新连接至 TLS 完成阶段**，
  在发送 HTTP 认证头之前；不能归因于凭据校验或 OAuth 续期。
  当前流量经过 Mihomo，但未隔离代理、上游线路、丢包等因素，
  **不能进一步认定是 Mihomo 本身的缺陷**。
- 一次刷新不是只取主 usage：并行查询三个接口且等待全部完成后发布快照。
  可选的 reset credits / token profile 出现长尾也会延长 Loading；
  本轮没有证明某一个可选接口持续异常。
  [fetchSnapshot:221–248](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/account-usage/src/commonMain/kotlin/io/github/stream29/kodex/openai/accountusage/CodexAccountUsageStoreImpl.kt#L221)
- 程序化并发刷新在整个网络请求期间持有 operationMutex，后来的刷新排队后重新请求，
  不是 single-flight；双 RPC 请求实验验证了这一行为。
  [refresh:187–218](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/account-usage/src/commonMain/kotlin/io/github/stream29/kodex/openai/accountusage/CodexAccountUsageStoreImpl.kt#L187)
- 打开 OpenAI 设置页会自动刷新，但 Loading 时界面禁用 Refresh。
  **纠正上一轮推断：不能把程序化并发排队当作用户正常重复点击造成慢的证据。**
  [SettingsViewModel:24–39](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/SettingsViewModel.kt#L24)
  [refreshEnabled:56–59](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/viewmodel/src/specMain/kotlin/io/github/stream29/kodex/app/accountusage/AccountUsageViewModel.kt#L56)
- 通用请求默认 90s 超时、最多 4 次重试，遇到暂态错误可能放大等待；
  **本次未复现该路径**，不能当作已确认根因。
  [请求超时:5–13](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClientConfig.kt#L5)、
  [重试配置:41–48](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClientConfig.kt#L41)
- 没有复现几十秒卡顿。当前证据不支持凭据失效，也不足以排除其他时刻的
  供应商长尾、重试、连接池或应用调度问题。

## 第二轮：真实前端端到端实验

- 2026-10-17 15:09–15:13（Asia/Singapore），两次启动用户实际的 `kodex` 命令，
  解析到 `Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`。
  自有前端 PID 分别为 433833、450639。
- 前端使用正常 Home、当前 Kodex 凭据和现有 backend，工作目录 `/tmp`；
  独立真实 PTY 160×50，不接管用户窗口、不打开已占用 Session。
- 用终端 SGR 鼠标事件点击屏幕上的 Settings → OpenAI → Refresh。
  pyte 只解码实际前端 ANSI 输出，不模拟 ViewModel、RPC 结果或网络响应。
  以 `Refreshing usage…` 的出现/消失验证状态，并用 monotonic clock 计时。

| 操作 | 界面可见等待 |
| --- | --- |
| 前端 1 首次进入 OpenAI | Refreshing 从出现到消失 **4.091s** |
| 前端 1 手动 Refresh，第 1 次 | 点击到完成 **0.820s** |
| 前端 1 手动 Refresh，第 2 次 | 点击到完成 **0.994s** |
| 前端 1 手动 Refresh，第 3 次 | 点击到完成 **1.348s** |
| 前端 2 首次进入 OpenAI | 点击到完成 **1.010s** |
| 前端 2 手动 Refresh，第 1 次 | 点击到完成 **0.815s** |
| 前端 2 手动 Refresh，第 2 次 | 点击到完成 **1.131s** |

- 前端 2 测量期间，以独立认证 RPC 连接每约 100ms 只读 Get usage 状态。
  三次操作观察到后端 Available 的时刻分别为点击后
  1.062s、0.835s、1.186s；对应界面完成 1.010s、0.815s、1.131s。
  19–56ms 的观察差在轮询分辨率内，**没有观察到后端完成后界面再滞留数秒**。
- 手动点击后约 16–29ms 出现 Refreshing；第二个前端首次进入约 32ms。
  输入到 Loading 的路径在这些样本中没有秒级延迟。
- 确实观察到真实前端一次约 4s 的等待，不能再只用 RPC 的约 1s 样本概括体验。
  该 4s 前端样本没有逐接口/传输阶段计时，**尚不能证明它也是 TLS 慢**。
- 本轮仍未复现十几秒或几十秒的前端等待；具体长等待根因未确定。
  尚不能将第一轮冷连接抖动或程序化排队认定为用户所遇场景的根因。

## 第三轮：按一秒预算定位慢阶段

### 方法与边界

- 用户进一步要求解释 4s，并把超过 1s 作为性能关注点。
  本轮只做诊断，不修改业务代码或用户代理设置。
- 用 PycURL 7.48.0 / libcurl 8.22.0 / OpenSSL 3.6.4 建立独立 Multi，
  关闭 multiplex，与生产 Curl engine 的并发模式一致；
  三个接口并行，开启响应压缩，测量新连接与同一个池的复用连接。
- 这不是生产 native backend 的埋点；生产二进制包含 OpenSSL 3.6.2。
  另用系统 curl 8.18.0 做同时间对照，避免把诊断库差异当成业务根因。
- 主实验 18 个认证 GET，TLS 参数对照另 8 个认证 GET，全部 HTTP 200，
  不配置重试。凭据只在实验进程内存中使用，不记录认证头或正文。
- 另做 10 条不发送 HTTP、不使用任何凭据的 TLS 连接；
  记录 ClientHello、ServerHello、Finished 的方向与时刻，不记录报文或密钥。

### 建连与复用

| 同一轮并行请求 | usage | reset credits | token profile | 整轮 |
| --- | --- | --- | --- | --- |
| 新池冷连接，第 1 轮 | 5.096s | 5.069s | 5.030s | 5.097s |
| 新池冷连接，第 2 轮 | 5.460s | 5.659s | 5.799s | 5.800s |
| 持续池首次冷连接 | 5.573s | 5.810s | 5.527s | 5.810s |
| 同池复用，第 1 轮 | 0.590s | 0.813s | 0.868s | 0.869s |
| 同池复用，第 2 轮 | 0.573s | 0.647s | 0.797s | 0.798s |
| 同池复用，第 3 轮 | 0.553s | 0.718s | 0.863s | 0.863s |

- HTTP 协议均为 HTTP/2；PycURL 的 INFO_HTTP_VERSION 返回数值 3，
  对应 CURL_HTTP_VERSION_2，不是 HTTP/3。
- 冷样本的 TCP 完成后到 TLS 完成耗时 **4.061–4.748s**；
  随后从请求就绪到首字节约 **0.773–1.175s**。
  后一个指标包含网络往返和服务端时间，不能单独称作服务端计算。
- 热样本 new_connections=0，建连时间为零。主额度请求稳定在本组的
  0.553–0.590s，附加统计让整个快照比主额度多等约 0.224–0.310s。
- 压缩后 profile 约 6KB，解压后的 JSON 约 52KB；
  Python JSON 解码约 0.16–0.63ms，不能解释数秒等待。
  该解码计时不是 Kotlin decoder 的基准，但结合前端输入/显示计时，
  未发现本地解析或渲染造成秒级等待的证据。

### TLS 与路由对照

- 同时比较默认 TLS、限制 X25519、限制 TLS 1.2、系统 curl。
  第 1 轮四组总耗时 2.06–2.31s；
  第 2 轮 4.76–5.41s，建连阶段分别约 3.94–4.34s。
  没有证据支持用切换 TLS 版本/曲线来消除慢等待，
  不提出 TLS 降级作为修复。
- 无 HTTP / 无凭据探针也复现：

| 入口 | TLS 总时长 | ClientHello 到 ServerHello 的主要等待 |
| --- | --- | --- |
| TUN，慢样本 1 | 4.277s | 约 4.262s |
| 本机 SOCKS，慢样本 1 | 4.284s | 约 4.261s |
| TUN，慢样本 2 | 5.318s | 约 5.308s |
| 本机 SOCKS，慢样本 2 | 5.320s | 约 5.307s |

- 同一组随后两条恢复到约 0.67–0.68s；另两轮探针范围约 0.51–1.24s。
  多秒等待发生在 ClientHello 发出后、首个 ServerHello 收到前，
  后续 Finished 处理只占毫秒级到十余毫秒。
- 本机 TCP 与 SOCKS 协商均约毫秒级；这只说明本机代理迅速接受连接，
  **不保证其上游连接已经完成**，不能据此排除代理/上游路径。
- 只读查询 Mihomo controller 的现有 chatgpt.com 连接，规则路由到
  ChatGPT 分组下手动选择的日本 W03 节点。未切换分组、节点或规则。
  TUN 与显式 SOCKS 均出现同类等待，不支持将问题归结为仅 TUN 入口。

### 可以下的结论

- 已复现足以解释 4s 体验的明确机制：
  **当前访问 chatgpt.com 的路径有数秒 TLS 首个服务端响应长尾**，
  在任何 HTTP 凭据校验或 usage 请求之前就可以独立发生。
  这比“可能是网络慢”更具体，但仍不能细分为节点、线路、重传或上游哪一项。
- 原始 4.091s 前端样本没有逐请求传输埋点，不能追溯性证明该次的全部分段；
  本轮是相同目标与当前路由下重复复现的机制证据。
- 1s 可以作为优化目标，本次热连接整轮已有 0.80–0.87s 的样本；
  不能把 4–5s 的新连接长尾当成必要的正常界面开销。
- 把主额度先发布、附加统计异步补齐，按本组热样本可去掉约 0.22–0.31s 的等待，
  但**不能单靠这个改动消除主额度自身的多秒建连等待**。
  此处仅是实验支持的设计方向，未获准实施，也没有用旧缓存冒充刷新完成。
- backend 本来就持有共享 OpenAiClient，不能声称每开一个前端都会重建 HTTP client。
  为何某次生产请求需要新连接（连接繁忙、空闲失效或其他原因）尚未逐请求观察。
  [客户端与账号状态创建](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L61)

## 第四轮：定位到当前供应商的 ChatGPT 域名路径

### 独立节点对照

- 2026-10-17 下午，在当前 Mihomo v1.19.27 的本机 Unix controller 上，
  仅调用指定叶子节点的 `/proxies/{name}/delay`。
  未调用组测速（可能清除固定选择）、PUT、PATCH、DELETE、reload 或连接关闭接口。
- 每组最多并行 3 个控制器探测。URLTest 发起公开地址的 HEAD，不使用 OpenAI 凭据；
  另测 TLS / 公开 trace，所有证书验证均开启。
- `🤖 ChatGPT` 一直选择 `🚀 手动选择`，后者一直选择 iKuuu 日本 W03；
  每组实验前后均核对，未改变当前路由。
- 主机开启 `unified-delay=true`。匹配版本的实现先做一次 HEAD，
  第二次成功时用第二次计时替代首次；第二次失败则仍报告首次。
  因此本轮记录**完整 controller 请求耗时**，不把 `delay` 一律视作冷连接或热连接。
  “完整耗时减 delay”也不能普遍当作首次耗时。
  [v1.19.27 URLTest 源码](https://github.com/MetaCubeX/mihomo/blob/v1.19.27/adapter/adapter.go#L191-L261)、
  [指定节点 API](https://github.com/MetaCubeX/mihomo/blob/v1.19.27/hub/route/proxies.go#L98-L137)

- 同供应商三轮对照，单位秒，均为完整探测耗时，非带凭据的 usage 请求：

| 节点 | chatgpt.com | 普通 Cloudflare 站点 | Google gstatic |
| --- | --- | --- | --- |
| iKuuu 日本 W03 | 1.279–2.732 | 0.446–0.469 | 0.419–0.465 |
| iKuuu 日本 W01 | 1.276–2.736 | 0.447–0.685 | 0.459–0.502 |
| iKuuu 新加坡 W01 | 3.543–5.472 | 1.315–4.487 | 3.450–3.701 |

- 换同供应商日本编号没有明显改善；新加坡节点更慢，
  不支持“随便换一个地理位置更近的节点”作为建议。
- 随后跨供应商、同一时段三轮对照：

| 节点 | ChatGPT 三轮完整耗时 | 普通 Cloudflare 范围 |
| --- | --- | --- |
| 当前 iKuuu 日本 W03 | **4.431s、5.808s、0.630s** | 0.443–0.799s |
| KittyNetwork Japan 01 | **0.566s、0.248s、0.269s** | 0.492–0.762s |
| FlowerCloud 日本标准 IEPL 专线 1 | **0.520s、0.328s、0.313s** | 0.427–0.753s |

- 两个替代节点都已存在于用户配置中，且可被 `🤖 ChatGPT` 直接选择。
  仅探测，未实际切换。三轮小样本不能证明长期 p95，也未验证替代节点下
  真实账号的 usage 刷新完成时间。

### 固定地址与同 IP / 不同 SNI

- 本机 Mihomo DNS 查询 chatgpt.com 返回 `203.111.254.117`；
  Google 公共 DoH（本机请求及外部只读核验）返回
  `104.18.32.47`、`172.64.155.209`。
  [Google DoH](https://dns.google/resolve?name=chatgpt.com&type=A)
- 本机 DNS 配置使用 doh.pub / dns.alidns.com，未配置 nameserver-policy、
  fallback 或 OpenAI hosts 覆盖。这是另一个应核查的解析异常，
  但**没有把它直接认定为本次延迟根因**。
- 显式 SOCKS 分别连接域名和两个公共 DoH 地址，TLS SNI 始终为 chatgpt.com。
  三者同轮仍共同出现约 4.06s、3.94s 的慢握手；
  因此固定地址没有消除延迟，不能建议用永久 hosts / 固定 IP 解决。
- 进一步做 3 轮 × 2 IP × 2 SNI 的配对对照：
  目标地址分别为 `104.18.32.47`、`104.16.124.96`，
  SNI 分别为 chatgpt.com、www.cloudflare.com。
  两者证书均验证成功，随后公开 `/cdn-cgi/trace` GET 均 HTTP 200。
  不提交任何个人凭据；trace 只保留 colo / loc 等字段，不记录出口 IP。
- 用每条自有 socket 的 sourcePort 核对 controller 元数据：
  所有固定地址对照均走相同 Match 本地规则、相同手动组、
  相同 iKuuu 日本 W03 叶子节点；未发生 SNI 覆盖目的地址。

| 固定目标 IP | SNI=chatgpt.com 的 TLS 时长 | SNI=www.cloudflare.com 的 TLS 时长 |
| --- | --- | --- |
| 104.18.32.47 | **1.657、4.307、5.272s** | **0.192、0.307、0.183s** |
| 104.16.124.96 | **1.657、4.364、5.238s** | **0.192、0.861、0.273s** |

- 慢时间仍主要在 ClientHello 到 ServerHello。相同目标 IP 不是充分解释；
  已把问题进一步缩小到**当前 iKuuu 路径上，与 ChatGPT TLS 域名相关的连接处理**。
- 无法从客户端单独区分供应商内部的 SNI 分流/中继，还是 Cloudflare/OpenAI
  对该供应商出口的特殊处理。**不把这些机制推断写成已证实的限速、审查或故障归属。**

### 原因与按收益排序的建议

1. **首要原因与建议：当前供应商到 ChatGPT 的域名特定 TLS 长尾。**
   不需要个人凭据，也不需要调用 usage 就能复现；
   同 IP / 不同 SNI 和跨供应商对照支持这一定位。
   建议经用户确认后，让 `🤖 ChatGPT` 单独选择 **KittyNetwork Japan 01**，
   以 **FlowerCloud 日本标准 IEPL 专线 1** 为备选，
   不修改共享 `🚀 手动选择`，以免影响其他流量。
   切换后必须验收真实 usage 的首次进入、连续刷新、空闲后刷新，
   按用户的一秒目标观察分布，而非用公开 HEAD 的最好样本保证结果。
2. **次要实现成本：主额度等待附加统计。**
   本组热连接附加等待约 0.22–0.31s；
   可考虑主额度成功后立即发布该部分，重置券明细 / Token 统计异步补齐。
   须保留同账号隔离、明确部分更新/失败与真实新鲜度，
   不以旧缓存冒充已刷新，也不能靠该改动解决多秒建连。
3. **诊断与验收：记录真实关键路径，而非只看代理面板延迟。**
   建议在后续获准的局部改动中记录 usage 操作排队、各接口耗时、重试、
   新建/复用连接与状态发布时间，均不记录 token、认证头或个人正文。
   按首次、热连接、空闲重连区分样本。不要用 1s 硬超时、TLS 降级或固定 IP
   代替根因处理。

## 第五轮：用户切换路由后的真实复测

- 2026-10-17 15:43–15:48（Asia/Singapore），用户已自行将
  `🤖 ChatGPT` 改为 **FlowerCloud 日本高级 IEPL 专线 1**，
  不是上一轮探测的日本标准 1；共享手动组仍为 iKuuu 日本 W03。
- 复测前后 controller 所见 13 条 chatgpt.com 存量连接均走该 FlowerCloud，
  没有旧 iKuuu 连接；三条自有新 TLS 连接也逐条核对了相同叶子节点。
- 启动用户实际 native 二进制的独立真实 PTY 前端，PID 602139，
  正常 Home、当前凭据、现有 backend，真实鼠标事件点击 Settings → OpenAI / Refresh。
  屏幕只在内存中解码，不输出账号正文。
  每次界面结束后用只读 Get 验证 Available 且 fetchedAt 比操作前更新。

| 真实前端操作 | 点击到界面刷新完成 |
| --- | --- |
| 首次进入 OpenAI | **0.714s** |
| 连续手动 1 | **0.805s** |
| 连续手动 2 | **0.778s** |
| 连续手动 3 | **1.245s** |
| 连续手动 4 | **0.912s** |
| 本前端不操作 125s 后 | **1.098s** |
| 空闲后连续手动 1 | **1.070s** |
| 空闲后连续手动 2 | **0.792s** |

- 8 次全部 Available 且快照更新；本组中位数 **0.859s**、最大 **1.245s**，
  **5/8 低于 1s，3/8 超过 1s**，不能宣称稳定达到一秒目标。
  点击后约 13–47ms 出现 Loading/Refreshing。
- “125s 空闲”仅指自有前端没有操作；共享 backend 和用户其他活动继续运行，
  不能据此声称 backend 的整个连接池完全空闲或必然发生了重连。
- 独立、无 HTTP / 无凭据的新 TLS 握手：
  **0.481s、0.200s、0.394s**，均走新 FlowerCloud 节点，
  不再复现旧路径的 4–5s 首个 ServerHello 等待。
- 另用独立 Multi 做三轮三接口并行，每轮重建池、每个请求 new_connections=1，
  开启压缩、不重试，9 次均 HTTP 200：

| 强制新连接轮次 | usage | reset credits | token profile | 整轮 |
| --- | --- | --- | --- | --- |
| 1 | 1.321s | 1.351s | 1.576s | **1.577s** |
| 2 | 0.753s | 0.951s | 1.145s | **1.145s** |
| 3 | 0.893s | 0.897s | 1.223s | **1.223s** |

- 这 9 条连接的 TLS 建立约 **0.201–0.244s**；
  剩余时间已不主要是旧路径的数秒 TLS 等待，而在 HTTP 响应/下载阶段。
  三轮 Token profile 均最后结束，比主额度多等约 0.25–0.39s；
  但主额度自身也有一次 1.321s，不能承诺仅异步附加统计就一定全部小于 1s。
- 独立网络探针与前半段前端测试时间有重叠；
  空闲后 3 次前端刷新没有并行的自有外网探针。
  本轮是当前共享环境下的真实功能复测，不是隔离负载基准。
- **结论：切换已明显消除了本轮可见的主要 4–5s 建连长尾，
  但“稳定低于 1s”尚未达成。**
  当前可继续优化的成本是接口响应长尾及等待全部附加统计的关键路径；
  某次前端超一秒仍未逐请求埋点，不将独立探针结果当作其精确逐段计时。

## 验证与清理

- 完成真实接口、运行中 backend 与真实前端的只读实验；
  未改业务代码，因此未启动编译或测试构建。
- 仅新增本任务记录，保留原有工作区修改。
- 两个自有前端均已退出并回收，观察连接已关闭，无其 Home lease 遗留；
  前端 1 正常退出 0，前端 2 由 SIGINT 退出。共享 backend PID 1471498 保持运行。
- 临时脚本、脱敏计时文件与自有前端的空日志在记录结果后移除。
- 第三轮所有独立 Multi、socket 与 TLS 连接已关闭；
  临时分段、TLS 对照脚本与脱敏计时文件在结果记入本文件后移除。
- 第四轮指定叶子测速只更新正常健康测试历史；手动组与 ChatGPT 组选择保持不变。
  所有独立 controller / socket / TLS 连接已关闭；
  临时路由、地址/SNI 对照脚本、脱敏结果与自有 pycache 在记录后移除。
- 第五轮自有前端 602139 正常退出 0、无其 Home lease 遗留，
  本次自有空日志和临时复测脚本/脱敏数据在记录后移除；
  共享 backend 1471498 保持运行，用户的新节点选择未被修改。
