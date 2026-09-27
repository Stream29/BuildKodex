# Task Tree

- `Review confirmed designs and committed implementation baseline`()
- `Decompose the remaining implementation into scoped batches`()
- `Review batch feasibility and record unresolved semantic conflicts`()
- `Consolidate cross-batch boundaries in canonical checklists`()
- `Record the finalized RPC contract and mandatory blocker review gate`()
- `Resolve import semantics without changing the RPC contract`()
- **`Await scoped implementation authorization`()**
- `Implement Session settings CAS`()
- `Implement backend Session lifecycle`()
- `Implement backend settings models and MCP`()
- `Implement backend authentication OAuth and usage`()
- `Implement runtime and timeline services`()
- `Implement Session catalog and management`()
- `Implement notification delivery and frontend Hooks`()
- `Implement frontend Session views`()
- `Implement frontend settings and authentication`()
- `Prepare the frozen settings split migration`()
- `Cut over the single CLI and activate migration together`()
- `Run integration acceptance and close the task`()

# Details

## 实施计划

- 2026-09-27：用户确认设计已完整，要求在看板拆分实现计划。本轮只规划，不启动源码实施、不重新审批已定业务语义。
- 父任务及下列十二个子任务均在 planning；每个调用对应独立任务文档，列明依赖、范围、验收与交接。获得实施授权后，才将对应子任务移入 executable 并更新调用位置；创建任务不是自动执行授权。
- 开始规划时两仓工作区干净，均为 refactor/rpc。基线为根仓库 `a830638`、Kodex `e6154848`，根仓库 gitlink 一致；本轮不动 main 或提交代码。
- 已完成、无需重做：八个 RPC 契约及值模型、四种已知异常、内存 JSON 连接、RestoringRpcClient、状态/CAS 初始化封装、两侧 CachedIndexVersioned、统一运行槽位、目录字段、两侧设置 store 与后端文件 CAS。详见[源码范围](#已落地的源码范围)和[主线同步验证](../done/2026-09-27-sync-main-into-rpc-refactor.md)。
- 尚未完成：真实业务服务、后端 Session 所有权/TTL、前端聚合与页面接入、通知运行、文件迁移和实际 CLI 切换。以下计划只安排这些差额，不把契约或值测试通过当成生产接入完成。
- 全部批次遵循 [分批实施边界](../../checklist/rpc-architecture.md#分批实施边界)；本任务只维护调度、具体交付和验收记录，不另立一套架构规则。
- 用户进一步确认当前 RPC contract 已定稿，所有批次适用 [契约冻结门禁](../../checklist/rpc-architecture.md#rpc-契约冻结)。本计划不包含改约授权；必须改约才能推进时，停止实施并重新审阅遗漏的阻塞点，不把改约放入局部实现或后续批次代办。

### 批次与依赖

| 批次 / Task Tree 调用 | 子任务与独立交付 | 前置 |
| --- | --- | --- |
| P01 `Implement Session settings CAS` | [Session settings 同锁 CAS](2026-09-27-rpc-session-settings-cas.md)：补实际写入原语，保留运行期编辑及 turn-state 交错语义 | 已提交基线 |
| P02 `Implement backend Session lifecycle` | [后端 Session 生命周期](2026-09-27-rpc-backend-session-lifecycle.md)：真实 repository、接受操作所有权、保活/失活/清理 | 已提交基线；测试注入领域依赖 |
| P03 `Implement backend settings models and MCP` | [全局设置、模型与 MCP](2026-09-27-rpc-backend-settings-models-mcp.md)：后端资源及完整设置 CAS 的业务接线 | 已提交 store/领域基线 |
| P04 `Implement backend authentication OAuth and usage` | [认证、OAuth 与用量](2026-09-27-rpc-backend-auth-oauth-usage.md)：两文件来源、attempt、usage/reset、MCP logout | P03 |
| P05 `Implement runtime and timeline services` | [运行与 timeline 服务](2026-09-27-rpc-runtime-timeline-services.md)：六条服务、Runtime、currentFlow、自动标题 | P01、P02、P03、P04 |
| P06 `Implement Session catalog and management` | [目录与管理命令](2026-09-27-rpc-session-catalog-management.md)：真实目录日期、创建/管理/fork/批量交接 | P02、P05 |
| P07 `Implement notification delivery and frontend Hooks` | [通知、Hook 与后端装配](2026-09-27-rpc-notification-hooks.md)：补齐通知后组装完整服务，供前端批次真 RPC 验收 | P03、P04、P05、P06 |
| P08 `Implement frontend Session views` | [前端 Session 视图](2026-09-27-rpc-frontend-session-views.md)：保活/恢复、历史和流输出、提交/工具/目录消费者 | P05、P06、P07 |
| P09 `Implement frontend settings and authentication` | [前端设置与认证](2026-09-27-rpc-frontend-settings-auth.md)：两侧设置编辑、字段冲突、OAuth/选券/MCP UI | P03、P04、P08 |
| P10 `Prepare the frozen settings split migration` | [冻结设置迁移](2026-09-27-rpc-settings-split-migration.md)：转换、重入状态表、隔离升级测试，暂不激活 | 已提交两侧文件格式 |
| P11 `Cut over the single CLI and activate migration together` | [CLI 整体切换](2026-09-27-rpc-cli-cutover.md)：完整服务注册、前端工厂、文件读取与迁移激活同步替换 | P01–P10 完成 |
| P12 `Run integration acceptance and close the task` | [整体验收](2026-09-27-rpc-integration-acceptance.md)：真实调用链、资源/数据/平台验证与文档收尾 | P11 |

- 建议按编号串行推进；P01/P02/P03/P10 在逻辑上可独立准备，不据此自动开并行会话或竞争构建资源。Task Tree 是默认执行顺序，表格区分真正的前置关系。
- 先在隔离 fixture 中验证组件，再切生产入口；服务内部拆分不改变一个 GlobalRpc 的对外边界，也不通过假成功/空返回拼出“已实现”的服务。
- 新模块或具体内部类型名称在相应批次实施时沿既有依赖收敛；不为任务拆分强制创建十二个模块或增加 RPC。

### 计划阻塞点审查

- 本轮按用户要求审查可执行性，未开始源码实施。显式依赖表没有环，原分批文字的跨批验收和编译缺口已修订；导入语义冲突经用户澄清后关闭，处置如下。

| 项目 | 影响与处置 | 状态 |
| --- | --- | --- |
| 完整 GlobalRpc 组装放到 P11 太晚 | P08/P09 要求真 RPC，但又不能用半实现 GlobalRpc；改由 P07 汇合全部后端组件并提供隔离装配，P08 依赖 P07，P11 只负责生产选择与退出 | 计划已修订；实现待验收 |
| P08/P09 修改共享接口可能破坏旧 CLI | 原 Application 构造仍依赖实体参数，renderer 消费 execution；新工厂/视图先隔离验证，无法独立保持编译的共享签名/旧消费者删除集中到 P11，逐批编译旧 CLI | 计划已修订；实现待验收 |
| 内存 repository 不是现成 timeline RPC fixture | 原内存 Session 没有 ObservableKodexAgentStorage/cacheNonce；P05 及完整往返改用隔离文件 repository，内存 transport 不等于内存存储，不为测试扩展后端缓存 | 计划已修订；实现待验收 |
| MCP 导入的旧 Replace 副作用 | 用户确认新路径仅更新配置值，不具有独立 Replace 语义；同值不产生变化，不迁移额外 invalidate，无需修改 RPC contract | 已澄清；设计阻塞关闭，运行接线待验收 |

- MCP 的澄清及旧实现依据见 [P03 值更新记录](2026-09-27-rpc-backend-settings-models-mcp.md#已澄清导入仅更新配置)。撤回迁移旧 Replace 额外副作用的要求，不新增 RPC 或改变 CAS；实际配置变化所需的校验、持久化与连接协调继续保留。
- P03 的 models 依赖已认证 client，P04 又依赖 P03 settings/MCP：按组件注入和 P07 汇合处理，P03 测试用 mock client，不反向调用旧 Application 创建资源；这不是要求两个生产 store 同时启动。
- P10/P11 的目标版本尚未指定，是后续激活门禁，不阻止 P01/P02：登记 future entry 不代表当前二进制会执行迁移，必须核对实际生成版本、读取切换和授权后才能激活。
- 当前静态审查记录中的设计阻塞已闭合，未运行编译或交错实验；各批次仍待实施与验收，不据此宣称运行行为已完成或保证未来不会发现遗漏。
- 按用户要求，跨批改动、分层、验证及切换约束已收录 [架构 checklist](../../checklist/rpc-architecture.md#分批实施边界)，[文件切换门禁](../../checklist/rpc-settings.md#文件切换门禁)及 [MCP 接入等价性](../../checklist/rpc-mcp.md#接入等价性边界)分别归领域 checklist；各子任务已链接共同约束。导入阻塞依据用户本次澄清关闭，不是实施阶段自行更改 contract。
- 导入语义澄清后检查：16份文档的326个本地链接/锚点、25处源码范围及任务依赖检查通过；十二批授权状态不变。仅文档修改，Kodex 工作区干净，未运行构建或提交。

### 契约与消费者覆盖

| 已定能力 | 后端批次 | 前端/组合/验收批次 |
| --- | --- | --- |
| GlobalRpc settings Get/Flow/CAS、models Get/Flow | P03 | P09、P11、P12 |
| GlobalRpc authentication Get/Flow、统一 OAuth、removeAuthentication | P04 | P09、P11、P12 |
| GlobalRpc usage Get/Flow、refresh、指定券 reset | P04 | P09、P11、P12 |
| GlobalRpc MCP Get/Flow、reconnect、Codex import Get / logout | P03 / P04 | P09、P11、P12 |
| GlobalRpc catalog、create、createSuggested、keepAlive、archive/unarchive、fork、history fork、delete | P02、P06 | P08、P11、P12 |
| AgentRuntimeRpc 全部状态、输出流与命令 | P02、P05 | P08、P11、P12 |
| 六条 timeline 及 settings CAS | P01、P05 | P08、P09、P12 |
| Notification.Stop 四分支与 getNotificationFlow | P07 | P07 执行器、P09 配置、P11、P12 |
| 两侧文件、宽度临时态、旧 Hook/源文件处理 | P03、P10 | P09、P11、P12 |

### 执行门禁与完成标准

- 本轮计划会产生新的文档改动；进入首个源码批次前，先在用户授权下保存计划并再次核验两仓工作区干净，不把上一轮干净状态永久沿用。
- 每批按 [阶段交付与验证边界](../../checklist/rpc-architecture.md#分批交付与验证)完成其内联验收，记录实际执行命令、数量、跳过原因和未覆盖项，不复制旧结果作为新批通过。
- 默认使用 mock provider/模型、隔离 Home 和测试命令；不读取真实凭据、消费 reset、执行用户 Hook、升级真实 Home 或抢占用户设备/窗口。
- P01–P10 保持现行 CLI 的可用入口；P07 提供完整隔离后端，P08/P09 验证新前端并保持旧入口可编译，P11 成组切换并清理过渡路径，不留两份设置真源或前端实体所有权。
- P11 核对 [生产切换门禁](../../checklist/rpc-architecture.md#生产切换门禁)后才能切入口；相关实际版本与冻结迁移证据由 P10 交接，不能只按批次编号推断已满足条件。
- 每批完成后更新父任务进度、移动该子任务到 done 并修正链接；仅 P12 验收通过后关闭父任务。分批提交须有用户授权，不因任务完成自动提交或推送。
- 发现涉及其他批次的真实冲突时，按 [改动范围与冲突](../../checklist/rpc-architecture.md#改动范围与冲突)同步影响和依赖，不将新的范围调整隐含在当前实现里。

## 历史审查过程

- 下列树保留此前逐项设计/原语落地的经过，未完成的接入项已归入上方 P01–P12；它不再作为当前调度树或实施授权。

<details>
<summary>展开历史逐项审查记录</summary>

- 规划单 CLI 内存 RPC 重构并审查可编译契约
  - [done] 确认一期不独立部署后端，保留单 CLI 分发
  - [done] 恢复取消前的契约、模型与必要序列化
  - [done] 落实首版当前设置契约与 SuspendMutableStateFlow 声明
  - [done] 落实普通泛型父接口与六条 timeline 契约
  - [done] 合并当前设置与 settings timeline 边界
    - [done] 确认只有 settings timeline 对前端可写
    - [done] 确认迁移 CAS 与删除重复服务的范围
    - [done] 迁移 CAS 并删除 AgentSettingsRpc
    - [done] 同步当前设置投影与服务归属
    - [done] 编译验证合并后的生成结构
  - [done] 重建两侧设置模型并落实 GlobalRpc settings/models
  - [done] 确认 notification-only 并从后端设置移除 hooks
  - [done] 确认两种认证来源完整生命周期与对称管理
  - [done] 落实认证摘要、统一 OAuth 与各自 logout 契约
  - [done] 落实 MCP 运行态、重连与 Codex 配置 Get
  - [done] 核对 MCP 设置页的状态观察
    - [done] 核对原 invalidate 与同值 CAS 的差异
    - [done] 撤回改变后端副作用的候选
    - [done] 核对页面消费者与原状态投影
    - [done] 确认既有 Get/Flow 覆盖观察需求
  - [done] 落实 Session 目录 Get 与四个持久化管理命令
  - [done] 将已定设计迁入 checklist
    - [done] 按主题建立迁移目标与现行约束入口
    - [done] 精简任务为引用、缺口和研究验证记录
    - [done] 检查决策覆盖、引用与工作区范围
  - 继续审查 Session 创建与资源生命周期
    - [done] 确认创建与首次提交分开
    - [done] 确认前端周期保活且无 close RPC
    - [done] 修订为后端失活以异常结束已有 Flow
    - [done] 核实 kRPC 取消传播与断线重连职责
    - [done] 确认目录运行布尔值与快照 spinner
    - [done] 确认创建签名、保活命名与目录运行口径
    - [done] 添加创建/保活契约及目录运行字段
    - [done] 验证 Session 契约与目录值测试
    - [done] 落实目录日期模型与两处读取边界
      - [done] 核对菜单消费者与未激活 Session 的读取路径
      - [done] 核对现有 timeline 覆盖并撤回专用读取候选
      - [done] 确认 tab 走 timeline 且目录自带两个日期
      - [done] 确认目录模型改动及必要引用调整
      - [done] 添加目录日期字段与值测试
      - [done] 验证独立模型并记录 RPC/UI 构建边界
      - [done] 依赖恢复后复验 RPC 与目录显示引用
    - [done] 确认显式激活准入与 SessionNotActive 方向
    - [done] 确认失活时已有 Flow 的处理
    - [done] 添加 SessionNotActive 与恢复测试
    - [done] 编译验证 SessionNotActive 的跨平台恢复
    - [done] 确认未消费运行资源的 TTL 回收边界
      - [done] 核对 pendingSteer 与 shell 尾输出的存储归属
      - [done] 整理沿用 owner 的保守回收候选
      - [done] 确认 runningTurn 自保活及 TTL 唯一判据
    - [done] 确认 TTL 参数与前端调用间隔
      - [done] 整理同一期限与双侧续期的时间参数
      - [done] 确认固定默认值与前端保活范围
    - 确认可观察状态、重型 owner 与缓存身份的分离
      - [done] 核对原缓存和窗口代际的作用域
      - [done] 整理重建时分配新代际的候选
      - [done] 确认以内存随机标记替代有序代际
      - [done] 统一 cacheNonce 契约与异常命名
      - [done] 细化缓存标记校验、失效与迟到结果规则
      - [done] 编译验证 cacheNonce 契约及异常恢复
      - [done] 审查 timeline 元数据的事实源
        - [done] 核对缓存与存储原语的现状
        - [done] 整理缓存层发布元数据的候选
        - [done] 确认后端元数据所属层
      - [done] 落实缓存层最小元数据
        - [done] 核对原追加回退与缓存淘汰路径
        - [done] 确认只改内部缓存类与测试
        - [done] 添加只读元数据与提交后发布
        - [done] 验证初值追加回退及缓存淘汰
      - [done] 落实服务读取元数据的只读接口
        - [done] 核对 Session 到缓存的类型链
        - [done] 确认使用只读观察接口
        - [done] 确认接口形状及 CachedIndexVersioned 命名
        - [done] 添加两个只读接口并调整内部实现
        - [done] 验证只读视图与原存储回归
      - [done] 核对读取竞态与最终一致性
        - [done] 核对现有缓存锁与回退发布时序
        - [done] 区分前端暂态与后端迟到加载
        - [done] 临时复现并撤回强制同步入口候选
        - [done] 确认两侧独立实现并收敛只读保证
      - 落实版本标记与前端旧结果拒绝
        - [done] 核对只读接口与远程查询的覆盖
        - [done] 收敛单 timeline 前端缓存的实施候选
        - [done] 确认前端实现与边界测试的授权
        - [done] 实现单绑定视图与缓存失效
        - [done] 验证追加迟到结果及生命周期
        - [done] 同步实际验证与未接入范围
        - 将前端视图接入 Session 与历史消费者
      - [done] 落实幂等 open 与目录活跃标记
        - [done] 核对活跃 Session 查找与原 Job
        - [done] 撤回先注入查找回调的实施顺序
        - [done] 核对 open 幂等性并撤回 getActive 候选
        - [done] 区分目录 isActive 与 running
        - [done] 确认补齐两个标记与单用户全量查询
        - [done] 添加目录字段与原登记表采样映射
        - [done] 验证未打开、空闲、运行及关闭快照
        - [done] 同步全量读取与未来分页边界
  - 补齐其余 RPC 切面
    - [done] 落实账号 usage 读取与刷新
      - [done] 批准三方法与原值最小序列化
      - [done] 添加契约、序列化及五项值测试
      - [done] 编译并验证生成接口与值测试
    - 审查账号 usage reset
      - [done] 核对原 attempt 准备、消费与失效时机
      - [done] 撤回跨 RPC attempt 的候选
      - [done] 确认必须指定重置券并核对现有选择 UI
      - [done] 确认单次 reset 与手动再次操作边界
      - [done] 批准具体契约及原值序列化
      - [done] 添加单次契约及两项结果值测试
      - [done] 编译验证 reset 契约与测试
      - [done] 审查确认期间账号变化的准入
        - [done] 核对旧 attempt 的账号绑定与新参数缺口
        - [done] 确认维持 creditId 并依赖供应商校验
      - 在前端接入时移除自动选券回退并验证日期展示
    - 审查 Agent 命令、运行状态与未落盘流式输出
      - [done] 核对原 AgentRuntime 与前端 ViewModel 的语义差异
      - [done] 审批 pendingSteer 的 Get/Flow/CAS 契约
      - [done] 添加 AgentRuntimeRpc 与原队列值测试
      - [done] 编译验证 Runtime 首批契约
      - [done] 确认 resume 沿用原完整 operation 等待语义
      - [done] 添加 resume 契约声明
      - [done] 编译验证 resume 生成签名
      - [done] 审查 runningTurn 的布尔投影与显式取消
      - [done] 添加运行布尔 Get/Flow 与当前 turn 取消
      - [done] 编译验证运行控制三方法
      - [done] 收敛运行控制与原 state 的流式输出边界
        - [done] 核对每输出项 SharedFlow 的 replay 与索引时序
        - [done] 确认独立流标识与 state Flow 发布
        - [done] 收敛为当前活跃流的随机标记校验
        - [done] 确认前端流切换、过期处理及上游取消
        - [done] 确认 Long 标记、状态模型与取流三方法
        - [done] 添加传输模型、契约与值测试
        - [done] 编译验证 state/currentFlow
      - [done] 批准并添加 forcedCompact 声明
      - [done] 澄清并添加独立 clearPending RPC
      - [done] 验证 forcedCompact/clearPending 契约
      - [done] 追溯消息写入与 markNewTurn 历史差异
      - [done] 核对调用点并撤回 injectHistory RPC 候选
      - [done] 按统一 runningTurn 复核单一用户消息写入
      - [done] 审批 appendUserMessage 契约
      - [done] 添加消息写入声明与原内容值测试
      - [done] 编译验证消息写入生成签名
      - [done] 核对自动标题触发与显式改名协调
      - [done] 确认消息接受后由后端自动触发标题
      - [done] 确认手动与自动改名的字段级冲突策略
      - [done] 确认标题子协程与失活处理
        - [done] 核对原一次性门闩及历史回退恢复
        - [done] 整理按保留历史恢复资格的候选
        - [done] 确认依循 Runtime 作用域且不补生成
      - [done] 核对 execution、tokenCount 与广义 Stop
      - [done] 核实请求取消、异常与 Native 类型边界
      - [done] 确认已知错误联合与未知异常通道分离
      - [done] 审查 currentFlow 与 timeline 的具体错误结果
        - [done] 收敛 currentFlow 的事件与过期二分支候选
        - [done] 核实 Flow 与 NoMatch 外层联合的框架限制
        - [done] 核查异常承载已知错误值的可行性
        - [done] 核对保留字段并整理最小 helper 候选
        - [done] 核实 Native 类名查询与远端类名丢失点
        - [done] 深查解码前扩展、主分支与官方异常讨论
        - [done] 确认专用 message 协议承载已知错误
        - [done] 确认裸 JSON 与序列化库自动异常分派
        - [done] 确认单一泛型异常还原 helper
        - [done] 添加 helper 模块与 commonTest
        - [done] 编译并验证 JVM/Native helper 行为
        - [done] 重命名恢复函数并添加自动编码异常基类
        - [done] 验证基类 message 编码与还原
        - [done] 审查集中 sealed 异常与无显式类型参数方案
        - [done] 确认集中建模及首个 NoMatchException 分支
        - [done] 实现 sealed 基类与无类型参数恢复
        - [done] 验证集中异常的 JVM/Native 行为
        - [done] 审查一期已识别的生产异常分支
          - [done] 核对 timeline 查询与历史命令的代际失败
          - [done] 审批 GenerationMismatch 分支与恢复边界
          - [done] 添加代际异常、恢复测试与契约说明
          - [done] 编译验证代际异常恢复
          - [done] 审查缺失 Session 的失败分支
            - [done] 核对缺失校验与 delete 的 false 返回
            - [done] 确认 SessionNotFound 与保活恢复边界
            - [done] 核对现有 helper 与往返测试的落地范围
            - [done] 确认本项源码与测试实施
            - [done] 添加已批准异常及恢复测试
            - [done] 编译 helper 并运行缺失异常的值测试
            - [done] 依赖恢复后运行新增异常的 RPC 往返测试
          - [done] 审查删除后 Session index 的身份复用
            - [done] 核对原空号分配与 nonce 覆盖范围
            - [done] 确认保留编号复用并撤回水位与迁移方案
        - [done] 确认客户端还原接入与往返验证范围
        - [done] 确认保留顶层事件流并用已知异常拒绝绑定
      - [done] 统一普通运行与手动压缩的 runningTurn 和 Stop
      - [done] 确认两种压缩仅发布 Compacting 并撤回输出流
      - [done] 落实原 runtime 的统一运行槽位
        - [done] 核对外层手动压缩与内层自动压缩的调用路径
        - [done] 收敛不改压缩算法及任务所有权的最小批次
        - [done] 确认运行槽位实现与回归测试的授权
        - [done] 添加外层压缩的同槽位准入与清理
        - [done] 验证手动互斥及自动压缩嵌套
        - [done] 同步运行语义及未接入的所有权边界
      - [done] 审查压缩期间新 steer 的提交
        - [done] 核对 composer 入队与压缩的消费路径
        - [done] 确认保留排队且不补消费的原行为
      - [done] 复核 execution 派生字段与历史操作忙碌期
      - [done] 撤回 execution RPC 并确认废除旧聚合模型
      - [done] 确认目录 running 直接取统一 runningTurn
      - [done] 核对旧 execution 消费者所需的真实原语
        - [done] 核对 Agent 全局 latestIndex 与稀疏 timeline 的区别
        - [done] 审批全局 latestIndex 的 Get/Flow
        - [done] 添加全局索引读取声明
        - [done] 编译验证全局索引生成方法
    - [done] 审查工具交互、草稿与宿主能力
      - [done] 核对 completeToolCall 的两处前端交互调用
      - [done] 审批结果提交契约与原联合序列化
      - [done] 添加 completeToolCall 与三项值测试
      - [done] 编译验证工具结果提交契约
      - [done] 核对 shell 登记表、完成观察与显式关闭
      - [done] 收敛并落实整体 shell 状态与 close
      - [done] 添加 ShellSessionState 与三项值测试
      - [done] 编译验证 shell 契约与模型
      - [done] 核对子任务完成结果所需的 storageUri
      - [done] 审批 storageUri 单一读取契约
      - [done] 添加存储 URI 读取声明
      - [done] 编译验证存储 URI 读取签名
      - [done] 核对子任务命名可复用 settings CAS
      - [done] 确认子任务的自动标题抑制边界
        - [done] 按最新标题生命周期复核实际调用
        - [done] 确认只依赖名称判断并撤回抑制 RPC
      - [done] 审查前端驱动的专属批量创建 RPC
        - [done] 核对创建清理与父工具提交的独立边界
        - [done] 确认前端交互驱动与后端批量执行
        - [done] 核对原参数和批次返回的信息缺口
        - [done] 确认是否一并提交首条消息和启动
        - [done] 精简为任务列表与原 AgentSettings 参数
        - [done] 收敛返回模型及未知失败边界
        - [done] 添加专属契约、返回模型与值测试
        - [done] 编译验证批量创建契约
      - [done] 审查工作目录选择的文件系统归属
        - [done] 核对 cwd 选择入口与原目录浏览器
        - [done] 区分前端交互与目录数据读取
        - [done] 确认一期仍由前端本地读取
    - [done] 审查历史回退与边界 fork
      - [done] 核对前端菜单、原操作与代际来源
      - [done] 审批两项命令及后端代际绑定
      - [done] 添加历史命令与已定设计
      - [done] 编译验证历史操作契约
    - [done] 确认一期 OAuth 句柄保留与取消边界
      - [done] 核对一期准备阶段取消与句柄交付
      - [done] 确认待回调登录的失效策略
      - [done] 细化过期时长与前端等待清理
    - [done] 确认 Codex 认证存储的兼容范围
      - [done] 核对原文件读取与官方存储模式
      - [done] 整理文件完整生命周期与新增存储支持的区别
      - [done] 确认一期仅覆盖固定 auth.json
    - [done] 确认刷新成功但回存失败的处理
      - [done] 核对原写入发布顺序与维护重试
      - [done] 确认沿用原失败路径且不保留待回存状态
    - [done] 审查认证页面的原 Reload 入口
      - [done] 核对按钮、原 store 命令与测试
      - [done] 区分摘要观察与凭据重新加载
      - [done] 确认移除前端手动入口且不新增 RPC
  - 完善迁移接入与验证计划
    - [done] 复核剩余一期协议阻塞
      - [done] 核对工具提交的本地结果与远程命令
      - [done] 核对设置页观察与后台刷新所有权
      - [done] 核对 Hook 清理所需的原进程接口
      - [done] 清理已解决的待审节点并区分接线缺口
    - [done] 落实独立两侧设置 store
      - [done] 核对原加载器与新模型的依赖
      - [done] 核对迁移激活与旧 CLI 读取的冲突
      - [done] 整理独立 store 范围与临时目录测试
      - [done] 确认两侧 store 的源码实施授权
      - [done] 添加独立文件加载与串行更新
      - [done] 验证默认值隔离失败及取消清理
      - [done] 同步验证与未启用迁移的边界
    - [done] 落实后端设置存储的比较写入
      - [done] 核对全量 CAS 与原配置写入的共享边界
      - [done] 区分文件比较原语与业务副作用接入
      - [done] 确认最小文件 CAS 的实施范围
      - [done] 添加同锁比较与零写入语义
      - [done] 验证 CAS 与 update/reload 的交错
      - [done] 同步失败边界与业务接线待办
    - 确认内存 RPC 接入与单进程宿主生命周期
      - [done] 核对现行 CLI 组合根及测试通道的所有权
      - [done] 整理共享连接与两侧生命周期候选
      - [done] 确认连接终止时的宿主处理
      - [done] 收敛独立内存连接模块的最小实施范围
      - [done] 确认生产连接模块及生命周期测试的实施授权
      - [done] 实现 scoped 连接及端点终止传播
      - [done] 添加隔离与资源清理测试
      - [done] 编译并运行 JVM/Native 生命周期测试
      - 核对共享 repository 的所有权接入
        - [done] 核对原工厂与独立 owner 的创建
        - [done] 核对 tab、目录及临时操作的关闭路径
        - [done] 区分标记已落地与宿主尚未共享
        - [done] 明确后端实体与前端 RPC 视图
        - [done] 核对原前端工厂的实体类型依赖
        - 收敛服务与前端所有权迁移的实施范围
      - 将连接模块接入单 CLI 组合根
    - [done] 确认设置、Hooks、TTL 与 reset 同属一期交付
    - [done] 确认配置目录、旧 settings.yml 迁移与默认值加载
      - [done] 核对现有设置目录、宽松加载与 Home 迁移入口
      - [done] 确认两侧文件目录及正常加载规则
      - [done] 确认旧字段、Hooks 与迁移中断的处理
        - [done] 核对旧字段与脚本输入的转换差异
        - [done] 确认新 hooks 为空且完成后删除旧源文件
        - [done] 明确两文件转换的重入状态
          - [done] 整理源文件最后删除的重入状态表
          - [done] 确认冲突和歧义状态的拒绝规则
    - [done] 确认一期前端通知事件与配置规则
      - [done] 核对旧错误通知来源与已有异常通道
      - [done] 确认后端生产联合事件与单一通知 Flow
      - [done] 确认通知 replay=0 且不补播
      - [done] 确认通知事件分支、字段和投递语义
        - [done] 核对七类原 Hook 的触发点与纯值载荷
        - [done] 裁剪为停止与错误相关通知
        - [done] 确认统一 Stop 与四种原因分支
        - [done] 确认保留外层 Notification.Stop
        - [done] 核对具体载荷、复用类型及全局错误差异
        - [done] 确认四分支载荷与多项宿主请求处理
        - [done] 添加通知契约与联合值模型
        - [done] 添加通知序列化与校验测试
        - [done] 编译验证通知契约与值模型
        - [done] 核对无 replay 与慢消费者背压的区别
        - [done] 确认通知缓冲溢出的投递取舍
        - [done] 确认缓冲容量与前端 Hook 执行资源边界
          - [done] 核对旧命令并发、超时与进程清理
          - [done] 整理有限缓冲与串行执行候选
          - [done] 审批容量、执行预算及失败处理
      - [done] 确认前端通知配置与命令输入
        - [done] 核对原配置、通知编码与本地进程能力
        - [done] 整理精确匹配、JSON 输入与本地环境候选
        - [done] 确认单个 Hook 用 types 选择多种通知
        - [done] 审批配置模型、输入与执行环境
        - [done] 添加多类型 Hook 模型及前端设置字段
        - [done] 添加配置序列化与合法性测试
        - [done] 编译验证并同步配置设计
    - [done] 确认客户端 CAS 等待与订阅基础设施
      - [done] 核对值合并与失败后等待的进展缺口
      - [done] 确认无新订阅值时的 CAS 重试边界
      - [done] 同步有节制重试规则与契约注释
      - [done] 核对统一异常及 Flow 保护的客户端入口
      - [done] 确认公共 RpcClient 适配及测试的实施范围
      - [done] 实现公共异常还原与 Flow 保护
      - [done] 添加委托边界和真实 RPC 往返测试
      - [done] 编译并运行 JVM/Native 客户端验证
      - [done] 落实本地挂起 StateFlow 的最小封装
        - [done] 核对原库更新扩展与异步本地值的区别
        - [done] 确认只读委托、CAS 回调及更新扩展的实施范围
        - [done] 添加最小封装与三个更新扩展
        - [done] 验证50ms等待、取消与比较结果返回
      - [done] 落实只读状态初始化与订阅归属
        - [done] 核对原库 stateIn 与既定 Get/Flow 顺序
        - [done] 收敛最小接入与实施范围
        - [done] 确认提取初始化工厂及测试
        - [done] 添加薄工厂和共享订阅测试
        - [done] 编译并验证初始化与 owner 释放
    - 列明内存 RPC 及单 CLI 的运行回归范围
      - [done] 核对测试专用内存 transport 与框架公开入口
      - [done] 确认最小真实 RPC 往返测试的实施范围
      - [done] 添加测试专用通道与异常/取消 fixture
      - [done] 运行 JVM 与本机 Native 往返验证
      - [done] 复现普通下游异常的订阅释放缺口
      - [done] 验证零缓冲协程边界的取消效果
      - [done] 确认生产客户端的 Flow 保护边界
  - [done] 分批保存当前重构检查点
    - [done] 核对两仓改动并建立 refactor/rpc
    - [done] 按职责提交已实现源码
    - [done] 提交设计、任务记录与子模块指针
  - `Sync current main into refactor/rpc`()
  - `Review main-state compatibility and Session settings consumers`()
  - `Distinguish existing CAS contracts from implementation gaps`()
  - `Review submission, recovery and disposal contract coverage`()
  - `Hand off remaining integration work to the implementation plan`()

</details>

- 以下专题保留审查依据、历史批准范围及分批验证；其中“本轮”“候选”“待接入”以当时记录为准。当前调度和范围以上方[实施计划](#实施计划)及下方[状态与授权](#状态与授权)为准，不从历史提问重新推导待决事项。

## 恢复工作与主线同步

- 用户批准先合入两仓最新 main，处理必要兼容并验证后保存合并提交，再继续接入审查；已完成[主线同步子任务](../done/2026-09-27-sync-main-into-rpc-refactor.md)。
- 此前分批提交与验证是旧基线的检查点，不等于已适配主线的 TokenCountSnapshot、turn-state 和客户端签名。
- Kodex 的新合并检查点为 `e6154848`：token-count RPC 与缓存观察接口已复用原 TokenCountSnapshot；JVM424项、Native422项回归以及直接受影响四模块两端各200项强制重跑通过。跨目标与 CLI 编译范围、macOS 跳过见子任务。
- 本轮完成后留在两侧 refactor/rpc；main 保留用户暂停期间的新提交，不重写也不推送。

## 本轮复核：提交、恢复与释放

- 按用户要求继续设计审查，不把已定 CAS、宿主或服务尚未实现重新包装成需要选择的协议阻塞。本轮沿真实调用点核对，未发现新增 RPC 签名、值模型或业务语义选择。

| 路径与原实现依据 | 既有设计覆盖与迁移检查 |
| --- | --- |
| [普通消息提交](../../Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt#L200-L231)、[append 后启动运行](../../Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt#L524-L545) | 复用 appendUserMessage、pendingSteer CAS 和 resume；前端提交反馈与完整运行等待分开，不增加 submit/start RPC。运行中仍按已定规则入队，不借迁移补消费。 |
| [新 Session 物化](../../Kodex/app/viewmodel/new-session/src/commonMain/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModel.kt#L115-L140) | 原 catch 会 rollbackCreated；按已定创建/提交分离替换这条编排，创建成功后后续失败不自动删除，也不因丢失回执盲目再次创建或追加。不是新提出的回滚政策。 |
| [用户回答](../../Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/RequestUserInputViewModel.kt#L83-L146)、[子任务确认](../../Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskViewModel.kt#L48-L112) | 草稿/revision 留前端；completeToolCall、专属批量创建及后续 resume 已覆盖。父工具结果与孩子执行不合成事务，失败不自动重放批次，不增加草稿 RPC。 |
| [关闭 tab](../../Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L365-L382)、[显式删除](../../Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L430-L448) | 按已定资源边界替换 release 的后端关闭责任；本地关闭、Stop、删除仍是不同操作。导航更新留前端，不增加 closeSession 或删除结果推送。 |
| [单 timeline 绑定失活](../../Kodex/rpc/client/src/commonMain/kotlin/io/github/stream29/kodex/rpc/client/RpcCachedIndexVersioned.kt#L178-L182) | 现有缓存会失效，但不自行保活重订阅；按已定规则由仍存活的前端视图恢复。SessionNotFound 停止缺失绑定的自动恢复，关闭页面不重建，恢复不重放写命令。外层聚合接入仍未实现。 |
| [MCP 登录取消与 logout](../../Kodex/mcp/impl/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt#L245-L277) | 已定统一 OAuth 区分取消请求、后台清理和凭据移除；cancel 返回不证明清理完成，logout 仍需后端准入。原 Authorizing 状态可经现有 MCP Flow 观察，但不能把展示状态当作清理完成屏障，不新增清理确认 RPC。 |

- 对应规则分别以 [Session](../../checklist/rpc-session.md)、[AgentRuntime](../../checklist/rpc-agent-runtime.md)、[状态恢复](../../checklist/rpc-state.md#订阅释放与重连)和 [OAuth](../../checklist/rpc-authentication.md#提交取消与退出)为准；表格只记录映射及待验收点，不新增业务保证。
- 本轮结论：当前已审路径没有剩余的新增设计选择，后续工作归入下方[实施组](#本轮复核协议覆盖与实施前置)，不为推进轮次继续制造接口或重复审批既有决定。尚不能据此宣称所有运行竞态已排除、CLI 已迁移或一期验收完成。
- 本轮仅读取源码、既有测试和设计，更新任务记录；未运行测试、启动服务、操作真实配置或修改生产源码。不另行发起后端宿主/整体接入的实施授权，等待用户确定后续工作范围。

## 已核对：Session settings 的实际比较边界

- 用户要求继续审查阻塞点，而非立即实施后端宿主；该候选暂缓。本轮只核对代码与现有约定，不把审查转为源码实施授权。
- 用户指出后端 CAS 已有、应由前端正确使用。复核区分：原子 CAS 已是 SettingsTimelineRpc 的确定契约，不是新的设计阻塞；全局 BackendSettingsStore 已有文件 CAS，Session settings 目前仍只有 RPC 声明和底层无条件更新，公共客户端封装也只是委托 CAS 回调。下列内容保留为实现接线检查，不再据此要求重新选择 CAS 归属或协议。
- 主线新增 turn-state 不破坏 append-only 缓存：响应头通过 [appendTurnStateIfAbsent](../../Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L526-L539)在最新全局位置之后追加 settings/timestamp，并发布 latestIndex；不是覆盖原 exact 记录。token-count 同样追加原 TokenCountSnapshot，上一轮已完成契约兼容，无需新的 RPC 或标记。
- turn-state 仍是[原 Session 协议状态](../../checklist/codex-turn-state.md)，不是可编辑默认值；前端改 model/threadName 等字段时，按已定[目标字段冲突规则](../../checklist/rpc-state.md#settings-的字段冲突)保留最新其他字段。后台追加改变完整 expect 时可比较失败，不因此增加 revision、忽略内部字段或清空 turn-state。
- **后端接入前置**：SettingsTimelineRpc.compareAndSet 已有声明，但 AgentState 目前只有无条件的 [updateSettings](../../Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L427-L435)。在 RPC 层先读取比较、再调用该方法，中间仍可插入后台写入；服务自己的 Mutex 不能覆盖响应头等直接进入 AgentState 的写入。
- 例如读取 A 并比较成功后，[响应头写入](../../Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L526-L539)追加含 turn-state 的 B，随后无条件更新提交由 A 构造的值，就不是已承诺的完整值 CAS。这是错误接法的静态时序推导，不是已运行 RPC 服务的故障。
- 也不能简单套 `modify`：其 [mutate 准入](../../Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L672-L696)拒绝非稳定状态，而原 [运行期设置测试](../../Kodex/agent-state/impl/src/commonTest/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImplTest.kt#L953-L1003)明确保留响应期间提交设置、当前请求使用旧快照、下一请求使用新值的行为。不能为接 CAS 禁止这类编辑，或在非重入锁内再次调用 updateSettings。
- 最小接入方向是在 AgentState 原写入边界提供完整 expect/update 比较并追加的能力，再供既有 SettingsTimelineRpc 委托；不公开 Mutex，不改缓存算法或扩大 modify 准入。具体内部 API 及源码尚未实施。全局 BackendSettingsStore 的文件 CAS 已落地，但它操作另一份设置，不能当作 Session settings 原语已经完成。
- **前端配套接入**：旧 [SessionSettingsDataSource](../../Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/SessionSettingsDataSource.kt#L49-L87)先校验本地 revision，再调用字段命令，最后以 configurationOverride/nameOverride 主动发布提交值；原 [AgentRuntimeViewModel](../../Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt#L454-L475)也直接回写 mutableSettings。这些路径不能机械包一层 RPC 后保留：持久化 Session 应等待已有 timeline 投影更新，CAS 回执不覆盖已收到的新状态。本地草稿仍可直接编辑，弹窗目标/revision 不变成远端版本协议。
- 标题和 plan 的原扩展也经过 updateSettings，已核对[原调用](../../Kodex/agent-state/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentStateExtensions.kt#L13-L39)；本轮不顺带修改其既有先读后写行为，也不把 plan 与工具完成改成新事务。已批准的标题 CAS 迁移仍须使用同一实际比较边界。
- 后续接入验收需覆盖：运行期 CAS 保留 RequestResponse；响应头先提交后旧 expect 返回 false；比较失败/同值成功不追加 settings 或 timestamp；成功追加保留旧 exact 值；目标字段重试保留新 turn-state；回执迟到不覆盖订阅新值。当前值测试和旧无条件 update 测试不能替代这些验收。
- 本组未发现需要新增协议或重新选择后端归属的事项；阻塞是上述原子原语与消费者尚未接线。仅做静态审查及任务更新，未修改源码、运行竞态实验或重跑构建；不以合并前的测试结果声称这些未实现路径通过。

## 暂缓候选：后端 Session 宿主

- 此处保留上轮暂缓经过；本轮已拆为 [P02 后端生命周期](2026-09-27-rpc-backend-session-lifecycle.md)，服务与 CLI 分别归 P05/P06/P11，不再把这段候选作为单个整体实施批次。
- 本轮继续对照主线核查，没有发现需要为 TokenCountSnapshot 新增 RPC 方法；当前主要阻塞是已定后端所有权尚未接线，不重新讨论原语归属。
- [Application 原工厂](../../Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L339-L341)仍按前端 owner 创建 repository；[运行槽位](../../Kodex/agent-runtime/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L180-L189)仍登记当前调用 Job。直接将服务方法委托给该 runtime，不能满足取消等待与已接受执行的隔离。
- 下一批建议落实独立后端 Session 宿主及测试：后端创建和持有真实 repository，复用其幂等 open；承接已接受执行的 Job；落实60/20秒保活、运行自续期、失活与宿主关闭清理。
- 实体及目录标记仍以同一 repository 为来源；不另造前端实体代理或第二份活跃真源，不新增 getActive/closeSession RPC，不改存储和压缩算法、steer/shell 保活规则。
- 验收覆盖重复保活同实例、取消等待不停止运行、显式 Stop、无前端长运行、最后续期到期关闭、目录活跃快照、清理后重建及宿主退出；优先内存/隔离文件 fixture，使用 mock 模型调用。
- 该批暂不切换 CLI、设置 migration 或前端工厂，不以占位成功实现未完成的 GlobalRpc。具体宿主/服务接入的代码实施尚待用户授权；不是本次主线合并已完成的功能。

## 状态与授权

- 当前状态为 Planning。用户确认设计已完整并要求拆分实现计划；本轮建立 P01–P12，不开始源码实施，不因看板存在而自动进入 executable。
- 上轮三份文档已按授权分批提交，开始规划时两仓干净；用户本次另行授权分批提交当前 checklist 与实施计划，并在提交后核验开工条件。该授权不包含源码实施或推送，不能沿用提交前的状态代替最终干净检查。
- 最新一期范围：单一 CLI 分发不变，同进程内部隐藏 frontend/backend，经 in-memory RPC 通信；独立进程、daemon、多 CLI 共享后端及网络部署留未来。具体边界以 [一期部署与交付](../../checklist/rpc-architecture.md#一期部署与交付)为准。
- 历史独立授权已完成的原语、缓存、模型、文件 store 等保留为[实现基线](#已落地的源码范围)，不是待重新实现的任务；具体批准与验证见各专题。
- 后端 CAS、资源所有权、只读最终一致性、两侧缓存差异及业务协议均已定；后续缺口按实施接线处理，不重新要求用户选择归属。
- 连续推进至真实阻塞点的协作约定继续有效，不反复询问“是否继续”；有业务范围、数据保留或源码授权的实质变化才停下确认。
- 不借重构增加未获批功能、网络工程或修复既有行为；一期已接受的 Hooks、TTL、设置拆分、reset 等调整必须交付，不因本约束后移。
- 不修改 Draft、其他任务、真实配置或凭据；具体范围与验证门禁见各子任务。
- 两仓留在 refactor/rpc，不改 main、不推送。当前源码检查点为 `e6154848`；早期 `92b572d4`、回切 main 及逐轮版本信息仅是历史记录，不作为本轮基线。
- 复用仓库锁定的 Kotlin、协程、序列化和 kRPC 版本，不借实施计划升级依赖或重新选择网络 transport。

## 已定设计的唯一记录

- [RPC 总边界](../../checklist/rpc-architecture.md)：单 CLI 内存 RPC 一期范围、后续独立部署、模型复用与服务分组。
- [状态与本地投影](../../checklist/rpc-state.md)：Get/GetFlow、完整 CAS、SuspendMutableStateFlow、订阅与重连职责。
- [timeline 与缓存](../../checklist/rpc-timeline.md)：六条 timeline 的代际读取、settings 特例 CAS、前端缓存和索引优化范围。
- [设置与通知](../../checklist/rpc-settings.md)：两个文件、字段拆分、完整 BackendSettings、models 与 notification-only。
- [认证与 OAuth](../../checklist/rpc-authentication.md)：两种来源加载/续期/回存、统一登录、前端回调与分别退出。
- [MCP](../../checklist/rpc-mcp.md)：运行态、重连、Codex 单一读取与前端预览/CAS。
- [账号用量](../../checklist/rpc-account-usage.md)：只读状态、显式 provider 刷新与账号隔离。
- [Session](../../checklist/rpc-session.md)：目录快照、运行标记、管理命令、创建分离与 TTL。
- [AgentRuntime](../../checklist/rpc-agent-runtime.md)：pendingSteer 原队列的 Get/Flow/CAS。
- 上述文件是迁移目标，不是已完成实现报告；原 checklist 已添加适用范围链接，继续约束尚未迁移的进程内代码。

## 已落地的源码范围

- 契约目录：`Kodex/rpc/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/`；模型目录：`Kodex/rpc/models/src/commonMain/kotlin/io/github/stream29/kodex/rpc/models/`。
- 现行 CLI/Application 尚未调用 RPC 或新 store；已有独立内存连接、公共异常/Flow 适配、本地 CAS 更新扩展、单 timeline 前端缓存及测试服务，但没有业务 RPC service、Session/Agent 聚合投影或界面接入。settings/filesystem 为两个新 store 增加 rpc/models 依赖，使模型间接进入旧调用方依赖图，不等于启动服务或切换设置文件。

| 已落地项 | 当前形状与进度 |
| --- | --- |
| SettingsTimelineRpc 当前设置 | 原当前值 CAS 已迁入；独立 AgentSettingsRpc 已删除，当前值从 timeline 重建 |
| AgentRuntimeRpc | storageUri Get；latestIndex Get/Flow；state Get/Flow/currentFlow；appendUserMessage/resume/forcedCompact/clearPending/completeToolCall/revertHistory；shell Get/Flow/close；runningTurn Get/Flow/cancel；pendingSteer Get/Flow/CAS，共二十一方法 |
| ShellSessionState | 整体进程状态 Map 的值，原 ExecCommandArguments 加 completed Boolean |
| AgentStateValue | 保留原分支与 ToolPending 原事件，五种输出变体以 Long nonce 替换 SharedFlow |
| SuspendMutableStateFlow | 原接口由 rpc/client 的 StateFlow 委托实现；三个更新扩展在比较失败后等待50ms，不回写本地值 |
| TimelineRpc 与六个服务 | 普通只读父接口；每条显式 override 10 个读取方法，含 getCacheNonce/getCacheNonceFlow；SettingsTimelineRpc 额外声明 CAS，共 11 方法 |
| BackendSettings | 六字段、完整 MCP 配置；CliFrontendSettings/CliSidebarSettings 已拆分，无宽度 |
| 两侧设置 store | 原 settings/filesystem 新增 BackendSettingsStore/CliFrontendSettingsStore，独立文件加载、只读状态和串行更新；后端另有同锁文件 CAS，未接入 CLI、完整业务副作用或 migration |
| OAuthTarget/OAuthAuthorization | 两种目标、attemptId/url；不使用前端 effect 类型 |
| Notification/Stop | 四分支携带 Session index 及获批事件载荷，原请求按类型聚合 |
| GlobalRpc settings/models | 五方法 |
| GlobalRpc 通知 | 单一 getNotificationFlow，replay=0，无初值 Get |
| GlobalRpc 认证/OAuth | 认证 Get/Flow、统一 start/complete/cancel、removeAuthentication |
| GlobalRpc usage | getAccountUsage/getAccountUsageFlow/refreshAccountUsage/consumeUsageReset |
| GlobalRpc MCP | 运行状态 Get/Flow、reconnect、logout、getCodexMcpSettings |
| GlobalRpc Session | getSessionCatalog、createSession、createSuggestedSessions、keepSessionAlive、archive、unarchive、fork、forkSessionHistory、delete；目录含 running/isActive |
| SessionCatalogEntry | 原条目新增 createdAt，lastActivityAt 改名 updatedAt；日期皆可空，实际 createdAt 填充及菜单快照接入未实现 |
| CreatedSuggestedSession | 批次结果项，只补 sessionIndex 并复用原 SuggestedSessionMeta |
| rpc/in-memory | 单一 scoped JSON 连接，注册服务后在 block 内使用 client；17项独立测试，不接入现行 CLI |
| rpc/client | RestoringRpcClient、本地 StateFlow/CAS、rpcStateIn 及单 timeline 前端缓存；84项测试（新增29项缓存/生命周期测试），不接入现行 CLI |
| CachedIndexVersioned 元数据 | 公开只读接口与 ObservableKodexAgentStorage 已落地；internal CachedIndexVersionedImpl 维护原两项状态，RPC 校验及服务组装未接入 |

- GlobalRpc 当前共 30 个方法；创建/批量创建/保活、通知 RPC 与 SessionCatalogEntry.running 已添加，尚无服务接入、事件发布或 TTL 实现。
- 必要序列化增量限于实际使用的设置嵌套类型、认证摘要/枚举、MCP 运行值/导入候选、SessionCatalogEntry；没有序列化原始 OpenAI 请求凭据或整个 ViewModel。
- 已撤回旧 GlobalSettings 公开投影、rpc/model 单数模块、两类前端授权 effect 的临时序列化、独立模型目录服务、整窗 HistoryIndexRpc、分离 OAuth/await 方法、后端 MCP preview/apply 与目录 Flow 草案。

## 下一步待审与落地缺口

- 本节沿用原标题保存研究与验收证据；已定事项不再待审。剩余工作现按 P01–P12 调度，旧局部“待实施”不表示新增独立任务或授权。

### Session

- 用户明确当前前端暂不需要传初始标题，创建命名按 [Session 创建边界](../../checklist/rpc-session.md#创建与前端关闭)收敛；撤回 useDefaultThreadName 与空串表示命名意图的候选。
- 保活命名已确认为 `keepSessionAlive(sessionIndex)`；一期固定 TTL 60 秒、两侧每 20 秒续期，不增加参数配置；实际调度、过期竞态和清理仍待运行接入验证。
- 用户已改为 keepSessionAlive 显式激活后才能访问 live 属性，撤回普通访问 cache miss 按需加载。仍需落实激活/回收与同时到达操作的协调，不能同时重建两个占用同一 Session 资源的 owner。
- 失活时结束旧订阅、关闭重型 owner 的规则已确认，不再选择保留轻量上游；仍需将原 shutdown 的取消/等待与后端服务失活信号一起接入。
- owner 重建的版本身份已选择内存随机标记，撤回全局计数器和保留轻量代际真源的候选；后端元数据与前端旧结果拒绝已分别实现，尚未经业务服务连接到 Session 消费者。
- 目录 running/isActive 字段及原 repository 的采样映射已落地，分别读取 runningTurn 和 owner 存活状态；手动压缩槽位随后已统一，实际 RPC 服务与 TTL 尚未接入，不增加全任务聚合。
- delete 造成的旧订阅/引用失效与 TTL 卸载不是同一情形，其失败传播还需落实。

#### 已落地：目录日期模型

- 用户明确两种入口分流，规则收录[菜单日期](../../checklist/rpc-session.md#菜单日期)：已打开 tab 走 timestamp timeline，Sessions view 直接展示 catalog 的 createdAt/updatedAt。撤回专用日期 RPC、菜单临时激活及放宽 inactive timeline 准入的候选。
- 原目录菜单每次右键单独读取的调用在 `Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt:1054-1066`；迁移后改为目录快照，日期随整个 catalog 刷新。这是用户本次明确的读取边界，不改变独立时间戳任务或当前运行实现。
- 原 repository 不打开 runtime 即可精确读取 timestamp[0] 或最新时间（`Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt:86-102`）；因此无需激活整个目录，也无需新增日期 RPC。
- 用户批准并落实原模型 createdAt: Instant? 和 updatedAt: Instant?，均默认 null；旧 lastActivityAt 字段已移除，无兼容别名或重复序列化字段（`Kodex/app/contract/session-catalog/src/commonMain/kotlin/io/github/stream29/kodex/app/sessioncatalog/contract/SessionCatalogEntry.kt:8-22`）。
- 该批次构造仅机械改为 updatedAt = entry.lastActivityAt，repository 日期字段及排序不变；后续活跃标记批次补齐两项布尔映射，当前见 `Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt:606-623`。目录相对时间标签及其测试同步改名，测试使用不同的 createdAt/updatedAt，原预期仍为更新时间。GlobalRpc 仅补注释，仍30方法，没有新增日期 RPC。
- 目录模块新增三项值测试，覆盖六字段 JSON、纳秒时间往返、两日期独立可空/缺省和列表值/顺序；仅 commonTest 增加既有版本 serialization-json。RPC 原目录值测试同步字段名，不新增生产依赖。
- 环境核查发现旧 Daemon 已退出、无可复用实例；通过现有 IDEA 进程定位源码后，显式使用原 Temurin 25 路径启动 Gradle 9.5.1 Daemon 208224，没有切换 JDK/设备或改动其他项目。
- 验证：独立目录模块 JVM、Linux x64/ARM64、Windows x64、macOS ARM64 库编译实际通过；JVM 和本机 Linux x64 Native 各4项测试通过（新增3项），零失败/跳过。构建40秒成功，35任务中12执行、23 up-to-date，配置缓存保存。
- 初次验证时 RPC/UI 被 utils-coroutines 临时 main.kt 的编译错误阻塞；公共客户端批次核实该文件已不存在后，按原待办实际补跑，未修改或排除该文件。
- 补充验证：rpc-contract 四目标编译、JVM/Native 各57项测试通过（macOS RPC 因既有依赖跳过）；app-viewmodel-session 与 app-view-application 的 JVM 编译、后者测试源码编译通过；按原测试名过滤的目录更新时间标签测试实际1项通过。没有重跑全部 UI 测试或进行实际菜单交互验证。
- 本组未实现 createdAt 的实际填充或菜单接入；原目录构造继续使用其默认 null，现行菜单仍沿原日期读取方法运行，不能将值模型通过当成界面已迁移。两处运行接入留后续获准阶段。

FIXME:

#### 已定：运行自保活与 TTL 唯一判据

- 用户明确 TTL 是 Session 生死的唯一指标，后端 runningTurn 自行保活，steer/shell 不参与；确定规则见[保活与回收](../../checklist/rpc-session.md#保活与回收)。撤回以队列或登记表非空阻止卸载的候选，以及旧“只有前端续期、所有在用任务都阻止回收”的笼统约束。
- 运行中的 Session 不因前端关闭而自动卸载，由后端续期保障；未来独立后端可继续运行。一期仍是单 CLI 同进程，整个进程退出清理两侧，不因该决策扩大独立部署范围。
- 原 pendingSteer 在构建 runtime 时初始化为空，消费以 getAndUpdate 清空后注入历史（`Kodex/agent-runtime/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt:71-86,108-131`）；没有从存储恢复该队列的路径。取消 resume 清理的是 pending tools，不是 steer 队列；外层 finally 释放 runningTurn（同文件 :154-191）。不能据正常消费路径假定所有退出时队列必空。
- shell 的 activeSessions 明确保留 completed=true 但未读最终结果的项目；outputFor 读到 exitCode 才移除，close 不移除登记，discardInactiveSessions 也保留 exitCode 已完成项（`Kodex/tool/unified-exec/impl/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolClient.kt:47-57,123-143,176-180,235-248`）。重建 owner 不能从已持久化历史恢复尚未交付的尾输出或原进程句柄。
- 原测试明确 close 后 completed=true 仍保留同一实例，writeStdin 读取最终结果后才移除（`Kodex/tool/unified-exec/impl/src/commonTest/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolsTest.kt:579-606`）。本轮只查源码和测试，没有运行 TTL 或进程实验。
- 用户接受无人消费 shell 不阻止 TTL 关闭，不因尾输出未读而延长驻留；残留 steer 应排查消费/退出逻辑，不作为新的持久化或独立保留需求。撤回拆分队列/进程资源生命周期的替代候选。
- 到期关闭仍要协调写入事务、清理及同 index owner 重建；这不是额外续期或无限等待消费的理由。具体取消/提交交错留运行接入验证，不能用 TTL 允许破坏半个存储事务。
- 原 CachedValueTtl=60.seconds 只作用于 timeline 值缓存，且 expireAfterAccess 会因读取续期（`Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt:118-144,309`）；不是本次 Session 保活 TTL 的现成参数或实现。
- 已同步 Session/Runtime/架构 checklist 及 GlobalRpc.keepSessionAlive、resume/forcedCompact 注释；未改签名、运行代码或原 shell/steer 实现。TTL 和续期间隔已在下节确认，不增加引用计数或 busy DTO。
- 验证：复用已打开的 IDEA 导航后修改注释；Global 29/Runtime 21 方法及剥离注释后的源码哈希不变，四份文档链接/锚点、任务结构和两仓空白检查通过。本轮无可执行行为改动，未重复编译或运行测试，实际自保活与到期清理仍待接入。

FIXME:

#### 已定：保活时间参数

- 用户已确认一期 TTL 60 秒、两侧每 20 秒续期，前端覆盖全部已打开持久化 tab；规则见[保活参数与范围](../../checklist/rpc-session.md#保活参数与范围)。不新增参数字段、返回 DTO 或批量 RPC，数值与原 timeline 值缓存的 60 秒无共享含义。
- 接入验证覆盖：无前端的长运行持续保活、后台 tab、停止所有续期后失活、steer/shell 不阻挡、旧 turn 续期任务退出、到期/续期竞态及单调时钟；异常恢复仍复用 SessionNotActive/cacheNonce，不盲重放结果不明的写请求。
- 本组仅固化设计，不实现 timer/cache 或迁移前端 tab，不修改方法签名；未运行实际保活/回收测试。按用户要求继续审查标题资格等剩余接入边界，不以参数确认代表运行接入完成。

FIXME:

#### 已定：timeline 内存随机缓存标记

- 原缓存实现没有 generation；后续已加入随机 cacheNonce 并更名 CachedIndexVersionedImpl，见 `Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt:118-156,282-302`。原前端 HistoryIndexViewModel 从 0 起算并局部递增，不能原样转发（`Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexViewModel.kt:44-50,115-124`）。
- 原 History 另以 closed 和当前 generation 校验旧项（`Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt:218-231`）。新 RPC 按 Session index 寻址重建后的 owner，不能只让每个新对象从 0 递增，否则旧缓存/请求可能被误认作同代；这是假设照搬局部计数的迁移风险，不是已运行 RPC 的故障。
- 用户选择与 currentFlow 类似的内存随机标记，不需要有序 generation 或持久化；撤回全局 Long 分配源及保留轻量代际表候选。用户进一步指定名称为 cacheNonce，限于缓存有效性，不泛化为业务版本；确定规则见 [timeline 缓存设计](../../checklist/rpc-timeline.md#缓存标记与前端缓存)，不在任务重复完整规则。
- 前端收到 SessionNotActive 后使旧绑定/在途读取失效；重新保活后按初始化路径绑定元数据，新标记清空旧缓存，旧请求的迟到结果不能回填。新 owner 拒绝旧标记请求走 CacheNonceMismatch；历史命令不自动替换标记重试。标记校验不替代读取与破坏性写入的同步边界。
- 随机标记提供概率性的身份区分，不宣称数学上的绝不重复；不为此增加持久化唯一性登记或共享分配器。未来独立后端的重连、重新初始化仍需接入设计，随机标记不等于已实现网络恢复。
- 用户要求调整细化后，已将六条服务及父接口统一为 getCacheNonce/getCacheNonceFlow、cacheNonce: Long；两项历史命令参数为 expectedCacheNonce。异常改名 CacheNonceMismatch，保留无字段 message 协议，不加旧名别名或 SerialName 兼容映射。currentFlow 的 nonce 与 NoMatchException 不变，settings CAS 仍比较当前完整值。
- helper 测试已调整为新类型，并增加旧 GenerationMismatch 载荷不被识别为兼容别名的断言；旧名测试只验证未知载荷保留原失败，不实现兼容。没有新增依赖、DTO、随机数生成器或运行接入。
- 验证：复用 IDEA 与空闲 Temurin 25 Daemon 2385167；RPC JVM、Linux x64/ARM64、Windows x64 实际编译通过，macOS ARM64 因既有依赖 cinterop 条件跳过。helper 五目标库实际编译通过，含 macOS；contract 33 项 JVM 测试与 helper JVM/Linux x64 Native 各 15 项测试实际通过，零失败/跳过。
- 生成检查：六条 timeline（settings 11 方法，其余各 10）、Runtime 21、Global 28 的 stub/invokator/callable 完整，cacheNonce/expectedCacheNonce 参数元数据正确。旧 getGeneration/异常生成类与中间 getNonce 名称已清除，CacheNonceMismatch serializer 存在，普通父接口仍不生成服务。既有 Mosaic 配置缓存警告未扩展处理。
- 任务下文的旧批次使用当时 generation/GenerationMismatch 命名，当前接口以上表及 checklist 为准；未修改旧 CLI 的窗口计数或持久化格式。缓存重建/回退的标记发布后续已实现并测试，但迟到结果拒绝、RPC 查询校验与生命周期接入仍未完成。

FIXME:

#### 已批准：Session 契约收尾

- 用户已整组批准两项 GlobalRpc 方法与目录 running 字段；源码已添加并完成编译/值测试。下一组继续 Agent，不等待完整缓存实现，也不将 TTL 参数提前写入模型。
- 已批准签名与语义以 [Session checklist](../../checklist/rpc-session.md)为准，不在此保留重复草案。
- 原 NewSessionViewModel 确实保留 rename/explicitThreadName 路径（NewSessionViewModel.kt:40-66,111-126）；用户选择不将这项能力纳入当前 RPC 创建范围，不据此删除现行 UI 或修改旧物化实现。具体前端接入留待后续实施。
- SessionCatalogEntry 已新增 `running: Boolean = false`；默认值保留现有构造调用，后续 RPC 服务必须填真实查询值，不能据默认值把未知/断连伪装为空闲。
- 最初运行标记参考现行 `SessionTabBar.kt:45-57`（app/view/application）及其 execution 投影；用户随后废除该聚合模型，目录迁移改为直接采样统一 runningTurn，不保留 historyOperation 的旧聚合口径。
- 本轮仅修改契约/条目 KDoc 与值测试，不修改原 Session 创建、缓存或 UI，无新依赖/序列化类型；新增归档与运行布尔值独立组合/默认值测试，并更新原序列化字段断言。
- 创建默认命名来自 NewSessionViewModel.defaultNameForIndex；目录布尔值只反映查询时的统一 runningTurn，不把后台标题请求、history 操作或 shell 会话的资源寿命折叠进去。现行 tab 对旧 execution 的消费另在迁移接入时替换。
- 一期内存 RPC 同样采用以上边界；TTL 和前端调用间隔在接入前统一确定，不为本组加入客户端 lease、到期时间返回值或网络协商。
- 验证：复用 IDEA 导航与 Temurin 25 Daemon 1652546；RPC 的 JVM、Linux x64/ARM64、Windows x64 实际编译通过，macOS ARM64 仍因既有依赖 cinterop 条件跳过。contract 24 项与原 catalog 1 项 JVM 测试均实际运行通过；27 个 GlobalRpc 方法的 stub/invokator/callable 完整，无 closeSession。配置缓存仍因既有 Mosaic 问题丢弃；未运行内存 RPC、TTL、Session 创建或目录 spinner。

FIXME:

### 其余 RPC 与状态适配

- 新准入方向：keepSessionAlive 成功返回时 Session 已活跃；普通访问不隐式激活，未活跃时抛已知 SessionNotActive。已定规则见[保活与回收](../../checklist/rpc-session.md#保活与回收)；异常已放入既有 utils/rpc-exception 的 sealed 联合，复用 message/restoreRemoteException，无需新增通用异常或传激活句柄。
- 用户确认失活时已有上游也以 SessionNotActive 结束，撤回 TTL 卸载不影响旧订阅的方案。后端生命周期适配层负责检测业务失活并失败结束，不能只取消 owner scope 而假定框架会自动产生该异常；前端保活成功后重新订阅。
- SessionNotActive 与 NoMatchException 分开：前者缺少可访问的 live Session，后者在可访问 Session 内当前输出/nonce 不匹配。不存在/已删除 Session、目录和归档等不依赖 live owner 的方法不机械套用该分支；具体服务覆盖继续核对。
- 已添加无字段的 Serializable SessionNotActive，继承现有 RemoteException；恢复函数无需改动。激活准入/订阅结束说明已覆盖 GlobalRpc.keepSessionAlive、AgentRuntimeRpc 及 timeline；当时的独立 settings 服务后续已合并，见下节。
- 新增两项 helper 值测试覆盖与 NoMatch 分支区分、自动 message 编码，以及模拟上游失活时的异常还原和 finally 清理；原取消测试覆盖两种载荷。它们不是实际 Session 卸载或 kRPC 往返测试。
- 验证：复用 IDEA 和空闲 Temurin 25 Daemon 2385167，utils-rpc-exception 五目标库实际编译通过（含 macOS ARM64），JVM/Linux x64 Native 各 12 项测试实际通过，零失败/跳过。RPC 模块本轮仅 KDoc 修改，签名未变，不重复编译或运行其测试；文档链接、旧规则残留及空白检查通过。
- 尚未实现激活、访问校验、失活终止或前端恢复。失活恢复不得盲目重试结果未知的写操作，TTL 到期与安全卸载仍须区分；各 Global 管理命令的活跃准入不在本组扩大。
- 一期契约盘点：GlobalRpc 已含单次 reset、Session create/保活及目录 running 字段。六条 timeline（含 settings CAS）不覆盖 Agent 运行命令或实时输出，不能据这些契约已编译宣称业务切面齐全。
- AgentRuntimeRpc 与历史命令的已添加声明见上表；全局 latestIndex 已添加，tokenCount 复用 timeline 读取。标题生命周期、子任务命名及通知规则已确认；execution 聚合 RPC 已撤回，契约完成不等于运行接入完成。
- History 的 streamingItem.Output 仍内嵌前端 SharedFlow<ResponsesStreamEvent>（`Kodex/app/contract/history/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/AgentHistoryViewModel.kt`）；迁移按已定 state/nonce/currentFlow 重建本地输出，完整 replay 与绑定失败语义已确定，不以 UnstableTimelineRpc 代替未落盘增量。
- 工具 Pending 的草稿、revision、提交阶段留前端，后端事件通过 state 和 completeToolCall 交互；shell 行为 handle 改由已定整体状态及 close 命令适配。原 contract 的本地形状不是新增 RPC 的依据。
- usage 三方法已获批并添加，设计见 checklist；原公开状态及五个子类型、八个原快照/嵌套值声明已补序列化与必要 nullable KDoc，openai-account-usage-contract 补既有插件/core，无新模型或版本升级。
- usage 原投影位于 GlobalSettingsViewModel.kt:657-667（app/viewmodel/settings）；加载/账号校验在 CodexAccountUsageStoreImpl.kt:46-87,185-219（openai/account-usage/impl）。本轮不修改这些运行实现。
- reset 已收敛为单次消费，账号绑定及供应商幂等接入仍待落实；Session 创建/保活/运行字段只有契约，不随本组签名落地视作运行行为已实施。
- 普通运行与手动压缩已统一到 runningTurn，前端 Stop 使用 cancelRunningTurn，撤回 cancelExecution 候选；原前端等待/编辑忙碌状态不作为新的后端聚合原语，具体接线需保留原操作准入。
- 工具交互复用 call id、revision 和既有准入，草稿及提交状态留前端；未实现的自动决策能力和未来多端仲裁不成为一期新增功能。
- History：后端 CachedIndexVersionedImpl 与前端单 timeline 缓存分别实现只读接口；初始加载、标记/尾索引与迟到结果已有独立测试，尚未替换原历史消费者，不共用文件缓存实现。
- CAS：实际 AgentState 当前只有锁内全量 update，没有 expect 比较入口；需把比较纳入同一写入边界，不能只修改镜像。
- KodexAgentSettings 含 turnId/windowNumber/plan 等运行字段；接入全量 CAS 仍须保留合法性，不以契约编译证明任意旧快照可写。
- CAS 重试的50ms等待、取消与异步本地值已在独立客户端测试覆盖；业务接入仍需验证字段冲突、后端实际比较和订阅时序，不把 helper 测试当作完整设置写入验收。
- 一期审查内存 RPC 状态初始化、取消、错误及宿主关闭；网络重连触发/退避与跨连接缓存恢复移至后续部署，不再阻塞当前契约。结果未知的写调用仍不能任意自动重放。
- 超长 Session 有界索引内存单独保留，不阻塞迁移；分页、滚动锚点、按条数限制与区间缓存未获设计/实施授权。

#### 已批准：历史回退与边界 fork

- 用户批准两个挂起命令及后端 index timeline 代际绑定；源码声明已添加，设计以 [历史操作](../../checklist/rpc-timeline.md#历史操作)为准。
- 原前端确有需求：`Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt:833-871` 中回退/分支传选中条目 index + 1，回退并编辑传条目 index；确认弹窗、编辑草稿与打开目标 tab 留前端，不新增 request/confirm/dismiss RPC。
- 原实现参考：`Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt:309-433`；`Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt:302-359`。原测试 `AgentHistoryActionTest.kt:33-132` 覆盖稀疏边界、旧代拒绝与回退等待取消；本轮只阅读，未重跑。
- 适配差异：原 expectedGeneration 校验 `history.historyItems.value.generation`，由 History ViewModel 自增（`Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt:421-432`），并非后端缓存标记。后续接入须改为 IndexTimelineRpc 的 cacheNonce。
- 上述绑定复用已定 timeline 版本协议，不新建历史版本 DTO；后端需在实际写入/源快照捕获边界验证标记与准入。缓存实现后续已发布随机标记并提供只读视图，但历史命令的校验与 owner 准入仍未接入，不能由元数据测试替代验证。
- 本轮仅新增两个原始类型签名，无新模型、序列化或依赖；尚未实现后端操作、代际适配及前端接入。
- 验证：复用 IDEA 导航及 Temurin 25 Daemon 1951973，RPC JVM、Linux x64/ARM64、Windows x64 实际编译通过；macOS ARM64 仍因既有依赖条件跳过。31 项既有 contract JVM 值测试实际运行通过，Runtime 14 / Global 28 方法及新增 Int/Int/Long 参数、Unit/Int 返回的 stub/invokator/callable 检查通过。无新值类型故未新增序列化测试；未运行历史操作、代际适配或内存 RPC。既有 Mosaic 配置缓存问题导致缓存丢弃，未扩展处理。

FIXME:

#### 已落实：Shell 进程观察与关闭

- 原前端边界为 AgentShellSessionRegistry.activeSessions 与 AgentShellSession 的 sessionId/arguments/completed/close（`Kodex/app/contract/agent/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/AgentState.kt:51-69`）；不需要把 UnifiedExecToolClient 的 execCommand/writeStdin 或输出消费入口给前端。
- 用户要求按一个 StateFlow 加 close 理解此切面；撤回逐进程 completed Get/Flow 的五方法草案。
- 用户要求继续后已落实三方法及两字段 ShellSessionState；已定设计见 [Shell 进程](../../checklist/rpc-agent-runtime.md#shell-进程)，不再保留逐进程订阅候选。
- ShellSessionState 位于 rpc/models，复用原已序列化 ExecCommandArguments 与既有传递依赖，不修改原进程模型或模块构建。新增三项测试覆盖完整/默认参数、登记完成态与缺失的区别、完成值变化及必需字段；不是运行投影测试。
- 原 ID 在当前登记表内随机分配且只排除当前占用值（`Kodex/tool/unified-exec/impl/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolClient.kt:184-190`），不能声称跨 owner 重建或移除后永不复用；本契约按当前登记项寻址，不提供旧对象身份的严格等价保证，也不新增句柄注册表。
- 证据：侧栏 `SessionSidebar.kt:647-707,941-956` 使用完成观察、原 command 和显式 close；历史 `AgentHistoryView.kt:626-641` 按进程 id 查 registry 并观察 completed；原 close 的保留尾输出约定见 UnifiedExecToolClient.kt:234-248。前两文件分别位于 `Kodex/app/view/application` 与 `Kodex/app/view/history` 的 mosaicMain 包下。
- 本轮只落实契约、值模型与测试，不实现 registry/完成流合并、进程控制或前端接入；运行时的取消、尾输出读取及 TTL 到期清理仍待实施验证，shell 不参与保活的最新决定见 Session 设计。
- 验证：复用 IDEA 导航与 Temurin 25 Daemon 1951973，两模块 JVM、Linux x64/ARM64、Windows x64 实际编译通过，macOS ARM64 因既有依赖条件跳过。models 15、contract 31 项 JVM 测试实际运行通过（合计 46，新增 3）；Runtime 17 方法生成结构及 Map<Int, ShellSessionState>/Flow/Unit 签名完整，无逐进程 completed 方法。配置缓存仍因既有 Mosaic 问题丢弃；未运行真实进程或内存 RPC。

FIXME:

#### 已定：统一 runningTurn 与压缩状态

- 用户确认普通运行和手动压缩都属于 runningTurn；核对现有压缩实现后撤回 SharedFlow 要求，手动与自动均只发布 Compacting 状态。已定设计见 [Running turn](../../checklist/rpc-agent-runtime.md#running-turn)与[压缩状态](../../checklist/rpc-agent-runtime.md#压缩状态)；独立 cancelExecution 仍撤回，现有 cancelRunningTurn 承担统一 Stop。
- 已同步 AgentRuntimeRpc 的 resume/forcedCompact/runningTurn/取消注释，没有改变方法签名、依赖或运行实现。需要统一的是后端任务所有权和准入，不是额外维护 OR 合并后的布尔镜像；普通运行的取消清理不机械套用到压缩。
- 原底层槽位只覆盖 resume；下方最小实现已让外层显式 compact 共用同一槽位（`Kodex/agent-runtime/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt:144-191`）。前端仍以 mutableRuntimeOperation 承载等待并在 Stop 时取消两处（`Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt:233-257,551-590`）；移除该旧模型及后台任务所有权留后续接入，不混称已完成。
- 两种压缩当前已共用 compact(trigger, reason, phase)：forcedCompact 使用 Manual/UserRequested/StandaloneTurn，自动路径使用 Auto/ContextLimit/PreTurn 或 MidTurn（`Kodex/agent-state/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentStateExtensions.kt:43-48`；`Kodex/agent-runtime/decorator/compact/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/compact/KodexAgentCompactionRuntime.kt:100-157`）。
- 当前并非自动压缩已发布 SharedFlow：OpenAiClient 在 createRemoteCompactionV2Response 内收集 SSE 为最终值；事件解析复用 ResponsesStreamEvent，内部另有 Done 标记和重试预算（`Kodex/openai/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt:220-244,468-547`）。AgentState 仅发布无流的 Compacting，取得结果后校验所有权并提交 checkpoint（`Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt:252-299,523-568`）。
- 继续保留后端聚合/结果验证/重试职责，不增加压缩输出发布、回放或新的订阅接口；前端观察现有 state Flow，展示 Compacting，结果提交不由前端 collector 驱动。
- 传输模型的 Compacting 保持无字段 object，撤回 Compacting(nonce) 候选及压缩 currentFlow 接入，不再审查压缩流标记或重试流切换；普通 Responses 的 nonce/replay 设计不变。
- 当时仅修改设计与注释；运行槽位及互斥/清理/自动压缩嵌套随后在下方批次完成。真实 RPC 状态发布、后台任务及 TTL 仍需后续接入验证。

FIXME:

#### 已落实：原 runtime 的统一运行槽位

- 用户批准后，仅修改原 AgentRuntimeImpl 与契约说明，并在原内存 Session 测试模块添加8项测试；未增加模块、依赖、公共方法或生产服务。现行槽位规则收录[AgentState 与 AgentRuntime](../../checklist/agent-state-and-runtime.md)。
- 外层 compact 与 resume 共用私有 withRunningTurn，继续登记调用 Job、原子拒绝重复进入并在 finally 释放（`Kodex/agent-runtime/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt:144-191`）。不再仅由前端 Job 表示手动压缩；但本批没有删除旧前端等待模型。
- 保留 forcedCompact 扩展和 Manual/UserRequested/StandaloneTurn 参数，继续委托原压缩装饰器；不增加运行类型 DTO 或第二个忙碌状态。复用原 ConcurrentAgentRuntimeResumeException 类型，说明及诊断文案扩展为两类执行互斥，不把它加入跨 RPC 异常联合。
- 自动压缩在内层 KodexAgentCompactionRuntime.resume 中调用自己的 compact，不需要再次占用外层槽位（`Kodex/agent-runtime/decorator/compact/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/compact/KodexAgentCompactionRuntime.kt:38-43,89-106,140-145`）。普通运行的取消 clearPending 保留，不能机械应用于手动压缩。
- 新测试覆盖手动压缩的调用 Job/Compacting、checkpoint 与原 Int 返回值、turn/window/tokenCount/请求元数据、双向互斥及重复压缩拒绝、未知失败和准入失败后的空槽、取消清理中的槽位保留、成功/取消均不消费 steer，以及自动 PreTurn/MidTurn 不二次占槽（`Kodex/agent-session/in-memory/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/inmemory/AgentRuntimeRunningTurnTest.kt:42-288`）。使用原 mock provider 与内存 Session，不调用真实模型或 shell。
- 编译：复用 Temurin 25 Daemon 622303，JVM、Linux x64/ARM64、Windows x64 的库及测试源码通过；相关 macOS ARM64 任务因原传递 cinterop 条件跳过，不声明 Mac 运行通过。Native 批次1分11秒，840项任务149执行；既有 Mosaic 配置缓存3550项问题导致缓存丢弃，未修改其构建。
- 实际 JVM/本机 Linux x64 Native 最终各79项通过：内存 Session20（新增8）、压缩装饰器11、文件 Session36、Agent ViewModel12。新增测试所在内存模块两端各20项又强制重跑通过（19秒），不是复用旧报告或仅编译测试源码。
- 首轮 JVM 的旧 HistoryIndexViewModelTest 等待 generation 更新超时（`Kodex/app/viewmodel/agent/src/commonTest/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexViewModelTest.kt:102-157,295-301`）；该用例直接使用存储与前端 StateFlow，没有调用修改的 runtime。原样复跑12项通过，Native12项也通过；保留这次不稳定结果，不顺带修改历史缓存、测试延迟或将复跑称为已修复问题。
- 本批只完成占用、互斥与清理可见性；当前仍登记调用 Job，不能直接把 RPC handler 当后端任务 owner。已接受工作由后端持有、调用者只等待的规则不变，独立任务归属与 TTL 在后续服务接入中落实，不声称已实现脱离前端运行。
- 存储算法、原压缩装饰器、前端 execution 模型、通知、目录接口及生产 RPC 接线均未修改；原队列消费行为不变，不补消费或自动 resume。剩余服务接入须成组维护后端所有权与前端释放责任，不据本组授权启动整体迁移。

FIXME:

#### 已定：压缩期间沿用原排队行为

- 用户明确保留允许排队但不补消费的现有行为，规则见[Pending steer](../../checklist/rpc-agent-runtime.md#pending-steer)。撤回压缩期间禁止提交、补消费及自动 resume 候选，不把它们作为本期阻塞点。
- 原 submitComposer 在 execution.running 时清空草稿并将内容加入 pendingSteer；running 又包含手动压缩的 mutableRuntimeOperation（`Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt:200-230,491-502`）。输入框只按宿主交互等待禁用，未按 Compacting 禁用（`Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt:186-199`）。
- 原 SteerRuntime 只在 resume 的前后调用 take/injectHistory（`Kodex/agent-runtime/decorator/steer/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/steer/SteerRuntime.kt:22-41`）；forcedCompact 直接调用 compact，结束后没有进入该消费循环（`Kodex/agent-state/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentStateExtensions.kt:43-48`）。只合并 Job 槽位不会补出消费者。
- 保留 pendingSteer 原 CAS 和后续正常 resume 的消费路径，不增加该场景专用的拒绝异常、运行类型 DTO 或持久化；runningTurn、TTL 和手动压缩不自动 resume 的已定规则均不变。
- 本轮只查源码并同步用户决定，未修改原队列、输入框、契约或运行实现，也未运行压缩/提交实验。

FIXME:

#### 已定：废除 Execution 聚合模型

- 用户否定原 AgentExecutionState 的前后端边界，要求迁移时废除而非跨 RPC 保留；getExecution/getExecutionFlow 两方法及原三类型序列化/模块依赖候选均撤回，未创建源码。确定方向见 [前端执行投影](../../checklist/rpc-agent-runtime.md#前端执行投影)。
- 原 phase 直接映射 state，capabilities 由 state/running/cancelable 计算，activityVersion 只是前端投影变化计数（`Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt:477-502,709-748`）；保持旧 UI 消费形状不构成新增后端 DTO 的理由。
- 修正上轮推论：mutableHistoryOperation 定义在前端 ViewModel，用于回退过程的等待与 busy（同文件 :131,368-435），并非已存在的独立后端历史活动原语。其观察区间确实不同于 ExternalWrite，但不能据此要求后端复制该 Job 或新增 history-busy RPC。
- 后续逐项检查原消费者实际需要的后端原语，允许前端基于已暴露值派生 UI；不再以“完整复原旧 execution”作为契约完备性标准。后端仍保留操作准入、历史代际校验和已接受任务的所有权。
- 用户已确认目录 running 直接取统一 runningTurn，不依赖旧 execution.running；包含普通运行和手动压缩，不包含 history 操作或前端等待。目录仍为按需快照，不新增 Flow；条目和 GlobalRpc 注释已同步，没有新增字段、运行实现或依赖。
- 本项仅文档与注释变更，未重复编译/运行测试；实际目录服务取值、spinner 与统一任务槽位仍需在迁移接入时验证。
- tokenCount 原值来自 storage.tokenCount[index]（同文件 :129,471-475），已有 TokenCountTimelineRpc 可读取；Agent 全局 latestIndex 与各稀疏 timeline 尾索引须区别，不能未经核对就视作相同。
- 本轮只更新迁移决定和纠正候选，没有删除现行 UI 模型或修改运行实现；旧消费者移除及相关回归留到获准的迁移接入阶段。

FIXME:

#### 研究：请求取消与异常边界

- 核查范围为已锁定 kotlinx.rpc 0.10.3 的 kRPC 实现，不混用 gRPC 预览能力。项目目前只有契约/core，尚未接入 kRPC client/server；下面不是 Kodex 已运行的保证。
- 请求可取消：挂起调用内部同样经 callServerStreaming(...).first()；客户端捕获 CancellationException 后发送 REQUEST cancellation，服务端取消对应 handler Job/collector。[客户端](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-client/src/commonMain/kotlin/kotlinx/rpc/krpc/client/KrpcClient.kt#L237-L310)、[请求释放](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-server/src/commonMain/kotlin/kotlinx/rpc/krpc/server/internal/KrpcServerService.kt#L334-L404)。
- 这是协议桥接的两个 Job，不是跨两侧的同一结构化父子树。取消消息异步发送，本地 cancelAndJoin 不等于服务端 finally 已结束；协作取消也不撤销已提交副作用。[发送路径](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/commonMain/kotlin/kotlinx/rpc/krpc/internal/KrpcEndpoint.kt#L17-L45)。
- 单次请求失败/取消不由框架自动关闭其他请求或整个业务 owner；官方 CancellationTest 覆盖请求取消、并行请求不受单调用异常影响、服务端取消与 Flow 取消。测试采用的本地 transport 用 Channel 传 KrpcTransportMessage；同进程传输也不自动保留异常对象身份。[测试](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-test/src/commonTest/kotlin/kotlinx/rpc/krpc/test/cancellation/CancellationTest.kt#L23-L115)、[本地承载](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-test/src/commonTest/kotlin/kotlinx/rpc/krpc/test/LocalTransport.kt)。
- 后端挂起方法抛错作为 CallException 返回，输出 Flow 抛错作为 StreamCancel 返回；客户端在调用/collect 处抛出还原的 Throwable。调用方本地 launch/async、catch 与 SupervisorJob 决定其父级/兄弟协程是否受影响，不能把框架请求隔离解释为调用方未捕获异常绝不扩散。[服务端](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-server/src/commonMain/kotlin/kotlinx/rpc/krpc/server/internal/KrpcServerService.kt#L128-L205)、[Flow 错误](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-server/src/commonMain/kotlin/kotlinx/rpc/krpc/server/internal/KrpcServerService.kt#L249-L327)、[客户端解码](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-client/src/commonMain/kotlin/kotlinx/rpc/krpc/client/KrpcClient.kt#L312-L365)。
- CancellationException 单独归一化并恢复为 CancellationException，具体子类/对象身份不保留；可以区分取消与一般失败，但不能依赖原 TimeoutCancellationException 等子类。[公共异常处理](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/commonMain/kotlin/kotlinx/rpc/krpc/internal/ExceptionUtils.kt#L10-L39)。
- 普通异常不保证原类型：JVM 尝试有限构造器反射恢复，不符合条件则通用 DeserializedException；Native 非取消异常直接变成该通用 Throwable。Native 原栈提取为空，未传自定义字段和 suppressed 列表；catch(具体业务异常)，甚至仅 catch(Exception)，都不能当成跨 Native RPC 的可靠错误分支。[序列化值](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/commonMain/kotlin/kotlinx/rpc/krpc/internal/SerializedException.kt)、[JVM](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/jvmMain/kotlin/kotlinx/rpc/krpc/internal/ExceptionUtils.jvm.kt#L35-L55)、[Native](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/nativeMain/kotlin/kotlinx/rpc/krpc/internal/ExceptionUtils.native.kt)。
- 对既定所有权的影响：只读查询/订阅可直接沿请求取消；resume、压缩、回退等已接受工作若按现有决策独立归后端 owner，取消 handler 只终止等待，不能直接把业务调用运行在 handler Job 中。反过来，后端明确取消该业务任务可经仍在等待的调用传播 CancellationException。后台任务已脱离等待或方法已返回后的异常没有旧 RPC 回执可用，需原后端监督/通知接入，不能假设自动回传。
- 因此显式 Stop 的理由是已选“前端停止等待不停止后台工作”，不是框架不会取消；当前已定由统一 cancelRunningTurn 承担，cancelExecution 候选已撤回，不改变已接受任务的生命周期。
- 前端 Flow 转 stateIn 后又有一层异常边界：上游失败结束共享协程，由其 scope 处理，不直接向本地 StateFlow 订阅者抛错；须在上游/owner 明确处理，否则可能留下最后值而失去更新。[coroutines 1.11.0 stateIn](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/state-in.html)。
- 用户已确认[取消与业务错误规则](../../checklist/rpc-state.md#取消与业务错误)：需要前端解析的已知非取消失败用联合类型，未知异常不进入兜底分支。该规则不是所有方法必须返回统一 Result，也不将取消转换为失败值。
- 具体落地仍待逐项审查：currentFlow nonce 过期和 timeline 旧代是现有明确需求；需确定各自最小结果分支、成功值形状及 Flow 终止语义。现有契约仍为原返回类型，尚未新增错误模型、映射实现或调整签名。
- 一期运行验证需覆盖调用方取消/超时、后端取消、普通调用与 Flow 异常、并行请求隔离、已接受任务与 handler 分离、Native 错误类型和 stateIn 上游失败；本轮只读官方源码/测试与本地调用链，未运行这些测试、制作原型或改代码。

FIXME:

#### 已撤回：currentFlow 的事件包装

- 用户未批准 Flow<Event | Stale>，随后选择异常承载绑定失败；不再保留 CurrentFlowResult 的实施候选。
- 最终契约保留顶层 Flow<ResponsesStreamEvent>，无当前可绑定输出或标记不匹配使用 NoMatchException；timeline 的旧缓存标记使用 CacheNonceMismatch。业务空查询与未知异常不混入这两个分支。
- 格式、恢复及取消规则已有 sealed helper 和实际 RPC 往返验证；未完成的是业务绑定与前端切流接线，不是再次审批错误结果形状。

FIXME:

#### 研究：Flow 与 NoMatch 外层联合

- 业务语义确实是一次绑定选择：失败 NoMatch，成功后只有原事件流；不是每条事件都可能发生绑定失败。候选本地形状可为 Match(events: Flow<ResponsesStreamEvent>) | NoMatch，但不等于它能作为 RPC 返回值。
- 当前固定 kRPC 0.10.3 不支持这种服务端返回：官方 strict mode 将返回数据类中包含 Flow 列为非顶层服务端流限制，0.8.0 起不可关闭。[官方约束](https://kotlin.github.io/kotlinx-rpc/strict-mode.html#not-top-level-server-flows)。
- 不仅是语法检查：普通挂起返回走值序列化，只有顶层非挂起 Flow 才走 sendFlowMessages；注册的 ServerStreamSerializer.serialize 直接报错，客户端 ClientStreamSerializer.deserialize 也直接报错，不能靠 sealed 子类、Contextual 或漏掉诊断让该返回可用。[分发路径](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-server/src/commonMain/kotlin/kotlinx/rpc/krpc/server/internal/KrpcServerService.kt#L143-L169)、[服务端 serializer](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-server/src/commonMain/kotlin/kotlinx/rpc/krpc/server/internal/ServerStreamSerializer.kt#L22-L28)、[客户端 serializer](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-client/src/commonMain/kotlin/kotlinx/rpc/krpc/client/internal/ClientStreamSerializer.kt#L22-L28)。
- 编译检查器会递归检查可见可序列化属性并报告非顶层流；本轮没有编译 sealed 反例，不宣称每种封装都会被静态检查抓到。运行路径的明确限制已足以否定按官方 kRPC 直接传该联合。[检查器](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/compiler-plugin/compiler-plugin-k2/src/main/kotlin/kotlinx/rpc/codegen/checkers/FirRpcStrictModeClassChecker.kt#L68-L92)。
- 若仅要求业务侧得到该形状，可另审客户端本地适配：维持一次真实流绑定，在明确接受/拒绝后返回本地 Match/NoMatch，并接续同一次订阅；这不是把 Flow 联合跨线传输，也不能先 first() 取消再 collect() 建第二个请求。具体握手和本地生命周期尚未选择或实现。
- 不能用普通 canBind(nonce) 预检查加 currentFlow(nonce) 宣称等价：两次调用之间输出可能被替换；要保证预检查成功就保留旧流需另外持有绑定/资源，与当前不增加旧流 registry 的简化目标不同。本轮不引入这种机制。
- 已定的已知错误联合规则不变；具体 wire 形状仍待用户选择。没有创建 CurrentFlowResult、修改 currentFlow 或制作原型，本轮仅官方资料/源码核查与记录。

FIXME:

#### 研究：异常通道承载已知错误值

- 当前决定：用户已选择专用 message 协议并要求简洁，确定规则见 [已知错误的 message 协议](../../checklist/rpc-state.md#已知错误的-message-协议)；下文保留选型研究，具体格式/helper 与运行接入尚待审查。修正此前“保留纯 Flow 必须改框架”的过强判断：应用层专用异常载荷加客户端适配也可保持原返回类型。
- 固定 kRPC 0.10.3 的 serializeException 明确复制 Throwable.message；Native 的 DeserializedException 保留 message，JVM 通用还原也保留。普通异常的类型/字段不保留，不妨碍显式序列化载荷随 message 传输。[公共编码](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/commonMain/kotlin/kotlinx/rpc/krpc/internal/ExceptionUtils.kt#L10-L21)、[Native](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/nativeMain/kotlin/kotlinx/rpc/krpc/internal/ExceptionUtils.native.kt#L8-L24)、[JVM](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/jvmMain/kotlin/kotlinx/rpc/krpc/internal/ExceptionUtils.jvm.kt#L10-L24)。
- 已选载荷使用裸 JSON，由首字符 `{` 进入解码；撤回此前“保留文本前缀加联合值”的候选。仅编码明确已知的异常联合，后续 generation 错误逐项纳入；不解析已有文案、不传任意 Throwable、不添加 Unknown/Exception 业务分支。
- 这不是 kRPC 的官方 typed-error codec。当前 KrpcConfig 公开项只有 serialization/connector，无异常映射 hook，KrpcClient 调用入口为 final，消息处理为 private；不可声称配置一个异常 serializer 就能覆盖内部 serializeException。[配置](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/commonMain/kotlin/kotlinx/rpc/krpc/KrpcConfig.kt)、[客户端](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-client/src/commonMain/kotlin/kotlinx/rpc/krpc/client/KrpcClient.kt#L237-L365)。
- 客户端可组合公开 RpcClient，统一适配 call/callServerStreaming；withService 把接收者 client 交给生成 stub，因而可以在创建业务代理时使用该包装器，不逐方法复制代理或修改生成代码。[RpcClient](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/core/src/commonMain/kotlin/kotlinx/rpc/RpcClient.kt)、[withService](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/core/src/commonMain/kotlin/kotlinx/rpc/withService.kt#L49-L54)。
- 还原由异常基类的 serializer 直接构造已知子类，不再手写值到异常的分派；不依赖远端异常类名/反射。CancellationException 先原样传播；非载荷、畸形载荷或不认识的分支仍为未知失败，不构造 NoMatch，不自动重试，也不递归扫描任意 cause/message。
- Flow 应在 RPC 上游做异常转换，避免把前端下游 renderer 自己的异常误当成远端业务失败；生成 Flow 当次抛出与后续 collect 抛出须分别覆盖，不能用包住整个 emitAll 的 catch-all 混淆下游错误。[Flow.catch 透明性](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/catch.html)。
- 后端可在绑定点直接抛专用载荷异常；currentFlow 保持 Flow<ResponsesStreamEvent>，客户端在 collect 时得到本地可识别的 NoMatch。成功路径不增加事件封装或两次绑定，不需要旧流 registry；timeline 的已知失败也可复用这一适配入口，而非机械改变所有成功返回值。
- 已接受的方向把异常 message 定义成应用协议，不再只是诊断文本；标记不作为可信身份/权限凭据，载荷应只含业务所需值，不放凭据，日志可能记录 message。具体格式仍待审查。
- 验证缺口：JVM/Native 对专用载荷的实际往返、普通调用/Flow 的绑定前后错误、取消原样传播、非载荷与非法/未知分支不误分类、下游异常透明性、stateIn 前处理。当前只是源码支持的可行性判断，没有制作原型、增加依赖或运行 RPC。
- 精确字段核查：内部 SerializedException 只有 toStringMessage、message、stacktrace、cause、className 五字段；stack 元素只有 clazz/method/fileName/lineNumber。null message 被替换为 `"Unknown exception"`；cause 递归编码，取消 className 归一化，其余类名不可取得时为空串。未传 suppressed、自定义属性、对象身份或 Job/CoroutineContext。[线协议值](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/commonMain/kotlin/kotlinx/rpc/krpc/internal/SerializedException.kt#L10-L28)。
- 区分线上字段与前端可用字段：Native 普通异常保留 message、toString 文本和递归 cause，忽略接收的 className/stacktrace；Native 发送栈为空。JVM 通用 fallback 保留上述文本/栈/cause；若反射重建原类，则其构造器决定 message/cause/toString，零参或仅 cause 构造路径不能承诺恢复所有原信息。取消走专门重建路径，不保留具体子类/原栈。[JVM 构造选择](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/jvmMain/kotlin/kotlinx/rpc/krpc/internal/ExceptionUtils.jvm.kt#L83-L111)。
- helper 仍只需编码载体与识别还原两个操作，旧 RpcFailure 值加手工工厂候选已由“直接解码异常联合”取代，具体签名见下文。识别失败返回 null 仅表示未还原已知异常，调用方须保留原异常而非吞掉。
- 取消处理由边界先分流，decoder 不将取消识别为业务错误；不自动做 retry、结果缓存或新订阅。RpcClient 装饰器可以后续复用这两个 helper，不作为本轮默认实施范围。
- 受控载体必须验证 JVM/Native 解码后的 message 确实原样保留，不能把原异常任意构造器的行为当成保证；测试还需覆盖转义/Unicode、空载荷、非法/未知分支、普通异常与取消不误识别，以及 cause 中有标记但顶层无标记的情况。用户本轮询问保留信息及 helper 必要性，未创建源码或改已有设计规则。
- 用户提出按已知异常 class name 白名单重建，并放入独立 `util/rpc-exception` 模块。模块与有限工厂映射是候选组织方式；现有代码目录实际为 `Kodex/utils/`，具体落点与承载机制尚未落实，不因提议创建未可用的模块。
- Kotlin 2.4 Native 支持 KClass.simpleName/qualifiedName；但它们查询当前对象类型，RPC 接收端普通 Throwable 已是 kRPC 的 DeserializedException，结果不是远端原类名。Native 实现构造参数 className 没有 val/var 且未使用，并非一个可反射读取的隐藏属性。[KClass 文档](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.reflect/-k-class/qualified-name.html)、[丢失点](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/nativeMain/kotlin/kotlinx/rpc/krpc/internal/ExceptionUtils.native.kt#L8-L24)。
- 白名单重建本身不需要动态反射构造：一旦取得可靠类型标识，可用明确工厂构造有限已知异常，其余原样抛出；自定义属性仍须额外显式载荷，不能仅凭类名恢复。取消保留专门通道，不按普通白名单重建。
- 因此仅在 RpcClient 外层 catch(Throwable) 后读取 ::class 不能实现原方案。若要直接用线上 SerializedException.className，需要在 Native 还原丢弃前保留该字段，涉及内部异常解码/消息适配；现行无公开 hook，不能当作普通 util 的现成功能。另一候选是显式载荷携带类型标识再做白名单映射。
- 原 toString 文本保留不等于原 className 属性仍存在；从默认 toString 拆类名受覆写/格式影响。若使用文本，必须另行约定受控格式并确认替代此前的已知错误编码方向，不能默认为稳定反射方案。本轮仅记录事实与设计限制，无源码或运行验证。

#### 深查：className 的可达性与替代入口

- 核查基线为 0.10.3（fc69ae6288cb4c8596493cd0b7ea2d79cac78f0d），并对照本轮官方 main（675755ef7e6f5a46d563f7420ca278e46d97275c，2026-09-09）。后者 Native 异常文件仅移除文件级诊断抑制，仍丢弃 className；公共异常编码和 KrpcConfig 相同，客户端变化未提供异常映射 hook。不是已升级依赖或验证 gRPC 预览版。[固定主分支 Native 源码](https://github.com/Kotlin/kotlinx-rpc/blob/675755ef7e6f5a46d563f7420ca278e46d97275c/krpc/krpc-core/src/nativeMain/kotlin/kotlinx/rpc/krpc/internal/ExceptionUtils.native.kt)。
- 完整路径：serializeException 生成带 className 的 SerializedException → CallException/StreamCancel 携带该值 → connector 解码 KrpcMessage → KrpcClient 的私有处理函数直接 deserialize → Native 创建不保存类名的 Throwable → 调用方 catch。类名确实到达客户端，但在普通业务 catch 之前丢失；扫描 cause 也会遇到同一递归还原限制。[消息形状](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/commonMain/kotlin/kotlinx/rpc/krpc/internal/KrpcMessage.kt#L184-L247)、[丢弃前的调用位置](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-client/src/commonMain/kotlin/kotlinx/rpc/krpc/client/KrpcClient.kt#L364-L400)。
- 公开 RpcClient 装饰器只能观察已还原 Throwable；KrpcClient.call/callServerStreaming 为 final，connector 和消息处理不可替换。仅新增 `Throwable.remoteClassName` 扩展或白名单工厂，无法补回已丢弃字段。
- 序列化格式是公开扩展点，但不是异常 codec：可以注册自己的 StringFormat/BinaryFormat，理论上观察解码后的内部 KrpcMessage；普通 contextual serializer 不能替换已经显式生成的 SerializedException 序列化，也不能替换平台 deserializeUnsafe。若在格式解码处直接抛已知异常，connector 会把它当作消息解码失败处理，不会正确投递到对应调用。[格式扩展](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-serialization/krpc-serialization-core/src/commonMain/kotlin/kotlinx/rpc/krpc/serialization/KrpcSerialFormat.kt)、[消息解码与错误处理](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/commonMain/kotlin/kotlinx/rpc/krpc/internal/KrpcConnector.kt#L279-L318)。
- 自定义 KrpcTransport 可以观察文本/二进制消息，但不会自动改变随后丢弃类名的行为。要用这个入口，仍须解析内部协议并改写载荷、额外关联调用结果，或接管消息处理；不能在 receive 抛业务异常来替代某一调用失败。此路并非技术上绝对不可行，但不是普通、无状态的异常 helper，不为它新增关联表或读取框架调试日志。[Transport](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/commonMain/kotlin/kotlinx/rpc/krpc/KrpcTransport.kt)、[connector](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/commonMain/kotlin/kotlinx/rpc/krpc/internal/KrpcConnector.kt)。
- 官方仓库 issue #213 中，维护者曾表示计划改善异常处理；2026-04 的用户 workaround 也是在代理处拆 toString 并匹配类名，不是维护者提供的稳定 API。不能复用其中 contains(simpleName) 的宽松分类。PR #170 修复的是 JVM 反射构造器选择/字段计数，不是 Native 类型注册机制。[问题与 workaround](https://github.com/Kotlin/kotlinx-rpc/issues/213#issuecomment-4269866627)、[历史修复](https://github.com/Kotlin/kotlinx-rpc/pull/170)。
- 若要保留真正的 className，较直接的框架改法是让通用远端异常通过公开属性/接口暴露该值，或增加异常还原 hook，再由本地白名单工厂构造已知异常；只在 Native 内部加 private 字段仍不够。这需要上游改动或维护 fork，尚未获授权。
- 在不修改框架、不绑定内部协议的前提下，推荐继续审查受控 message 载荷；toString 即使定成受控格式也只是另一套文本协议，没有减少协议责任。载荷可用稳定错误标识及必要参数，不必传 Kotlin 全限定类名；如果暂时只有无参数 NoMatch，也不需要通用异常属性序列化系统。
- 用户在本轮深查后选择 message 协议，方向已迁入 checklist；未创建 `utils/rpc-exception`。下一步需确定有限错误类型、载荷格式和 helper API，见以下最小候选。
- 本轮仅源码、版本差异和官方讨论核查；没有运行 JVM/Native RPC 往返、验证自定义格式拦截或创建临时原型。此前列出的专用载体 message 保真、Flow 透明性及取消测试仍必需。

FIXME:

#### 已定格式与单一 helper

- 用户确认取消额外前缀：message 首字符为 `{` 就尝试按我们的异常联合解码，复用 kotlinx.serialization 的类名判别与子类构造；确定规则见 [message 协议](../../checklist/rpc-state.md#已知错误的-message-协议)。
- 官方 sealed 多态方案会生成 type 分派，默认值为完整序列化类名。编码与解码必须显式使用同一基类型，否则按具体子类编码可能缺失 type；无需自行解析 type、反射构造或维护 when 分派。[官方说明](https://github.com/Kotlin/kotlinx.serialization/blob/master/docs/polymorphism.md#sealed-classes)。
- RemoteException 最初为普通抽象基类，现已按用户批准的集中方案改为 Serializable sealed 基类，并加入首个 NoMatchException；最终形状见下节及 [异常还原 helper](../../checklist/rpc-state.md#异常还原-helper)。不把框架的异常反序列化与应用层 JSON 解码混为一谈。
- 用户将 helper 收敛为单一 inline reified 包装函数后，进一步改名为 restoreRemoteException；撤回旧 catch 命名和两个编码/识别 API 的候选，不留兼容别名。实际签名见 [异常还原 helper](../../checklist/rpc-state.md#异常还原-helper)；为保留任意 block 返回值增加可推断的 R，不另建 RpcFailure 值或手写工厂。
- `{` 只是进入解码的约定，不证明一定是我们的异常；普通 JSON 文案可能碰撞。没有已知 type、JSON 非法或分支不认识时按原约定保留原 Throwable，不能把解码失败或空结果吞掉。取消先原样传播；不扫描 cause。
- 不添加版本协商、Base64、通用 payload/参数 Map、注册中心、缓存或自动重试。统一 RpcClient 接入另审，模块具体依赖与有限错误清单尚待确认。
- 已新增 `Kodex/utils/rpc-exception`，自动沿现有 utils 模块扫描纳入构建；采用五目标 KMP 约定与序列化插件，main 仅依赖 serialization-json，test 使用协程。没有修改版本、现有 RPC 签名或生产调用点。
- 实现只在 block 抛出非取消且 message 首字符为 `{` 时解码；最终固定使用 RemoteException。解码的 IllegalArgumentException（含序列化失败）保留原异常，成功后在解码 catch 之外抛出已还原异常，避免把它误当作解析失败。没有吞掉原失败或自动重试。
- 新增九项 commonTest：成功返回/单次执行/null/Unit/非局部返回、自动类型构造、参数与转义文本、非法及未知 JSON、首字符与顶层限制、取消、挂起 block、上游 Flow 还原和下游异常透明性。测试使用自有封闭异常联合与通用 Throwable 载体，不运行真正 kRPC 或引入生产 NoMatch 类型。
- 验证：复用 IDEA 导航及空闲 Temurin 25 Daemon 1951973；模块 JVM、Linux x64/ARM64、Windows x64、macOS ARM64 五目标库编译全部实际通过。本模块不依赖阻碍原 RPC 模块 macOS 编译的 cinterop，不沿用其跳过结论。
- JVM 与本机 Linux x64 Native 各九项测试实际运行通过，零失败/跳过；没有在 Windows、macOS 或 ARM64 设备运行测试。真正 JVM/Native kRPC message 往返仍须在运行接入时验证，helper 测试不能替代框架传输验证。
- 本轮未改生产调用方，未重复运行既有 RPC 值测试；此次构建成功且配置缓存已保存，未处理其他模块的既有弃用/cinterop 配置警告。源码/文档空白检查通过，临时构建日志已清理，无提交。
- 后续基类修订：restoreRemoteException 的 T 收紧为 RemoteException；基类 message 无 backing field，不在 JSON 内递归编码自身，也不缓存可能过时的编码结果。普通基类允许业务异常族在其他模块声明，各族显式选定自身 serializer，避免不安全强转、反射或手写子类分派。
- 测试改为从异常自身 message 取得载荷，新增 JSON 字段集合/重复编码/还原后再编码，以及直接抛出已知异常的两项检查。旧函数和测试文件一并重命名，无兼容别名。
- 基类修订验证：复用同一 IDEA/Temurin 25 Daemon，五目标库实际重新编译通过；JVM 和 Linux x64 Native 各十一项测试实际通过，零失败/跳过，配置缓存复用。源码旧名/空白检查通过，临时日志清理；未在其他平台运行测试或执行真实 kRPC 往返，未改生产 RPC 调用。

FIXME:

#### 已落实：集中 sealed 远端异常

- 用户批准集中 sealed 方案及首个 currentFlow NoMatch 分支；已改造 RemoteException/restoreRemoteException，新增无参数 NoMatchException。确定语义与模块边界见 [异常还原 helper](../../checklist/rpc-state.md#异常还原-helper)，不再保留每异常族 serialize 或 reified T 入口。
- 官方要求 sealed 直接子类在同模块、同 package，普通 KMP sealed 还要求同 source set；本实现符合该约束，默认 final 具体异常子类不开放外部扩展。[Kotlin sealed 规则](https://kotlinlang.org/docs/sealed-classes.html)、[自动多态序列化](https://github.com/Kotlin/kotlinx.serialization/blob/master/docs/polymorphism.md#sealed-classes)。
- 测试改用正式 NoMatchException，移除测试专用 KnownFailure 及 GenerationMismatch 和对应参数往返项；没有为了保留测试新增虚构生产错误。generation 的失败分支与参数仍待审，其他原错误结果不随本次集中而替换。
- 未新增依赖或修改 currentFlow 返回类型、服务实现和客户端接入；本模块的异常类型已可用于后续绑定实现，但不宣称后端已抛出 NoMatchException。
- 验证：复用项目 IDEA 与空闲 Temurin 25 Daemon 1951973；五目标库实际重新编译通过，JVM/Linux x64 Native 各十项测试实际通过（零失败/跳过），包括正式 sealed 类型的 message 无递归、自动恢复及无显式泛型调用。测试数减少一项源于移除尚未成为协议的参数型示例异常，非测试跳过。
- 配置缓存复用，旧泛型/测试异常清理与源码/文档空白检查通过，临时日志已清理；未运行其他平台测试、真实 kRPC 往返或现有业务操作，没有创建提交。

FIXME:

#### 当前审查：AgentRuntime

- 用户认可向前端暴露 AgentRuntime 的能力及部分 Flow；本轮优先复用原运行层，而非把 AgentRuntimeViewModel 的 UI 命令全搬过来。
- 原接口见 `Kodex/agent-runtime/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/contract/AgentRuntime.kt`：继承 KodexAgentState，声明挂起 resume、可写 pendingSteer、带 Job 的 runningTurn 和 unifiedExecToolClient。
- 首批三个方法已获批并添加，签名与语义见 [AgentRuntime checklist](../../checklist/rpc-agent-runtime.md)。复用已序列化的原列表元素和既有模块依赖，无新 DTO、序列化注解或依赖改动。
- 原消费使用 `pendingSteer.getAndUpdate { emptyList() }`（KodexAgentRuntimeComposition.kt:84-86，agent-runtime/impl）；原 submitComposer 在运行期间追加 StableUserMessage（AgentRuntimeViewModel.kt:217-220，app/viewmodel/agent）。新契约适配不能绕过 CAS 恢复已消费内容；用户已允许无新订阅值时有节制重试，具体合并活性验证未完成，见下方 CAS 小节。
- 用户确认 RPC resume 沿用原 AgentRuntime.resume 语义，具体设计见 [Resume](../../checklist/rpc-agent-runtime.md#resume)；不再把启动后立即返回列为待选路线。用户要求继续完善后，已添加 `resume(sessionIndex: Int): Unit` 挂起契约，尚无运行适配。
- resume 验证：复用 IDEA 与 Temurin 25 Daemon 1652546；RPC 四个可用目标实际编译通过，macOS ARM64 仍因既有依赖条件跳过；27 项既有 JVM 值测试实际运行通过。AgentRuntimeRpc 四方法的生成结构及 resume 的 Int/Unit 挂起签名检查通过；无新值类型，不增加模拟“声明会执行”的测试。没有运行 resume 或证明其取消/并发适配；既有 Mosaic 配置缓存警告未处理。
- 原底层 resume 等待完整 operation，前端 ViewModel.resume 则启动 owner 任务后返回；接入仍需区分等待与后端任务所有权，不能直接把 RPC handler 的 Job 当作任务 owner。submit 包含 append、启动运行和自动标题，不是 appendUserMessage 的简单别名，留后续审查。
- runningTurn 三方法已获批并添加，签名及语义见 [Running turn](../../checklist/rpc-agent-runtime.md#running-turn)；不再将布尔投影或取消当前 turn 列为未决。state 的 RequestResponse 变体内含 SharedFlow，仍须单独设计流式传输，不直接给整个原 state 补 Serializable。
- 原 Runtime 在 resume 结束的 finally 清空 runningTurnSlot，因此取消请求发出后可以暂时仍为 true；直到实际退出再发布 false。原 ViewModel.cancel 还取消 mutableRuntimeOperation（AgentRuntimeViewModel.kt:240-244），本候选不宣称覆盖手动 compaction 等全部 Stop 行为，不把这些额外操作的取消范围暗中并入。
- 本组无新 DTO、序列化依赖或 Job ID/远程句柄；仅声明方法，无取消服务或前端接入。复用 IDEA/Temurin 25 Daemon 1652546，四个可用目标实际编译通过，macOS ARM64 仍跳过；27 项既有 JVM 值测试实际运行通过，七个 Runtime 方法及新增 Boolean/Flow<Boolean>/Unit 的 stub/invokator/callable 检查通过。未运行 turn、取消清理或 Flow 时序测试；既有 Mosaic 配置缓存警告仍在。
- clearPending 原义是把待处理工具调用完成为 user interrupt（KodexAgentStateExtensions.kt:57-65，agent-state/contract），不是清空 pendingSteer。storage/settings 复用已有 timeline/CAS；不暴露 CoroutineScope、通用 modify(lambda) 或 unifiedExecToolClient 行为对象。
- 本组仅添加三方法契约及三项原值测试（四类 Steerable、顺序/重复项、空队列/null、未知类型）；没有客户端 CAS 或运行服务。运行命令与流式输出不随“暴露 AgentRuntime”自动全量获批。
- 验证：复用 IDEA 导航及 Temurin 25 Daemon 1652546；RPC 的 JVM、Linux x64/ARM64、Windows x64 实际编译通过，macOS ARM64 因既有依赖条件跳过。contract 27 项 JVM 值测试实际运行通过（新增 3），AgentRuntimeRpc 三方法的 stub/invokator/callable 及 Steerable 泛型返回/参数检查通过。配置缓存仍有既有 Mosaic 警告并丢弃；未运行内存 RPC、CAS 消费竞争或客户端状态恢复。

FIXME:

#### 当前审查：输出流引用与完整 replay

- 用户将独立流标识进一步收敛为随机标记：每个 Agent 只提供当前活跃流，currentFlow 校验活跃状态和标记，匹配后 replay，不保存按编号寻址的旧流。已定方向见 [输出流引用](../../checklist/rpc-agent-runtime.md#输出流引用)，撤回旧流保留窗口/registry 候选。
- 原 StreamingOutput 在每次 OutputItemAdded 时创建，先 emit Added 再更新 state；其 MutableSharedFlow 的 replay=Int.MAX_VALUE。直接订阅同一原流可取得保留的前缀和后续事件，不需要额外实现 replay/live 拼接。源码：`Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt:148-169,504-516`。
- 当前 latestIndex 是已提交的全局存储快照索引，不是输出流序号；Started/OutputItemAdded 只更新 state，不增加 latestIndex。OutputItemDone 才写对应历史，之后 emit Done 并恢复 Started；流期间 updateSettings 也能推进存储索引而不更换输出流（同文件 :202-243,398-406,429-444,569-580）。因此只监听现有 latestIndex 可能直到输出落盘才得到变化，不能可靠发现新流。
- 原 History 同时独立收集 latestIndex 和 agentState.state，后者驱动 streamingItem（AgentHistoryViewModel.kt:101-110,136-140，app/viewmodel/history）；已选方案保留两类观察来源。
- 重试核查：compaction runtime 在 Retryable/Continue 后再次调用 requestResponseApi，抛出的异常则向外传播（`Kodex/agent-runtime/decorator/compact/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/compact/KodexAgentCompactionRuntime.kt:39-90`）；每个新 OutputItemAdded 都产生新 StreamingOutput。随机标记绑定该输出流实例，而不是一个 resume、逻辑 turn、存储 index 或供应商 output_index；重试即使没有存储推进也不能复用旧流身份。
- 原前端 remember(events)/LaunchedEffect(events) 按流对象重新建立 accumulator（`Kodex/app/view/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/StreamingRequestResponseView.kt:151-179`）；接入后按随机标记隔离本地流，不能把上次未完成输出的 delta 与重试流串接。已提交输出仍由历史读取恢复；这只是沿用现有展示方式，不增加请求重试日志。
- 用户明确以简化后端状态为目标：当前实现已有 StreamingOutput/state/全量 replay，后续主要补随机标记、传输投影及 currentFlow 校验适配，不新建历史流缓存。发布标记后若前端请求时流已结束或被替换，则校验失败，不再承诺为迟到订阅保留旧流。
- 校验应在服务端实际绑定订阅时读取同一当前状态快照并捕获流实例；之后的新流不能使已捕获引用变成另一个对象。原始 SharedFlow 不自行正常完成，已绑定订阅在完成/失败/取消/状态切换后的结束和释放仍需落实；这不要求新增历史流 registry。
- 随机标记类型、传输模型与 currentFlow 签名已获批并落下声明；生成方式和跨线错误表达仍待接入审查，无运行原型。Started 无输出流时不可伪造可用流，标记也不是认证凭据。
- 用户已接受前后端流程审查：前端独立观察 state、按标记复用/切换本地流、过期不重试旧标记、状态离开或关闭时取消旧上游；已迁入 checklist，不再列为候选。后端额外主动结束已绑定流未被要求，不据此新增终止协议；运行时序仍未验证。
- 官方 SharedFlow 1.11.0 文档确认新订阅先接收 replay 后接收新值，且 SharedFlow 不正常完成：[文档](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/)。本轮只核查源码/文档，未修改模型、流实现或契约，未运行 RPC/replay 时序测试。

FIXME:

#### 已批准：state 与 currentFlow

- 用户批准本组后要求继续下一组；已添加状态/取流三方法，确定签名见 [RPC AgentRuntime](../../checklist/rpc-agent-runtime.md#输出流引用)。新增四项状态模型测试与一项原事件 replay 值序列化测试；测试不执行真正订阅。
- 在 rpc/models 新增传输用 AgentStateValue，保持原 KodexAgentStateValue 的分支命名和结构：Empty/UserMessage/AssistantMessage/ToolPending(events)/ToolCompleted/ExternalWrite/Compacting，以及 RequestResponse 下 Started/Message/AgentMessage/Reasoning/ToolCall/Unknown。
- 只把五种携带 SharedFlow 的输出变体改为持有 `nonce: Long`；Started 保持无流，不用 null/空标记伪造输出。Long 为不透明随机标记，不编码序号、索引或时间；具体生成实现不在本组范围。
- ToolPending 继续复用 List<PendingToolEvent> 并保留非空约束；ResponsesStreamEvent 直接使用原自定义 serializer。原类型已可序列化，本组不修改它们或原 KodexAgentStateValue，也不引入 phase/nullable payload 的平行组合模型。
- 改动限于 rpc-models 显式 api 依赖 agent-storage-clean-models、新传输模型、Runtime 三方法及值测试；不实现 mapper、nonce 生成、currentFlow 服务或客户端状态恢复。该新状态类型是为替换不可直接传输的流引用，不复制工具参数/事件 DTO。
- 校验失效与其他错误须可区分的方向已定，具体跨线错误表达仍留实现接入审查，不宣称声明方法已经保证类型化异常传输。
- 验证：复用 IDEA 导航及 Temurin 25 Daemon 1652546；rpc-models/rpc-contract 的 JVM、Linux x64/ARM64、Windows x64 均实际编译通过，两个 macOS ARM64 任务因既有依赖条件跳过。models 12 项、contract 28 项 JVM 测试实际运行通过（共 40 项，新增 5），涵盖全部 13 个状态变体、原待处理工具值、Long 范围与事件自定义 serializer 的未知载荷保留。Runtime 十方法生成结构及状态/事件类型元数据检查通过；未运行 kRPC 动态序列化解析、replay/过期竞态或 UI，既有 Mosaic 配置缓存警告仍在。

FIXME:

#### 已批准：手动压缩与 pending tools 清理

- 用户批准 forcedCompact，并在澄清 pendingSteer 与 pending tools 区别后批准独立 clearPending；已添加两个挂起 Int 方法，无新模型、序列化或依赖。确定语义见 [手动压缩](../../checklist/rpc-agent-runtime.md#手动压缩)与[待处理工具清理](../../checklist/rpc-agent-runtime.md#待处理工具清理)。
- pendingSteer 清空继续使用原队列 CAS；clearPending 逐个完成待处理工具为 user interrupt，涉及持久化与状态转移。撤回“不增加独立 clearPending”的中间决定，不将两类 pending 合并。
- 原扩展见 `Kodex/agent-state/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentStateExtensions.kt:43-65`；clearPending 是逐事件提交循环，不是整个列表的单事务。原实现不在本轮删除或修改。
- 该批次未实现服务、compaction Hook 迁移或 UI；其后用户已确认普通运行与手动压缩统一 runningTurn/Stop，见统一设计，实际任务槽位适配仍待实施。
- 验证：复用项目 IDEA；原 Daemon 已退出，确认无运行实例后沿此前许可在同一本机 Temurin 25 JVM 启动 Daemon 1951973。RPC JVM、Linux x64/ARM64、Windows x64 实际编译通过，macOS ARM64 仍因既有依赖条件跳过；28 项既有 contract JVM 值测试实际运行通过，Runtime 十二方法及两项挂起 Int 签名的生成结构检查通过。未运行工具清理、远程压缩或取消行为；既有 Mosaic 配置缓存问题导致缓存丢弃。
- 消息写入的旧文档差异已追溯，见下一组；本清理/压缩组不顺带修改消息行为、原指导或其他工作。
- completeToolCall 的原参数是跨 timeline 联合，后续工具结果组已单独获批并补齐该子接口序列化；清理/压缩组本身没有顺带修改其类型。

FIXME:

#### 已落地：用户消息写入

- 用户批准单一挂起 appendUserMessage，已添加声明与两项原内容值测试；确定边界见[用户消息写入](../../checklist/rpc-agent-runtime.md#用户消息写入)。原参数类型已序列化且依赖可用，无新 DTO、原类型注解或依赖改动；injectHistory RPC 已撤回。
- injectHistory 当前生产调用仅有后端 SteerRuntime 的投递（`Kodex/agent-runtime/decorator/steer/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/steer/SteerRuntime.kt:35-40`）及旧 TurnHookRuntime 的 stop 续跑/附加上下文（turn-hook 同级模块，TurnHookRuntime.kt:118-141）；app 下没有调用。前端已有 pendingSteer CAS，由后端决定何时领取并写入，不需要绕过它直接注入。
- 旧控制型 Hooks 将按一期设计裁剪，但后端 steer 仍需原 injectHistory；不暴露 RPC 不等于删除原方法。没有具体前端调用需求时，不因它属于 AgentRuntime 继承能力就复制一份接口。
- 事实追溯：Kodex 提交 80e4d277（2026-08-31）明确移除了 markNewTurn，把轮次判断放入 appendUserMessage，并把 injectHistory 改为遍历 stable clean events 时按同样条件更新 turnId。旧 checklist 的独立 markNewTurn 指导来自 wrapper 提交 57e71267（2026-07-31）；shared-context/agent-state-abstraction-levels.md:27 仍是该旧流程。此处记录历史，不擅自修改这些文件。
- appendUserMessage 当前在同一写入边界追加用户事件、timestamp，并按此前最近的用户/助手事件决定是否轮换 turnId：非 Commentary 助手消息之后的下一条用户输入轮换；初始化、连续用户消息或 Commentary 后的追加不轮换（KodexAgentStateImpl.kt:341-359,749-764）。测试已覆盖 final 后轮换、连续用户追加不重复轮换（KodexAgentStateImplTest.kt:869-905）；本轮仅读测试，未重跑。
- injectHistory 原 KDoc 与轮次实现的差异只保留为后端研究记录，不再作为本组 RPC 审批内容，不顺带修改原方法。
- 原 submitComposer 在运行时追加 pendingSteer，空闲时经 acceptNewTurn 调用 appendUserMessage、启动 resume 并触发标题生成（同文件 :200-224,524-546）；接入改用统一 runningTurn 判定，后端仍以当前真实状态校验，不能把前端观察当作准入保证。
- UserMessageContentValuesTest 覆盖三种原 ContentItem、图片 detail 默认值、顺序和重复项，以及未知/null/缺字段拒绝；空列表序列化成功不代表后端允许提交空消息。
- 验证：复用项目 IDEA 与 Temurin 25 Daemon 1951973；RPC JVM、Linux x64/ARM64、Windows x64 实际编译通过，macOS ARM64 因既有传递依赖 cinterop 条件跳过。33 项 contract JVM 值测试实际运行通过（新增 2）；Runtime 十八方法及 appendUserMessage 的原内容列表/Int 生成签名检查通过。既有 Mosaic 配置缓存问题导致缓存丢弃，未扩展处理。
- 未实现消息写入服务、迁移前端提交编排或修改原 turnId 算法；未运行真实消息提交、轮次行为测试或内存 RPC 往返。

FIXME:

#### 已定：标题触发与改名协调

- 名称读写已统一至 SettingsTimelineRpc，不新增名称 Get/Flow 或普通 rename RPC。手动和自动标题均使用 CAS，按目标字段判定冲突；撤回成功改名即取消自动任务的候选，见下节。
- 原前端 submit 的 acceptNewTurn 在 append 后启动 resume，再调用 startAutomaticTitle；后者读取全局标题设置并交给 AgentTitleGeneration，不等待生成结束（`Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt:505-546`）。
- 用户选择消息接受后由后端自动触发标题，撤回独立 startAutomaticTitle RPC 候选；同步修订 appendUserMessage 原“无标题副作用”约定。已定规则见[自动标题](../../checklist/rpc-agent-runtime.md#自动标题)，方法签名不变。
- 用户进一步明确原因：前端不执行模型调用，自动标题必须保留后端；该通用执行归属已记录到 RPC 架构业务边界，不把自动标题当作前端展示计算。
- 原生成器只取首个非空 InputText；图片-only 不消耗机会，关闭生成或非默认名遇到首条文本会消耗机会。默认名检测为 `Session [0-9]+`；任务与 consumed 均为 owner 内存状态（`Kodex/app/shared/session-title/src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/AgentTitleGeneration.kt:25-85`）。最新选择依循 Runtime 子协程生命周期且不补生成，见下节，不新增跨 owner 恢复要求。
- 原 renameThread/updateSettings 会失效自动任务，生成结果回写检查 attempt 与旧名称（同文件 :116-158）；迁移时仍须遵守原标题写入准入，不能先读 settings 再无条件写旧全量值。现有测试包含显式改名胜出和历史回退资格恢复；本轮仅读源码，未运行测试。
- suppressAutomaticTitle 的原生产调用来自建议子任务创建（`Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt:150-176`）。用户已选择仅依赖名称判断，不复制 suppress RPC；批次命名改由专属后端创建入口负责，见下方子任务小节。
- 标题触发批次仅更新设计及 appendUserMessage KDoc，未改方法签名、模型或运行实现；后续 CAS、作用域及子任务名称规则已确认但尚未接入，不因设计获批而自动实施。

FIXME:

#### 已定：标题依循 Runtime 子协程生命周期

- 用户明确生成标题是 AgentRuntime CoroutineScope 下的子协程，依循作用域取消即可，不需要补生成；确定规则见[自动标题](../../checklist/rpc-agent-runtime.md#自动标题)。不新增跨 owner 任务恢复、持久化进度或专门保活。
- 原实现确实由传入 scope.launch 创建标题任务并传播取消（`Kodex/app/shared/session-title/src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/AgentTitleGeneration.kt:58-83`），close 取消 activeJob（:134-135）；当前 ViewModel 将 scope 传给该对象（`Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt:126`）。迁移改由后端 Runtime scope 持有，不改变为独立进程任务。
- 原 consumed 属于实例，历史回退通过 retained-text 判断恢复资格（AgentTitleGeneration.kt:94-113，AgentRuntimeViewModel.kt:774-783）；现有测试 `AgentTitleGenerationTest.kt:106-189` 覆盖回退两种情形。本轮只读，未执行。
- 上轮提出的激活时额外扫描历史、建立跨 owner 一次性保证不作为新增要求；保留原输入资格和回退逻辑。不恢复/补跑旧任务与之后新消息的正常资格判断不同，不据“不补生成”新增全 Session 持久化门闩。
- 生成结果仍受已定 settings CAS 和 owner 关闭约束；后续用户已明确裁剪子任务显式 suppress，不增加跨 owner 抑制状态。只更新设计记录，没有修改契约或运行代码。

FIXME:

#### 已定：CAS 失败后的等待进展

- 原[全量 CAS](../../checklist/rpc-state.md#全量-cas)要求 false 后等待订阅推进，但普通 StateFlow 不保证每次真实变化都可观察；用户已选择有节制重试，不重新引入 ABA 检测或 settings revision。
- 原队列确为 MutableStateFlow，运行器通过 getAndUpdate 清空（`Kodex/agent-runtime/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt:69-86`）。允许的时序：前端观察空队列；另一提交使后端非空；本次以空队列为 expect 的 CAS 返回 false；运行器清空，而订阅未观察到中间非空值。此后无新变化时，等待不同值可能一直挂起。
- coroutines 1.11.0 明确允许跳过快速更新、抑制与上次发出值相等的值；本地 sources.jar 的 StateFlow.kt:386-406 在收集时读取最新状态并按 equals 决定是否 emit。[官方 StateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/)。上述为源码支持的时序推导，未运行并发实验。
- 原 SuspendMutableStateFlow、GlobalRpc、AgentRuntimeRpc 与 SettingsTimelineRpc 的等待注释已随最新规则修订；随后本地委托和 update 扩展已在 rpc/client 落地，但尚未接入业务状态，不能将此记录描述成已发生的 CLI 故障。
- settings timeline 的尾索引/cacheNonce 可提供独立于设置值的进展，适配时应保留；但不能据此假定全局 settings、pendingSteer 或仅观察派生值的通用 helper 都有同样保证。
- 用户批准保留 Boolean CAS，仅放宽客户端等待条件；确定规则已归入 checklist，不添加比较快照返回值、Get 轮询、DTO 或版本状态。字段冲突与取消边界不变。
- 本轮只修改设计和四个文件的 KDoc，未添加重试实现或固定等待数值。接入测试须覆盖 A→B→A 被合并、回执前订阅已更新、取消/失活及目标字段真实冲突。
- 验证：复用已打开的 IDEA 导航；四个契约文件剥离注释后的代码哈希保持不变，五份文档的 95 个相对链接/锚点、源码路径、任务结构与空白检查通过。本轮无可执行行为变更，未重跑编译/测试；未创建临时文件。

FIXME:

#### 已定：settings CAS 与标题冲突

- 用户指出当前设置与历史读取的重叠，进一步明确只有 settings timeline 需要前端写入：普通 TimelineRpc 和其他五条保持只读，SettingsTimelineRpc 额外提供 CAS；撤回给所有 timeline 增加 CAS 的初步方向，不新增通用 MutableTimeline 抽象。可写性决定已同步 checklist。
- 用户已批准完整合并：原 compareAndSet 已移入 SettingsTimelineRpc，AgentSettingsRpc 三方法及文件已删除；确定边界见[settings 读写](../../checklist/rpc-settings.md#rpc-读写)及 [timeline](../../checklist/rpc-timeline.md#读取边界)。只在具体 settings 服务增加一项，不改其他五条只读服务或原模型、依赖。
- 本地 SuspendMutableStateFlow 保留，其说明已覆盖 timeline 驱动的投影；settings 适配尚未实现，通知驱动取值不是轮询初始化 Get。删除重复 Get/Flow 后已更新相关指南，不将声明合并视为客户端接入完成。
- 合并验证：复用 IDEA 和空闲 Temurin 25 Daemon 2385167；RPC JVM、Linux x64/ARM64、Windows x64 实际编译通过，macOS ARM64 因既有依赖条件跳过。33 项既有 JVM 值测试实际通过；settings 11 方法、其余五条各 10 方法及原 Boolean CAS 签名的生成结构检查通过，旧 AgentSettingsRpc 生成类已移除，普通父接口无服务生成。
- 无新值类型故未新增值测试，未实现后端原子 CAS 或客户端 timeline 投影；既有 Mosaic 配置缓存问题未扩展处理。源码旧服务引用、文档链接和空白检查通过，未运行真实 settings 更新/重试或内存 RPC。
- 用户纠正：手动与自动改名都是 CAS，失败后检查本次目标字段；仅其他字段变化可用新快照重试，目标字段变化则为真正冲突，不再自动覆盖。确定规则见[Settings 的字段冲突](../../checklist/rpc-state.md#settings-的字段冲突)和[自动标题](../../checklist/rpc-agent-runtime.md#自动标题)。
- 撤回“成功手动改名即取消自动任务并消耗机会”的候选；不让前端改名自动优先于已提交的自动标题，也不把 CAS false 解释为发生过写入后撤销。对 settings 的通用无条件重算 update 不足以保留此次用户编辑的基准。
- 现行事实：AgentTitleGeneration.renameThread/updateSettings 先 invalidate 再写，原 updateThreadName 扩展先读全量 settings 再调用持锁 updateSettings；不存在可直接转发的原子 CAS。分别见 `Kodex/app/shared/session-title/src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/AgentTitleGeneration.kt:116-153`、`Kodex/agent-state/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentStateExtensions.kt:13-16`、`Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt:427-435`。原改名测试不构成新 CAS/标题提交竞态的证明；主线合并后的具体接入限制见[实现核对](#已核对session-settings-的实际比较边界)。
- 本组仅更新设计，不新增 DTO、RPC 方法、字段冲突 helper 或原子写入实现；未编译或运行竞态测试。普通改名的界面校验与冲突提示在接入时继续收敛；子任务抑制已裁剪，不由同值 CAS 表达“保留这个默认标题”的额外意图。

FIXME:

#### 已落地：Agent 全局 latestIndex

- 原始后端已有 `KodexAgentState.latestIndex: StateFlow<Int>`，表示事务完成后发布的全局可见存储位置，不是旧 execution 自造字段（`Kodex/agent-state/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentState.kt:141-148`）。
- 真实前端消费者包括 History 刷新/pending tools、HistoryIndex 及当前 tokenCount 投影：`Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt:102-108,477-487`；`Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt:143-146,164-166,471-475`。
- 用户已批准并添加 AgentRuntimeRpc 的 getLatestIndex/getLatestIndexFlow，确定语义见[全局存储位置](../../checklist/rpc-agent-runtime.md#全局存储位置)；无新模型、序列化或依赖。
- 验证：复用项目 IDEA；原 Daemon 1951973 已退出且无其他运行实例，沿已有许可使用同机 Temurin 25 启动 2385167。RPC JVM、Linux x64/ARM64、Windows x64 实际编译通过，macOS ARM64 因既有依赖 cinterop 条件跳过；33 项既有 JVM 值测试实际通过，Runtime 二十方法及新增 Int/Flow<Int> 的 stub/invokator/callable 检查通过。
- 本组只有原始类型签名，没有新增值测试；既有 Mosaic 配置缓存问题仍导致缓存丢弃。未实现全局发布、缓存、订阅或 UI 适配，未执行事务发布/回退或内存 RPC 运行验证。

FIXME:

#### 已落地：代际不匹配异常

- 用户批准并添加 utils/rpc-exception 的无字段 GenerationMismatch；复用现有 message 编解码，恢复函数无需修改。TimelineRpc 与两项历史命令 KDoc 已同步；确定的使用和恢复规则见 [RPC timeline](../../checklist/rpc-timeline.md)，不新增方法、依赖或返回联合。
- 原生产实现目前仍用 require 比较前端 history window generation（AgentRuntimeViewModel.kt:309-316；SessionViewModels.kt:310-327），不是该新错误或后端 timeline 代际校验。后端 owner 重建身份及 generation 来源仍需单独落实，不能以新增异常证明已解决。
- 新增两项 helper 测试：新分支的仅 type 载荷/自动恢复及与既有分支区分；失败只执行一次 block、不隐式重试。原取消测试扩展到三个已知异常载荷。
- 验证：复用 IDEA 与空闲 Temurin 25 Daemon 2385167；helper 五目标库编译通过，JVM/Linux x64 Native 各 14 项测试实际通过，零失败/跳过。RPC 仅 KDoc 改动、签名未变，未重复编译其模块；文档与空白检查通过。
- 未修改现行代际算法、历史操作或客户端恢复；未运行真实 RPC、代际失效或 owner 重建验证。

FIXME:

#### 已落地：root Agent 存储 URI

- 子任务接受结果复用原 `SuggestedSessionMeta(uri, name)`；当前 ApplicationViewModel.createSuggestedSessions 在创建后读取 rootAgent.storageUri 构造该值（`Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt:142-190`）。completeToolCall 已允许提交原结果，但现有 RPC 只有 Session index、settings 名称，没有 URI 读取入口。
- 用户已批准并添加 AgentRuntimeRpc.getStorageUri；原 AgentViewModel.storageUri 来自 session.storage.uri（AgentRuntimeViewModel.kt:96）。确定边界见[存储定位](../../checklist/rpc-agent-runtime.md#存储定位)，无新模型、序列化或依赖。
- 原 CachedAgentStorage.uri 也经过 requireActive（`Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt:58-63`）；filesystem 的 URI 格式由后端平台实现负责，本组不重写格式或假设所有后端均为 file URI。
- 验证：复用 IDEA 与空闲 Temurin 25 Daemon 2385167；RPC JVM、Linux x64/ARM64、Windows x64 实际编译通过，macOS ARM64 因既有依赖条件跳过。33 项既有 JVM 值测试实际通过，Runtime 二十一方法及 String 挂起返回的 stub/invokator/callable 检查通过；仅原始类型签名，没有新增值测试。既有 Mosaic 配置缓存问题未扩展处理。
- 未实现 URI 读取服务或子任务创建，未进行真实 RPC/活跃准入验证。后续用户已要求专属批量创建 RPC，撤回“不增加 createSuggestedSessions”的候选；批次命名、返回信息和启动范围见下方审查，普通创建契约不随之撤销。

FIXME:

#### 已落地：SessionNotFound

- 用户已批准并添加无字段 SessionNotFound，规则见[Session 管理](../../checklist/rpc-session.md#管理命令)；它是 utils/rpc-exception 的第四个封闭分支，不代表 Session 服务已接入。
- 已补 GlobalRpc 的保活/归档/取消归档/完整 fork 与 delete 注释，添加两项 helper 值测试并扩展取消载荷测试；现有内存 RPC fixture 纳入新类型的 unary/Flow 还原与零容量边界验证。复用原依赖，无新方法或异常包装器。
- 验证范围为 helper 的值还原及已有测试专用 kRPC 的 JVM/本机 Native 往返；不实现 Session 查找/删除或把通用传输失败映射为不存在。deleteSession 缺失返回 false 的已定语义不变。
- keepSessionAlive、archive/unarchive、fork 等要求已有 Session 的入口，在确认缺失时使用该分支。deleteSession 缺失仍返回 false；目录查询仍返回实际条目，不因列表为空报错。没有新增 RPC 或返回 DTO。
- live Session 缓存未命中仍可快速返回 SessionNotActive，不要求每个属性访问都额外查文件。前端按需保活，若保活确认 SessionNotFound，就停止该旧绑定的自动保活/重订阅并提示缺失，不隐式新建 Session；导航和草稿处理留前端。
- SessionNotActive 只说明 owner 不可访问，不证明持久数据仍存在；已有订阅因 owner 关闭而结束也不必预先推断删除结果。归档、数据损坏、权限/传输失败和不存在的 shell ID 不归入 SessionNotFound。
- 原路径见 `Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt:157-162,190-204`；repository 的 getEntry/createFork 均 requireEntry，开根目录还验证目录存在（`Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt:79-84,154-157,190-202`）。未来从明确校验点抛出已知异常，不解析原 require 文案。
- 原 repository 用 smallestMissing 分配目录 index（同文件 :174-187），删除后可能复用；用户最新决定保留原规则，见下节。SessionNotFound 本身不承担身份分配，不为该异常增加 nonce。
- 已添加异常与测试，未实现服务适配；原生命周期/删除运行行为不变。
- 验证：复用 IDEA 和空闲 Temurin 25 Daemon 2385167；helper 的 JVM、Linux x64/ARM64、Windows x64、macOS ARM64 库编译通过，JVM 与本机 Linux x64 Native 各17项测试实际通过（新增2项），零失败/跳过。
- 初次 RPC 复验因另一处工作区的 utils-coroutines 临时 main.kt 存在未解析引用而受阻；当时的54项旧报告没有计作新增异常通过，本组未修改、回滚或从构建排除该文件。
- 公共客户端批次核实该文件已不存在后完成原复验：rpc-contract 四目标编译及 JVM/Native 各57项实际通过，含新增 SessionNotFound 往返和既有零容量边界覆盖；macOS RPC 仍因既有依赖跳过。GlobalRpc 方法/参数不变，没有实现真实 Session 查找服务。
- 构建35秒后失败，helper 的独立任务已完成；Mosaic 配置缓存仍丢弃。错误已记录，临时日志已清理，不重复占用正在编辑的依赖进行构建。

#### 已定：保留 Session 编号复用

- 用户重新评估后决定保留原空号复用，按单用户使用场景接受其边界；最新规则见[Session 编号](../../checklist/rpc-session.md#session-编号)。撤回上轮不复用、持久水位、旧库迁移及替换分配器/测试的要求，不增加 RPC 身份参数。
- 两类 nonce 的校验范围不扩大：timeline/历史命令及 currentFlow 分别校验原标记，不等于普通 CAS、append、完整 fork 等都带 Session 身份。这里是用户接受当前寻址方式，不是证明单用户不存在异步时序或所有旧请求都会被 nonce 拒绝。
- 原文件分配见 `Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt:142-187`，内存分配见 `Kodex/agent-session/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/inmemory/InMemoryKodexSessionRepository.kt:152-155`。两种原测试均断言删除后重新取得 sourceIndex：对应 FileSystemKodexSessionRepositoryTest.kt:420-447 和 InMemoryKodexSessionRepositoryTest.kt:201-227；这些预期不因本组审查而修改。
- 上轮未修改分配器、真实目录或测试，本轮无需回滚源码；仅清理已撤回的设计和任务项。SessionNotFound 的错误语义不随编号决定撤回，最新源码进度见上节。

FIXME:

#### 已定：子任务仅依赖名称判断

- 用户选择只依赖名称判断，接受默认格式的子任务名仍可能被自动标题替换；确定规则见[自动标题](../../checklist/rpc-agent-runtime.md#自动标题)。撤回独立 suppressAutomaticTitle RPC，不增加持久化抑制字段或修改 createSession 签名。
- 用户后续明确新增专属批量创建 RPC，撤回由前端逐项组合创建/命名的候选；标题生成仍全部在后端，保留其他原资格条件。批次操作范围与返回信息见下一节，父调用过期及失败清理继续审查。
- 与原行为的差异已明确：ApplicationViewModel.kt:170-175 在提交前显式 suppress，AgentTitleGeneration.kt:89-92 使当前尝试失效；默认名条件见同文件 :51-54，历史回退另按保留文本更新资格（:94-113）。不把普通 CAS 改名等同于执行原 suppress。
- 本轮仅确认裁剪与更新设计；未删除现行调用、修改生成器或新增契约，未运行子任务/标题测试。

FIXME:

#### 已落地：专属子任务批量创建 RPC

- 用户最新决定：前端负责交互确认与发起，后端提供专属批量创建 RPC，避免逐项往返；确定职责见[建议子任务批量创建](../../checklist/rpc-session.md#建议子任务批量创建)。撤回“不增加批量 RPC”和前端逐个创建/命名的候选，不将前一条短暂表述解释为取消前端确认。
- 原 ApplicationViewModel.createSuggestedSessions 先创建全部 Session，再发布 tab、异步提交子任务，最后返回 URI/name；外层 catch 调用 rollbackCreated 清理已加入 created 列表的项（`Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt:145-192`）。单个创建失败还有自身清理（`Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt:79-108`）；批次清理不是数据库事务，清理自身仍可能失败。
- 父 completeToolCall 发生在 dispatcher 返回之后；该提交失败不会进入上述批次清理，子任务可能已开始。见 `Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskViewModel.kt:66-98`；原后端在实际写入时检查 ToolPending/callId（`Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt:362-394`）。不能声称旧流程保证父子整体成功或失败。
- 用户要求直接复用 KodexAgentSettings，并摊平仅含 tasks 的参数包装；已撤回 SuggestedSessionConfiguration 的 RPC 复用、序列化及 app-contract-agent 依赖候选，不直接搬运 SuggestSubagentTaskState/ViewModel。
- 原 SuggestedSessionMeta 只有 uri/name（`Kodex/tool/multi-agent/contract/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/SuggestSubagentTaskModels.kt:59-63`），前端开 tab/保活还需 index；已添加 CreatedSuggestedSession(sessionIndex, meta)，不从 URI 解析编号或要求逐项补查元数据。
- 用户已确认创建并启动：后端完成批量创建/命名并拥有各孩子的首条消息提交和运行启动，返回不等待模型运行结束，不要求前端再逐孩子 append/resume。确定范围已收录 checklist，不改变单次普通 createSession。
- 界面配置/确认和返回后的导航归前端；普通 createSession 的创建/提交分开规则不变。父工具提交仍用原独立 completeToolCall，不隐式绑定父 Session 或替前端确认；该边界与孩子运行成功是两件事。
- 已添加 GlobalRpc 的挂起 createSuggestedSessions(tasks: List<SuggestedSubagentTask>, initialSettings: KodexAgentSettings): List<CreatedSuggestedSession>；确定签名、创建时元数据和结果顺序见 checklist，不新增参数包装。
- 原 SuggestedSessionConfiguration 确实只有 model/reasoningEffort/serviceTier/cwd/requestUserInputMode（app-contract-agent 的 AgentViewModel.kt:135-142）；旧创建器再用这些字段构造 KodexAgentSettings，其余取默认值。新 RPC 不复制这一投影，各元素及原 settings 均已有序列化且通过现有依赖可见，无需额外构建依赖。
- 每个任务使用共同 initialSettings，将 threadName 替换为该任务 name；继续经过原新建校验和 initialize，为各 Session 独立生成 turn/window 身份，而非复用输入中的身份（`Kodex/agent-storage/contract-ext/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/ext/AgentStorageInitialization.kt:12-25`）。此为专属命名输入，普通 createSession 仍采用默认名称。
- 仅返回包装放 rpc/models，解决原 meta 无 index 的实际缺口；不修改原工具 SuggestSubagentTaskArgs/SuggestedSessionMeta 协议，也不删除现行 UI 配置类型。
- 用户将失败收敛为普通未知异常、后端尽力减少副作用；撤回准备期整批清理保证和部分结果协议的候选，不加入专属 RemoteException 分支。不存在已设计的类型化创建失败，不等于 I/O、初始化或运行永远不会失败。
- 成功只表示创建和后台启动任务交接完成，不证明首条消息/模型执行成功；回执丢失不重放创建，取消前端等待不撤销已接受的后端工作。不实现批次 journal、结果缓存或跨 Session 事务。
- 新增模型与契约各三项值测试：结果顺序/原 meta 编码/必填字段，以及摊平任务列表/完整 settings/非法值；仅测试传输，不模拟执行创建、初始化或启动。
- 本轮只添加声明、返回模型和六项测试，无原类型或依赖改动。未创建或删除真实 Session，也未实现后台批次、清理、tab 接入或模型调用。
- 验证：复用 IDEA 与空闲 Temurin 25 Daemon 2385167；models/contract 的 JVM、Linux x64/ARM64、Windows x64 实际编译通过，两模块 macOS ARM64 因既有依赖条件跳过。models 31 项、contract 36 项 JVM 测试实际通过，共 67 项，含新增六项，零失败/跳过。
- GlobalRpc 30 个生成 stub 方法/invokator/callable、摊平 tasks/initialSettings 参数及新返回 serializer 检查通过。构建仍有既有 Mosaic 配置缓存问题，缓存被丢弃；未运行 Native 值测试、实际 RPC、批量创建或错误清理验证。
- 原 SuggestSubagentTaskFeedbackTest 只用 mock dispatcher 覆盖接受/拒绝反馈，不是批次中断测试；本轮仅读该测试，未执行。

FIXME:

#### 已定：通知模型与事件契约

- 原 AgentNotification/PersistedSessionNotification 是前端 ViewModel 的 id/level/message/detail 状态，捕获操作异常时生成、前端按 id 清除；不是原 AgentRuntime 原语。来源为 AgentRuntimeViewModel.kt:556-605、SessionViewModels.kt:494-505。本轮检索 app/view 未找到这些通知值的消费，不能声称已有对应 UI 必须整体跨线。
- Application.kt:322-330 的真实报告路径记录错误并调用旧 unhandled_error Hook；该 Hook 将按 notification-only 迁移，不能照搬旧控制 Hook。既有后台日志职责不因前端通知而取消。
- 用户纠正：新通知是后端生产事件、前端订阅并按联合分支调用 Hook，用于替代旧 Hook 系统。单一 getNotificationFlow 已确定；撤回“只处理调用失败、不增加通知流”的候选，不将旧 UI notification DTO 当作该协议。已定边界见 [Frontend notification only](../../checklist/rpc-settings.md#frontend-notification-only)。
- GlobalRpc 的 `fun getNotificationFlow(): Flow<Notification>` 已获批并添加；外层 Notification 一期只有 Stop，内部为四种停止原因，放 rpc/models。全局归属、Session index 及载荷已定，见下节，不迁移旧 Hook request/result 全部类型。
- 用户明确通知是纯 Flow、无 replay，按 replay=0 的 SharedFlow 语义；新订阅不读历史，无订阅者时不保存供未来消费，重订阅不补播。已同步 checklist，不影响 currentFlow 的普通模型输出完整 replay。
- 用户先裁剪为停止/错误相关通知，随后统一为 Stop 的四种原因，不保留其他五类或旧控制结果。具体载荷见下节；溢出丢弃旧通知已另行获批，不从 replay=0 本身推导丢弃策略。
- 后端事件不依赖某个前端仍在等待 resume 回执；运行结果不能只由 runningTurn=false 推导，Hook 执行仍在前端且不控制后端。生命周期事件是协议值，不能与 RemoteException 的失败分支混为同一联合。
- 通知声明、联合值与前端配置模型已落实；缓冲和执行资源策略也已确认，但事件发布、执行器及缓冲尚未实现；子任务显式抑制已决定裁剪。

FIXME:

#### 已定 Stop 四分支

- 用户要求 Stop 区分 AssistantMessage、RequestUserInput、SuggestSubagent，并将 UnhandledError 也归入 Stop；已定范围见 [Frontend notification only](../../checklist/rpc-settings.md#frontend-notification-only)。不再保留外层 Stop/UnhandledError 二分，也不将前三类压成同一 lastAssistantMessage 字符串。
- 用户明确保留外层 Notification.Stop，以便以后增加其他 Notification 类型；撤回单层 StopNotification 候选。一期仅 Stop 分支和内部四种原因，不添加其他通知占位类型。实际载荷已在下一节获批，不携带 Hook 名称、命令配置、stopHookActive 或控制结果。
- 原 Stop 在 delegate 返回后发现 AssistantMessage 或宿主交互等待时触发（TurnHookRuntime.kt:77-105），不是任意 runningTurn=false 或点击 Stop；原 helper :215-221 返回两类宿主事件的列表。用户已批准同类聚合、混合类型各发一条，不取 first 丢弃其他请求。
- UnhandledError 分支是通知数据，不是序列化任意 Throwable 或新增 RemoteException 通用兜底；错误产生位置、可用上下文与去重范围仍需细化。取消不自动归入未处理错误。
- 四个停止原因和单一 getNotificationFlow 的声明已落地，不引入 hook-contract 依赖。replay=0、溢出 DROP_OLDEST、容量及前端执行资源边界已定；发布和前端 Hook 运行尚未实现。

FIXME:

#### 已批准：通知契约与具体载荷

- 用户批准单一全局通知 Flow、四分支载荷、同类请求聚合及 Stop 错误范围；已添加 GlobalRpc 声明与 rpc/models/Notification.kt。确定规则见 [Stop 通知载荷](../../checklist/rpc-settings.md#stop-通知载荷)，不在任务重复完整协议。
- 上述三个领域值已有 Serializable，rpc/models 已依赖 clean-models，无需修改原类型或新增模块依赖。源码见 `Kodex/agent-storage/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableMessageEvent.kt:27-43`，两类 pending 值位于同 package 根的 `unstable/PendingRequestUserInputToolEvent.kt:9-18`、`unstable/PendingSuggestSubagentTaskToolEvent.kt:9-18`。
- 原 Application.reportUnhandledError 还接收目录/设置等全局错误（`Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt:322-330,396-401`），并非新 Stop 发布器；后续接入须按已批准的运行最终失败边界生产事件，而不是直接转发所有旧报错。本组不修改该运行入口。
- 新增六项 NotificationModelsTest：外层/Stop 两级 sealed 往返、原值编码不变、调用列表顺序/重复项/内嵌问题任务、可空诊断、空分组拒绝及必需字段/未知分支拒绝。它们不是事件聚合、投递或 Hook 运行测试。
- 验证：复用 IDEA 与空闲 Temurin 25 Daemon 2385167；首次构建因新测试漏导入 encodeToJsonElement 扩展失败，补齐后同环境重跑通过。两次构建合计完成 models/contract 的 JVM、Linux x64/ARM64、Windows x64 实际编译，重跑复用部分产物；两模块 macOS ARM64 均因既有依赖条件跳过。
- models 21 项（含新增 6 项）、contract 33 项 JVM 测试实际通过，共 54 项，零失败/跳过。GlobalRpc 29 方法的 stub/invokator/callable 及顶层 Flow<Notification> 签名完整，四个 Stop 子类 serializer 存在，无通知快照 Get。既有 Mosaic 配置缓存问题未扩展处理；未运行 Native 通知测试或实际内存 RPC。
- 不实现发布、Hook 执行、缓冲或前端状态编排；投递策略及容量已在后续批次确认，前端通知配置模型也已另行落实。未新增依赖或修改旧 Hook 模型/持久化。

FIXME:

#### 已定：通知慢消费者策略

- coroutines 1.11.0 的 replay 与额外缓冲独立：默认 SharedFlow 无缓冲，emit 等待订阅者接收；无订阅者且 replay=0 时直接丢弃。仅将 emit 换为 tryEmit 不足以解决投递，默认配置在有订阅者时 tryEmit 返回 false。[官方 SharedFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/)、[tryEmit](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-shared-flow/try-emit.html)。
- 用户已接受有限缓冲、溢出丢弃旧通知的尽力投递语义；确定规则见[通知投递](../../checklist/rpc-settings.md#通知投递)。在线慢消费者允许漏通知，不再列为待选项；容量已在下节确定，不新增设置字段。
- 本地源码交叉核对来自缓存的 coroutines-core 1.11.0 sources.jar，SharedFlow.kt:404-474：DROP_OLDEST 推进落后 collector 的位置，无订阅者分支在 replay=0 时不存储。仅源码/官方文档核查，未运行慢消费者实验。
- 有界条目数不等于载荷字节或端到端资源已有上限；实际 RPC 缓冲、前端 Hook 并发和超时仍随接入核对。
- 本组仅同步设计，不修改 GlobalRpc/Notification 模型、不创建通知源或 Hook 执行器；文档检查通过，未重跑编译测试或执行慢消费实验。

FIXME:

#### 已定：通知容量与前端 Hook 执行

- 用户已批准 64 条后端缓冲、前端串行 Hook、单条 10 秒执行预算及失败清理规则；确定设计见[通知投递](../../checklist/rpc-settings.md#通知投递)和[前端 Hook 执行](../../checklist/rpc-settings.md#前端-hook-执行)。这些数值是一期默认值，不是实测容量或端到端内存上限，不新增参数配置。
- 使用普通 collect 而非 collectLatest 的依据：后者在新值到达时取消上一条处理，与不因新通知中止当前 Hook 的选择不同。[官方 collectLatest](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/collect-latest.html)。实际 RPC 缓冲及退出等待仍须接入验证。
- 原实现依据：`Kodex/hook/impl/src/commonMain/kotlin/io/github/stream29/kodex/hook/impl/HookExecution.kt:21-36` 对每条配置 async；同目录 `ShellClientHook.kt:25-49,90` 使用 use 关闭进程、传播取消并设 600 秒超时。新方案不要求复用旧并发和控制结果模型。
- 原进程契约明确 close 只请求终止、须观察 exitCode（`Kodex/utils/shell-client/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ProcessSession.kt:43-88`）；已查 JVM/POSIX/Windows 实现对应 close 路径。原测试 `Kodex/hook/impl/src/commonTest/kotlin/io/github/stream29/kodex/hook/impl/ConfiguredHooksTest.kt:459-482` 覆盖取消脚本，本轮只阅读，未重跑，不能据此证明新执行器已正确清理。
- 该批次只固化容量与执行设计；后续配置、stdin 载荷、cwd/shell 规则已确认，配置模型见下节。容量批次未修改旧 Hook 或实现执行器，未执行真实 Hook。

FIXME:

#### 已落地：前端通知配置模型

- 用户确认有序 hooks、多类型选择、原 Notification JSON 输入与前端本地环境方案；已定规则见[前端 Hook 配置](../../checklist/rpc-settings.md#前端-hook-配置)及[命令输入与环境](../../checklist/rpc-settings.md#命令输入与环境)。本轮仅落实值模型和测试，不增加 RPC、后端字段或依赖。
- 已添加 NotificationHook、NotificationHookType 与 CliFrontendSettings.hooks，保留其余默认值；直接构造、copy 和反序列化都校验非空白名称/命令、非空类型集合及列表名称唯一性。类型集合去重但不按命令去重不同 Hook。
- 新增七项 NotificationHookModelsTest，覆盖四类序列化值、命令原样保留、列表顺序/集合相等性、默认空列表、构造与解码合法性、必填/null/未知类型/旧单 type 拒绝及重复类型归一；原 SettingsModelsTest 更新字段和默认值断言。
- 原配置为名称 Map，HookBody 只有 type/command，旧 HookType 含已裁剪事件（`Kodex/hook/contract/src/commonMain/kotlin/io/github/stream29/kodex/hook/contract/HookSettings.kt:11-49`）；没有修改或复用该旧协议。Notification 原值序列化测试继续覆盖实际输入联合，不另写 HookInput。
- 进程参数可复用 `Kodex/utils/shell-client/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ProcessSession.kt:8-32` 的 cwd/default shell/login/tty 和既有环境继承；只是接入依据，不代表配置模型已启动命令。
- 验证：复用项目 IDEA 和空闲 Temurin 25 Daemon 2385167；models/contract 的 JVM、Linux x64/ARM64、Windows x64 均实际编译通过，macOS ARM64 因既有传递依赖条件跳过。models 28 项（新增 7）、contract 33 项 JVM 测试实际通过，共 61 项，零失败/跳过；新模型 serializer 已生成。
- 构建仍有既有 Mosaic 配置缓存问题，缓存被丢弃，未扩展修复。未运行 Native 值测试、YAML 文件存取、命令匹配/执行、旧配置迁移或实际通知/RPC；测试命令字符串仅为数据，没有执行。

FIXME:

#### 已批准：工具结果提交

- 本工具结果批次未包含消息写入；appendUserMessage 后续已单独获批并添加，见上节，injectHistory RPC 仍不暴露。
- 用户已批准并添加挂起 completeToolCall；签名与边界见 [工具结果提交](../../checklist/rpc-agent-runtime.md#工具结果提交)，编译验证完成。
- 实际前端入口：RequestUserInputViewModel.kt:121-134 构造 StableRequestUserInputToolEvent，SuggestSubagentTaskViewModel.kt:78-91 构造 StableSuggestSubagentTaskToolEvent，均调用 runtime.completeToolCall 后独立 resumeRuntime（`Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/`）。不像 injectHistory，这里有具体前端调用需求。
- 问题、调用 id 与原参数已通过 AgentStateValue.ToolPending(events) 提供；前端保留选择/草稿/revision 与确认，不增加这些编辑操作的 RPC。后端按原 ToolPending 准入和完成结果的 projected callId 校验后提交 stable/work 及 unstable 更新；调用失效或已完成时失败，不重复追加。
- 方法只完成一个工具，不创建建议的 Session、不消费前端草稿、不自动 resume；具体创建编排及回答合法性校验的接入仍需保留，不以本组完成事件可序列化证明整个交互迁移完成。原底层校验是 pending callId，不宣称已验证全部前端题目/参数约束。
- 已仅为原 `StableCleanEvent.CompletedTool` sealed 子接口补 Serializable，外层 StableCleanEvent 不序列化；不改字段、持久化所用联合或构建依赖，不创建新 DTO。
- 新增三项值测试：前端两类完成事件及失败分支、work timeline 完成事件的联合 round-trip；所选事件在新联合与原 index/work 联合编码一致；非工具/未知/null 值拒绝。未实现工具提交服务、问题校验或 UI。
- 验证：复用 IDEA 导航和 Temurin 25 Daemon 1951973；RPC JVM、Linux x64/ARM64、Windows x64 实际编译通过，macOS ARM64 因既有依赖条件跳过。31 项 contract JVM 值测试实际运行通过（新增 3），Runtime 十三方法及 CompletedTool/Int 参数返回的生成结构检查通过。未执行工具提交、前端问答或内存 RPC；既有 Mosaic 配置缓存警告未扩展处理。

FIXME:

#### 已落地：usage reset

- 用户要求简化：不把 attempt 暴露给前端，失败后刷新用量，由用户决定是否再手动操作；撤回 prepare/consume 两个 RPC 与 attempt 序列化提案。
- 用户要求必须选择具体重置券，日期差异需对用户可见；已定设计见 [重置券选择](../../checklist/rpc-account-usage.md#重置券选择)，撤回 nullable creditId 回退。
- 已批准并添加单一挂起 `consumeUsageReset(creditId: String): CodexRateLimitResetOutcome`，不提供默认参数，空白值须拒绝；原 outcome 枚举已补序列化，不序列化 attempt。两项新值测试覆盖结果 round-trip 及未知/null 拒绝，已通过 JVM 测试。
- 现有源码并非完全没有选券：UsageResetPickerDialog 遍历 options 并调用 selectUsageReset，选择页和确认页均展示 expiresAt（`Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/CodexAccountUsageSettings.kt:153-169,200-255`）。无明细时生成 creditId=null 的 Full reset（GlobalSettingsViewModel.kt:722-752，app/viewmodel/settings），并提示由后端选择（上述 view 文件 :134-141）；这条退化路径不满足新要求，需在前端接入时移除。未运行 UI，不能据源码断言用户所见界面已正常覆盖。
- 原 usage 快照已传 credits 的 id/grantedAt/expiresAt，足够前端选券，无需新增列表 RPC。原 UsageResetOption 未含 grantedAt；是否补展示发放日期及具体日期格式继续随 UI 接入审查，不为本轮先改模型。
- 后端可内部复用创建/消费 attempt 和供应商幂等键，不要求前端持有、恢复或重放句柄；新调用代表用户的新操作，而不是跨 RPC 重试同一逻辑请求。不新增终态缓存/查询接口。
- 失败后前端调用已批准 refreshAccountUsage，由 usage Flow 更新展示，不调用初始化 Get 代替刷新。刷新只能帮助取得当前用量，不能证明上次 reset 没执行；若刷新也失败，不能伪装已有最新结果。
- 不自动再消费；用户重新查看并确认后再次调用是新操作，必须仍显式指定券，不因旧券失效换用其他券。若原调用可能仍在后端执行，刷新与消费须沿用后端已有串行化边界，不以瞬时旧快照证明操作未发生。
- 保留 Reset/NothingToReset/NoCredit/AlreadyRedeemed 与“已明确得到 reset 结果后，用量刷新失败不抹掉该结果”的原边界；已接受提交、账号校验和凭据仍归后端。确认期间账号变化已选择不增加前端绑定，见下节，不取消供应商请求前的后端账号校验。
- 原实现供接入参考：create 在锁内替换单个有效 key/账号，consume 明确收到结果后移除 key，供应商异常保留 key；这是现有内部行为，不再作为前端契约。[原 store](../../Kodex/openai/account-usage/impl/src/commonMain/kotlin/io/github/stream29/kodex/openai/accountusage/CodexAccountUsageStoreImpl.kt#L88-L181)。原前端 PreparedReset 流程见 GlobalSettingsViewModel.kt:350-414,548-584（app/viewmodel/settings）。
- 本轮仅落下单次 RPC、原 outcome 序列化和值测试；原 UI/store 可空参数尚未裁剪。无真实 reset、刷新请求或运行验证，契约声明不等于已实现参数校验、消费或前端选券行为。
- 验证：复用 IDEA 导航及 Temurin 25 Daemon 1652546；rpc-contract JVM、Linux x64/ARM64、Windows x64 实际编译通过，macOS ARM64 因既有传递依赖 cinterop 条件跳过。rpc-contract JVM 23 项值测试实际运行通过（含两项新增 reset），无失败/跳过；25 个 GlobalRpc 方法的 stub、invokator 和 callable 生成检查通过。构建仍报告既有 Mosaic 配置缓存问题并丢弃缓存，未扩展修复范围；未做内存 RPC 或供应商消费运行验证。

FIXME:

#### 已定：reset 确认期间的账号边界

- 用户明确接受依赖 OpenAI 对当前账号与指定券的校验，维持单一 creditId，不增加确认期账号参数或标记；规则见[重置券选择](../../checklist/rpc-account-usage.md#重置券选择)。撤回绑定选券快照所属账号的候选，不恢复远程 attempt。
- 旧前端先 createResetAttempt，再显示 Confirming，确认后消费同一 attempt：`Kodex/app/viewmodel/settings/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/GlobalSettingsViewModel.kt:362-401,548-553`。前端主动切换 authSource 会关闭旧确认（同文件 :247-252），但这不是后端准入保证。
- 原 store 在准备时记录 AccountKey，消费时核对当前账号，并将当次 account 传给 HTTP 客户端再次检查：`Kodex/openai/account-usage/impl/src/commonMain/kotlin/io/github/stream29/kodex/openai/accountusage/CodexAccountUsageStoreImpl.kt:88-135`；`Kodex/openai/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt:293-308`。
- 新 consumeUsageReset 只有 creditId，已有 usage 快照无账号字段；后端在收到该调用时临时准备内部 attempt，只能绑定当时的账号，不能据此还原前端选券/确认所属账号。认证与 usage 两条流也没有联合原子性保证。
- 原 store 的 AccountKey 可回退到 token，不跨线发送该私有类型。这里记录用户接受的校验边界，不据此宣称已验证供应商跨账号券 ID 的唯一性或行为。
- 当前只同步设计，未改契约、消费实现或执行真实 reset；不增加自动消费重试，禁止替换所选券的决定保持不变。

FIXME:

### 配置、认证与宿主

#### 已定：单 CLI 的内存 RPC 组合根

- 用户确认最小宿主方案；已定规则收录[内存连接与宿主生命周期](../../checklist/rpc-architecture.md#内存连接与宿主生命周期)。本次仅确认设计，不构成生产模块或 CLI 接入授权。
- 现行入口先准备 Home 与日志，再打开 Application 和 Mosaic，退出在 finally 调用 shutdown（`Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt:11-54`）。Application 已是组合根，当前 shutdown 先关闭 ViewModel/基础设施，最后释放 Home handle（`Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt:84-116`）；本组不顺带修改原资源实现。
- 测试 fixture 已使用双向有界 Channel、JSON、一个 client/server 及各自子 Job，收尾先取消双方再等待（`Kodex/rpc/contract/src/commonTest/kotlin/io/github/stream29/kodex/rpc/contract/InMemoryRpcFixture.kt:105-167`）。这是测试实现依据，不证明生产组合根、多服务共用或异常退出已经验证。
- 连接模块已验证多服务、启动失败和取消边界；实际 renderer 退出与业务资源清理仍随 CLI 接入验证，不能由独立模块测试替代。此前受阻的 RPC/目录显示复验已在公共客户端批次完成。

#### 已落地：独立内存连接模块

- 用户已授权并落实 `Kodex/rpc/in-memory`（`:rpc-in-memory`）；入口及生命周期规则收录[内存连接模块](../../checklist/rpc-architecture.md#内存连接模块)。不让 contract/models 或现行 CLI 依赖它。
- 公开入口是 `withInMemoryRpc(registerServices: RpcServer.() -> Unit, block: suspend CoroutineScope.(RpcClient) -> R)`；内部固定 JSON、双向通道及端点监视（`Kodex/rpc/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/rpc/inmemory/WithInMemoryRpc.kt:25-102`）。没有新 RPC、连接 DTO、重连器或自动重放。
- 复用现有版本别名，显式指定 serialization JSON 1.11.0，避免采用 kRPC 传递的1.9.0；无库升级或业务依赖，目录自动纳入现有模块扫描（`Kodex/rpc/in-memory/build.gradle.kts:1-18`）。未修改受阻的 utils-coroutines 文件。
- 新增17项测试：14项真实连接生命周期、3项通道行为；覆盖双服务及并发调用、Flow、未使用连接、注册/使用失败、外层取消、订阅与调用取消隔离、端点关闭、主失败及子协程清理失败、FIFO/满缓冲挂起及同时取消收发。仅使用测试服务。
- JVM 首轮的4项失败来自异常对象身份断言，已改为允许协程栈恢复副本并检查原失败仍在 cause 链；未关闭栈恢复。另修正测试 cause 遍历的泛型推断错误。
- 首次重复运行在 Native 复现外层取消被误报为端点死亡：transport 先于同级监视协程收到取消。现已同时检查连接作用域，取消测试每轮重复32次；没有把该失败作为框架故障或修改框架。
- 最终库及测试源码均通过 JVM、Linux x64/ARM64、Windows x64、macOS ARM64 五目标编译；JVM 与 Linux x64 Native 各17项实际通过，随后两轮 `--rerun` 均再次通过。其他三平台仅编译，未执行测试。
- 显式复用 Temurin 25 Daemon 208224，离线构建，最终 configuration cache 复用；最初缺少传递1.9.0的 Native 缓存由固定既有1.11.0依赖解决，没有切换设备或修改全局环境。
- 本组未访问真实 Home/凭据、创建 Session 或执行模型/shell/OAuth；没有实际 CLI 启动退出验证。该轮保留的 rpc-contract/目录显示复验后来在公共客户端批次完成，不计入这17项连接测试。

#### 已落地：公共客户端异常与 Flow 适配

- 用户确认公共入口机制后批准实施；该批在新 `rpc/client` 添加 RestoringRpcClient，规则收录[公共客户端适配](../../checklist/rpc-state.md#公共客户端适配)。旧代理不自动替换，现行 CLI 尚未使用包装器；本地状态封装随后单独获批，见下节。
- 生产实现只委托公开的 call/callServerStreaming，复用原 helper，并按 buffer(0)→catch 的顺序保护流（`Kodex/rpc/client/src/commonMain/kotlin/io/github/stream29/kodex/rpc/client/RestoringRpcClient.kt:10-36`）。没有新增状态、重试、恢复或连接所有权。
- 该批生产依赖仅 RPC core、协程及 utils/rpc-exception，测试依赖 rpc/in-memory；后续状态封装再引入原 contract。复用已有版本、模块发现和构建惯例，不改原契约、连接模块或异常 helper。
- 新增28项测试：6项委托边界、22项真实 JSON 往返。覆盖四种已知异常的普通调用/取流拒绝/流中失败、即时取流表达式失败、未知或非法载荷、取消、无自动重试、下游 JSON 文案异常不被误还原、退订/take/unary 取消释放、冷流及旧原始代理不自动包装。
- 五目标库及测试源码编译通过；JVM、Linux x64 Native 各28项实际通过，并各强制重跑一次通过。其他三平台只编译，没有执行；本组源码首轮无需修正。
- 显式复用 Temurin 25 Daemon 208224、离线构建；本模块首次构建1分47秒成功，configuration cache 保存。未升级依赖、改全局环境或切换设备。
- 原阻塞文件已不存在，本轮另按原授权补跑 rpc-contract 四目标编译及 JVM/Native 各57项、目录两模块 JVM 编译及1项标签测试，全部实际通过；没有更改旧运行源码。RPC macOS 跳过，联合复验3分45秒成功，Mosaic 的既有配置缓存问题导致 cache 丢弃，不修改其构建。
- 本轮不包含真实 CLI 接入、业务服务、Session 恢复/CAS/缓存、目录日期填充或菜单交互验证；临时日志清理后不保留测试以外的运行资源。

#### 已落地：本地挂起 StateFlow 的最小封装

- 用户批准最小委托、三个更新扩展及50ms节奏；已定规则收录 [SuspendMutableStateFlow](../../checklist/rpc-state.md#suspendmutablestateflow)。原接口保持不变，实现在 `Kodex/rpc/client/src/commonMain/kotlin/io/github/stream29/kodex/rpc/client/SuspendMutableStateFlow.kt:10-69`。
- `asSuspendMutableStateFlow` 委托原 StateFlow 与挂起 CAS 回调，不创建镜像或收集任务；update/getAndUpdate/updateAndGet 只在 false 后延迟重算，返回成功比较的值，检查取消且不吞异常。
- 复用项目 IDEA 导航和空闲 Temurin 25 Daemon 208224；rpc/client 新增既有 rpc-contract 依赖，测试使用既有1.11.0协程测试库。不迁移接口、升级依赖或改变 RPC 签名。
- 新增15项 commonTest：读取/收集委托、可空参数、三个扩展的滞后本地值与返回值、精确50ms等待及重算、无新发射时重试、等待取消、已取消准入、CAS/transform失败不重试；虚拟时间测试不依赖实际睡眠。
- 四目标库及测试源码（JVM、Linux x64/ARM64、Windows x64）编译通过；JVM/Linux x64 Native 各43项实际通过，含15项新增及28项原公共适配测试，零失败/跳过。未执行其他平台测试。
- 新引入 contract 的既有传递依赖使本模块 macOS 库/测试编译跳过，不能沿用上一批五目标结果；未切换设备或扩大模块拆分。构建1分7秒成功，既有 Mosaic 配置缓存问题3550项/167类导致缓存丢弃，不扩展修复。
- 本组没有初始化订阅、失活恢复、timeline 缓存或字段冲突策略，未接入真实 CAS/业务状态或 CLI。新增测试验证本地委托与循环，既有28项回归验证异常/Flow往返，不宣称已运行完整业务状态同步。

FIXME:

#### 已落地：Get/Flow 的本地状态初始化

- 用户批准薄工厂与测试；`rpcStateIn` 仅组合一次 Get 与原库 Eagerly stateIn，规则收录 [Get 与 GetFlow](../../checklist/rpc-state.md#get-与-getflow)。实现见 `Kodex/rpc/client/src/commonMain/kotlin/io/github/stream29/kodex/rpc/client/RpcStateIn.kt:9-26`，没有新增依赖或独立状态容器。
- 锁定 coroutines 1.11.0 的 [stateIn 实现](https://github.com/Kotlin/kotlinx.coroutines/blob/1.11.0/kotlinx-coroutines-core/common/src/flow/operators/Share.kt)承担共享与作用域生命周期；现行 Application 也使用 Eagerly stateIn（`Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt:270-291`）。工厂不包装已由 RestoringRpcClient 保护的流。
- 新增12项测试：Get/Flow 顺序、首值纠正、可空值、三类初始化失败、立即取流失败、初始化取消、多个观察者共享及无观察者同步、owner 取消、上游失败归属、正常完成不重启；其中两项使用既有真实内存 RPC 验证订阅释放与已知异常进入 owner，连接仍可用。
- 四目标库及测试源码编译通过，JVM/Linux x64 Native 各55项实际通过（原43项加12项），零失败/跳过，并各强制重跑一次通过。macOS 仍因 contract 的传递依赖限制跳过；其他两平台只编译，未执行测试。
- 显式复用 IDEA、空闲 Temurin 25 Daemon 208224，离线构建43秒成功；既有 Mosaic 配置缓存问题仍导致缓存丢弃，未扩展修复。
- 未接入实际全局/Session 业务状态；失活重建、缓存与业务 owner 仍未实现。正常完成保留末值、上游失败不直接传给 UI StateFlow 订阅者均按原库行为验证，不把保留末值当作恢复成功。

FIXME:

#### 已落地：后端 timeline 最小元数据

- 用户确认缓存层归属并批准最小实施；原 internal CachedIndexVersioned（后更名 CachedIndexVersionedImpl）新增只读 cacheNonce/latestIndex，构造、追加与实际回退按[缓存标记规则](../../checklist/rpc-timeline.md#缓存标记与前端缓存)发布。
- 该批生产修改仅原缓存文件21行增量，保留普通 IndexVersioned、原 latestIndex() 及六种查询，当时未改公开类型或构建依赖；后续只读视图见下节。字段与提交位置见 `Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt:127-134,255-302`。
- 原 set/revert 都先完成 delegate，再在 NonCancellable 中更新缓存；本组只在该完成路径发布，不移动原锁或改文件提交/补偿算法，不宣称未知失败绝无磁盘副作用。
- 新增8项测试：空/稀疏初值及当前值订阅、追加发射、破坏性回退及清空、无变化/非法回退、重建、注入 backing 追加/回退失败；原 TTL/容量淘汰测试补充元数据保持断言，原 owner 取消测试保留。
- 首轮 JVM 两项新增订阅测试误用虚拟时间超时覆盖真实文件 I/O；已沿原测试的 Dispatchers.Default 有限并行做法改为真实时间等待，未为测试修改生产行为。
- 四目标库与测试源码编译通过；最终 JVM/Linux x64 Native 各33项通过（13项缓存、20项原 repository），零失败/跳过，并各强制重跑一次通过。macOS 因既有传递依赖跳过，未运行 Windows/ARM64 或 JS 测试。
- 复用项目 IDEA；开始时无存活 Gradle Daemon，显式沿用原 Temurin 25、离线构建。最终33秒成功并复用配置缓存，没有修改全局环境或切换设备。
- 测试使用临时目录和原模拟客户端，不访问用户真实 Home/凭据或执行模型请求；fixture 负责取消资源与删除临时目录。未接入 RPC 标记校验、SessionNotActive 流结束或前端迟到结果拒绝，两个 StateFlow 不等于已满足这些保证。

FIXME:

#### 已落地：服务层读取缓存元数据的只读接口

- 原类型链将元数据隐藏在模块内：FileSystemKodexAgentSession 是 internal，storage 声明为 MutableKodexAgentStorage（`Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexAgentSession.kt:21-32`）；repository 的 openRoots/open 使用 KodexAgentSession（`Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt:42-58,190-213`）。这些签名未改，缓存与各 timeline 的具体实现仍 internal。
- 用户选择只读接口并指定 `CachedIndexVersioned` / `CachedIndexVersionedImpl` 命名；撤回 ObservableIndexVersioned 名称、公开具体实现及收窄 repository.open 的候选，确定边界见[缓存标记规则](../../checklist/rpc-timeline.md#缓存标记与前端缓存)。
- 新增 `CachedIndexVersioned<T> : IndexVersioned<T>`，只增加两项 StateFlow，复用六种查询及 latestIndex()，没有 set/revert；见 `Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/CachedIndexVersioned.kt:5-21`。
- 新增 `ObservableKodexAgentStorage : KodexAgentStorage`，将六条属性协变为该接口；原缓存仍 internal，同时实现可写接口与只读视图，内部字段收窄为同一个 CachedIndexVersionedImpl，无新增数据副本、依赖或 MutableObservable 层级。
- 保留原 Session.storage 和 repository.open 签名；真实 repository 测试通过非泛型 assertIs 取得整个读视图，验证六条 timeline 与原属性为同一对象。生产服务组装尚未接入，不增加泛型强转或兼容 fallback。
- 新增两项测试：单 timeline 的同一 StateFlow、全部查询、追加/回退及取消；真实临时 Session 的六条视图、初始化、时间戳追加/回退及关闭后拒绝访问。原33项缓存/repository 测试一起回归。
- 四目标库与测试源码编译通过；JVM/Linux x64 Native 各35项实际通过（14项缓存、21项 repository），零失败/错误/跳过。macOS 因既有依赖条件跳过，未运行 Windows/ARM64/JS 或真实 CLI/RPC 服务。
- 开始时没有存活项目 IDE 或 Gradle Daemon，未擅自启动 IDE；显式沿用本机 Temurin 25，离线构建启动 Daemon 409719，1分钟成功并复用配置缓存。仅测试临时目录和模拟客户端，没有访问真实 Home 或凭据。
- 原查询、锁、文件提交与补偿算法未变；只读类型并不自动使标记校验与读取原子化，是否需要这种保证已在下一节按最终一致性重新核对。

FIXME:

#### 核对结果：读取竞态与最终一致性

- 用户指出局部未同步未必影响最终一致性，要求核对；此前仅凭“先检查后读取”就提出受保护入口，推论过强，撤回把该入口当作必需前置的候选，不新增锁或修改业务行为。
- 对前端投影，按[既定缓存失效规则](../../checklist/rpc-timeline.md#缓存标记与前端缓存)：旧请求先返回则暂存结果在新 nonce 到达时清掉；新 nonce 先到达则拒绝旧请求回填。条件是通知正常收集、失效/重建及绑定检查落实；实际 RPC timeline 适配尚未实现，不能把逻辑推演称为已验证的前端运行行为。
- 旧 checklist/TimelineRpc 注释中“单次查询不能跨有效期返回”的表述比上述最终一致性更强。调查时未自动修改；随后用户确认后端沿用机制、前端按 nonce invalidate，并明确两侧独立实现，现已同步为只读最终一致性，不据暂态要求所有查询复用 AgentState.modify。
- 原查询加载值在 SafeRw 索引读锁外；revert 先操作 backing，后清值缓存、改索引与 nonce（`Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt:163-195,228-251,282-302`）。这些行为早于 RPC 元数据改动，本组未修改。
- 核对固定 cache4k 0.14.0 的本地 sources 与[官方 RealCache 源码](https://github.com/ReactiveCircus/cache4k/blob/0.14.0/cache4k/src/commonMain/kotlin/io/github/reactivecircus/cache4k/RealCache.kt)：loader 完成后先查当前缓存，已有值时采用已有值，否则放入刚加载的值；invalidateAll 只清条目与队列，不取消在途 loader。
- 用原 CachedAgentStorageTest fixture 添加并运行三个临时探针：在文件旧值已经读取、loader 尚未返回的位置设置 barrier；回退后分别不重写、重写原 index、重写后让新值被容量淘汰，再释放旧 loader。容量设为1以确定性制造淘汰，生产默认仍为1024；没有将测试时序或发生频率当作用户现场事实。
- JVM/Linux x64 Native 各38项通过预期断言，含原35项和3个探针。仅移除时旧调用可能返回旧值，但新 getExact 返回 null；重写且新值仍缓存时旧 loader 采用新值；新值已淘汰时旧 loader 把旧值回填，磁盘为 replacement、新 nonce 未变，但后续 getExact(0) 返回 first。
- 第三个探针另用原 FakeTimeSource 每次推进59秒并读取，连续6次仍返回旧值，证实 expireAfterAccess 并非“60秒后一定恢复”；没有测试自然空闲淘汰、真实默认容量的发生概率、真实模型调用或持久化副作用。
- 这是原后端缓存的独立迟到加载问题，不只是前端暂态，前端新 nonce 不能自动纠正它；未复现磁盘损坏或用户业务事故，也不宣称所有竞态都无害。按用户“不顺带修 bug”的要求记录事实，不据此扩大 RPC 重构或批准修复。
- 探针和测试专用 barrier 已移除，保留原获批的35项测试；清理后 Native 35项重跑通过，JVM 恢复同源码的构建缓存结果，不另称重跑。未新增生产 API、异常、锁或依赖。历史命令的写入准入与前端拒绝旧结果仍独立保留，不把只读最终一致性的讨论扩展为放宽写入校验。
- 用户随后确认 CachedIndexVersioned 的两侧实现应分开，确定规则见[两侧缓存分工](../../checklist/rpc-timeline.md#缓存标记与前端缓存)。同步共享接口的 KDoc，明确 nonce 由后端产生、前端仅观察，不要求公共实现或新的可写接口。
- 该轮只调整设计和两处 KDoc，不新增前端 Impl、同步入口、锁或业务服务；非注释代码保持不变，不将前一轮35项测试称为本轮重跑。

FIXME:

#### 已落地：单 timeline 的前端缓存实现

- 用户批准最小前端实现与测试；已添加 rpcCachedIndexVersioned 工厂和内部 RpcCachedIndexVersioned，返回原接口，不新增 RPC、DTO 或可写接口。接入规则见[前端单 timeline 实现](../../checklist/rpc-timeline.md#前端单-timeline-实现)。
- 工厂为一次绑定建立子 Job，复用 rpcStateIn，异常/取消清理不关闭共享客户端；实现见 `Kodex/rpc/client/src/commonMain/kotlin/io/github/stream29/kodex/rpc/client/RpcCachedIndexVersioned.kt:31-92`。普通查询属于调用者，同时挂接绑定结束取消；不存在自动激活、保活或恢复循环。
- 同步 cache4k get/put 与短临界区协调失效及回填，远程查询不持锁，也不用库内异步 loader 自动回填；原后端实现、锁及已记录的迟到 loader 问题均未修改。公开参数不暴露容量/时钟，测试通过 internal 工厂控制它们。
- rpc/client 仅新增 agent-storage-contract 与既有 cache4k 依赖；TimelineRpc 只更新“尚无客户端实现”的旧注释，所有方法和序列化模型不变。
- 新增29项测试：元数据共享与不轮询、六种稀疏查询、exact/null/范围回填、未来查询追加、同尾索引换 nonce、六类迟到结果拒绝、失配不重试、容量/访问过期、初始化失败/取消、查询错误隔离、SessionNotActive、owner 取消、上游失败/取消/结束。
- 其中两项使用现有内存连接及测试专用 TimelineProbe，实际经过 kRPC 与 RestoringRpcClient，验证迟到值拒绝、两条远端订阅释放、失活异常到达 owner，以及共享连接仍可调用。它们不等于实际业务服务或 CLI 已接入。
- 复用空闲 Temurin 25 Daemon 409719；未发现运行中的项目 IDE，未另启 IDE 或切换设备。四目标库/测试源码编译通过；JVM/Linux x64 Native 各84项实际通过（原55项加29项），零失败/错误/跳过，并各强制重跑一次通过。
- macOS ARM64 沿既有传递依赖/cinterop 条件跳过；未执行 Linux ARM64/Windows 测试或真实界面。平台构建26秒、483任务中54执行，重跑8秒；既有 Mosaic 配置缓存问题导致缓存丢弃，未扩大修复。
- HistoryIndex 的原 getExact/indexesIn 消费者未改（`Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexViewModel.kt:107-112,140-167`）；六条组合视图、Session 恢复、后端查询服务和实际界面接线仍待后续授权。

FIXME:

#### 已落地：幂等 open 与目录活跃标记

- 用户指出 repository.open 已有幂等性，应由目录携带 isActive；撤回 getActive API 与其实施候选，也不先创建 rpc/server 或另一份活跃登记。
- 原接口明确重复打开同一活跃条目返回同一 KodexAgentSession（`Kodex/agent-session/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/contract/KodexSession.kt:64-69`）；文件与内存测试已有同实例断言（`Kodex/agent-session/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepositoryTest.kt:420-428`；`Kodex/agent-session/in-memory/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/inmemory/InMemoryKodexSessionRepositoryTest.kt:201-209`）。
- 两实现均复用 openRoots 中 Job 活跃的对象，否则重新打开（`Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt:49-57,190-213`；`Kodex/agent-session/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/inmemory/InMemoryKodexSessionRepository.kt:37-45,123-137`）。无需为重复打开实现第二套 owner 管理。
- 原目录源码只有 running；用户随后批准同时补齐两个标记。现保留 running，新增默认 false 的 isActive，均是快照；[目录规则](../../checklist/rpc-session.md#目录与运行标记)明确空闲 owner 的 active/running 区别及单用户、小规模全量读取假设，未来扩大规模必须改为分页，本期不预设分页参数或结果模型。
- KodexRootSessionEntry 增加两项只读布尔值；文件/内存 repository 从各自原 openRoots 中采样 Job 活跃及 runningTurn 非空，listEntries/getEntry 均提供同一语义。没有为读取目录调用 open、续期、增加登记表或改存储算法。
- 原 SessionCatalogViewModel 映射透传两字段，不再把 running 留为默认值；它观察的是所用 repository，不跨独立实例推测其他 owner。实际 RPC 后端仍须复用其共享 repository，不把本地映射测试称为 CLI/RPC 宿主已接线。
- 幂等 open 只保证活跃时复用，未活跃时仍会加载；目录快照也不是持久访问许可。本次不据此撤销已定的显式保活、普通访问不隐式激活及 SessionNotActive 规则；具体服务准入接线未实现，不把目录字段当作已经解决了该接线。
- 新增五项测试：目录默认值与三种状态值往返两项、两种 repository 生命周期各一项、ViewModel 快照映射一项；RPC 原值测试同步七字段。覆盖未打开读取不激活、空闲、运行、归档独立、旧快照不变、关闭与重开，模型响应均使用模拟客户端。
- 五模块 JVM/Linux x64 Native 各125项实际测试通过：目录6、文件 repository/缓存36、内存 repository12、Session ViewModel14、RPC contract57；零失败/错误/跳过。值序列化、真实临时文件生命周期与本地 ViewModel 映射均运行，不仅是编译验证。
- 五模块 JVM、Linux x64/ARM64、Windows x64 库与测试源码编译通过；独立目录模块另通过 macOS ARM64 编译，其余四模块因既有传递 cinterop 条件跳过该目标。未在 Windows/ARM64 执行测试，未做实际 CLI 菜单/远程目录运行验收。
- 开始时没有存活项目 IDE 或 Gradle Daemon；显式沿用 Temurin 25 启动 Daemon 622303，Native 阶段复用同一 Daemon。JVM 构建40秒成功；Native/跨平台构建1分16秒成功，既有 Mosaic 配置缓存问题导致该阶段缓存丢弃，未扩大修复。
- 该目录批次未改原 open/delete、运行槽位或生命周期；统一槽位随后单独获批并完成，见上方记录。TTL、实际业务 RPC、目录 spinner 或分页未实现，不把目录标记测试当作已解决失活准入。

FIXME:

#### 接入阻塞：后端 repository 实体与前端 RPC 视图

- 用户要求继续审查；本项没有新增业务协议选择，阻塞在已定[后端资源所有权](../../checklist/rpc-session.md#创建与前端关闭)尚未取代旧组合方式，不重新审批后端归属或增加 getActive。
- 用户进一步明确真实 repository 及其协程作用域必须完整保留在后端，前端只是 RPC 视图；确定规则收录[实体与视图边界](../../checklist/rpc-session.md#repository-实体与前端视图)。此前“共享 repository”仅指后端各服务共享，不指把该对象借给前端。
- 原组合根的 KodexSessionRepositoryFactory 每次按调用方 ownerScope 创建 repository（`Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt:339-341`）；持久化 tab 的 createOpened 又新建子 scope 并使用该工厂（`Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt:190-223`），目录另有自己的 repository（同文件 :661-662）。
- open 的同实例保证只覆盖同一 repository；不同实例各有 openRoots。上轮目录字段及映射测试证明同实例观察正确，不证明现行 CLI 目录能看见其他 tab 的 owner；不能为补足该信息把文件 lease 当作 running 或枚举其他前端对象。
- 原 registry.release 调用 ViewModel.shutdown，后者取消 ownerScope（同文件 :181-188,451-467）；旧测试明确断言释放 tab 后 repository 和文件 lease 都关闭（`Kodex/app/viewmodel/session/src/commonTest/kotlin/io/github/stream29/kodex/cli/session/FileSystemSessionViewModelOwnershipTest.kt:71-110`）。这是原行为，不是本轮发现并顺带修复的 bug；迁移时应按已定前后端生命周期替换这项预期。
- 不可只把工厂改成返回共享实例：目录 delete/reloadRepository 会直接 cancelAndJoin 其 repository（`Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt:571-600`）；临时 withRepository 则创建并取消自己的 owner（同文件 :226-234）。这些“取得即拥有”的路径须替换为 RPC 视图操作，前端也不应直接借用后端实例。
- 原 KodexSessionRepository 继承 CoroutineScope，open 返回 KodexAgentSession，后者包含后端 runtime 与 MutableKodexAgentStorage（`Kodex/agent-session/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/contract/KodexSession.kt:21-31,71-112`）。这不是可直接跨线返回的视图；不因前端需要目录或状态便暴露全部实体能力。
- 原 AgentRuntimeViewModel 直接接收该 session，读取 storage 并调用 runtime（`Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt:81-102,659-670`）。后续迁移须把这些构造依赖换成本地 RPC 视图与前端 scope；现有共享只读接口可按语义复用，但本轮不预先增加一套完整 repository 代理接口或远端句柄。
- 统一运行槽位完成后继续复核：runtime 仍登记 currentCoroutineContext().job，直接在 RPC handler 中调用 resume/forcedCompact 会让请求取消中断运行，不满足已定后端所有权。原前端 launchOwnedOperation/runInOwnerScope 已区分自身 owner 与等待者（同文件 :558-589）；迁移需要把接受后的执行归属移到后端，而不是只改调用名称或再增加一套运行状态。这里是已定规则的接线前置，不是新的并发政策选择。
- 必须成组迁移的范围：后端 scope 拥有同一 repository，目录与 Session 业务访问复用它；前端 tab/目录只拥有 RPC 投影、草稿和保活循环，释放不取消后端；后端 Session 按既定 TTL、显式删除与宿主退出清理。不能先让旧 CLI 失去释放责任，却尚无已批准的保活/回收接入。
- 后续验收应同时覆盖：打开后的目录标记、关闭前端观察不停止已接受运行、目录关闭/删除一项不关闭其他 Session、TTL 和显式删除释放目标 owner、宿主关闭释放全部资源。复用已定 60/20 秒和 runningTurn 自保活规则，不再提出保留队列、引用计数或额外 lease。
- 本轮只读取组合根、调用点和旧测试，没有修改运行源码、构造真实 repository 或重跑测试。完整服务/前端接入仍需收敛实施范围，不创建半实现的 GlobalRpc、平行 owner 管理器或假成功占位。

FIXME:

#### 已定：一期工作目录浏览仍在本地

- 本轮继续核对前端入口；消息提交、工具回答与子任务确认可使用已审契约，未因此提出新的批量事务或自动恢复。工作目录选择器的读取归属已确认，不重开 repository 所有权。
- Agent 设置与子任务确认分别把当前 cwd 交给选择器（`Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt:290-330`）；Session 设置入口也从配置取得 workingDirectory（`Kodex/app/viewmodel/settings/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/SessionSettingsViewModel.kt:82-105`）。这些目录最终用于后端 Agent，不是前端 Hook 的本地执行目录。
- 原 DirectoryPickerBrowser 使用本机 CoroutineFileSystem 和 userHome，展开 ~、解析路径、检查目录并列出直接子目录（`Kodex/app/viewmodel/path-picker/src/commonMain/kotlin/io/github/stream29/kodex/app/pathpicker/DirectoryPickerBrowser.kt:27-81`）；前端工厂直接创建该浏览器（`Kodex/app/viewmodel/path-picker/src/commonMain/kotlin/io/github/stream29/kodex/app/pathpicker/DirectoryPickerViewModel.kt:154-163`），现有 RPC 尚无这段读取的映射。
- 已定[后端设置归属](../../checklist/rpc-settings.md#文件与字段归属)要求路径按后端环境解释；一期同进程下本地读取仍能工作，本轮没有证明当前故障，也不把跨进程或跨操作系统浏览兼容列为一期新功能。
- 用户选择一期仍由前端本地读取；确定规则收录[工作目录选择](../../checklist/rpc-architecture.md#工作目录选择)。撤回后端目录读取候选，不再审查其签名、载荷或异常，不新增目录监视、文件内容读取、上传或写入 API。
- 现有选择器行为以[路径选择指导](../../checklist/path-picker.md)为依据；该一期本地读取特例不允许前端直接访问后端 Session、设置或凭据文件。独立部署前再调整边界，不将未来适配列为一期阻塞。
- 本轮仅读取源码与设计，没有浏览用户工作目录、改动选择器或新增契约，也未运行测试；原后端业务逻辑不变。

FIXME:

#### 其他接入缺口

- 设置字段归属、同 dataDirectory、宽松加载、旧字段/Hooks 转换及拆分迁移重入已确认；两个独立 store 已实现，实际 CLI 文件切换与 migration 未实施，多个 CLI 的重载协调后移，不重开已定设计。
- notification-only 的事件、配置模型与命令执行规则已确认；旧配置处理、文件加载及执行器接入未完成。多端触发、网络重连补触发/去重后移，不作为单 CLI 一期必做工作。
- OAuth：一期已确认待回调 attempt 的10分钟上限及前端 listener 清理；具体接入与命令交接仍未实现。网络失联重连发现 attempt、独立后端重启后的句柄恢复后移，不增加永久 registry。
- Codex 认证：已确认仅覆盖固定 auth.json，并沿用原回存失败路径，不新增存储兼容或待回存状态；账号变化、迟到结果和认证失败处理的实际接入仍待核对。已选完整生命周期，不再要求用户改选只读/独立登录/broker。
- 全局 MCP CAS 接入尚须执行原 manager 的配置合法性与连接/登录身份协调；新模型不是已完成的写入实现。
- 一期接入：共享 JSON 内存连接模块已落地，两侧实际 scope 与 CLI 接入尚未实施；daemon 启动/发现/避免重复实例、是否按 Home 隔离、独立空闲退出、网络 shutdown 与跨进程版本匹配属于未来工程。
- 宿主：一期工作目录选择继续前端本地读取；本轮未发现需要新增文件内容或图片读取服务的具体入口，不预建通用文件 RPC。OAuth 浏览器与回调前端归属已定，不再列作开放选择。
- 一期需验证 Native CLI 中内存 RPC 的链接/运行、状态流与取消及慢消费者资源边界；不要求网络引擎验证。此前平台表与 WebSocket 研究保留为后续参考，不证明内存接入已实现。
- 用户已确认此前设置、Hooks、TTL 及 reset 调整均可接受并与一期同批交付，范围以 [一期部署与交付](../../checklist/rpc-architecture.md#一期部署与交付)为准；不再将其阶段归属列为未决。具体参数、接入与尚未获批的契约修改继续按各节审查，不据此推进 Executable 或直接实现运行逻辑。

FIXME:

#### 本轮复核：协议覆盖与实施前置

- 本轮继续沿真实入口审查，没有因下列调用点发现新的 RPC 或可恢复异常分支；这是静态核对结论，不宣称所有业务服务可运行或已完成端到端验收。
- 工具回答的 StaleCall/StaleRevision/Busy/Incomplete 是前端草稿准入结果，成功提交使用原 completeToolCall；批量子任务使用已定专属调用，其余失败仍显示普通诊断。依据：`Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/RequestUserInputViewModel.kt:83-146`、`Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskViewModel.kt:48-100`。不为这些本地编辑状态新增远端异常。
- 设置页当前只观察 models/auth/usage/MCP 并投影展示（`Kodex/app/viewmodel/settings/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/GlobalSettingsViewModel.kt:118-169`）。模型初始刷新属于 catalog 自有 scope，usage 随 auth 变化刷新；迁移保留 store 所有者即可，不补前端轮询或模型刷新 RPC。依据：`Kodex/openai/model-catalog/impl/src/commonMain/kotlin/io/github/stream29/kodex/openai/modelcatalog/OpenAiModelCatalogImpl.kt:25-80`、`Kodex/openai/account-usage/impl/src/commonMain/kotlin/io/github/stream29/kodex/openai/accountusage/CodexAccountUsageStoreImpl.kt:32-87`。
- 新通知执行器需要的终止请求、exitCode 和 session Job 已由原 ProcessSession 提供；ShellClient 拥有进程，命令的等待者不是进程 owner。原测试包含 close 后等退出码及 Job 完成，不因本轮审查修改进程底层或重新审批清理规则。依据：`Kodex/utils/shell-client/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ProcessSession.kt:47-94`、`Kodex/utils/shell-client/src/commonTest/kotlin/io/github/stream29/kodex/utils/shellclient/ProcessSessionTest.kt:308-323`。新10秒 Hook 的取消/启动交接仍须在执行器接入时实测，不能用这些源码断言代替验收。
- 清理旧任务标记：currentFlow 结果、一期已知异常、工具交互及一期 OAuth/通知规则已定；网络失联恢复、多端触发和新功能不列为当前阻塞。已有后端缓存问题不顺带修复，MCP 写侧仍须保留原 manager 行为，不能由观察路径已覆盖推导写入已完成。

| 后续实施组 | 必须共同验证的范围 |
| --- | --- |
| 两侧设置存储 | 各自文件、默认值解析、非法输入、原子写入、取消清理；不提前激活旧文件删除 |
| 后端资源与服务 | 同一真实 repository、Session scope、已接受运行与调用等待隔离、60/20秒保活、失活结束上游；不得先取消前端释放责任却没有后端回收 |
| 全局业务适配 | 原 auth/usage/models/MCP store 的观察与命令、完整设置 CAS 校验、副作用及 OAuth 交接；不以半实现 GlobalRpc 或假成功占位 |
| 前端业务视图 | Session/历史缓存、currentFlow、本地草稿与设置投影、关闭 tab 只释放本地资源；不注入真实后端实体 |
| 通知接入 | 后端 Stop 发布、前端单订阅与本地命令，慢消费者和取消下的资源释放 |
| 单 CLI 切换 | 注册完整服务、替换旧工厂及设置读取、激活拆分 migration、退出清理和 Native CLI 验收；不保留两套持久化真源 |

- 以上是已定行为的实施依赖，不代表本轮获准整体改写。没有仅为了继续审查而新增协议、通用管理层或并发保证。

FIXME:

#### 已落实：独立两侧设置 store

- 旧 KodexSettingsStore 在 settings.yml 缺失时返回 defaults（`Kodex/app/shared/settings/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/KodexSettingsStore.kt:42-73`）；Application 在 Home 准备后仍调用它（`Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt:242-265`）。因此拆分 migration 激活必须与 CLI 改读两侧文件协调，不能先删除旧文件。该顺序已收录[目录与加载](../../checklist/rpc-settings.md#目录与加载)，不是当前已经发生的数据丢失。
- 用户批准后，在原 settings/filesystem 模块新增两类及工厂：`Kodex/app/shared/settings/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/BackendSettingsStore.kt:18-80`、`Kodex/app/shared/settings/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/CliFrontendSettingsStore.kt:13-52`。分别持有自己的只读 StateFlow 与更新锁，不创建 owner scope 或通用 store 框架。
- 正常加载只读所属文件，缺失/嵌套稀疏字段应用所属 defaults，不读旧文件或另一侧；私有 file codec 保留 snake_case，显式 null 只用于可空领域字段，未知字段忽略，非法配置拒绝。RPC 模型没有新增字段或默认值，也没有改变 serializer 名称。
- 更新在本 store 锁内读取当前文件、应用 transform、私有临时文件原子替换后发布状态；NonCancellable 清理预算10秒，仅删除本次 temporary，并把清理失败附加到主失败。替换完成后的取消不承诺回滚，测试确认文件可能已更新而内存尚未发布。
- 模块仅新增 api 依赖 rpc/models，复用 KAML/序列化及 filesystem 依赖，无新版本或测试库。原 KodexSettingsStore、旧测试与调用方未改；新依赖不形成 settings/contract → filesystem 的回环。
- 新增35项 commonTest，覆盖两侧默认值、嵌套字段、空集合/可空标题、完整 MCP 假凭据和多类型 Hook 值往返、非法配置保留、另一侧/旧文件隔离、16次并发更新、临时写入失败、取消、suppressed 清理错误和丢失完成后的磁盘结果。所有文件均在隔离临时目录，测试不执行 Hook 命令或连接 MCP。
- JVM 首轮51项中两项新断言失败：跨 withContext 的清理 IOException 可被协程栈恢复复制，原测试错误地比较 suppressed 对象身份。只改为验证单项类型和内容，仍断言主失败是原对象；生产实现未因此修改。
- 修正后 JVM51项、Linux x64 Native50项实际通过，并各强制重跑一轮通过，均含35项新测试；差一项为原 JVM 专属文件权限测试。JVM、Linux x64/ARM64、Windows x64 库与测试源码编译通过；macOS ARM64 受既有 cinterop 依赖条件跳过，未在其他设备执行测试。
- IDE 仅打开 PersonalSkills，复用空闲 Temurin25 Daemon622303；三次构建依次10秒（断言失败）、14秒（跨目标及两端测试成功）、5秒（强制重跑成功），配置缓存保存。没有因新依赖修复其他模块或迁移底层进程实现。
- 未修改 Application、旧 store 调用方、migration registry 或应用版本，没有真实 Home 读写/迁移。新 store 不是 GlobalRpc 完整业务 CAS；MCP 校验/副作用、认证接线及文件切换仍待实现。

FIXME:

#### 已落实：后端文件 CAS 原语

- 用户确认该比较本来就属于后端；不再重复审批归属，也不以此前前端 SuspendMutableStateFlow 的回调包装代替后端原子性。
- BackendSettingsStore.compareAndSet(expect, update) 与 update/reload 共用原锁，读取当前文件完整值后比较，匹配才原子写入并发布。见 `Kodex/app/shared/settings/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/BackendSettingsStore.kt:47-67`；未添加新的锁或依赖。
- mismatch 返回 false 而不写文件；三值相等返回 true，不创建目录/文件、不规范化稀疏 YAML 或删除未知字段。读到已变更的文件会更新本地观察，不发布被拒绝的 proposal；解析、I/O、取消失败原样作为失败，不转成 false。
- 原 MCP 配置写入通过 settings.update 的 transform 完成（`Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/KodexMcpConfigurationStore.kt:21-40`）；后续迁移必须走同一后端实际写入边界。本组只提供文件比较，不改原 manager 行为，也不声称 GlobalRpc 的业务校验与副作用已接入。
- 新增12项测试覆盖值相等与不同实例、缺文件/同值零写入、过期比较观察实际文件、全量提交、8个同期待 CAS 仅一个成功、CAS/update 双向交错、reload 等待同锁以及解析/写入/取消失败。复用原批隔离目录和故障 fixture，未运行真实业务。
- 四目标库与测试源码编译通过，JVM63项、Linux x64 Native62项实际测试通过，并各强制重跑一轮通过；均包含35项 store 测试与12项 CAS 测试，macOS ARM64 仍因既有 cinterop 条件跳过。继续复用 Daemon622303，构建4秒、重跑1秒，配置缓存复用；临时日志清理。本轮未新增前端 CAS、RPC 方法、服务、迁移或版本修改。
- 剩余接入仍按上方实施表：将实际后台写入、manager 校验/副作用与完整服务放到后端，前端只观察和发命令；不再把“CAS 应在哪侧”列为设计阻塞，也不把文件 CAS 成功等同于业务迁移完成。

FIXME:

#### 已核对：MCP 设置页观察原后端状态

- 此段记录此前读侧审查，当时用户要求先关注观察和展示，未批准导入行为调整；后续已明确新路径只有配置值更新，最新要求以[导入提交语义](../../checklist/rpc-mcp.md#导入提交语义)为准，不再把旧 Replace 副作用列为迁移前置。
- 原 applyCodexImport 的 Replace 另行触发 invalidate，源码与原断言保持不变（`Kodex/mcp/impl/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt:375-419`；`Kodex/mcp/impl/src/commonTest/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImplTest.kt:167-233`）。完整 CAS 与业务命令的写侧适配仍未实现，不把本轮状态观察核对称为已完成该适配。
- 原设置 ViewModel 直接 collect mcpManager.servers，再做 McpManagedServerState.toSettingsState 纯映射（`Kodex/app/viewmodel/settings/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/GlobalSettingsViewModel.kt:141-144,670-721`）；该映射提供原 Disabled/认证阻塞/Connecting/Healthy/Failed/Closed 展示，不需要跨 RPC 传输新的 UI DTO。
- 原列表只展示紧凑名称/状态按钮，详情呈现同一值的认证、连接、工具数及脱敏配置字段（`Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/McpSettingsContent.kt:19-82`；`Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/McpSettingsDialogs.kt:243-354`）。详情以 serverName 从最新 StateFlow 列表查找，不固定为打开瞬间的旧对象（`Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt:241-263`）。
- 现有 getMcpServers/getMcpServersFlow 已返回同一原后端 List<McpManagedServerState>，含禁用项（`Kodex/rpc/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/GlobalRpc.kt:195-204`）。可复用已落地 rpcStateIn 初始化与共享观察，前端继续原纯映射，无需新增方法、模型或逐服务器订阅。
- 确定分工见[前端观察与展示](../../checklist/rpc-mcp.md#前端观察与展示)：运行显示跟随后端流，不从 CAS/reconnect 回执推断 Healthy；完整 BackendSettings 仍是编辑来源，与运行状态允许异步到达。
- 原测试已覆盖从 Failed 收到 Healthy/工具数3的更新，以及页面发起 reconnect 的转交（`Kodex/app/viewmodel/settings/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/SettingsViewModelTest.kt:574-624`）。本轮只读源码与测试，没有重跑或实现 RPC UI 接线。
- 本组观察能力没有新增协议阻塞，剩余为按既定规则接线与验证；没有读取真实设置/凭据、运行 MCP 连接、修改生产代码，原后端副作用保持不变。

FIXME:

#### 已定：移除认证页面的手动 Reload

- 继续按“不改后端业务行为、核对前端实际需求”审查；本项不是新增凭据维护能力，而是现有按钮尚无跨线映射。
- OpenAI 设置页在 Codex 与 Kodex 两种来源都提供 Reload（`Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt:570-627`）；原 ViewModel 把操作直接交给 authenticationStore.reload，并保留前端操作中/失败状态（`Kodex/app/viewmodel/settings/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/GlobalSettingsViewModel.kt:298-304`）。
- 原 reload 在 updateMutex 内读取当前配置选中的来源并发布结果，不是只返回内存摘要；加载失败按原规则发布 Unavailable，取消继续抛出（`Kodex/app/shared/auth/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/FileSystemKodexAuthStore.kt:129-144,248-297`）。它不等同于强制供应商 token 续期、重新登录或刷新 usage。
- 现有 getAuthentication/getAuthenticationFlow 只承载认证摘要（`Kodex/rpc/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/GlobalRpc.kt:82-90`）；不能为承接按钮而让 Get 隐式重读文件、重建订阅或由前端读取后端凭据。
- 用户选择移除手动入口；确定规则收录[认证来源约定](../../checklist/rpc-authentication.md#codex-与-kodex-来源)。撤回 reloadAuthentication RPC 候选，后台加载/续期仍由后端独立负责。
- 实际页面接入时删除两种来源的 Reload 按钮及其专用前端处理，不删除后端 store.reload，不改其 Unavailable 发布、后台维护、来源选择或保存失败策略；登录/退出等其他操作的前端等待状态不随之裁剪。
- 原测试分别覆盖外部文件更新后的 reload 和页面命令转交（`Kodex/app/shared/auth/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/cli/auth/FileSystemKodexAuthStoreTest.kt:152-172`；`Kodex/app/viewmodel/settings/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/SettingsViewModelTest.kt:308-346`）。本轮仅读源码与测试，未运行测试、读取真实凭据或新增契约。
- 本轮只固化用户明确选择的前端能力裁剪；尚未修改页面、ViewModel 或后端代码，没有新增 RPC 或运行测试。

FIXME:

#### 已定：Codex 认证仍只兼容文件

- 用户明确此前只支持文件，本期非必要不增功能；文件管理范围收录[Codex 与 Kodex 来源](../../checklist/rpc-authentication.md#codex-与-kodex-来源)。不撤回已批准的加载、续期、回存、登录与移除。
- 现行 CodexCliStorage 只读取注入目录的 auth.json；文件缺失返回 null，没有 keyring 或存储模式选择（`Kodex/openai/codex-cli-storage/src/commonMain/kotlin/io/github/stream29/kodex/openai/codexclistorage/CodexCliStorage.kt:10-24`）。认证 store 直接使用该读取结果，缺失映射 CredentialsNotFound（`Kodex/app/shared/auth/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/FileSystemKodexAuthStore.kt:248-260`）。
- 组合根固定 ~/.codex，既有指导不读取 CODEX_HOME，常规认证也不读取 config.toml；见[Codex 文件边界](../../checklist/codex-cli-storage.md)。这些读取边界不变。
- 官方文档区分 file、keyring、auto、ephemeral：auto 优先系统凭据库，不可用时回退文件；ephemeral 只在对应进程内存存活。因此文件不存在不代表官方 Codex 一定未登录，删除文件也不等于删除系统凭据库中的登录。[官方存储说明](https://developers.openai.com/codex/auth/#credential-storage)、[配置参考](https://developers.openai.com/codex/config-reference/)。
- 撤回一期接入 keyring/auto 的备选，不将其平台适配或配置读取列为前置条件；没有新增 RPC、来源枚举、依赖或 provider。
- 该轮只查仓库源码和官方公开文档并同步设计，没有读取真实 auth.json/config.toml/keyring，没有新增存储实现、调用登录或重跑构建；SessionNotFound 的 RPC 复验随后在公共客户端批次完成。

FIXME:

#### 已定：刷新成功但回存失败

- 用户确认沿用原失败路径，规则见[续期回存失败](../../checklist/rpc-authentication.md#续期回存失败)，不增加待回存状态。原私有来源先请求刷新，构造新值，再 writeAuthFile，成功返回后才 publish；写入异常不会执行这次 publish（`Kodex/app/shared/auth/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/FileSystemKodexAuthStore.kt:215-246`）。
- 后台维护捕获非取消异常、记录日志并延迟重试；下次刷新前重新读取文件（同文件 :198-225）。原实现没有专门保留新凭据以重试落盘的状态，也没有在这条异常路径清空旧凭据。
- 现有测试只验证成功刷新后的发布和文件内容（`Kodex/app/shared/auth/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/cli/auth/FileSystemKodexAuthStoreTest.kt:323-367`），没有覆盖供应商已返回新 token、写入却失败的场景。本轮仅读源码，未执行故障注入。
- 用户接受这次返回的新 token 不专门留待再次保存，撤回额外保留/重试保存的备选；不宣称刷新令牌轮换会立即使旧 access token 失效。
- 仅同步设计，没有修改 store、进行真实刷新或故障注入；OAuth complete 成功必须已提交凭据的规则不变。

FIXME:

#### 已定：OAuth 未交付句柄的清理

- 用户已确认待回调 attempt 有限存活，规则收录[待回调登录的过期](../../checklist/rpc-authentication.md#待回调登录的过期)；具体时长和前端等待清理也已在下节确认。
- 已定接口由 startOAuthLogin 返回后端 attemptId，cancelOAuthLogin 要求 exact attemptId；尚未收到回调时没有独立 token 交换可完成。start 回执丢失不证明未创建，见 `Kodex/rpc/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/GlobalRpc.kt:128-181`。
- 一期同进程也需处理前端在 start 返回值交付期间取消：后端可能已准备并保留 attempt，前端却未取得 ID。清理仍在执行的 start handler 与清理已完成、但结果未被前端取得的 attempt 不是同一件事；仅在持有 ID 时调用 cancel 不能覆盖后者。
- 原前端拿到对象后检查 closed/pendingAttemptId，并直接 attempt.cancel（`Kodex/app/viewmodel/settings/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/OpenAiLoginViewModel.kt:38-62`）；RPC 后不能在前端补取这个内存对象。原 MCP 保留每服务器单一登录限制，finally 才移除（`Kodex/mcp/impl/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt:171-174,252-258`）。
- 当前统一 RPC 尚无运行实现；这是按契约检查出的交付/清理缺口，不是已复现的生产登录泄漏。本轮未运行 OAuth、浏览器或异常交付实验。
- 用户接受浏览器等待过久后重新登录的代价，撤回不自动过期的备选；没有新增源码、配置、运行超时或 RPC 方法，不将设计确认当作清理已经可用。

FIXME:

#### 已定：OAuth 等待期限与前端清理

- 后端过期不会自动通知前端持有的 loopback listener；现有认证摘要也不是某次 attempt 的完成回执。前端必须结束自己的等待，不能在后端已过期后无限等浏览器回调。
- 用户已确认一期固定10分钟及前端清理规则，见[待回调登录的过期](../../checklist/rpc-authentication.md#待回调登录的过期)；撤回待定参数表述，任务不重复维护完整规则。
- 本轮只更新设计，未实现 listener、计时或取消调用，也未重跑 OAuth/编译测试；实际接入须验证过期与回调接受竞争、过期名额释放、前端超时清理及已接受交换不受该期限影响。

FIXME:

#### 已定：两侧设置文件的目录与加载

- 用户已确认同 dataDirectory、两侧独立读写、宽松加载与既有迁移入口；规则收录[目录与加载](../../checklist/rpc-settings.md#目录与加载)。不因 RPC 快照全量必填而要求磁盘配置写满字段。
- 事实依据：Application.openGlobalSettings 将 dataDirectory 作为设置目录（`Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt:255-261`）；旧 store 用私有可空文件值叠加 defaults，写入同目录临时文件再替换（`Kodex/app/shared/settings/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/KodexSettingsStore.kt:48-73,82-93,126-162,316-326`）。
- Home 已有版本表和迁移独占入口，正常打开前先执行迁移（`Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigration.kt:55-152`）；具体数据转换须遵循 [Home migration](../../checklist/kodex-home.md#migration)，不因“两个文件”自行增加 backup、journal 或跨文件事务。
- 本组只固化目录与正常加载规则；旧 hooks 的输入协议、已裁剪字段和部分转换失败继续单独审查。未读取真实设置、运行迁移或决定目标 release；新模型的 JSON 测试不等于 YAML 文件加载已实现。

FIXME:

#### 已定：旧字段拆分与源文件删除

- 普通字段按已定归属转换：auth_source、shell、context_sources、new_session、session_title、mcp_servers 进入后端文件；new_line_key 和 sidebars.left/right 进入前端文件。字段命名/嵌套编码由对应冻结迁移规则处理，不直接把旧 YAML 当作新模型 JSON 解码。
- sidebars.left_width/right_width 已决定不再持久化；不把其数值搬到后端或新增隐藏兼容字段。旧文件字段依据为 `Kodex/app/shared/settings/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/KodexSettingsStore.kt:126-155,226-247`。
- 旧 HookConfiguration 是 name→单 type/command 的 Map，不能仅改成列表就保证脚本兼容。Stop 的包装载荷见 `Kodex/hook/impl/src/commonMain/kotlin/io/github/stream29/kodex/hook/impl/projection/HookProjection.kt:15-43`；旧错误输入是 message.orEmpty，见 `Kodex/hook/impl/src/commonMain/kotlin/io/github/stream29/kodex/hook/impl/KodexHooksImpl.kt:163-171`。旧 Stop 还解析控制结果并使用 Agent cwd（同文件 :89-106）；新协议已明确取消这些语义。
- 用户选择迁移后删除：普通字段拆分，新前端 hooks 为空，两份目标确认完成后删除旧 settings.yml，不保留副本；确定规则见[旧设置拆分](../../checklist/rpc-settings.md#旧设置拆分)。撤回保留旧源文件供查阅的建议，未转换的旧命令不再保留。
- 本组只确定迁移设计，未执行真实文件删除或迁移；两目标已存在、部分转换中断、旧文件缺失/损坏的处理已在下节确认，不以文件存在代替目标有效性校验。

FIXME:

#### 已定：两文件迁移的重入状态

- 已沿上轮审查收录[拆分迁移重入](../../checklist/rpc-settings.md#拆分迁移重入)，不再逐轮确认非阻塞收尾；实际源文件删除与版本推进仍未实现。
- 原 coordinator 先等待 migration.action 正常返回，再写版本（`Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigration.kt:143-148`）；现行 SOP 已要求冲突/歧义拒绝，不需要另建备份、journal 或跨文件事务。
- 后续验证需覆盖每个写入/删除边界中断、已存在目标冲突、缺一侧歧义及版本尚未提交时重跑；本轮未运行中断测试，不把文档状态表当作迁移已可执行。

FIXME:

#### 已验证：测试专用内存 RPC 往返

- 此前只有 helper 的 Throwable(source.message) 模拟载荷及契约值测试；本批已增加真实 KrpcClient/KrpcServer 的测试专用往返，不改生产依赖或业务契约。
- 固定 0.10.3 的公开 KrpcTransport 只要求 CoroutineScope、send 和 receive，交付已编码的 StringMessage/BinaryMessage；可用成对有界 Channel 构造测试内存通道，不需要 Ktor、端口、daemon 或框架 fork。[接口](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/commonMain/kotlin/kotlinx/rpc/krpc/KrpcTransport.kt)、[transport 文档](https://kotlin.github.io/kotlinx-rpc/transport.html)。
- InitializedKrpcClient 与 KrpcServer 接受该 transport 及公开配置；JSON 是支持的测试格式。采用它测试不提前决定生产 transport 容量或新增网络重连职责。[客户端](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-client/src/commonMain/kotlin/kotlinx/rpc/krpc/client/KrpcClient.kt#L49-L87)、[服务端](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-server/src/commonMain/kotlin/kotlinx/rpc/krpc/server/KrpcServer.kt#L20-L47)、[配置](https://kotlin.github.io/kotlinx-rpc/configuration.html)。
- `Kodex/rpc/contract/build.gradle.kts:21-26` 的 commonTest 新增同版本 client/server/serialization-json 与 utils-rpc-exception；main 未新增依赖。fixture 位于 `Kodex/rpc/contract/src/commonTest/kotlin/io/github/stream29/kodex/rpc/contract/InMemoryRpcFixture.kt:35-164`，生成的 Probe stub 只在 test classes。
- 通道只传 JSON 编码协议消息；双方 transport 各有 operation 下的 owner，退出同时取消双方并在有限 NonCancellable 中等待清理。使用 Deferred 同步点而非 sleep 推断首值或服务端 finally；测试容量/超时不作为生产默认值。
- `Kodex/rpc/contract/src/commonTest/kotlin/io/github/stream29/kodex/rpc/contract/InMemoryRpcBoundaryTest.kt:40-212` 新增 18 项测试：正常调用/流、三个已知异常的 unary/绑定拒绝/已收到首值后失败、未知和非法 JSON、远端取消、调用/订阅主动取消隔离、前端下游异常和 fixture 退出清理。
- 发现直接 collect 的普通下游异常不会触发服务端取消，见下一节；测试保留该限制的特征断言，另验证 buffer(0) 的保护效果，不以测试全绿声称框架已修复。
- 验证：复用 IDEA 与 Temurin 25 Daemon 2385167；JVM 和本机 Linux x64 Native 各 54 项测试实际通过（36 项原值测试 + 18 项新测试），零失败/跳过，随后再次重跑通过。Native 包括测试编译、可执行文件链接及执行，不只是 KLIB。
- Linux ARM64/Windows x64 的测试源码编译通过，未链接/运行；macOS ARM64 因既有 transitive cinterop 条件跳过。构建仍报告 Mosaic 配置缓存问题并丢弃条目，未按日志建议停止共享 Daemon 或改其构建。
- 首轮误用 Throwable 对象身份断言，改为断言未触发恢复、类型与 message 保持后，暴露实际订阅残留；追加测试时漏写的括号也已修复。JVM 协程栈恢复可复制普通 Throwable，[固定版本源码](https://github.com/Kotlin/kotlinx.coroutines/blob/1.11.0/kotlinx-coroutines-core/jvm/src/internal/ExceptionsConstructor.kt#L50-L75)支持这一边界；不承诺跨挂起点的对象身份。
- 该轮未读取真实设置/凭据、创建 Session 或执行模型/shell/OAuth，也未实现生产 transport、Session TTL 或状态投影；后来获批的独立连接实现见[内存连接模块](#已落地独立内存连接模块)。测试专用服务不证明 CLI 组合根及全部业务设计已完成；临时构建日志已清理，无提交。

FIXME:

#### 已定：普通下游异常的订阅释放

- 用户已确认公共客户端的零容量 Flow 保护，规则收录[订阅释放与重连](../../checklist/rpc-state.md#订阅释放与重连)；这里保留实测依据与覆盖限制，不将设计确认视为生产适配已实现。
- 实测 JVM/Native：`rpc.heldValues().catch { ... }.collect { throw localError }` 不误还原前端错误，但对应后端 collector 仍驻留；显式 cancel、有限流完成、服务端抛错与整个 endpoint 关闭是不同路径。
- 源码吻合：[KrpcClient.callServerStreaming](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-client/src/commonMain/kotlin/kotlinx/rpc/krpc/client/KrpcClient.kt#L278-L309) 仅 catch CancellationException 时发送 REQUEST cancellation；普通异常进入 finally 清除本地登记，不等于已释放服务端请求。因此不能把此前正常取消结论推广为任意 collect 失败。
- 两端对 buffer(0) 的测试已验证服务端 finally 到达、同连接后续调用可用、前端错误未误还原，原远端异常/取消仍可解析；分离上游协程是 [buffer 的原有机制](https://github.com/Kotlin/kotlinx.coroutines/blob/1.11.0/kotlinx-coroutines-core/common/src/flow/operators/Context.kt#L33-L62)，零容量不等于省略该算子。
- 覆盖限于已测试的下游失败场景，不泛化为框架内部解码失败或所有连接故障都已覆盖；当前仅测试使用，未新增生产包装器或 fork 框架。本次只同步设计，不重跑上一批编译及往返测试。

FIXME:

## 原实现映射与研究证据

- 以下是源码核查/外部研究，不是 checklist 决策或运行证明；引用的原实现继续作为后续接入入口。

| 入口 | 已核查事实与源码 |
| --- | --- |
| CLI/Application | Main 直接 openDefault/shutdown；Application 持有 DI/auth/MCP/scope/Home，并非共享服务。`Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt:30-52`；`Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt:91-152` |
| Session 生命周期 | open 复用/取消归档，release 会 shutdown；资源回收不是单纯丢值。`Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt:61-105,180-210,451-472` |
| NewSession | 原 materialize 包含创建、首次 submit 和失败删除；`Kodex/app/viewmodel/new-session/src/commonMain/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModel.kt:111-167` |
| Agent 运行 | execution.running 包含 runningTurn/runtimeOperation/historyOperation；submit 返回不等待整轮结束。`Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt:491-502,524-546` |
| 目录 | repository inventory 不是自动共享更新源；目录按需查询须避免陈旧盘点。`Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt:49-77,142-166,192-234` |
| timeline | 六条缓存共用 CachedIndexVersionedImpl，通过 CachedIndexVersioned 暴露 cacheNonce/latestIndex；set 追加，revert 删除 suffix。`Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt:118-160,255-302` |
| History | 主 History/HistoryIndex 各自从 0 计代际，旧请求捕获 owner/代际，分页还检查窗口身份。`Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt:60-73,169-242,421-484`；`Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexViewModel.kt:44-78,115-173` |
| 全索引 | HistoryIndex 追加拼完整 indexes，侧栏按全列表位置绘制；LazyColumn 不等于索引内存有界。上述 HistoryIndex 文件 `:115-149`；`Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt:360-419` |
| Agent settings | 原 updateSettings 在 writeMutex 内写完整值；`Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt:398-406` |
| models | 目录有独立 models StateFlow 与 provider 刷新，不只是 settings.map；`Kodex/openai/model-catalog/impl/src/commonMain/kotlin/io/github/stream29/kodex/openai/modelcatalog/OpenAiModelCatalogImpl.kt:34-47,68-71` |
| 认证 | Codex 原每分钟重读但不主动 refresh；私有来源有续期，旧登录固定写 auth.yml 并切来源。`Kodex/app/shared/auth/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/FileSystemKodexAuthStore.kt:183-246,305-333,386,468-471` |
| OpenAI OAuth | 原准备持有 listener，awaitCompletion 实际执行交换/保存，并非纯观察；`Kodex/app/shared/auth/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/LocalKodexLogin.kt:25-105,110-211` |
| MCP OAuth/导入 | 原 manager.login 发 URL 后等待提交；effect 是分摊消费的 receiveAsFlow，preview 只有一个 activePreview。`Kodex/mcp/impl/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt:60-61,171-299,301-419`；`McpOAuth.kt:406-525`（同目录） |
| 旧 Hooks | 原 turn/tool 控制结果影响执行；迁移须裁剪，不能只改名。`Kodex/hook/contract/src/commonMain/kotlin/io/github/stream29/kodex/hook/contract/tool/ToolHooks.kt:4-8`；`turn/TurnHooks.kt:4-8`（同 contract 目录） |

### kotlinx.rpc 0.10.3

- 本节为先前固定版本研究；其中 WebSocket、断网和多进程恢复不再是一期开工条件，不因保留证据而恢复网络部署范围。
- 2026-09-08 核对时 Latest 为 0.10.3，0.11.0-grpc-189 为独立 gRPC 预览；不据版本号更大选用预览。后续以固定 0.10.3 源码核查，不声称此记录永远最新。[发布](https://github.com/Kotlin/kotlinx-rpc/releases)、[版本表](https://kotlin.github.io/kotlinx-rpc/versions.html)。
- 支持普通 Flow，不直接支持 StateFlow/SharedFlow；顶层返回 Flow 的方法非 suspend。按调用背压统计消息条数，不保证大 payload 内存有界。[能力](https://kotlin.github.io/kotlinx-rpc/features.html)、[配置](https://kotlin.github.io/kotlinx-rpc/configuration.html)。
- 平台表覆盖 Linux x64/ARM64、macOS ARM64、Windows x64；Ktor 接入使用 WebSocket，也允许自定义 transport。一期已选内存通信，具体接入待落实；此处平台研究不是运行验证。[平台](https://kotlin.github.io/kotlinx-rpc/platforms.html)、[Ktor](https://kotlin.github.io/kotlinx-rpc/krpc-ktor.html)、[transport](https://kotlin.github.io/kotlinx-rpc/transport.html)。
- 插件按声明生成 stub/descriptor/invokator；泛型服务被拒绝，扫描跳过 fake override，生成入口要求 interface。已用本地 2.4.0-0.10.3 插件字节码交叉核对；普通父接口加显式 override 已编译验证，空子接口/抽象类未制作失败样例。[检查器](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/compiler-plugin/compiler-plugin-k2/src/main/kotlin/kotlinx/rpc/codegen/checkers/FirRpcServiceDeclarationChecker.kt#L27-L44)、[扫描器](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/compiler-plugin/compiler-plugin-backend/src/main/kotlin/kotlinx/rpc/codegen/extension/RpcDeclarationScanner.kt#L22-L37)、[生成入口](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/compiler-plugin/compiler-plugin-backend/src/main/kotlin/kotlinx/rpc/codegen/extension/RpcIrServiceProcessor.kt#L15-L18)。
- 客户端取消实际 collect 会发送 REQUEST cancellation；服务端移除对应 request 并取消 handler/collector，不关闭共享业务 StateFlow 或 Session。[client](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-client/src/commonMain/kotlin/kotlinx/rpc/krpc/client/KrpcClient.kt#L241-L310)、[server](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-server/src/commonMain/kotlin/kotlinx/rpc/krpc/server/internal/KrpcServerService.kt#L334-L406)。
- 官方 testCancellingNonSuspendable 覆盖客户端取消、服务端 collector 终止而连接仍活；GC 相关测试 Ignore（KRPC-169），不把丢引用当退订。本轮仅读测试，未运行。[取消测试](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-test/src/commonTest/kotlin/kotlinx/rpc/krpc/test/cancellation/CancellationTest.kt#L432-L484)。
- 官方 KtorRpcClient 成功初始化后绑定一次 WebSocket；transport 生命周期结束会取消 client 请求，没有自动重连/恢复 Flow。恢复需新的 RPC client/代理，可复用有效 HttpClient；不要求每条 Flow 各建连接。[Ktor client](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-ktor/krpc-ktor-client/src/commonMain/kotlin/kotlinx/rpc/krpc/ktor/client/KtorRpcClient.kt)、[transport](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-ktor/krpc-ktor-core/src/commonMain/kotlin/kotlinx/rpc/krpc/ktor/KtorTransport.kt#L12-L14)、[client scope](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-client/src/commonMain/kotlin/kotlinx/rpc/krpc/client/KrpcClient.kt#L137-L161)。
- 重复 collect 创建新 callId，不是断点续传；connector 没有重建 transport 路径。物理断网的发现依赖传输层，不能承诺立即/固定时间清理。[connector](https://github.com/Kotlin/kotlinx-rpc/blob/0.10.3/krpc/krpc-core/src/commonMain/kotlin/kotlinx/rpc/krpc/internal/KrpcConnector.kt#L260-L278)。
- 在旧代理上加 retry 不会更换失效 client，且不重试取消；stateIn 上游失败可结束共享协程而 UI 仍保留最后值。StateFlow 不因生产 owner 取消自动完成，也不会自动绑定新 owner。[retry](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/retry.html)、[stateIn](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/state-in.html)、[StateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/)。
- 无参 stateIn 可挂起等流首值，技术上并非没有 Get 就不能初始化；仍按用户选定的显式 Get 设计。CAS equals 不检测 ABA，update transform 可能重算；等待活性仍须实现验证。[CAS](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-state-flow/compare-and-set.html)、[update](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/update.html)。

### Codex 多实例凭据研究

- 证据为官方 2026-09-09 提交 `17e64839eb1e30632eef4a0147862345fccb61cc`；本地先读快照为 `5ecb3afd1bf405149e2159bfda50093b0c1b5fab`。未读取真实 auth.json/auth.yml/keyring，也未运行 app-server、登录或续期。
- 官方支持 file/keyring/auto/ephemeral，固定 auth.json 只覆盖部分；无官方进程活动时文件不会自动续期。[认证存储文档](https://developers.openai.com/codex/auth/#credential-storage)。
- AuthManager 内 Semaphore 串行刷新，锁内先重读并核对账号；不是跨进程锁，文件 load/save 也未覆盖整个读—刷新—回存的共享锁。[manager](https://github.com/openai/codex/blob/17e64839eb1e30632eef4a0147862345fccb61cc/codex-rs/login/src/auth/manager.rs#L2041-L2065)、[storage](https://github.com/openai/codex/blob/17e64839eb1e30632eef4a0147862345fccb61cc/codex-rs/login/src/auth/storage.rs#L195-L223)。
- A 回存新凭据后，B 刷新前重读同账号更新并跳过重复请求；401 恢复先重读、再有界刷新，成功保存轮换 token/last_refresh；永久失败按尝试快照记忆，来源更新后可恢复。[刷新](https://github.com/openai/codex/blob/17e64839eb1e30632eef4a0147862345fccb61cc/codex-rs/login/src/auth/manager.rs#L2803-L2899)、[回存](https://github.com/openai/codex/blob/17e64839eb1e30632eef4a0147862345fccb61cc/codex-rs/login/src/auth/manager.rs#L1572-L1594)、[恢复](https://github.com/openai/codex/blob/17e64839eb1e30632eef4a0147862345fccb61cc/codex-rs/login/src/auth/manager.rs#L1983-L2038)。
- 官方维护者 2026-02-01 说明 refresh token 有有限重复使用窗口，当时约小时量级；不是当前 SLA，不硬编码，也不能推导旧 access token 因轮换立即撤销。双方同时读旧值仍可能竞争，不能保证任意回存顺序安全。[官方回复](https://github.com/openai/codex/issues/10332#issuecomment-3831635259)。
- 官方测试覆盖磁盘先更新后重读、跳过坏旧 token、永久失败不重复、账号不匹配及有界恢复；是 mock/顺序改写，不是本轮双进程或真实服务端测试。[auth_refresh 测试](https://github.com/openai/codex/blob/17e64839eb1e30632eef4a0147862345fccb61cc/codex-rs/login/tests/suite/auth_refresh.rs#L432-L1384)。
- 曾研究 app-server 委托：account/read(refreshToken=true) 返回摘要，旧 getAuthStatus(includeToken, refreshToken) 有条件返回 token；用户已选择 Kodex 直接参与完整生命周期，不再作为待选路线。[账户 API](https://developers.openai.com/codex/app-server/#auth-endpoints)、[v1 协议](https://github.com/openai/codex/blob/17e64839eb1e30632eef4a0147862345fccb61cc/codex-rs/app-server-protocol/src/protocol/v1.rs#L184-L205)。
- 不能由无跨进程锁推导必须独占续期，也不能承诺凭据永久有效；撤销/过期等不可恢复失败仍须登录。共享来源具体失败处理尚未实现。

## 验证记录与边界

- 下列为早期批次实际执行记录；文档整理不视为重跑，后续批次见上方对应专题。

| 批次 | 编译/生成验证 | 测试记录 |
| --- | --- | --- |
| 首批 Agent settings、Get 修订、本地接口 | 当时 JVM 与四 Native 目标均通过；StateFlow 继承仅接口级 opt-in | 无实现，未做重试/网络测试 |
| 六条 timeline 与显式 override | JVM/Linux x64/ARM64/Windows x64 通过；每服务 10 stub/invokator/callable，普通父接口无服务生成 | 签名结构检查，非运行验证 |
| 取消后恢复 | 8 个服务当时共 68 方法完整；部分构建缓存 | 原设置 contract/filesystem 3+16 项强制重跑通过 |
| BackendSettings/Cli 模型重建 | 四可用目标通过，serializer 6/2/2 字段 | SettingsModelsTest 4 项通过 |
| OAuth 合并 | 两模块四可用目标通过，当时 GlobalRpc 15 方法完整 | contract 10、models 8 本轮运行；MCP 原 2 项缓存，共 20 |
| Codex MCP Get | 四可用目标通过，当时 GlobalRpc 16 方法完整 | contract 13、models 8、MCP 2，共 23 项运行通过 |
| Session 目录 Get | RPC 四可用目标通过；catalog 模块本身五目标通过；GlobalRpc 17 方法 | contract 16、catalog 1 本轮运行；models 8 未重跑，共 25 项报告通过 |
| Session 管理四方法 | RPC 四可用目标通过；GlobalRpc 21 方法与准确参数/返回值完整 | contract 既有 16 项运行通过 |
| 账号 usage 三方法 | RPC 四可用目标通过，macOS 跳过；usage-contract 本身五目标通过；GlobalRpc 24 方法生成完整 | contract 21（新增 5）、usage-impl 原映射 3，共 24 项本轮运行通过 |

- 增加 clean-models 等传递依赖后，RPC 的 macosArm64 因 cinterop/跨编译条件 SKIPPED，不计为通过；早期 settings-only 或 catalog 模块的成功不代表当前整个依赖图通过。
- 编译对应 JVM class 与 Native KLIB，不是 Native executable 链接或运行验证；Mosaic cklib configuration-cache 警告仍存在，缓存条目曾被丢弃。
- 账号 usage 批次的测试文件：contract 的 AuthenticationValuesTest 4、McpRuntimeValuesTest 6、CodexMcpSettingsValuesTest 3、SessionCatalogValuesTest 3、AccountUsageValuesTest 5；models 的 SettingsModelsTest 4、OAuthModelsTest 4。均使用虚构值，不证明后台业务实现。
- 测试验证序列化/原构造校验及必要默认值：不再测试旧 effect 返回值；Supported.transport 从配置恢复，BackendSettings 保留完整 MCP 值，CLI 模型无宽度。
- 已运行测试专用内存 RPC 的异常/取消边界，见上方专题；尚未运行生产连接/断网恢复、CAS 原子性与等待、跨 owner 缓存、TTL、真实 OAuth、Codex 多进程续期、目录运行标记或前端 spinner 接入验证。
- 原测试源码参考：`AgentHistoryModelsTest.kt:540-603`（app/viewmodel/history commonTest）、`HistoryIndexViewModelTest.kt:102-165`（app/viewmodel/agent commonTest）、`AgentHistoryActionTest.kt:33-103,134-198`（app/viewmodel/session commonTest）；核查阶段未重跑。
- OAuth 旧本机 fixture：`LocalKodexLoginAttemptJvmTest.kt:24-112`（app/shared/auth/filesystem jvmTest）、`McpOAuthIoTest.kt:46-202`（mcp/impl jvmTest）；已阅源码，不能当成新 RPC 切分的运行证明。
- 一期验证候选为内存 RPC 的状态初始化、CAS 更新/取消/ABA、历史读取与迟到结果、宿主存活时的观察取消、CLI 退出清理、OAuth 交接及 Native CLI 回归；TTL owner 重建验证随具体回收实现细化，属于一期。网络断线、跨进程恢复与独立 daemon 验证后移；实施仍需明确授权。
- 构建复用项目 IDE 与运行中 Daemon 的准确 JVM；已有权限在无 Daemon 时启动 `/home/stream/.gradle/jdks/eclipse_adoptium-25-amd64-linux.2`，不改全局 Java 或切换设备。最近使用 PID 1652546，不将历史 PID 当作下次可用性证明。
- 先前遇原 Daemon 忙而自动新建的构建已取消，并只在新实例空闲后清理它；随后复用成功，未以环境失败为由改设备/依赖。临时构建日志均已清理。
- usage 批次先遇 IDE 占用而启动竞争 Daemon，已取消自己的客户端并在新实例 1701508 空闲后结束它；未中断原实例。随后复用 Temurin 25 Daemon 1652546 构建成功（26 秒），Mosaic configuration-cache 警告仍在，不修改其构建。
- usage 新测试覆盖五种状态、完整嵌套快照/Long/Instant/列表顺序、无私有 attempt 字段、可空与空列表区别、默认值及原校验；javap 核对 24 个 stub/invokator/callable，并确认没有为 CodexRateLimitResetAttempt 或整个 CodexAccountUsageState 生成 serializer。本轮未运行真实 provider/RPC、reset 或 Native executable。

## 历史与本次文档整理

- 最初由 Discussion 获准移入 Planning，使用独立可编译源码逐项审查边界；没有实施完整迁移。
- 2026-09-09 曾按用户决定撤回本任务全部源码增量并归档 cancelled；当时子模块干净，原设置 19 项测试及相关 JVM 编译通过。
- 随后用户明确恢复任务及契约，移回本 Planning 路径并重新验证；取消记录不再代表当前状态，done 路径删除是此次重新打开的既有差异。
- 恢复后逐项落实当前模型/契约；Hooks、完整设置、统一 OAuth、前端导入与 Session 快照的最新决定已归入 checklist，不保留失效方案作为指导。
- 本次仅迁移文档：新增主题 checklist，在相关旧文件中补适用范围，将任务保留为进度、缺口、证据与验证摘要；不改源码、依赖或阶段。
- 文档校验通过：相对链接/章节锚点、完整源码路径、Task Tree/Details 结构及空白；源码静态核对确认 21 个 GlobalRpc 方法及尚未落地的 Session 字段/入口。未重新编译或运行测试，因为本轮仅改文档；无临时文件或提交。

## 本地分批提交检查点

- 用户要求暂存当前进度并腾出主分支；根仓库与 Kodex 均已建立本地 refactor/rpc，完整迁移仍处于 Planning。
- Kodex 分为九个提交：
  - `a2372a49`：原领域值的最小序列化。
  - `8ce9d9c7`：手动压缩与 resume 共用运行槽位。
  - `ada4b5a2`：缓存观察接口与后端元数据。
  - `68e0286f`：目录时间、活跃和运行快照。
  - `14bb1ced`：封闭远程异常与 JSON 还原。
  - `6b5e7b36`：RPC 契约、模型及值/往返测试。
  - `f5a78934`：内存连接及生命周期测试。
  - `4897a08d`：公共客户端、状态和前端缓存适配。
  - `2433dac3`：两侧设置 store 与后端文件 CAS。
- 根仓库先以 `86ca5fe` 保存设计，再由本检查点提交保存任务状态及指向 `2433dac3` 的 Kodex gitlink。
- 提交前后核对源码工作区与逐批暂存范围；各批空白检查通过。本轮未重跑构建，既有验证结果及未接入范围仍以上方专题记录为准。
- 两个 main 保持原提交：根仓库 `29b624d4`、Kodex `92b572d4`。交付时切回两侧 main；恢复重构时分别执行 `git switch refactor/rpc` 与 `git -C Kodex switch refactor/rpc`。
- 独立的订阅用量讨论文件保持原样、未纳入提交；本轮不推送远端。
