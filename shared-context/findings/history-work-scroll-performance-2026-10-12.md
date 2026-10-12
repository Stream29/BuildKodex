# History Work 滚动性能实测

## 范围与结论

- 基线：Kodex `4940c2ad`，BuildKodex `68e2ae8`；2026-10-12 在 remote-macbook（macOS 26.5.1 / ARM64）的独立源码快照测量。
- 本报告前述实测为修复前研究及失败的最小改动实验；随后用户已授权修复，参见
  [已验收修复任务](../../kanban/done/2026-09-28-plan-fix-history-work-scroll-after-rpc.md)。
  新版结果单独追加，不改写原始基线数据；RPC 缓存契约与滚轮步长不在修改范围内。
- **已复现逐项历史分页造成的位移损失**：边缘进入视口才加载一个结构块；输入到达当前数据边界时，未消费的行数不会在数据到达后补滚。
- **需要区分两种 Work 场景**：未封口、逐条显示的 Work 有频繁分页；已经展开的 WorkGroup 内部没有逐条历史分页，本次未复现位移损失，但首次展开存在串行时间戳读取开销。
- **追加最小改动实验未通过交付标准**：只删除边缘可见等待会提前加载，但停止输入后仍持续双向读取，不能单独作为修复。

## 测量方法与边界

- 使用生产 `readHistoryChunk`、`createAgentHistoryViewModel`、`AgentHistoryView`，以及生成的 kRPC 接口、JSON 通道传输和 `RpcCachedIndexVersioned`。
- 后端是合成 InMemory 数据，不读取真实用户历史；不是 filesystem、跨机器网络或 Native CLI/PTY 性能基准。
- 结构读取每组测三轮：257 个 Index 消息，或起始消息加 256 个工具 Work；封口组另加一条结束消息。分别使用 64 B / 16 KiB 工具输出；Index 消息内容不随工具输出大小改变。
- 滚轮测试使用真实 History renderer 的 Mosaic 测试宿主，80×12 视口；Index 消息为两行，折叠工具为一行。已展开 WorkGroup 含 256 个折叠工具子项。
- 每组发送 40 次 WheelUp，间隔真实时间 10ms，每次请求 3 行，合计请求 120 行；等待 300ms 后记录实际消费。三轮交替场景顺序，初始可滚动后才计数。
- `delay0` 保留 RPC 自身开销；`delay3` 在每次数据 RPC 服务调用中**人为增加真实时间 3ms**，仅用于敏感性实验，不代表用户机器的实际延迟。
- 测试宿主的 `sendMouseEvent` 是排队注入；探针记录的 `input_ms_*` 不是端到端输入或帧延迟，`frames` 也不是显示帧率。本次没有给出 UI 绘制的耗时占比。
- 结构计数不含连接初始化和 metadata flow；renderer 计数在初始视口就绪后重置。首次展开先等待初始行元数据读取，再单独计时。

## 结果一：实际消费的滚动行数

| 场景 | 无注入延迟，三轮 | 每个数据 RPC +3ms，三轮 | 一级窗口峰值 |
| --- | --- | --- | --- |
| Index 消息，持续跨分页边界 | 80 / 80 / 80 | 40 / 40 / 40 | 8 |
| 未封口逐项 Work，持续跨分页边界 | 40 / 40 / 40 | 14 / 17 / 16 | 14 |
| 已展开 WorkGroup，组内滚动 | 120 / 120 / 120 | 120 / 120 / 120 | 3 |

- 同样请求 120 行，零注入延迟时逐项 Work 仅消费 40 行，Index 也只消费 80 行。每次输入分别只能跨过已加载的一行工具 / 两行消息，下一块到达后不会补足本次剩余位移。
- Work 零注入延迟的每轮计数：`index.floor=40`、`time.exact=40`、`work.indexes=80`、`work.values=40`；40 个 payload。这不是缓存窗口内的纯滚动，而是逐行触发读取。
- 已展开组的 120 行滚动期间，三个被观测 timeline 的数据 RPC 都为零；一级窗口仅 3 项不代表只持有 3 个对象，其组内另有 256 个 child。
- 这些数字对应指定 fixture 和输入节奏，不能直接外推用户日常滚动速度或所有工具布局；但足以证明分页与未消费输入之间的因果边界。

## 结果二：结构读取与缓存

每次连续读取 64 个结构块，64 B 工具输出；耗时为三轮中位数：

| 场景 | 数据 RPC 数 | Work payload 数 | 耗时 |
| --- | ---: | ---: | ---: |
| Index，冷缓存 | 258 | 0 | 48.88ms |
| Index，重读同一区间 | 128 | 0 | 24.65ms |
| 未封口 Work，冷缓存 | 322 | 64 | 56.64ms |
| 未封口 Work，重读同一区间 | 256 | 64 | 42.66ms |

- 重读时 Index 的 exact 值和时间戳命中缓存，仅剩 `index.floor` 查询。
- Work 重读仍有 `index.floor=64`、`work.indexes=128`、`work.values=64`。`valuesIn` 返回后写入 exact cache，但自身每次仍跨 RPC。
- 16 KiB Work 输出的冷 / 重读中位数为 52.72 / 42.69ms；本次没有证据表明 payload 大小是主要差异来源，不据此推导序列化成本占比。
- 256 个工具形成的**折叠**封口组，加上两条消息，总共只读 2 个结构块、形成 3 个一级 item；冷读取 14 次数据 RPC、零 Work payload。不能把这一场景等同于未封口逐项 Work。

## 结果三：首次展开 WorkGroup

- 256 个工具组从折叠到全部 child header 可用：
  - 零注入延迟：80.52 / 76.18 / 71.87ms。
  - 每个数据 RPC +3ms：1189.99 / 1257.06 / 1247.46ms。
- 三轮调用结构一致：`work.values=1`、`work.floor=1`、`index.floor=1`、`time.exact=255`。
- 组 payload 已经一次批量读取；接着逐 child 计算相邻时间差，尚未缓存的时间戳仍逐条远程读取。此处是**展开等待**，不是已经展开后的每次滚轮等待。
- 本次只覆盖 256 个普通工具、折叠 child、短输出；没有证明极大组、展开工具详情、Patch/Markdown 渲染都无性能问题。

## 基线实现锚点与约束

- 本节行号对应 `4940c2ad`，不对应修复后的工作区。

- [IndexHistoryRead.kt:53](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/IndexHistoryRead.kt#L53)：未封口 Work 一块一项；封口组只投影结构。
- [AgentHistoryView.kt:338](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt#L338)：边缘实际进入可见区才发需求。
- [AgentHistoryViewModel.kt:613](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L613)：每次只读一个相邻块，保留可见块并淘汰反向远端；新缓冲策略必须同时处理需求触发与保留范围。
- [AgentHistoryViewModel.kt:365](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L365)：展开时批量读取 Work，随后逐条求时间差。
- [RpcCachedIndexVersioned.kt:143](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/rpc/client/RpcCachedIndexVersioned.kt#L143)：exact cache 与远程范围查询不同。
- [Scrollable.kt:47](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/Scrollable.kt#L47)：默认三行输入，仅发出非零消费 interaction，零消费尝试需要独立计数。
- 研究基线的 spec 明确一块一需求、viewport 保护和有界淘汰。当时 checklist 中的 64/128 batch、
  `get` 触发预取、全部 child 保留等段落与实现不一致；实施阶段已统一为访问提示与预取保护的有界分页，
  不能将旧 checklist 当作基线实际行为。

## 追加实验：只删除可见等待

- 用户要求验证“删掉错误条件能否修好”。只在隔离快照删除 `HistoryEdgeDemandEffect` 中的三行 `snapshotFlow(...visibleItemsInfo...).first { visible }`，保留原 `LaunchedEffect`、viewport 上报、`request()` 和全部 ViewModel / LazyColumn 逻辑。
- 同一份探针分别运行原版与实验版，各有三个研究测试：原矩阵、结构读取，以及额外的空闲 / 反向导航观测。两次运行均完成、无框架或加载异常；**探针完成不等于修复验收通过**。
- 每组仍请求 120 行，三轮实际消费如下；+3ms 仍是每次数据 RPC 的人工延迟：

| 场景 | 原版 | 只删等待 |
| --- | --- | --- |
| Index，无注入延迟 | 80 / 80 / 80 | 109 / 102 / 105 |
| Work，无注入延迟 | 40 / 40 / 40 | 64 / 60 / 57 |
| Index，+3ms | 40 / 40 / 40 | 41 / 44 / 44 |
| Work，+3ms | 14 / 16 / 16 | 15 / 15 / 16 |
| 已展开组，两种延迟 | 各轮 120 | 各轮 120 |

- 初始不滚动、等待一秒后，实验版的 Work 窗口从 17 填到 24 项并进入 Ready；原版停在 13 项。这证实删除等待确实接通了提前加载，并非改动完全无效。
- **但滚动后停手仍在读**：+3ms Work 的一次额外观测中，停止输入 1.5 秒，`work.values` 从 32 增至 105，`work.indexes` 从 66 增至 210；窗口仍为 15 项，消费行数仍为 22。反向输入再停手 1.5 秒，`work.values` 又从 140 增至 213。
- 同样的原版观测在每个空闲区间均没有新增数据调用；实验版 Index 的空闲区间也出现 `index.ceil` / `index.floor` 连续增长。因此问题不是仅启动时多读一些，而是中间位置持续触发双向分页。
- 代码解释：可见约束移除后，两侧已组合的标记都可发需求；`loadOlder` 只为反向端保留一个邻接块，`loadNewer` 对另一端同样裁剪。前端预组合要求更多缓冲，加载一端可能重新暴露另一端的需求，形成相互触发。与观测到的窗口大小基本不变、双向查询不断增加一致。
- 本次没有读完整历史导致窗口无限增长，但发生了**有界窗口内的重复读取**；也没有恢复 Work 在 +3ms 条件下的完整滚轮位移。故“只删三行”不能交付，需要让预加载需求与双向保留 / 淘汰策略相容。
- 完整原版 / 实验版证据位于 [condition-check](file:///home/stream/ACodeSpace/push/BuildKodex/out/work-scroll-747/condition-check/)，包含两份 XML、JSONL、环境记录、日志和实验源码。每版 100 条观测，不是 100 个测试。
- 实验版 `AgentHistoryView.kt` SHA-256 为 `98d8e6ccf1686738bcd9b29b22774978ea76c306c221a4f8ebc9fe88e53eb098`，共用探针为 `eccd8a525fd721883bdc90e2c612e834151ce587cb4efb164fa901ad6c68f565`；已核对远端一致。产品工作区未应用这三行删除，未提交或推送。

## 对照 Compose / Paging 的模型偏差

- 对照日期：2026-10-12。外部依据为 Android Developers 的当前 API 文档及 AndroidX `androidx-main` 分支；以下为局部摘要与本地比较，不声称本地使用了该版本的库。
- 本地 History 导入的是自研 `io.github.stream29.kodex.cli.components.LazyColumn`，不是 `androidx.compose.foundation.lazy.LazyColumn`，数据也没有接 `LazyPagingItems`。问题在自研分页协议，不是 Compose Runtime 天然无法处理双向预加载。
- 官方 [LazyPagingItems.get / peek](https://developer.android.com/reference/kotlin/androidx/paging/compose/LazyPagingItems) 区分“返回值并提交访问提示”与“只查看、不触发分页”。[对应源码](https://github.com/androidx/androidx/blob/androidx-main/paging/paging-compose/src/commonMain/kotlin/androidx/paging/compose/LazyPagingItems.kt#L88-L107) 的 `get` 把访问交给 presenter；本地 `HistoryItemWindow.get` 则仅 `ensureLoaded()`，下一块只能经 View 中的边缘标记触发。缺失的是分页访问提示，不是缺少另一个 UI prefetch scheduler。
- 官方 [PagingConfig](https://developer.android.com/reference/androidx/paging/PagingConfig) 把 `pageSize`、`prefetchDistance` 和 `maxSize` 作为相互约束的参数；有限 `maxSize` 至少要容纳 `pageSize + 2 * prefetchDistance`，目的就是避免预取后立即丢弃。[对应回收源码](https://github.com/androidx/androidx/blob/androidx-main/paging/paging-common/src/commonMain/kotlin/androidx/paging/PageFetcherSnapshotState.kt#L293-L321) 只有超出大小预算才考虑回收，并在回收会侵犯预取距离时停止。
- 本地恰好缺少这个保护：每次加载就按“可见块 + 反向一个块”裁剪，没有统一的预取保护范围与容量阈值；LazyColumn 的一屏 overscan / 两屏预取预算也没有参与 History 的回收判断。这是“刚加载又淘汰、停手仍读取”的直接模型偏差。
- 官方分页以内容访问和距边界的距离决定是否继续加载。我们的边缘标记在每次新窗口 / Ready 后重新建立 effect；有可见等待时加载过迟，删掉等待时又会受标记预组合与两端裁剪驱动而反复发请求。标记本身不是普遍禁用的技术，但当前将它作为双向有界分页的唯一控制器，缺少稳定的访问需求依据。
- 单个未封口 Work 被直接当成一次数据加载单位，进一步把一个滚轮跨过几行变成多个串行分页回合。单元素页不是必然错误，但没有填充目标或批次缓冲时会放大本地问题；结构分块的边界不应自动等同于传输分页的最佳粒度。
- 历史核对：`80e4d277e4483267f2e99a2b4bb58d994b64fb94`（2026-08-31，`refactor: snapshot`）把旧 `get -> onOlderDemand(this, index)`、提前 8 项的需求和 64/128 批次改为当前有界结构块 / 可见边缘协议。后续路径迁移不是这些规则的起点。旧版本没有当前这套双向回收问题，但也不能因此整段回退到无限保留历史。
- [2026-09-01 滚动设计记录](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-09-01-align-lazy-column-scroll-behavior.md#L78) 还写了“预取不得触发 History 存储读取”。需要区分“测量不能同步阻塞做 I/O”和“访问提示可以启动异步分页”；禁止后者会断开标准模型中的重要衔接。
- 修正原则：让现有 History 分页所有者依据同一套访问提示与预取保护区处理异步加载和回收；仅回收保护区之外且超过容量预算的页。UI 预组合不直接等价于无条件下一页请求。用户随后授权按此方向实施；没有引入 Android Paging 依赖。

## 实施范围与保留边界

- 滚动问题优先让**现有预加载与有界历史窗口衔接**：在真正触边前准备多个相邻项，保持稳定 key / 可见锚点，并让两端淘汰规则保留足够缓冲。不先改滚轮步长，也不直接放大整个 RPC cache；不能仅移除可见等待而保留相互冲突的裁剪策略。
- 单纯在一次请求内循环读取多个块可能减少发布回合，却仍保留串行 RPC；需用同一输入矩阵证明其实际消费行数改善，不能只看总读取耗时。
- 是否保留边界处未消费输入，应单独确定取消、方向反转、切换 Agent、generation 失效规则，避免将“补滚”变成迟到的跳动。
- 首次展开另评估时间戳的有界批量读取 / 计算；不混入组内滚动修复，也不因一个 256-item fixture 就改变所有 child 生命周期。
- 回归还需涵盖反向滚动、follow-latest、revert/nonce 失效、关闭取消、工具详情展开，以及真实 Native/PTY 的视觉与资源行为。

## 证据与复现

- 本地保留目录：[out/work-scroll-747](file:///home/stream/ACodeSpace/push/BuildKodex/out/work-scroll-747/)。
- 最终 `probe-r9.log`：`BUILD SUCCESSFUL`；XML 为 2 个研究测试、0 failure。包含 54 条结构记录、18 条滚动记录、6 条展开记录，不是 78 个独立测试。
- [最终 JSONL](file:///home/stream/ACodeSpace/push/BuildKodex/out/work-scroll-747/r9/results.jsonl)、[探针源码](file:///home/stream/ACodeSpace/push/BuildKodex/out/work-scroll-747/HistoryScrollProbe.kt#L45)、[探针专用构建配置](file:///home/stream/ACodeSpace/push/BuildKodex/out/work-scroll-747/probe-build.gradle.kts#L1)。
- 探针 / 构建配置的本地和远端 SHA-256 一致：
  - `10cddd1e02b91957e16e7b13c2988af293c6b194fd90e2257ad287428ffe2dc3`
  - `bc5bc82f30a1ed79c05f2e020b840fe719eff520d3b6387d5ec45d32a382f7aa`
- 对 `4940c2ad` 做 `git archive`，仅在隔离快照的 history impl/viewmodel 模块增加上述研究文件 / 构建依赖；重用 Java 25 Gradle daemon，离线、单 worker、不并行，执行：

```sh
./gradlew --offline --console=plain --no-configuration-cache \
  --no-parallel --max-workers=1 \
  -Dorg.gradle.java.home=/Users/stream/Library/Java/JavaVirtualMachines/graalvm-jdk-25/Contents/Home \
  :app-component-history-impl-viewmodel:jvmTest --tests '*historyScrollProbe*'
```

- 保留早期失败日志：r1–r3 是研究构建配置缺依赖；r4 是测试宿主并发 Compose 错误；r5 是误用虚拟超时；r6 是快照泵在空闲超时后退出。修正研究配置后 r7–r9 成功，不把这些失败归因于产品缺陷。
- 关键产品源码的本地 / 远端哈希一致；证据转存后，已删除任务专属远端快照并停止本任务创建的空闲 Gradle / Kotlin daemon。未触碰用户的运行中 CLI。

## 修复实现与最终对照

- 沿用现有 History owner；`get` 提交异步访问提示，`peek` 不触发加载。已移除边缘 marker 的读取 effect。
- 分页、预取与回收使用同一套一级项计数；配置和约束见
  [HistoryPaging](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/HistoryPaging.kt#L5)
  与[设计规则](../../checklist/cli-view-model-state.md#懒history数据源)。
- 只在超过预算后回收保护区外的整页；稳定 child key、viewport 和预取区参与同一判断。
- 未封口 Work 按页批量读取 payload 与时间戳；普通 append 不重读完整页，大批追加不拼接有未读间隙的窗口。
- 没有修改滚轮步长、RPC cache 协议或 WorkGroup 首次展开实现。

首轮及最终复测均使用与“只删可见等待”实验相同的探针、数据、RPC 注入延迟和输入节奏：

| 场景 | 原版，零延迟 | 修复，零延迟 | 原版，RPC +3ms | 修复，RPC +3ms |
| --- | --- | --- | --- | --- |
| Index，三轮消费行数 | 80 / 80 / 80 | 120 / 120 / 120 | 40 / 40 / 40 | 120 / 120 / 120 |
| 未封口 Work，三轮消费行数 | 40 / 40 / 40 | 120 / 120 / 120 | 14 / 16 / 16 | 120 / 120 / 120 |
| 展开 WorkGroup，三轮消费行数 | 120 / 120 / 120 | 120 / 120 / 120 | 120 / 120 / 120 | 120 / 120 / 120 |

- 18 组主矩阵和 3 组边界场景全部消费所请求的 120 行。
- Index 滚动阶段无被观测数据 RPC；Work 滚动阶段为一次 64 项预取，共 10 次数据 RPC，其中 `work.values=1`、`time.values=1`。
- 修复窗口峰值：Index 192、Work 256、WorkGroup 3 个一级项；组内 256 个 child 另计。
- 三种场景各检查初始空闲、向旧端滚动后空闲、反向滚动后空闲，共 9 段：数据 RPC 增量、滚动位移增量均为零，窗口大小稳定。未再出现双向反复读取。
- [首轮原始记录](file:///home/stream/ACodeSpace/push/BuildKodex/out/history-paging-fix-747/probe/results.jsonl)
  与[自动验收结果](file:///home/stream/ACodeSpace/push/BuildKodex/out/history-paging-fix-747/probe/acceptance.json)。
- 最终源码复测结果相同：[最终原始记录](file:///home/stream/ACodeSpace/push/BuildKodex/out/history-paging-fix-747/probe-final/results.jsonl)、
  [最终自动验收](file:///home/stream/ACodeSpace/push/BuildKodex/out/history-paging-fix-747/probe-final/acceptance.json)。
- 最终 History 定向回归：JVM 151 项、macOS ARM64 Native 151 项，均为零失败、零跳过；
  `app-cli` JVM / macOS ARM64 编译通过。覆盖新旧窗口、双向分页与锚点、延迟 payload、空闲收敛、nonce/revert、
  稀疏索引、封口组跨页、大批追加以及 forward 尾页不重读已加载后缀。
- 59 个 History Kotlin 源码/构建文件的本地与远端 SHA-256 一致，探针只在隔离快照添加；
  [验证日志与哈希](file:///home/stream/ACodeSpace/push/BuildKodex/out/history-paging-fix-747/)已保留。
  已清理任务专属远端快照和本任务创建的两个空闲编译进程；共享缓存、锁文件及用户 CLI 未改动。
- 这些结果证明指定 fixture 中的分页位移损失和空闲反复读取已消除；不是跨机器真实网络、RSS 内存或帧率基准。
  更大的初载会增加首次读取量，本次未证明该参数对所有历史/网络条件最优；首次展开 WorkGroup 的时间戳开销仍是独立后续事项。
