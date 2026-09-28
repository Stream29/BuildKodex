# Task Tree

- `Record confirmed value-only import semantics`()
- `Receive the P02 handoff`()
- `Move global settings models and MCP ownership behind the backend boundary`()
- `Connect full settings CAS to existing business side effects`()
- `Adapt global observations and MCP commands`()
- `Verify concurrent updates and shared-resource cleanup`()
- `Record evidence and complete this batch`()

# Details

## 位置与前置

- 实施批次 P03；父任务：[实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。已获顺序实施授权，P02 已完成。
- 状态：Done；复用 BackendSettingsStore、文件 CAS、原 model catalog 与 MCP manager，隔离组件已验收，完整 GlobalRpc 装配仍归 P07。
- 交付：GlobalRpc 的 settings/models/MCP 业务组件；P04 接入认证协调，P07 完成隔离后端装配，P11 切换生产 CLI。
- 规范：[分批实施边界](../../checklist/rpc-architecture.md#分批实施边界)、[设置读写](../../checklist/rpc-settings.md#rpc-读写)、[MCP](../../checklist/rpc-mcp.md)、[状态](../../checklist/rpc-state.md)。

## 实施范围

- 后端持有 settings store、模型目录、context/shell 设置投影与 MCP service/manager；后台刷新不依赖前端轮询。
- 接 getSettings/getSettingsFlow/compareAndSetSettings、getModels/getModelsFlow，读取完整实际值，不用默认值或脱敏假值冒充后端快照。
- 将原 MCP 配置更新、凭据续期与完整 settings CAS 接入同一实际写入边界；保留校验、持久化及连接/身份协调，不用只 CAS 镜像或外层私有锁替代。
- 接 getMcpServers/getMcpServersFlow、reconnectMcpServer、getCodexMcpSettings；logoutMcpServer 与登录并发的整体协调归 P04。
- 观察接入不改 manager/service 的读侧行为；导入仅提交配置值，不承接旧 Replace 的额外 invalidate。前端预览不成为后端合法性校验来源。
- 只复用现有 GlobalRpc 切面；允许内部分工，不新增按子页面拆分的 RPC，不注册返回假值/空实现的生产 GlobalRpc。
- 入口：[Application 组装](../../Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt)、[后端 MCP 配置适配](../../Kodex/rpc/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalState.kt)、[文件 CAS](../../Kodex/app/shared/settings/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/BackendSettingsStore.kt)、[MCP 实现](../../Kodex/mcp/impl)、[模型目录](../../Kodex/openai/model-catalog/impl)。旧 KodexMcpConfigurationStore 已在 P11 被后端适配替换。

## 装配边界

- 原模型目录依赖 OpenAiClient，原 Application 又由 authStore 创建 client；P03 独立准备配置/MCP 和模型观察组件，测试注入 mock client，不声称生产认证装配已完成。
- P04 提供新认证来源后，在 P07 统一装配真实后端资源。后端不能为了复用 Application 的工厂反向依赖前端 ViewModel；搬迁必要组装代码，不新建第二份设置或认证真源。

## 已澄清：导入仅更新配置

- 用户确认新路径不具有独立 Replace 语义，无变化更新忽略；以 [导入提交语义](../../checklist/rpc-mcp.md#导入提交语义)为准，撤回“完整保留旧导入额外副作用”的要求。
- 原 [applyCodexImport](../../Kodex/mcp/impl/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt#L393-L438)及[原测试](../../Kodex/mcp/impl/src/commonTest/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImplTest.kt#L208-L228)中的 Replace/invalidate 是旧路径事实，不是新 RPC 路径必须复现的语义。
- [compareAndSetSettings](../../Kodex/rpc/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/GlobalRpc.kt#L38-L56)保持原签名与同值无变化约定；不新增命令意图字段或自动 reconnect。真实配置变化继续由后端正常协调。
- 此项设计阻塞已关闭，P03/P09 按现有 contract 接线即可；关闭阻塞不等于运行实现或测试完成，本轮原后端源码未改。

## 验收

- settings CAS 与后台凭据更新交错：旧 expect 不覆盖新凭据，同值不产生额外写入/变化；非法配置和持久化失败不是 false。
- 原 MCP 编辑、改名/删除、启停及连接身份测试继续成立；新增新导入路径的同值成功零写入/零重建与实际变化协调验证，不沿用旧 Replace 强制 invalidate 的预期。reconnect 失败保留前代 catalog，状态不由命令回执伪造。
- 配置与运行流独立推进；禁用项可见，脱敏展示不污染可编辑完整配置；取消一个观察不关闭共享 client。
- Get 初始化失败不返回伪空列表；Codex 导入缺配置与解析失败保持区别。
- mock MCP/模型服务及隔离设置文件的 JVM/Native 回归；不得连接用户 MCP、打印配置凭据或修改真实文件。

## 交接

- 提供 P04 所需后端 settings/MCP 能力与 P05 所需领域依赖；不把真实资源注入前端。
- P07 组装完整 GlobalRpc 供 P08/P09 真 RPC 验收；P03 内先验证实际业务组件，不伪造尚未就绪的方法。
- P09 消费这些观察和命令；旧 settings.yml 加载器与生产入口在 P11 才统一切换。

## 实现与验证

- [BackendGlobalState](../../Kodex/rpc/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalState.kt) 持有设置、context/shell 投影、MCP 配置 adapter/service/manager 与 model catalog；共享 OpenAI client 由上层认证 owner 提供，不反向依赖 Application。
- 文件 CAS 的验证回调位于原 update/reload 写入锁内，先比较再验证、保存；manager 命令锁只协调命令/删除登录/认证覆盖状态，不冒充文件原子性。后台 MCP 续期仍更新同一文件 store。
- 完整值检查不改写 CAS update；原编辑的空名称/密钥名称合法性及 OAuth 身份约束仍在后端验证。身份变化必须提交 Uninitialized，不默默将旧登录凭据迁到新名称/URL。
- 模型目录增加后端 scope 工厂，旧工厂继续供未切换 CLI 使用；explicit Codex 导入读取移入后端，保留完整可合并值、排序、缺文件与解析失败区别。
- 新增13项组件测试：真实文件 CAS/凭据交错/零写入/失败、配置与运行观察、真实 MCP service 的启停/改名与同实例保留、Codex 解析、模型所有权与退出收尾；均使用隔离目录和假凭据，无用户服务访问。
- 四目标库/测试源码编译、旧 CLI JVM 编译通过；JVM 四模块119项、Native 四模块110项通过（server 两端各30项，含原17项宿主测试）。macOS 沿原 cinterop 条件跳过。
- server 两端各30项随后强制重跑通过；临时日志与测试目录已清理。后续 P04 修改同一 MCP 模块时仍须重跑这些回归，不能沿用本批结果冒充新验证。
- 首轮修正 explicit API 返回类型标注，以及文件并发测试不应使用会自动推进超时的虚拟调度器；改为确定性 barrier 与普通协程作用域，未修改存储算法或 RPC contract。
