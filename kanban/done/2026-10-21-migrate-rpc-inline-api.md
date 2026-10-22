# Task Tree

- `Confirm the four inline APIs and semantics preserving scope`()
- `Apply spec actual separation with subagent`()
- `Resolve a reusable Gradle JVM and run serial validation`()
- `Record exact lint reduction and remaining gaps`()

# Details

- 用户选择先迁 4 项：[Session1035](file:///home/stream/.kodex/sessions/1035) 已接受实施，
  已返回最终编辑回执，未运行构建或提交推送；父 Session 已完成下述验收。
- 范围：rpc/client 的 update、getAndUpdate、updateAndGet，utils/rpc-exception 的
  restoreRemoteException；不触及其余 30 项或 SafeRw/use。
  两模块内只做 spec 同签名 expect inline 与原函数 actual inline 分离，原函数体保留。
  CAS 三项同时保留 suspend。规范 KDoc 归 spec，不把机制搬 spec 或加转发层。
- 保留 CAS 最新投影重读、可重复计算、只在 false 比较后 50ms 可取消重试、
  成功不改本地值和正确返回值；异常原样逃出。@PublishedApi internal 延迟常量留实现层。
- 保留远程闭合异常层级/wire、取消绕过解码、坏 JSON/未知类型原异常身份、
  挂起与非局部 return，以及只包围实际 RPC 操作而非后续 Flow collection 的语义。
- 子任务只写两个模块及 app/rpc 新增的 RpcInlineApiConsumerTest；
  不改构建脚本/目标/其他 API。原有测试断言保留，补真实跨模块内联消费回归。
- 保护：`file:///home/stream/ACodeSpace/demo/kodex-detekt-998/rpc-inline-before/`，
  含 1521 文件哈希及两个模块快照。前序迁移与两层索引保持。
- 验收需覆盖两模块 metadata、JVM 原测试、app-rpc 消费者、代表性 Native 编译，
  并确认严格/报告只删除指定 4 项；未把派发时预期当成结果。
- 环境：派发时无正在运行的 Gradle daemon；用户已明确允许初始化一个固定 JDK 25
  daemon，以单 worker 串行验收。初始化后所有命令复用该 JVM，保留内存安全检查，
  不停止用户进程或改全局设置。不能把前序扫描或标准库先例当本轮通过。
- 依据：
  - [CAS 实现](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/client/src/commonMain/kotlin/io/github/stream29/kodex/rpc/client/SuspendMutableStateFlow.kt#L25)。
  - [CAS 语义测试](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/client/src/commonTest/kotlin/io/github/stream29/kodex/rpc/client/SuspendMutableStateFlowTest.kt#L64)。
  - [远程异常实现](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/rpc-exception/src/commonMain/kotlin/io/github/stream29/kodex/utils/rpcexception/RestoreRemoteException.kt#L6)。
  - [已有异常测试](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/rpc-exception/src/commonTest/kotlin/io/github/stream29/kodex/utils/rpcexception/RestoreRemoteExceptionTest.kt#L1)。

## 验收 — 2026-10-21 / Session998

- 四项均已按同签名 expect inline / actual inline 分离，CAS 保留 suspend；
  原实现只变 actual 和 KDoc 归属，规范化比对确认代码不变。
  原模块测试、接口、构建脚本、50ms 常量及异常 wire 均未改。
- 两模块及 app-rpc metadata 通过。JVM：rpc-client 151、rpc-exception 18、
  app-rpc 111，共 280 项全通过，含新增真实跨模块四 API 挂起与非局部返回消费。
- Linux：两模块各 151/18 项共 169 项全通过；
  app-rpc Native 测试源码、CLI JVM/Linux 和集成测试源码编译通过。
  未运行真实模型集成测试或非 Linux 平台运行测试。
- 首轮消费测试的三个私有 helper 缺少末尾返回，已补 -1 兜底并以断言确认
  实际从 inline lambda 非局部返回；没有给 CAS 重试函数添加错误的 EXACTLY_ONCE 契约。
  另明确 spec update 的 Unit 类型，不改变签名语义。
- 严格/报告实际 34→30 项/8 模块，剩余项逐条等于旧报告移除这 4 项；
  严格仍正确失败，53 个规则样例通过。剩余 18 窄平台、2 inline/contracts、10 桥接入口。
- 1467 份范围外文件哈希不变，未提交推送。初始化只按用户授权创建 JDK25 daemon，
  等待期间出现的用户 JDK26/2GB daemon 保持原状；后续验收均复用 JDK25、单 worker，
  未改全局配置或停止用户进程。可复用 daemon 留待后续迁移，当前无自有验证 worker。
- 完整命令、测试 XML、初次失败及源码保护：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/rpc-inline-validation-summary.json`。
