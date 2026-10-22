# Task Tree

- `Freeze preserved worktree and excluded declaration shapes`()
- `coroutineScope {`
  - `val views = async {`
    - `Migrate component Views with Session1013`()
  - `}`
  - `val providers = async {`
    - `Migrate utils and tool APIs with Session1014`()
  - `}`
  - `val agents = async {`
    - `Migrate Agent and OpenAI APIs with Session1015`()
  - `}`
  - `val rpc = async {`
    - `Migrate RPC and application APIs with Session1016`()
  - `}`
  - `Migrate MCP shared declarations in parent Session`()
  - `awaitAll(views, providers, agents, rpc)`
- `}`
- `Review implementation preservation and scope boundaries`()
- `Compile metadata platform consumers and tests serially`()
- `Run strict lint and record deferred declarations`()
- `Release temporary resources and complete the accepted subset`()

# Details

- 上游：[导出 API 迁移](2026-10-20-audit-exported-api-spec-boundaries.md)。
  用户明确先修无争议部分；窄平台拆分仍暂缓，不扩目标或新建平台模块。
- 派发时产品 HEAD `d212c43cf`，保留 157 份脏工作树文件；
  快照在 `~/ACodeSpace/demo/kodex-detekt-998/round2-before/`，只供复核，不创建提交。
  起点为 367 个结构发现/93 模块，仍需真实编译而非只按文件/符号计数验收。
- 唯一编辑所有权：
  - [1013](file:///home/stream/.kodex/sessions/1013)：`app/component/*/view/**`。
  - [1014](file:///home/stream/.kodex/sessions/1014)：`utils/**`、`tool/**`。
  - [1015](file:///home/stream/.kodex/sessions/1015)：`agent-context/**`、`agent-runtime/**`、
    `agent-session/**`、`agent-state/**`、`agent-storage/**`、`openai/**`。
  - [1016](file:///home/stream/.kodex/sessions/1016)：`rpc/**`（排除 krpc-utils-patch）、
    `app/application/**`、`app/session/**`、`app/backend-host/**`、
    `app/backend-daemon/**`、`app/settings/filesystem/**`、`app/test-support/rpc/**`、`hook/**`。
  - 父 Session：`mcp/**` 共享声明；必要时处理尚未分工的普通 `app/view` 导出，
    负责 kanban、范围复核和集中验证。
- 暂缓：需要项目/目标变更的窄平台导出，inline/reified/contracts 的特殊语义，
  CLI 启动入口、krpc-utils-patch 兼容桥接、两个已复现的旧录制测试失败。
  不凭暂无调用者隐藏旧 public，不新增 stub、无意义接口或 lint baseline/suppress。
- 复用[已验证声明形状](../../checklist/kmp-spec-impl-source-sets.md#declaration-shape-checks)：
  defaults 留 expect、构造属性分开、私有状态不进 spec、继承成员 metadata 校验、
  getter 注解目标和 const 语义保持；不把机制整体迁到 spec。
- 子任务只编辑、检索和静态检查；不执行 Gradle/编译器/测试，不操作用户 IDE/进程，
  不暂存/提交/推送或再次派生 agent。父 Session 等代码稳定后单 worker 串行验证，
  复用空闲 JDK25 daemon，资源压力下仅取消自己的调用。
- 验收需包含所有变更模块的 spec/common metadata、JVM 与适用消费者编译和测试，
  代表性 Native/JS、严格/报告 lint 一致性及无变化跳过；不执行真实模型集成测试。

## 完成范围

- 四个子任务已交付并由父 Session 验收；本轮处理 86 个模块：
  26 个组件 View、22 个 utils/tool、22 个 Agent/OpenAI、14 个 RPC/应用模块，
  以及 MCP 和根 View。此数包含部分声明仍暂缓的模块，不表示每个模块均已清零。
- 分离公开工厂、扩展、类及其成员，签名/defaults 所需依赖进入 spec；
  状态、IO、连接、编解码、UI 和算法机制保留在实现层。
  新增四个编译期常量回归用例，未改变已有常量值或业务算法。
- 父 Session 修正 `WebSocketRpcTransport.alive` 契约的 `val`/`var` 不匹配，
  保留原有 `var` 和私有 setter；没有修改运行逻辑。
- 按实际模块边界复核派发快照：本轮 86 模块以外的 147 份既有文件字节不变。
  根配置、convention、lint、版本、模块和目标集合未改；未提交推送。

## 验证结果

- 全部 86 个模块的 spec/common metadata 与 JVM 主代码编译通过。
  CLI 和 app/rpc JVM 消费者、integration-test 测试源码编译通过；
  未运行真实模型集成测试。
- Linux X64 CLI klib 编译通过，覆盖实际 UI/后端依赖图；
  Agent composition、MCP、ImageGeneration 的 JS 及 utils-coroutines 的 Wasm 编译通过。
  这些不是最终 CLI 链接或各平台运行验收。
- 定向 JVM 测试按各模块最新结果汇总：1778 项，1773 通过、5 失败、0 错误/跳过。
  26 个变更组件 View 的 425 项、Agent/OpenAI 的 510 项、MCP 的 43 项均通过。
  根 app/view 的两个已知旧录制失败未混入这个定向计数。
- 五项失败均在干净 `d212c43cf` 导出源码上复现：
  app/application 四项仍断言旧版本 `0.4.15`，实际为 `0.5.2`；
  app/backend-host 一项重连后 ViewModel 为 null 的断言失败。
  通知 Hook 测试还在两处把 KodexAgentSettings 传给要求 Path 的 createSession，
  基线与当前测试编译诊断一致，故该测试组未能执行，不计作通过。
  这些测试源码未改，没有调整断言、超时或排除用例。
- 文件租约首次有一项一分钟等待超时；基线和未改代码的重跑均通过，
  重跑完整 54 项无失败。原因尚未确定，不把这次超时断言为已证实的旧问题。
- 53 个规则样例强制重跑通过。报告/严格 lint 的 46 项指纹一致，
  严格任务实际退出 1，未使用 baseline、suppress 或放宽规则。
- 单 worker、复用同一空闲 JDK25 daemon；本轮验证未发生资源阈值取消或新 daemon 启动。
  验证日志在 `~/ACodeSpace/demo/kodex-detekt-998/round2-*.{log,json}`，
  汇总为 `round2-validation-summary.json`；
  `round2-baseline-comparison.json` 保留基线对照。

## 剩余讨论与专项验证

- 结构发现从 367/93 模块降为 46/17 模块；消除 321 项，不等于完成全仓语义审计。
  完整声明位置见 `round2-remaining.json` 和产品 SARIF。

| 类别 | 结构发现 | 保留原因 |
| --- | ---: | --- |
| 窄平台及其签名/defaults 依赖 | 27 | 仍暂缓平台拆分；含 MCP、迁移、codec 后端、三个宿主工厂及受其牵连的 RPC/View |
| inline/contracts | 7 | 保留非局部返回、callsInPlace、异常及取消语义，需专门确定契约形状 |
| CLI 入口与 kRPC 兼容桥 | 10 | 启动入口和框架符号兼容需单独验证，不机械搬迁 |
| Sidebar 两个 const | 2 | 单独移到新文件会改变 JVM 字段承载类，未引入未经验证的 multifile façade |

- 无变化 lint 为 UP-TO-DATE，端到端 0.523 s；两次配置缓存命中的强制全扫
  分别为 1.352/1.318 s，分析为 0.823/0.708 s。
  首次强制扫描需重建配置缓存，端到端 4.780 s；这些是本机回归样本，不是通用性能保证。
- 临时基线导出已清理；保留原始证据，不停止共享 IDE 或普通 Gradle daemon。
  本卡仅完成已接受的无争议子集；上游任务继续跟踪上述缺口及测试验收限制。
