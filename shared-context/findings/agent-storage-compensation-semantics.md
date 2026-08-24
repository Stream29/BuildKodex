# AgentStorage补偿语义

## 一致性边界

- `MutableKodexAgentStorage`只允许一个writer，调用方负责串行化写入。
- 无缓存的filesystem storage只是文件行为包装，不持有目录所有权。它的外层缓存decorator通过根目录的`lock.json`租约实施进程级独占；租约owner变化后该视图立即失效，不得继续访问storage。
- compound write不提供跨timeline的snapshot isolation；直接reader可能观察到执行中的合法前缀。
- 每个compound write必须按顺序设计，使任意已持久化前缀都能重新推导为合法AgentState。
- `KodexAgentState.latestIndex`只在compound write完整成功后发布。进程重启时则从已经持久化的合法前缀恢复。
- canonical storage初始化在index 0写入`tokenCount = 0L`；每个成功compaction checkpoint也在其边界写入synthetic `0`。零值是普通数值，不表示缺失或未知；compaction reset有意不声称它是provider报告值。

## 操作级补偿

- 删除`MutableIndexVersioned.transaction`与`MutableKodexAgentStorage.transaction`。
- 增加`MutableIndexVersioned.setWithTransaction(...) { ... }`。它先执行并持久化自己的`set`，后续block失败或取消时从原始tail边界回滚。
- 增加`MutableIndexVersioned.revertWithTransaction(...) { ... }`。它在`revert`前保存被移除的稀疏后缀；后续block失败或取消时先移除block新增的后缀，再按index升序恢复原后缀。
- compound write通过嵌套上述两个操作形成调用栈。异常向外传播时，Kotlin调用栈自然按LIFO顺序执行补偿，不引入Saga库。
- 补偿在`NonCancellable`中执行。补偿失败附加到原始failure；对应storage不得继续写入，必须重新打开或显式恢复。
- 该语义依赖单writer，否则按tail执行的补偿可能删除其他writer的数据。

## 崩溃语义

- 普通异常与协程取消恢复到compound write开始前。
- 进程崩溃允许保留已经持久化的操作前缀，不保证整组操作全部提交或全部撤销。
- 文件后端不需要为compound write维护durable begin、commit或补偿栈。
- 六个timeline目录及其数字编号记录文件是真源；每条timeline另存可重建的`latest.json`加速精确tail读取。合法指针必须是`-1`或指向存在的数字记录；缺失、损坏和悬空指针回退到目录扫描。
- append在发布数字记录前先原子替换`latest.json`，revert在移走数字后缀后再替换，使进程中断只留下合法或可O(1)识别的悬空指针。缺失指针可以用exclusive create安装；覆盖修复由取得租约并完成全量扫描的owner执行。
- 外层缓存decorator取得租约后只扫描一次timeline目录，对账`latest.json`，并在内存中增量维护完整的有序稀疏索引；LRU只缓存按真实stored index解码的值。无缓存的filesystem storage不保留这些内存状态。
- 单个timeline记录通过临时文件发布；打开storage时忽略并清理未完成的临时文件。
- `forkTo`只操作已经存在的目标storage，不负责创建、发布或修改目标identity。调用方必须独占目标，并决定迁移期间的可见性与失败处理。

## Session租约所有权

- Catalog读取根Session metadata时先直接验证`settings`与`timestamp`的`latest.json`；两个指针都有效时不取得租约，也不枚举timeline。
- 指针失效且根租约正被其他owner持有时，只扫描失效timeline的数字记录，不修改`latest.json`；取得根租约则重新验证指针，只对仍失效的timeline扫描并修复。
- Catalog恢复租约是Session Catalog ViewModel scope的后代；正常返回时显式`closeAndJoin`，ViewModel销毁负责取消仍在执行的恢复。
- 根Session、runtime、subagents与根租约是Session ViewModel scope的后代。显式关闭Session时取消并等待整棵资源树释放；切换Session tab不销毁ViewModel。

## 验证要求

- 覆盖每种compound write在每个操作边界失败或取消后的逆序补偿。
- 覆盖`revertWithTransaction`恢复稀疏后缀以及删除block新增后缀的行为。
- 覆盖补偿失败后storage拒绝继续写入。
- 覆盖文件尾部截断、任意合法操作前缀和跨内存/文件后端的双向`forkTo`。
- 并发reader只要求不崩溃且观察到结构合法的前缀，不要求只能看到完整旧状态或完整新状态。
