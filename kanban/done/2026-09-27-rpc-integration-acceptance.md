# Task Tree

- `Await scheduled execution`()
- `Run the complete in-memory CLI acceptance matrix`()
- `Audit frontend dependency boundaries and obsolete production paths`()
- `Run supported compilation regression and migration checks`()
- `Reconcile current guidance with the implemented architecture`()
- `Close child tasks and the parent only after scoped checks finish`()

# Details

## 位置与前置

- 实施批次 P12；父任务：[实施总计划](2026-09-08-explore-krpc-frontend-backend-separation.md#实施计划)，前置 [P11](2026-09-27-rpc-cli-cutover.md)。
- 状态：Done；P11 经最小 kRPC utils 替换后通过 Native 取消与隔离 CLI smoke 门禁，本批 JVM/Native 与跨目标矩阵已验收。
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

## 执行中记录

- P11 的本地 kRPC utils 快照补丁已通过 JVM/Linux x64 Native 值测试、真实并发取消往返及 Application Native24项；重链 Native CLI 在独占隔离 Home 启动、升级0.4.7并正常退出。补丁源码/许可证在 `Kodex/rpc/krpc-utils-patch`，未来上游贡献的未提交分支在 `~/ACodeSpace/fork/kotlinx-rpc`；未改冻结契约。
- 本批串行运行22个相关模块的测试任务，其中 `app-view-session` 无测试源码；其余21个有报告的模块，JVM 共743项、Linux x64 Native 共729项，均零失败、零跳过。原 History bounded-window 测试在忙碌宿主上三次到15秒时约完成1300–1850次逐项导航；Native 初次载入3.17秒超过原2秒，后来又触及 TestBalloon 默认60秒外层限时与120秒导航保护。该测试现明确设置360秒协程预算、300秒防挂起保护、5秒初次载入上限，保留1000条记录、12行视口与窗口上限；最终 JVM 50项/Native 50项复验通过，长用例分别约23.7秒/271.4秒。生产历史算法未改；该用例是有界性验证，不以跨平台固定吞吐量为验收条件。
- Linux x64 Native 首组在20分钟外层预算到期前完成12个模块的实际测试；视图模型设置测试的相对临时路径和历史索引测试依赖可能被 StateFlow 合并的短暂状态脉冲，分别改为有效绝对路径与已定 cacheNonce 失效输入。两模块 JVM/Native 各7/12项重新通过。其余 Native 视图模块也已完成，前述 729 项统计来自全部21份当前测试报告。
- kRPC 本地 utils 补丁与内存 transport 的 Linux ARM64、Windows x64、macOS ARM64 源码编译均通过；整条 CLI 的 Linux ARM64 与 Windows x64 源码编译通过。`dependencyInsight` 确认 JVM 传递依赖 `kotlinx-rpc-utils:0.10.3` 被替换为本地项目，Native 已在 P11 的真实链接和取消测试验证唯一 KLIB。macOS CLI 在 Linux 主机因既有 Mosaic/process/shell cinterop 限制跳过，仅补丁与内存 transport 完成源码编译，未宣称 macOS CLI 编译、链接或执行。
- 补丁接入后的 CLI JVM 源码及 `integration-test` JVM 测试源码编译通过；integration-test 中需真实网络/模型/凭据的测试未执行，也未将“能编译”记为外部服务集成通过。整体验收实际执行的是上述 mock/隔离矩阵、P11 的完整后端 RPC 往返与隔离 CLI smoke。
- 生产 frontend 模块的 common/mosaic 依赖核对与源码检索没有发现直接引用实体 repository、runtime 实现、旧认证/单文件 store、控制 Hook 或旧 execution；唯一接入后端实现的是负责组合的 `app/viewmodel/application`。CLI Main 通过 `withKodexApplication` 进入一对内存端点。冻结 `rpc/contract`、`rpc/models` 与已知异常文件相对基线无改动；分层检索不代替真实运行测试。
- 验证命令均显式使用 Temurin25 与 `--max-workers=1`；Mosaic Native/CLI 链接加 `--no-configuration-cache`。主要任务为上述22模块的 `jvmTest`/`linuxX64Test`、补丁及内存 transport 的三种交叉目标 `compileKotlin*`、CLI 的 JVM/Linux ARM64/Windows x64 编译、`integration-test:compileTestKotlinJvm` 和 `rpc-in-memory:dependencyInsight`；报告位于各模块 `build/test-results/{jvmTest,linuxX64Test}`。不以跨目标源码编译代替该平台运行；fork 上游完整仓库尚未构建或提 PR，本地等价补丁及真实 kRPC 往返已验证。
- 验证全部使用 mock provider/模型、隔离路径和测试命令；未触及真实 Home、凭据或 reset。多模块原生链接串行执行，不与用户窗口或设备竞争；临时日志及隔离 Home 已清理，无遗留测试 CLI/连接进程。

## 验收结论

- P01–P12 的约定范围已完成：本机 JVM 与 Linux x64 Native 实测矩阵无失败，生产 CLI 隔离启动/退出及旧 Home 升级通过，冻结 RPC contract 未改变。
- Linux ARM64 与 Windows x64 仅完成源码编译；macOS CLI 因既有 Linux-host cinterop 限制未编译。未执行依赖真实网络、模型或凭据的 live integration tests，也未发布、提交、推送或升级真实 Home；这些限制不被写成已通过的验证。
- 上游 fork 的未来贡献分支保留未提交补丁；本地等价 utils 替换已通过实际取消与链接回归，但 fork 全仓构建及上游 PR 不属于本批完成项。

## 完成边界

- 父子任务脚本及所属测试/清理全部结束后才移动到 done，同步全部入站链接。
- 本期不包含 daemon、网络重连、分页、多用户、真实 Home 发布升级；它们不是补齐一期验收的借口或隐含交付承诺。
