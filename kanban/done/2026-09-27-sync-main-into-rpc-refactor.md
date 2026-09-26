# Task Tree

- `Inspect branch divergence and preserve unrelated work`()
- `Merge main into both refactor branches without committing`()
- `Reconcile cache and RPC snapshot types and regression fixtures`()
- `Compile affected modules and run JVM and Native regressions`()
- `Resume RPC integration review`()
- `Record verification and commit both merges`()

# Details

- 用户批准将两仓 main 合入各自 refactor/rpc，处理必要兼容、运行回归并保存合并提交；不改写原重构提交，不推送。
- 父任务：[kRPC 前后端分离](../planning/2026-09-08-explore-krpc-frontend-backend-separation.md)。本子任务仅同步基线，不代表业务服务或 CLI 迁移完成。
- 主线基线：BuildKodex `f0390b4`，Kodex `54f46aaf`；原重构检查点分别为 `5c31532`、`2433dac3`。
- 主线 tokenCount 已使用可序列化的 TokenCountSnapshot；重构分支的观察接口、TokenCountTimelineRpc、缓存与新增运行测试需复用该类型。保留原 totalTokens 投影和主线诊断/turn-state 行为。
- 合并冲突：Kodex 的 CachedAgentStorage 同时保留 CachedIndexVersionedImpl/ObservableKodexAgentStorage 与新值类型；根仓库的 Kodex gitlink 在代码合并提交后更新。
- 验证覆盖 RPC 契约/客户端、文件与内存 Session、压缩和存储相关回归、两侧设置 store；使用隔离 fixture，不运行真实 Home 迁移或模型调用。
- 两份原有未跟踪任务保持原样：MCP CLI 讨论与 CodeClaw/OpenSpec 比较记录。不更改其他子模块的工作区。
- 开始时无运行中的 Gradle Daemon；IDE 打开 DesktopManagement，不在其中操作本项目。显式使用原 Temurin 25，后续构建复用 Daemon 2074351。

## 兼容改动

- CachedAgentStorage 保留两侧合并后的只读接口、原实现与存储算法，将 tokenCount 的值类型改为 TokenCountSnapshot；同步 ObservableKodexAgentStorage。
- TokenCountTimelineRpc 的 get/getExact/valuesIn 直接返回原快照；只新增原 agent-storage/contract 模块依赖，不增加 DTO、RPC 方法或新库版本。
- 运行槽位测试按主线值类型写入并读取 totalTokens；文件 Session 原观察视图测试补齐结构化值追加/回退后的元数据断言。
- 新增两项值测试与一项真实内存 RPC 测试，覆盖快照 kind、完整 usage/diagnostics、缺失与零值、Long 精度、稀疏索引及前端 exact 缓存。

## 验证

| 模块 | JVM | Linux x64 Native |
| --- | ---: | ---: |
| rpc/models | 31 | 31 |
| rpc/contract | 59 | 59 |
| rpc/client | 85 | 85 |
| rpc/in-memory | 17 | 17 |
| utils/rpc-exception | 17 | 17 |
| agent-session/filesystem | 36 | 36 |
| agent-session/in-memory | 20 | 20 |
| agent-runtime/decorator/compact | 11 | 11 |
| app/shared/settings/filesystem | 63 | 62 |
| agent-state/impl | 34 | 34 |
| agent-storage/contract-ext | 4 | 4 |
| app/viewmodel/session | 14 | 14 |
| app/migration/impl | 33 | 32 |
| 合计 | 424 | 422 |

- 两端表内测试均实际执行且零失败、零跳过；两项数量差异来自原 JVM 专属测试。
- rpc/contract、rpc/client、两个 Session 模块及 settings/filesystem 的库和测试源码在 JVM、Linux x64、Linux ARM64、Windows x64 编译通过；macOS ARM64 因原 cinterop 条件跳过，不记为成功或运行验证。
- JVM 首轮构建 4分47秒，Native/跨目标构建 4分45秒；Mosaic cklib 的配置缓存不兼容警告导致后者缓存条目被丢弃，构建成功，未修改其构建配置。
- javap 确认 TokenCountTimelineRpc 仍为10方法，返回值为原快照及其 nullable/Pair 列表，没有遗留 Long 值签名。
- rpc/contract、rpc/client、两个 Session 模块在 JVM/Native 各200项强制重跑通过；CLI 组合根的 JVM/Linux x64 编译与 integration-test 的 JVM 测试源码编译通过，构建1分34秒。本次不包含 Native CLI executable 链接或交互运行。
- 未执行真实凭据、模型、Hook 命令或真实 Home 迁移；migration 模块只运行隔离目录测试。三份文档106个相对链接/锚点/源码行链接及空白检查通过。

## 后续接入复核

- 主线模型变化没有引入新的 RPC 方法需求；目前的关键前置仍是后端实际资源宿主，而不是更多传输 helper。
- 原 Application 仍按前端工厂创建 repository；runtime 仍将调用 Job 登记为 runningTurn。完整服务直接委托这些入口，不能自动获得已定的共享后端所有权和“取消等待不取消执行”语义。
- 下一组建议成组收敛后端 Session 宿主：真实 repository 的创建/关闭、已接受运行归属、60/20秒续期与失活、目录与 live 读取共用同一来源。沿用已定规则，不增加第二份活跃真源或半实现 GlobalRpc。
- 此处只记录下一实施边界；当前合并不接入 TTL、业务服务、CLI 或设置拆分 migration。

## 合并检查点

- Kodex 合并提交 `e6154848`，父提交为 `2433dac3` 与 `54f46aaf`；代码工作区干净，保留原分批提交和主线历史。
- 根仓库本次合并以 `f0390b4` 为第二父提交，Kodex gitlink 指向 `e6154848`；两仓 main 保持同步前原值，工作分支均为 refactor/rpc。
- 后续 Session 宿主范围已回填父任务，仍待实施授权；本子任务完成的是主线同步与兼容验证。
