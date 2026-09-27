# Task Tree

- **`Await scoped implementation authorization`()**
- `Run the complete in-memory CLI acceptance matrix`()
- `Audit frontend dependency boundaries and obsolete production paths`()
- `Run supported compilation regression and migration checks`()
- `Reconcile current guidance with the implemented architecture`()
- `Close child tasks and the parent only after scoped checks finish`()

# Details

## 位置与前置

- 实施批次 P12；父任务：[实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)。
- 状态：Planning；依赖 [P11](2026-09-27-rpc-cli-cutover.md) 的真实生产切换，不以各模块 mock 测试替代。
- 交付：一期验收记录、必要的迁移文档收尾、明确的平台限制和未通过项；不是发布任务。
- 规范：[分批实施边界](../../checklist/rpc-architecture.md#分批实施边界)、[一期范围](../../checklist/rpc-architecture.md#一期部署与交付)、[资源清理](../../checklist/coroutine-resource-lifecycle.md)、[Home 验证](../../checklist/kodex-home.md#验证)。

## 验收矩阵

| 场景 | 必须观察的结果 |
| --- | --- |
| 创建、普通运行、工具回答、批量子任务 | 全部经过真实 RPC；后端拥有接受工作；未知结果不自动重放；前端草稿与导航保持本地 |
| 取消等待、Stop、关闭 tab、TTL、删除、CLI 退出 | 六种边界不混淆；运行自保活；shell/steer 不阻挡 TTL；退出后资源已释放 |
| timeline/输出/历史操作 | 六条原值、TokenCountSnapshot、nonce 失效和迟到拒绝；当前输出完整 replay；历史旧目标拒绝而非自动重试 |
| Session/global settings CAS | 完整比较、运行期编辑、后台 turn-state/凭据交错、目标字段冲突、回执迟到均不覆盖新值 |
| catalog 与菜单 | 单用户全量快照，running/isActive/date 来自后端；目录不加载 runtime，tab 使用 timestamp timeline |
| 认证、MCP、usage/reset | mock OAuth 全生命周期、账号/来源隔离、MCP 实际配置变化协调及同值导入无额外副作用、显式券选择；前端不访问真实凭据 |
| 通知 Hooks | 无 replay、慢消费丢旧、单订阅/串行命令、四分支范围正确，超时和取消后进程清理 |
| 升级和启动失败 | 两文件重入、旧源最后删除、版本激活与读取切换一致；失败不打开半准备业务数据 |

## 检查与记录

- 运行本批受影响模块及前批相关回归，区分 JVM、本机 Native 实际执行与其他目标仅编译/链接；保留失败、复跑、跳过和原因。
- 对关键交错采用可控时钟/屏障重复测试；不得用一次通过宣称不存在所有竞态，不修复无关既有问题。
- 审查模块依赖与构造入口，证明前端不持真实 repository/runtime/store，旧控制 Hook/Execution/单文件设置已无生产调用；必要移除项限定本次替代范围。
- 使用隔离 Home、mock 网络/模型和测试命令；禁止真实 reset/凭据操作，启动 CLI 不占用用户当前窗口。
- 更新仍标为“迁移目标”的 checklist 适用说明及相关现行 SOP，以实际切换结果为依据；不抹掉保留的历史 migration 规则。
- 在任务记录列出命令、测试数量、报告位置、平台限制和未完成验收；清理临时文件/进程/连接。
- 若某项失败，回到对应子任务完成修正与复验；没有全部通过或明确获准的验收调整，不关闭父任务。

## 完成边界

- 父子任务脚本及所属测试/清理全部结束后才移动到 done，同步全部入站链接。
- 本期不包含 daemon、网络重连、分页、多用户、真实 Home 发布升级；它们不是补齐一期验收的借口或隐含交付承诺。
