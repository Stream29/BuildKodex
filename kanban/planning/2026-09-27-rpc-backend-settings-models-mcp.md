# Task Tree

- `Record confirmed value-only import semantics`()
- **`Await scoped implementation authorization`()**
- `Move global settings models and MCP ownership behind the backend boundary`()
- `Connect full settings CAS to existing business side effects`()
- `Adapt global observations and MCP commands`()
- `Verify concurrent updates and shared-resource cleanup`()
- `Record evidence and complete this batch`()

# Details

## 位置与前置

- 实施批次 P03；父任务：[实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。
- 状态：Planning；复用已完成的 BackendSettingsStore、文件 CAS、原 model catalog 与 MCP manager，不依赖 P01/P02。
- 交付：GlobalRpc 的 settings/models/MCP 业务组件；P04 接入认证协调，P07 完成隔离后端装配，P11 切换生产 CLI。
- 规范：[分批实施边界](../../checklist/rpc-architecture.md#分批实施边界)、[设置读写](../../checklist/rpc-settings.md#rpc-读写)、[MCP](../../checklist/rpc-mcp.md)、[状态](../../checklist/rpc-state.md)。

## 实施范围

- 后端持有 settings store、模型目录、context/shell 设置投影与 MCP service/manager；后台刷新不依赖前端轮询。
- 接 getSettings/getSettingsFlow/compareAndSetSettings、getModels/getModelsFlow，读取完整实际值，不用默认值或脱敏假值冒充后端快照。
- 将原 MCP 配置更新、凭据续期与完整 settings CAS 接入同一实际写入边界；保留校验、持久化及连接/身份协调，不用只 CAS 镜像或外层私有锁替代。
- 接 getMcpServers/getMcpServersFlow、reconnectMcpServer、getCodexMcpSettings；logoutMcpServer 与登录并发的整体协调归 P04。
- 观察接入不改 manager/service 的读侧行为；导入仅提交配置值，不承接旧 Replace 的额外 invalidate。前端预览不成为后端合法性校验来源。
- 只复用现有 GlobalRpc 切面；允许内部分工，不新增按子页面拆分的 RPC，不注册返回假值/空实现的生产 GlobalRpc。
- 入口：[旧 Application 组装](../../Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt)、[MCP 配置 store 适配](../../Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/KodexMcpConfigurationStore.kt)、[文件 CAS](../../Kodex/app/shared/settings/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/BackendSettingsStore.kt)、[MCP 实现](../../Kodex/mcp/impl)、[模型目录](../../Kodex/openai/model-catalog/impl)。

## 装配边界

- 原模型目录依赖 OpenAiClient，原 Application 又由 authStore 创建 client；P03 独立准备配置/MCP 和模型观察组件，测试注入 mock client，不声称生产认证装配已完成。
- P04 提供新认证来源后，在 P07 统一装配真实后端资源。后端不能为了复用 Application 的工厂反向依赖前端 ViewModel；搬迁必要组装代码，不新建第二份设置或认证真源。

## 已澄清：导入仅更新配置

- 用户确认新路径不具有独立 Replace 语义，无变化更新忽略；以 [导入提交语义](../../checklist/rpc-mcp.md#导入提交语义)为准，撤回“完整保留旧导入额外副作用”的要求。
- 原 [applyCodexImport](../../Kodex/mcp/impl/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt#L375-L418)及[原测试](../../Kodex/mcp/impl/src/commonTest/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImplTest.kt#L208-L228)中的 Replace/invalidate 是旧路径事实，不是新 RPC 路径必须复现的语义。
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
