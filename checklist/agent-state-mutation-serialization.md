# AgentState 写入串行化

- 每个live `KodexAgentState`持有一个私有、公平的`Mutex`，串行化操作准入、全部storage写入和最终状态恢复；远程请求不因此自动成为覆盖全生命周期的临界区。
- `modify`、`compact`、history/turn/settings写入、工具完成和plan更新都必须先取得同一Mutex，再读取`state`、校验前置条件并捕获pending/settings快照。`requestResponseApi()`在请求准入、每次结果落盘和最终恢复时使用同一Mutex。
- `requestResponseApi()`开始collect时在Mutex内校验稳定状态、捕获固定storage snapshot并发布`RequestResponse.Started`，随后在网络和流式读取期间释放锁。每个`OutputItemDone`及`Completed`持久化步骤重新取得锁并确认state仍为`RequestResponse`；成功、失败或取消后在`NonCancellable`中持锁从实际storage恢复`latestIndex`与稳定state。
- `RequestResponse`是请求全生命周期的逻辑所有权。冲突的会话原子操作取得短临界区后根据该in-flight state以invalid-transition失败，不再通过占用Mutex等待请求结束；`updateSettings`可以在请求期间持锁提交且不得改变`RequestResponse`，当前请求继续使用开始时的settings snapshot。
- 等待短临界区Mutex的协程可取消；取消后不得执行写入。持锁写入无论成功、失败或取消，都必须保持storage事务语义，并在所属操作结束时恢复可观测索引与state。
- 状态转移在锁内不再以CAS作为并发准入机制；`ExternalWrite`、`RequestResponse`和`Compacting`继续只表达当前操作阶段。
- `compact`当前仍持有同一Mutex覆盖远程请求与checkpoint提交；缩短其锁生命周期需要先独立解决完成时的settings快照语义。
- 标题更新必须在取得AgentState写入准入后读取最新settings，并且只patch `threadName`。自动标题同时在该临界区校验预期旧标题；不匹配时不写入，不能用陈旧的完整settings快照覆盖plan或其他设置。
- `modify`的block保持独占且不得重入任何AgentState原子操作；`Mutex`不是可重入锁。
