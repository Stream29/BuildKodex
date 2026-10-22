# Task Tree

- `Trace cancellable Result helper production consumers`()
- `Compare an inline preserving declaration and implementation shape`()
- `Confirm the minimal migration and cross module validation scope`()
- `Implement the authorized inline preserving split with subagent`()
- `Run centralized metadata consumer and semantics validation`()

# Details

- 用户已说“去吧”，授权实施该最小分离；
  [Session1033](file:///home/stream/.kodex/sessions/1033) 已返回最终编辑回执，未构建或提交推送。
  写入仅 utils/coroutines 本函数及其测试，另允许 MCP backend 新增一个 cliHostTest
  验证公开 inline 的跨模块消费，不改 MCP 生产代码或构建脚本。
  实施前 6 文件快照：`file:///home/stream/ACodeSpace/demo/kodex-detekt-998/cancellable-inline-before/`。
- 主 Session 不修改其他子任务范围，不启动并发构建。
  当前仅处理 runCatchingCancellable 的 1 项发现，不把所有 7 项 inline/contracts 一次认定已解决。
- 当前生产消费者为 MCP backend 的客户端连接、目录读取和操作失败处理。
  函数属于通用协程工具，不含 MCP SDK 或 UI；无需新增 contract/backend 模块或改变平台覆盖。
- 保留语义：成功值进入 Result.success，非取消 Throwable 原样进入 Result.failure；
  CancellationException 原样抛出。保留 inline 和无 noinline/crossinline 的 block，
  允许原有挂起调用及非局部 return，不增加回调间接层或把回调改成普通函数参数调用。
- 已实施最小形状：specMain 新增同签名 public expect inline 声明，commonMain 的原函数
  仅增加 actual，原函数体留原文件；规范 KDoc 随声明归 spec。
  不整段搬机制到 spec、不放宽 lint，也不为预想限制先造 PublishedApi 转发层。
- 标准库有[expect inline / actual inline 的先例](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin/-auto-closeable.html)。
  官方先例没有被当成验收替身；本仓 Kotlin 2.4.20 和 commonMain -> specMain
  已通过下述真实编译及消费者测试。
- 原测试覆盖挂起成功、普通失败对象和取消对象身份；本次已补非局部 return，
  并在真实跨模块消费者中验证挂起/内联用法，而不仅验证同模块测试。
  定向 metadata、JVM 以及代表性非 JVM 编译通过后才能判定这一项完成。
- SafeRw 的 EXACTLY_ONCE contracts、IO use 的不可取消清理、远程异常还原及 RPC CAS
  各自继续单独验证，不能将此无显式 contracts 的小函数结论直接推广。
- 依据：
  - [原公共函数](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/coroutines/src/commonMain/kotlin/io/github/stream29/kodex/utils/coroutines/RunCatchingCancellable.kt#L5)。
  - [已有语义测试](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/coroutines/src/commonTest/kotlin/io/github/stream29/kodex/utils/coroutines/RunCatchingCancellableTest.kt#L10)。
  - [MCP 挂起消费者](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/backend/src/cliHostMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt#L262)。
- 验收前未把派发或编辑交付当作通过，42 项为旧检查点。

## 验收 — 2026-10-21 / Session998

- utils-coroutines metadata 与 JVM/JS/Wasm Node/Linux 各 8 项测试通过，
  包括取消/异常对象身份、挂起成功和非局部 return。
- MCP backend 的真实跨模块新增 2 项随 JVM 41 项通过；
  cliHost metadata、Linux 测试源码和 CLI Native 编译也通过。
- JVM 消费者 javap 确认非局部返回直接生成 ireturn，没有引入助手回调转发；
  原函数体逐字节保留，仅 actual 与规范 KDoc 所属改变。
- 本项 1 条发现实际消除，不代表其余 6 项 inline/contracts 完成。
  合并批次严格/报告 34 项逐条一致，53 个规则样例通过。
- 证据：`file:///home/stream/ACodeSpace/demo/kodex-detekt-998/parallel-validation-summary.json`。
  未提交推送。
