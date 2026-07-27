# Task Tree

- [done] 制定Agent文件系统持久化方案
  - [done] 梳理`AgentStorage`和settings现有契约
  - [done] 调研Codex session与设置存储格式及导入边界
  - [done] 设计独立session存储后端和Codex导入路径
  - [done] 设计项目原生设置后端和Codex设置后端
  - [done] 写出初步实施计划供用户复核
  - [done] 完成用户逐项复核
  - [done] 用操作级补偿替换storage级transaction
  - [done] 为`CoroutineFileSystem.sink`补充原子创建能力
  - [done] 建立session repository契约
  - [done] 实现文件系统AgentStorage、租约和缓存装饰器
  - [done] 将CLI接入session repository
  - [done] 实现Codex rollout读取与单向导入
  - [done] 建立全局settings store契约和原生文件后端
  - [done] 实现保留TOML结构的Codex settings后端
  - [done] 覆盖真实文件IO、重启、竞争、fork和导入测试

# Details

方案已经完成逐项复核，现按已确定的实施顺序执行。session使用项目独立后端，同时支持导入Codex格式；设置保留两套后端。

## 调研基线

- Kotlin代码以当前工作树为准。
- Codex源码以`origin/main`的`cf821e8ec850c6d8380feea0e84859dd8ff54cd0`为准，时间为2026-07-20。
- `CodexAgentStorage`已经表达单个thread的稀疏共享index空间，包含history、compaction、settings、timestamp和token count：`CodexLite/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/codex/lite/agentstorage/contract/CodexAgentStorage.kt:38`。
- 当前只有进程内实现，且CLI自己持有session列表和新session默认值：`CodexLite/agent-storage/in-memory/src/commonMain/kotlin/io/github/stream29/codex/lite/agentstorage/inmemory/InMemoryCodexAgentStorage.kt:20`、`CodexLite/cli/app/src/commonMain/kotlin/io/github/stream29/codex/lite/cli/app/model/SessionManager.kt:137`。
- 全局设置目前只有内存`StateFlow`：`CodexLite/cli/settings/src/commonMain/kotlin/io/github/stream29/codex/lite/cli/settings/CodexGlobalSettings.kt:41`。

## Codex存储的真实边界

- Codex的session真源是按顺序追加的rollout JSONL。每行是`timestamp + ordinal + RolloutItem`，item联合包含`SessionMeta`、`ResponseItem`、`Compacted`、`TurnContext`、`WorldState`和持久化event：`shared-context/codex/codex-rs/protocol/src/protocol.rs:3082`、`shared-context/codex/codex-rs/protocol/src/protocol.rs:3211`。
- 活跃session位于`$CODEX_HOME/sessions/YYYY/MM/DD/rollout-*.jsonl`，归档session位于`archived_sessions`，冷数据可压缩。`RolloutRecorder`只负责持久化和重放：`shared-context/codex/codex-rs/rollout/src/recorder.rs:84`。
- SQLite state DB是可从rollout回填的列表、检索和元数据投影，不是session的唯一真源。
- 最新Codex已抽出与介质无关的`ThreadStore`，但它接受的仍是Codex自身`RolloutItem`：`shared-context/codex/codex-rs/thread-store/src/store.rs:36`。
- Codex的`config.toml`是多层配置的user layer。写入时通过TOML AST做定点edit，保留其他key和注释，再原子替换文件：`shared-context/codex/codex-rs/core/src/config/edit.rs:692`。

## 已确定的设计

### Session真源

- Codex Lite使用自己的session存储格式，不向Codex rollout追加或回写。
- Codex session只是只读导入源。repository先创建目标storage，importer只向该目标写入，并在导入元数据中保留原thread id、路径和源版本。
- 导入是一次性拷贝，不做双向同步，也不在启动后持续监视Codex文件。
- session列表、创建、打开、归档、删除和导入属于repository边界，不应塞进单thread的`CodexAgentStorage`。
- `CodexAgentStorage.id`只是后端对象的本地字符串标识：内存实现使用进程内对象identity hash，文件实现使用规范化storage路径。
- OpenAI wire `thread_id`不是`storage.id`本身。AgentState请求投影使用固定的provider规则把本地id转换为符合OpenAI格式的稳定字符串，并用同一结果构造`window_id`。

### 原生文件格式

- 每个thread有一个`manifest.json`，以及`history`、`compaction`、`settings`、`timestamp`和`token-count`五个timeline目录。
- `manifest.json`只保存整体format version，不保存storage id。它既是未来数据迁移的入口，也是该目录已经初始化为合法AgentStorage的标识。
- thread根目录包含不带format version的`lock.json`租约。租约记录`pid`、不可变的`acquired_at`和续租用的`expires_at`；`pid + acquired_at`共同标识owner。
- 创建和删除storage以及构造带缓存的活跃视图前必须取得租约。活跃视图定期续租；租约owner变化后立即失效，不得继续读写或删除新owner的锁。
- 每个timeline的一个stored index直接对应目录下的一个数字编号JSON文件；目录内容是真源，不额外维护`index.json`。
- 无缓存的文件视图直接从数字文件推导当前索引，不持有目录所有权或长期内存状态。外层缓存视图取得租约后为每个timeline扫描一次目录，在内存中重建完整的有序稀疏索引；后续`set`和`revert`增量更新该索引。
- 每个timeline操作独立持久化。compound write按前缀安全的顺序执行，使任意已持久化前缀都能恢复为合法AgentState。
- 单个记录先写临时文件，完整关闭后再发布为对应数字文件；进程崩溃留下的临时文件在打开storage时清理。
- `revert`按index降序将后缀移出timeline目录；`revertWithTransaction`在自己的临时目录中保留这些文件，成功后删除，失败时按index升序恢复。
- `forkTo`只把源storage在指定边界的数据复制到已经存在的目标storage。它不创建、不发布目标，也不修改目标的`manifest.json`；源和目标可以使用不同后端。
- format version只存在于`manifest.json`。repository先识别整体版本，再用该版本对应的serializer直接解码timeline真实数据，不为每条记录增加version envelope。
- 数字编号记录本身已经避免重写完整history，不再引入分段日志或额外checkpoint格式。
- 普通记录使用“临时文件、关闭、原子替换”发布，不执行逐记录`fsync`；当前持久化契约不保证机器断电后的强持久性。

### 文件后端组成

- `utils:kotlinx-io-coroutines`为`sink(path, append = false, mustCreate = false)`增加跨平台原子创建语义。`mustCreate = true`时目标已存在必须失败，供`lock.json`首次抢占使用。
- `FileSystemIndexVersioned<T> : MutableIndexVersioned<T>`基于`KSerializer<T>`包装一条由数字JSON文件组成的timeline。它负责直接文件行为，不持有租约、索引缓存或值缓存。
- `FileSystemAgentStorage`组合五个`FileSystemIndexVersioned`并实现`MutableCodexAgentStorage`。它仍是无所有权的直接文件视图。
- 独立decorator取得并维持`lock.json`租约，缓存五条timeline的完整有序索引，并为解码值提供有界LRU。LRU以实际stored index为key；请求同一稀疏值的多个snapshot index不会产生重复缓存。
- filesystem repository在同一模块中负责创建、打开、列举、归档和删除storage。跨后端复制仍由`forkTo`完成。

### 写入与补偿语义

- storage只允许单writer，不提供compound write期间的跨timeline snapshot isolation。
- 删除timeline与storage级`transaction`，改用`setWithTransaction(...) { ... }`和`revertWithTransaction(...) { ... }`嵌套表达操作及其后续步骤。
- 普通失败或取消沿Kotlin调用栈逆序补偿；补偿不可取消，补偿失败后storage停止接受写入。
- 进程崩溃允许保留已持久化的合法操作前缀，不维护durable begin、commit或补偿栈。
- `CodexAgentState.latestIndex`只在compound write完整成功后发布；重启时从持久化前缀重新推导状态。
- 完整语义见`shared-context/findings/agent-storage-compensation-semantics.md`。

### Codex session导入

- 低层reader放在现有`openai:codex-cli-storage`，只负责发现、解压、逐行解码和报告源位置。
- `agent-storage:codex-import`负责把Codex rollout重放到调用方提供的目标storage，它不创建目标，也不向其他模块泄露Codex rollout DTO。
- importer需要处理active/archive、压缩rollout、fork lineage、paginated `history_base`、compaction replacement history、rollback和turn context。继承前缀在写入原生存储前展平，不保留对Codex源文件的运行时依赖。
- 不识别但明确不影响重放的event可记录警告后跳过。未知的model-visible item、compaction或rollback必须使该session导入失败，不得静默丢失语义。
- 目标storage的创建与可见性由调用方和repository编排；importer只负责完整重放与验证。

## Settings后端

- 建立一个`CodexGlobalSettingsStore`契约：暴露完整`StateFlow<CodexGlobalSettings>`，并在持久化成功后原子发布新快照。
- 项目原生后端在Codex Lite home内保存全部全局设置，包含Codex home、换行键和新session默认的model/reasoning/service tier/mode。当前`SessionManager.newSessionConfiguration`要迁入这个快照。
- Codex后端只编辑`$CODEX_HOME/config.toml`中有明确对应的key：`model`、`model_reasoning_effort`、`service_tier`、`tui.keymap.composer.submit`和`tui.keymap.editor.insert_newline`。Codex已对submit/newline提供结构化keymap：`shared-context/codex/codex-rs/config/src/tui_keymap.rs:103`、`shared-context/codex/codex-rs/config/src/tui_keymap.rs:153`。
- Codex home是该后端的构造参数，不从`config.toml`反向推导。项目专属且Codex无对应项的设置不写入`config.toml`。
- Codex后端使用能保留格式和未知key的TOML AST做定点修改，通过临时文件、flush和atomic replace落盘。不把`config.toml`先解码成不完整DTO再整体重写。
- per-session `CodexAgentSettings`继续存在AgentStorage settings timeline中，不与全局设置后端合并。

## 模块划分

- `agent-storage:repository-contract`：session发现、创建、打开、归档和删除契约。跨storage复制由独立的`forkTo`完成。
- `agent-storage:filesystem`：包含filesystem repository、无所有权的`FileSystemIndexVersioned`与`FileSystemAgentStorage`，以及持有`lock.json`租约的缓存decorator。
- `agent-storage:codex-import`：Codex rollout到原生repository的单向导入。
- `openai:codex-cli-storage`：扩展为Codex home的低层只读访问，不承担业务投影。
- `cli:settings`：保留快照和store contract；内存实现用于测试和短生命宿主。
- `cli:settings-filesystem`：项目原生设置后端。
- `cli:settings-codex`：Codex `config.toml`后端。

## 实施顺序

1. 用操作级补偿替换现有transaction，并审计所有compound write的前缀合法性。
2. 为`CoroutineFileSystem.sink`增加并跨平台验证`mustCreate`语义。
3. 建立repository contract，实现filesystem backend、租约和LRU decorator。
4. 覆盖租约竞争与过期接管、重启、取消、中途崩溃、revert、跨后端fork、长history和reader观察合法前缀的真实文件IO测试。
5. 让CLI从repository列举和打开session，内存后端仍可供测试显式注入。
6. 实现Codex rollout reader与importer，用不同年代、compaction、fork、rollback、paginated和压缩fixture做重放对照。
7. 提取settings store contract，先实现原生后端，再实现保留TOML结构的Codex后端。
8. 在Linux和macOS上做真实文件IO、跨进程重启和CLI导入手工验收。
