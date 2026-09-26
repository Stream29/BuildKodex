# RPC timeline 与缓存

- 按 [RPC 迁移适用范围](rpc-architecture.md#适用范围)使用本文件；状态初始化沿用 [Get/GetFlow](rpc-state.md#get-与-getflow)。

## 读取边界

- 按后端各条 CachedIndexVersioned 暴露 IndexVersioned 读取能力，不传缓存对象或完整 HistoryIndexWindow；前端可写性按下述 settings 特例区分。
- 六条 timeline 分别为 index、work、settings、timestamp、tokenCount、unstable；复用 CleanIndexEntry、StableWorkEvent、KodexAgentSettings、Instant、TokenCountSnapshot、List<UnstableCleanEvent>。
- tokenCount 直接传原结构化快照，保留 usage/diagnostics 的缺失与零值区别；只展示总量的前端消费者提取 totalTokens，不将整个 timeline 降为 Long。原文件迁移与记录语义见 [Token-count Timeline](token-count-timeline.md)。
- 用普通 `TimelineRpc<T>` 统一接口，六条具体服务显式重声明全部方法；以 `sessionIndex` 定位 persisted root Agent。
- 读取与元数据订阅遵循[Session 激活准入](rpc-session.md#保活与回收)；失活不是缓存标记失配或记录不存在，使用 SessionNotActive，重新激活后另行核对 cacheNonce。
- 每条分别提供 getCacheNonce/getCacheNonceFlow 与 getLatestIndex/getLatestIndexFlow；Get 用于初始化，Flow 包含当前值与后续变化。尾索引属于该 timeline，空值为 -1，不混用 Agent 全局尾索引。
- Agent 发布的全局可见位置使用 [AgentRuntimeRpc 全局存储位置](rpc-agent-runtime.md#全局存储位置)，不改变本文件各条尾索引的含义。
- 保留六种带 cacheNonce: Long 的数据查询：get、getExact、floorToIndex、ceilToIndex、indexesIn、valuesIn。
- get 读取该位置可见值，无前驱值时失败；getExact/floor/ceil 的 null 只表示对应记录或边界不存在。
- 区间为 inclusive，indexesIn/valuesIn 按实际存储 index 升序返回；空区间返回空列表。
- cacheNonce 不匹配必须失败，不伪装成 null 或空列表；结果归属于请求绑定的 owner 与缓存标记。
- 不匹配使用无字段 CacheNonceMismatch，通过[异常恢复 helper](rpc-state.md#异常还原-helper)识别，不把新 cacheNonce 放进异常替代订阅。
- 元数据 Flow 正常发布 cacheNonce 变化，不因此失败；查询失败后前端沿已有 cacheNonce Flow 更新缓存，再重发仍需要的读取，不忙重试同一旧标记。
- 普通 TimelineRpc 保持只读；只有 SettingsTimelineRpc 增加 settings CAS，其他五条不向前端开放 CAS、set 或 revert，写入继续通过后端业务命令。
- settings 的读取与 CAS 统一归入 settings timeline，不将所有 timeline 变成可写，也不因一个特例新增通用可写父接口。
- SettingsTimelineRpc.compareAndSet(sessionIndex, expect, update): Boolean 比较当前完整 KodexAgentSettings 并经业务写入边界提交，不接收历史 index，不原地覆盖历史记录；保留原设置合法性和运行字段约束。
- settings CAS 不增加 cacheNonce 参数，仍按当前值相等性判断冲突；实际追加只推进该 timeline 的 latestIndex，不因每次设置修改使已有 exact 缓存全部失效。
- 比较和写入共享后端原子边界；不匹配不写并返回 false，同值成功不产生变化，失活/校验/持久化失败不伪装成 false，不续期 TTL。重试沿[settings 字段冲突规则](rpc-state.md#settings-的字段冲突)，成功不保证前端投影同步。
- unstable 的空列表是合法已存值，不等于 getExact 返回 null。

## 缓存标记与前端缓存

- 后端各条 cacheNonce 与 latestIndex 的可观察元数据由现有 Session-owned CachedIndexVersionedImpl 持有并发布；六条 timeline 复用该实现，RPC 服务不另建索引或标记真源。
- 两项元数据只读暴露为 StateFlow；沿原缓存追加/回退的成功提交路径发布，不修改原文件提交与补偿算法。无后缀可删除的回退不更换标记。
- 后端模块间通过只读观察接口访问缓存元数据，保持具体缓存与文件 Session 实现内部可见；不为此公开具体类型或把 repository.open 改为返回具体实现。
- 在 agent-storage/contract 使用 `CachedIndexVersioned<T> : IndexVersioned<T>` 暴露两项只读元数据，`ObservableKodexAgentStorage` 将原六条属性收窄为该接口；原内部实现命名为 CachedIndexVersionedImpl。
- 前后端分别实现 CachedIndexVersioned，接口统一读取与观察能力，不统一缓存机制，也不为两侧提取公共实现基类；前端不实例化后端的文件缓存实现。
- 后端实现直接参与本地存储操作并产生 cacheNonce；前端实现通过 RPC 读取、跟随后端元数据并管理自身缓存，不生成后端标记，不把前端视图重建视为后端 nonce 变化。
- 原后端缓存同时实现既有可写接口与只读视图，共用同一对象与状态；前端仅实现只读视图，不继承 set/revert。无需增加 MutableObservable 层级、修改原 Session.storage 或把缓存对象作为 RPC 传输值。
- 元数据属于现有缓存 owner，不把内存标记加入磁盘存储协议，也不从 Agent 全局 latestIndex 冒充各条尾索引；普通 IndexVersioned 读取接口不因这项归属决定自动扩展。
- cacheNonce 是标识对应 timeline 缓存有效性的内存随机 Long，不是有序业务版本；只比较相等，不推导顺序或差值。不使用递增 generation、全局计数器、已卸载 owner 的代际表或持久化。
- 创建/重建后端 timeline owner，以及回退等破坏性内容失效时重新生成标记；不能只在构造时生成一次而忽略同一 owner 内的内容替换。
- 替换时生成不同于当前值的新标记，不维护历史标记登记表；随机值不构成严格的全局唯一性保证。
- 普通追加只推进 latestIndex；重复保活和单纯值缓存容量/TTL 淘汰不换标记。重建后端 owner 即使磁盘内容未变也更换标记。
- 标记按 Session 和 timeline 作用域校验，不与 History ViewModel 的局部计数或 currentFlow 的流实例 nonce 混用，也不把随机值当作访问权限。
- 查询在同一后端 owner 上比较请求标记并读取；只读投影采用最终一致性，允许查询与内容变化竞争造成的短暂不一致，不承诺每次读取的原子快照，不为此强制新增受保护读取入口。历史命令的写入准入与标记校验仍按下节执行。
- 前端收到新 cacheNonce 后使旧缓存和旧读取绑定失效，拒绝迟到结果回填；即使后端读取当时合法，回填时仍须核对本地绑定与标记。
- SessionNotActive 使旧订阅绑定失效；仍需访问时先保活，再重新初始化元数据并建立订阅。旧流的迟到值不能覆盖新绑定，不复用旧 cacheNonce 作为新 owner 的默认值。
- 前端按需获取索引和值并重建展示窗口，不连续传输随历史增长的完整索引列表。
- 同一 cacheNonce 下 exact 已存值可按实际存储 index 缓存；floor/ceil、空查询及未来区间仍会受追加影响，不能永久缓存为不存在。
- 不把独立 timeline 通知组合成强一致 Agent 状态；命令准入与执行状态仍由后端维护。
- 保留 secret answer 等既有展示约束；原值可序列化不等于可以任意显示或记录。
- 初期允许前端保留完整索引以复用既有逻辑；有界索引内存、窗口化和滚动重构不作为 RPC 迁移前置工作。
- 保留远程 get/floor/ceil 和范围读取能力，不要求前端先取得全索引才能查询。
- 不用这些持久化读取替代未落盘的响应流、运行状态或工具交互契约。

## 前端单 timeline 实现

- `rpc/client` 的 `CoroutineScope.rpcCachedIndexVersioned(sessionIndex, rpc)` 返回原 CachedIndexVersioned；使用已恢复异常的客户端及已激活 Session，不依赖后端 filesystem 实现。
- 一次工厂调用形成独立子 Job 绑定，元数据复用 rpcStateIn；初始化失败或取消时释放已建立的订阅，不返回半初始化视图。初始化失败的取消/等待清理预算为10秒，保留主异常。
- 本地只缓存非 null 的 exact 已存值；valuesIn 的带 index 条目可填入同一缓存。get、floor/ceil、indexesIn、valuesIn 仍查询后端，不将 get 的可见值当作请求 index 的 exact 值，不缓存不存在或范围完备性。
- 值缓存使用现有 cache4k，默认1024项、60秒访问过期；采用访问时的库内过期检查，不另建清理定时器或用户配置。此缓存期限不是 Session TTL。
- latestIndex() 读取最近收到的元数据，不重读初始化 Get；查询前后核对绑定与 nonce，迟到结果使用 CacheNonceMismatch 拒绝。命中检查、失效和校验后回填仅在前端短临界区协调，不持锁等待 RPC，不增加后端锁。
- 元数据失败使绑定失效并遵循 owner 的异常处理；元数据正常结束或取消也终止该绑定，不让已冻结标记继续服务查询。查询返回 SessionNotActive 时关闭该绑定并原样抛出；普通单次查询错误或取消不关闭其他观察。
- 绑定结束释放本地缓存、元数据观察及该绑定挂接的在途读取；只读 StateFlow 仍可保留末值，不把末值解释为绑定仍有效。重建和前端可用性展示由外层处理，不在缓存中自动激活、重订阅或重试。
- 不关闭共享 RpcClient，不替代 Session/Agent 聚合、settings CAS、历史窗口与实际界面接入。

## 历史操作

- AgentRuntimeRpc.revertHistory(sessionIndex, untilExclusive, expectedCacheNonce) 挂起等待回退完成；GlobalRpc.forkSessionHistory 使用相同参数并返回新 Session index，区别于完整 forkSession。
- expectedCacheNonce 绑定菜单目标使用的后端 index timeline 缓存标记，不使用前端窗口自增计数；确认时不得替换成当前标记使旧目标通过。
- 该校验失败使用 CacheNonceMismatch，前端须重新选择或确认有效目标，不给回退/fork 换新标记后自动重试；非法边界、准入失败和未知异常不合并为标记失配。
- 在实际写入或源快照捕获边界校验缓存标记与原非运行/能力准入；缺失 Session、标记失配和非法边界均失败。
- untilExclusive 排除该位置及以后内容，必须保留初始化且不超过当前 storage latestIndex + 1；允许稀疏间隙，不要求边界恰有事件。
- 回退保留原自动标题协调及成功回退后的 pendingSteer 清空；fork 保留边界 settings、标题处理和失败目标清理，不修改源历史。
- 确认弹窗、回退后编辑草稿及打开 fork 目标留前端，不增加后端确认句柄或 request/confirm/dismiss RPC。
- 已接受操作归后端持有，不因前端等待取消而终止，不续期 TTL；成功回执不表示前端缓存同步，丢失 fork 回执不得盲目重放。
