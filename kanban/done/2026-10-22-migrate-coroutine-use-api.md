# Task Tree

- `Trace resource scope consumers and cleanup semantics`()
- `Define the minimal inline preserving source split`()
- `Confirm the migration and cleanup acceptance scope with the user`()
- `Add the spec declaration and preserve the actual implementation`()
- `Add cross module cleanup and non local return regression tests`()
- `Run serial platform tests and compare lint findings`()
- `Record acceptance and remaining gaps`()

# Details

- 用户以“那补”授权补齐 CoroutineCloseable.use 的 1 项发现；
  不改 SafeRw、接口、目标覆盖或资源关闭策略。父 Session 直接实施并串行验收。
  修改前1523份已有源码的哈希及模块快照位于
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/coroutine-use-before/`。
  其中仅修改2份生产文件，其余1521份未变，另新增1份消费测试文件。
- use 是通用协程资源作用域助手，不是前端/后端业务装配。
  当前生产 Node 文件系统的 useSource/useSink 使用它，进程流消费者也有跨模块测试。
  类型边界为非空 T : CoroutineCloseable，close 是 suspend。
- 当前行为必须保留：block 正常返回、抛错或非局部 return 均执行 finally，
  在 NonCancellable 中尝试一次 close，允许关闭挂起。
  只有 close 失败时传播该错误；操作和关闭都失败时保留原操作错误并追加 suppressed。
  操作取消也按原 Throwable 路径保留，关闭失败不能覆盖它。
  不承诺关闭一定成功，不新增超时，也不把此 helper 改为关闭所属文件系统/进程。
- 已实现并验收的最小形状：在现有 specMain/CoroutineRawIoHelpers.kt 加
  public expect suspend inline fun <T : CoroutineCloseable, R> T.use(block: suspend (T) -> R): R；
  commonMain 原函数仅加 actual，保留 try/catch/finally/NonCancellable 函数体。
  不加 noinline/crossinline、PublishedApi 转发、接口或模块，也不移动整段清理机制到 spec。
  本函数原来没有显式 callsInPlace contract，不凭重试/清理需要添加新契约。
- system 模块已有取消后关闭、双失败 suppressed、单独关闭失败的三项跨模块测试。
  原断言保留，新回归复用其测试接线，不为 base 模块新建测试框架。
- 依据：
  - [当前 use 实现](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineRawIoHelpers.kt#L7)。
  - [可挂起关闭接口](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/src/specMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineRawIo.kt#L8)。
  - [Node 实际使用](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/system/src/jsMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/NodeCoroutineFileSystem.kt#L181)。
  - [既有关闭语义测试](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/system/src/commonTest/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineFileSystemTest.kt#L157)。

## 验收 — 2026-10-22 / Session998

- 实现文件逐字节比对仅插入 actual，
  CoroutineCloseable 接口、copyTo、原有测试、构建脚本及目标均未改。
- base/system metadata、Wasm 编译通过；CLI JVM/Linux 及集成测试源码编译通过。
  system 的 JVM/Node/Linux 各14项，进程流消费者 JVM 9项，共51项全通过；
  包含新增挂起返回、非局部返回清理和取消叠加关闭失败的三项跨模块回归。
  集成测试仅编译源码，未运行。
- 首轮新增断言未考虑 JVM 栈恢复复制异常，已根据本机 kotlinx-coroutines 1.11.0
  源码修正：关闭异常兼容原实例或以原实例为 cause 的恢复副本，
  仍验证单一 suppressed 的类型/消息及操作取消异常 assertSame。算法未改，重跑全通过。
- 严格/报告实际30→29项/7模块，逐条等于旧报告移除 use；没有新增发现，
  严格仍因其余29项正确失败。53个规则样例通过。
  剩余为18项窄平台及关联API、1项SafeRw contracts、10项CLI/kRPC桥。
- 两层索引均未暂存，未提交推送。
  复用用户已授权的 JDK25 daemon、单worker；无临时源码/worktree或遗留验证worker，
  不停止用户进程，不修改全局配置。未运行真实模型或 Windows/macOS 运行测试。
- 命令、结果、XML与精确范围保护：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/coroutine-use-validation-summary.json`。
