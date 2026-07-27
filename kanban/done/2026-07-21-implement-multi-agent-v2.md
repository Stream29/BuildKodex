# Task Tree

- [done] 实现Multi-agent V2
  - [done] 完成AgentStorage、AgentSession和multi-agent重构规划
    - [done] 调查上游V2工具、状态和会话关系
    - [done] 复核跨thread持久化、消息投递和并发边界
    - [done] 明确AgentState和AgentStorage只表达单Agent语义
    - [done] 将AgentSession定义为递归Agent树的实际节点
    - [done] 删除public Agent address和Agent handle抽象
    - [done] 将节点生命周期绑定到CoroutineScope和Job
    - [done] 明确一个进程独占整个Session，展开的Agent节点继承Session所有权
    - [done] 允许下游直接对spawn返回的原始child执行forkTo
    - [done] 保留无持久化sidecar的轻量SessionEntry列表投影
  - [done] 重构agent-session contract
    - [done] 为AgentState、AgentStorage和AgentSession补充单Agent与多Agent边界KDoc
    - [done] 让AgentSession直接实现MutableCodexAgentStorage并递归列举children
    - [done] 删除CodexAgentHandle、public CodexAgentAddress和基于address的open API
    - [done] 将spawn改为无initial settings并返回原始child AgentSession
    - [done] 删除AgentSession的AutoCloseable和suspend release生命周期API
    - [done] 将repository open/create/fork绑定到调用方拥有的CoroutineScope和Job
  - [done] 重构in-memory和filesystem AgentSession
    - [done] 将filesystem path、AgentPath和child ordinal locator收进实现边界
    - [done] 让root open取得并持有整个Session的独占lease，tree expand不再取得独立lease
    - [done] 让root Job递归拥有child Job、Session lease renewal和cleanup
    - [done] 在root owner coroutine的NonCancellable finally中释放Session lease
    - [done] 让repository topology lease保护顶层结构操作，让Session lease保护spawn和child ordinal分配
    - [done] 让spawn只保证原始节点创建并继承Session ownership，不验证AgentState合法性
    - [done] 保持repository list直接投影root settings和timestamp且不取得Session lease
  - [done] 重构coordinator和CLI调用链
    - [done] 用AgentSession对象关系替代address到handle的二次打开
    - [done] 用coordinator持有的对象关系表达UI、tool和callback ownership
    - [done] 让tree expand在现有Session ownership下直接装载节点
    - [done] 让turn Job只控制单次turn，不释放Session ownership
    - [done] 让Session close通过取消并join root Job递归完成cleanup
  - [done] 建立会话级multi-agent coordinator
    - [done] 实现节点状态、并发额度和运行期状态发布
    - [done] 实现spawn、send、follow-up、wait、interrupt和list语义
    - [done] 首版不建立跨重启coordination journal
  - [done] 实现六个Multi-agent V2工具
  - [done] 将Agent树和thread切换接入CLI
  - [done] 覆盖storage、lifecycle、消息、等待、中断和持久化测试
  - [done] 在真实PTY中验证多Agent链路

# Details

- 状态：已完成并归档。
- AgentSession V2、会话级coordinator、六个工具和CLI Agent树均已接入。
- Linux与macOS Native测试通过；真实PTY已验证spawn、wait、树切换、流式输出、中断、冷恢复、mode持久化和干净退出。
- 现有Session filesystem布局和直接SessionEntry投影来自[Session surface任务](2026-07-22-redesign-session-surface.md)。

## 规划后的抽象边界

```text
CodexAgentState = 一个Agent的运行状态
CodexAgentStorage = 一个Agent的五条持久化timeline
CodexAgentSession = AgentStorage + recursive children<AgentSession>
CodexSessionEntry = 未打开Session root的轻量列表投影
```

- `CodexAgentState`和`CodexAgentStorage`都只表达单Agent语义，不包含parent、children、spawn或multi-agent生命周期。
- `CodexAgentSession`是Agent树的实际节点；repository打开的Session是这棵树的root节点。
- contract KDoc必须明确上述单Agent与多Agent边界，避免把树拓扑重新放入AgentState或AgentStorage。
- `CodexAgentSession`直接暴露`MutableCodexAgentStorage`，下游可以使用原始timeline API。
- public contract不保留`CodexAgentHandle`或`CodexAgentAddress`；filesystem path、ordinal和lease slot都是实现细节。
- `CodexSessionIndex`只作为`CodexSessionEntry`及repository open/delete的root槽位key保留。
- storage-backed identity统一取自`CodexAgentStorage.id`；coordinator以当前Agent对象关系和ownership判断UI、tool与异步callback是否仍然有效。

目标contract形状：

```kotlin
interface CodexAgentSession : MutableCodexAgentStorage {
    suspend fun children(): List<CodexAgentSession>
    suspend fun spawn(): CodexAgentSession
}

interface CodexSessionRepository {
    suspend fun list(): List<CodexSessionEntry>

    suspend fun <R> open(
        sessionIndex: CodexSessionIndex,
        run: suspend CoroutineScope.(CodexAgentSession) -> R,
    ): R
}
```

- `create`和root `fork`返回的root同样只在调用方拥有的scoped owner block内有效。
- repository打开root时取得整个Session的独占ownership；`children()`只在该ownership下装载并缓存direct child，不建立per-node lease。
- `spawn()`在现有Session ownership下创建并返回新child，不接收initial settings，也不承诺该storage已经能构造合法AgentState。
- 下游可以直接执行`source.forkTo(until, child)`；失败、取消或进程崩溃后的AgentState合法性不由AgentSession层保证。
- Job结束只负责停止节点工作并释放资源，不把取消解释为storage transaction rollback或自动删除child。

## 生命周期与lease

- repository scoped open取得Session独占lease，coordinator创建并持有对应root owner Job；AgentSession本身不暴露`release()`。
- `/close`取消并join root Job；parent取消递归传播到已展开和spawn的child Job。
- Session lease renewal属于root owner Job，不属于repository全局scope或任一child Job。
- root owner coroutine在`finally`中使用`NonCancellable`完成runtime cleanup和Session lease release。
- root Job完成必须表示其全部child cleanup和Session lease release已经结束。
- turn Job只控制一次模型/tool turn；取消turn不关闭AgentSession，也不释放Session lease。
- repository topology lease只保护create、root fork、delete、迁移和顶层Session ordinal分配；Session内spawn和child ordinal分配由Session独占ownership保护。
- 同一Session不得被不同进程分别占有部分节点；发现这种情况必须视为严重的所有权协议错误。

## Session列表

- `CodexSessionRepository.list(): List<CodexSessionEntry>`保持为无需打开Session的轻量API。
- `CodexSessionEntry`继续只包含`sessionIndex`、root name和nullable timestamp。
- filesystem list直接读取root latest settings和timestamp，不读取history、compaction、token count或subagents，也不取得Session lease。
- 首版不建立per-Session sidecar、全局index、dirty marker或额外持久化identity。
- 只有实际测得列表投影成为性能瓶颈后，才单独规划可重建缓存。

## Multi-agent边界

- 六个工具为`spawn_agent`、`send_message`、`followup_task`、`wait_agent`、`interrupt_agent`和`list_agents`。
- 每个Agent节点拥有独立storage、state、runtime和turn lifecycle。
- `send_message`只投递消息；`followup_task`按节点运行状态启动turn或排队投递。
- `wait_agent`等待活动通知；`interrupt_agent`只取消当前turn，不销毁AgentSession。
- 首版恢复以filesystem Agent树和各节点timeline为事实来源；现有`CodexAgentStateValue`恢复逻辑决定冷恢复后的合法操作，不持久化role、fork来源或per-Agent lifecycle metadata。
- `Running`等运行期状态只由当前进程的runtime/coordinator发布；冷恢复时按持久化Agent状态投影工具所需状态，这不是实现阻塞点。
- 未投递消息、follow-up等待和exactly-once完成通知首版不保证跨重启；未来确有需求时使用独立coordination journal。
