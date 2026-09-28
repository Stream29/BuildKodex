# Task Tree

- `Confirm sequential implementation authorization and clean baseline`()
- `Add Session settings comparison at the existing write boundary`()
- `Verify response-header and settings write interleavings`()
- `Run affected JVM and Native regressions`()
- `Record evidence and complete this batch`()

# Details

## 位置与前置

- 实施批次 P01；父任务：[RPC 重构实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。
- 状态：Done，范围内实现和验收完成。用户已授权按批次顺序实施；从两仓干净的 refactor/rpc 基线（BuildKodex `08b9532`、Kodex `e6154848`）启动，不包含改 RPC contract、提交或推送授权。
- 交付：Session settings 的后端实际 CAS 原语，供 P05 委托；不重复实现已有 BackendSettingsStore 文件 CAS。
- 规范：[分批实施边界](../../checklist/rpc-architecture.md#分批实施边界)、[settings CAS](../../checklist/rpc-timeline.md#读取边界)、[写入串行化](../../checklist/agent-state-mutation-serialization.md)、[字段冲突](../../checklist/rpc-state.md#settings-的字段冲突)。

## 实施范围

- 在原 AgentState 的私有写入锁内读取当前完整设置、比较 expect，再按原路径追加 update；不在 RPC 层先比较再无条件 updateSettings。
- 保留运行中编辑设置的原准入：当前请求沿用已捕获快照，新设置供后续请求使用；不借用拒绝 RequestResponse 的 modify 改变该行为。
- mismatch 返回 false；同值成功不追加 settings/timestamp；合法提交继续发布原全局 latestIndex 和 timeline 元数据。
- 校验、I/O、取消不伪装成 false；内部 turn-state 等字段的合法性沿原领域约束处理，不为完整比较增加 revision 或忽略字段。
- 不公开 Mutex，不修改通用缓存/文件提交算法，不顺带改 plan 或工具完成的事务语义。
- 入口：[AgentState 实现](../../Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt)、[状态契约目录](../../Kodex/agent-state/contract)、[SettingsTimelineRpc](../../Kodex/rpc/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/SettingsTimelineRpc.kt)。

## 验收

- 旧 expect、相等但不同实例、同值更新及正常追加；比较失败/同值时历史文件与索引不变。
- 响应头先提交 turn-state 后旧 expect 失败；反向交错也不丢失最新值；不通过服务层专有锁绕过后台写入。
- RequestResponse 期间 CAS 不重置运行状态；已缓存 exact 旧记录不被原地覆盖。
- 写入失败、等待锁时取消与提交中取消保留原状态恢复/文件事务保证。
- 实际运行相关 JVM 与本机 Native 测试，并编译受影响目标；记录平台跳过，不用旧报告代替本批结果。

## 交接

- P05 使用此原语接 SettingsTimelineRpc，并迁入自动标题 CAS；P09 验证前端字段重试与回执不回写。
- 完成后更新父任务进度，移入 done 并修正引用；本任务不启动 CLI、迁移 Home 或自动提交。

## 实施记录

- 在原领域接口增加 `compareAndSetSettings`，实际比较与追加使用 `KodexAgentStateImpl.writeMutex`；抽取原 settings 追加代码供 CAS 与无条件 update 共用，没有新依赖、第二把锁或 RPC contract 改动。
- 新增12项 AgentState 测试，覆盖完整值比较、同值零写入、8个竞争 CAS、双向 update/CAS 排队、等待及提交中取消、读写异常、响应头先/后落盘和运行状态保持。
- 文件 repository 增加1项测试，验证真实缓存 metadata 发布、nonce 不变、旧 exact 记录保持和关闭重开后的持久结果；测试使用隔离临时目录。
- 首轮 JVM 的两个新增响应头测试失败：用例误把全局索引1当成必有 settings exact 记录，实际 settings 是稀疏 timeline。改用真实 settings latestIndex 捕获旧记录后复验；没有修改生产存储来迁就测试。
- IDE仅打开其他项目，不抢占窗口；构建前无 Gradle Daemon，显式以既有 Temurin25 启动，后续复用 Daemon `3219238`。

## 验证结果

- 使用 `JAVA_HOME=/home/stream/.gradle/jdks/eclipse_adoptium-25-amd64-linux.2`，在 Kodex 运行 `./gradlew`，没有升级依赖。
- `:agent-state-impl:{jvmTest,linuxX64Test}` 各46项、`:agent-session-filesystem:{jvmTest,linuxX64Test}` 各37项、`:agent-session-in-memory:{jvmTest,linuxX64Test}` 各20项通过，两端各103项，包含13项新增测试。
- 对 agent-state-impl / agent-session-filesystem 的两端测试逐个加 `--rerun` 强制重跑，各83项再次通过；不把 up-to-date 当作本批实际测试。
- 两个修改模块的 `compileKotlin{Jvm,LinuxX64,LinuxArm64,MingwX64}` 与相应测试源码编译通过；`:app-cli:compileKotlinJvm` 的依赖重新编译通过，旧入口未切换。
- `:agent-state-impl:compileKotlinMacosArm64` 沿既有传递依赖条件 SKIPPED；未声称 macOS 实测或 CLI 运行验收完成。
- 本批只实现领域写入原语与测试。P05 仍需通过原 runtime 委托此方法；没有实现 SettingsTimelineRpc 服务、前端字段编辑、TTL 或真实 Home 操作。原无条件 update 的追加/失败语义不变。
