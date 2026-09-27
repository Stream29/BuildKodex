# Task Tree

- **`Await scoped implementation authorization`()**
- `Connect backend projections and local frontend preferences`()
- `Migrate settings editing and field-aware CAS retries`()
- `Adapt OAuth listeners MCP controls and reset confirmation`()
- `Verify frontend-only effects and resource cleanup`()
- `Record evidence and complete this batch`()

# Details

## 位置与前置

- 实施批次 P09；父任务：[实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。
- 状态：Planning；依赖 [P03](2026-09-27-rpc-backend-settings-models-mcp.md)、[P04](2026-09-27-rpc-backend-auth-oauth-usage.md)、[P08](2026-09-27-rpc-frontend-session-views.md)，复用已有前端 store/Hook 模型。
- 交付：设置与认证等页面的 RPC 消费、前端效果和本地偏好；Hook 执行器由 P07 提供。
- 规范：[分批实施边界](../../checklist/rpc-architecture.md#分批实施边界)、[设置](../../checklist/rpc-settings.md)、[字段 CAS](../../checklist/rpc-state.md#settings-的字段冲突)、[认证](../../checklist/rpc-authentication.md)、[MCP](../../checklist/rpc-mcp.md)、[reset](../../checklist/rpc-account-usage.md#重置券选择)。

## 实施范围

- 后端 settings/models/auth/usage/MCP 分别 Get 初始化后共享 Flow；本地偏好读取 CliFrontendSettingsStore，不建立合并持久真源。
- 全局和持久化 Session 编辑用完整 expect/update CAS；固定目标字段初值，其他字段变化重算、目标变化停止。取消/失效/未知失败不进入 false 重试。
- 持久化设置以订阅为准，删除旧 configurationOverride/nameOverride 等回执强制回写；草稿和弹窗目标/revision 留前端。
- 侧栏宽度只作启动时四分之一及后续调整的临时状态；编辑 hooks 的 name/types/command，不保留控制型配置项。
- 两种认证来源保留登录/移除、去掉手动 Reload；前端先 listener 再 start，打开 URL 后交付 callback，10分钟等待及取消时清理 listener/尝试 exact cancel。
- MCP 页面复用原脱敏状态展示，导入读取/预览/选择在前端，完整 CAS 交后端校验；命令回执不伪造连接成功。
- reset 必须显示具体券与到期日期并二次确认；无明细不自动选券，失败后刷新供用户重新决定，不自动消费。
- 工作目录选择继续本地；浏览器和通知命令属前端，模型调用、凭据与后端文件读取不进入前端。
- 入口：[设置 ViewModel](../../Kodex/app/viewmodel/settings)、[设置视图](../../Kodex/app/view/settings)、[SessionSettingsDataSource](../../Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/SessionSettingsDataSource.kt)、[两侧 store](../../Kodex/app/shared/settings/filesystem)。

## 分批边界

- 复用 P07 的完整后端装配做真 RPC 测试；不等到 P11 才第一次验证 GlobalRpc 与前端页面组合。
- 沿 P08 的编译边界准备新工厂/数据源；旧加载器和旧生产入口仍可构建。共享界面的破坏性参数替换、旧 Reload/Hook 控件最终删除在 P11 与入口选择一起完成。
- 导入提交按 [P03 已确认的值更新语义](2026-09-27-rpc-backend-settings-models-mcp.md#已澄清导入仅更新配置)接线，不再等待 Replace 意图设计；页面观察与提交仍分别验收。

## 验收

- 初始 Get 失败不显假默认值，CAS 成功但 Flow 滞后/回执迟到不倒退；目标字段冲突停止，其他字段及新 turn-state 保留。
- Session 重建/页面关闭终止旧编辑；本地偏好仅写 frontend 文件，后端设置无本地写盘路径。
- OAuth mock callback、超时、浏览器失败、取消与完成竞争；不记录 URL/token，不泄漏 listener。
- MCP 列表/详情跟随真实后端状态；未观察变化不能用配置或回执推测 Healthy。
- 导入合并得到相同配置时，CAS 成功不额外发 reconnect 或其他业务命令；真实字段冲突和配置变化仍走原 CAS/后端协调路径。
- 选券日期未知明确可见，无明细不可提交；确定 reset 结果保留，即使用量刷新失败。
- JVM/Native ViewModel 测试、可用的视图编译与隔离交互检查；不运行真实 OAuth 或付费用量操作。

## 交接

- P11 注入新前端工厂和共享 client，移除旧直接 manager/store/模型依赖；新页面不得先与旧后端加载器混合上线。
