# AgentState 写入串行化

- 每个live `KodexAgentState`持有一个私有、公平的`Mutex`，串行化全部可能改变AgentState或storage的操作。
- `modify`、`requestResponseApi()`、`compact`、history/turn/settings写入、工具完成和plan更新都必须先取得同一Mutex，再读取`state`、校验前置条件并捕获pending/settings快照。
- `requestResponseApi()`在其Flow开始collect时取得Mutex，并持有至流式请求结束、取消或异常恢复；Responses item落盘与状态发布不得绕过该锁。compaction同样持锁覆盖远程请求与checkpoint提交。
- 任何排队操作仅在获得锁后按当时状态决定是否合法。它不应因观察到前一操作的短暂state而失败；若前一操作完成后语义前置条件仍不满足，才以invalid-transition错误失败。
- 等待锁的协程可取消；取消后不得执行写入。持锁操作无论成功、失败或取消，都必须从实际storage恢复并发布`latestIndex`与state后释放锁。
- 状态转移在锁内不再以CAS作为并发准入机制；`ExternalWrite`、`RequestResponse`和`Compacting`继续只表达当前操作阶段。
- 标题更新必须在取得AgentState写入准入后读取最新settings，并且只patch `threadName`。自动标题同时在该临界区校验预期旧标题；不匹配时不写入，不能用陈旧的完整settings快照覆盖plan或其他设置。
- `modify`的block保持独占且不得重入任何AgentState原子操作；`Mutex`不是可重入锁。
