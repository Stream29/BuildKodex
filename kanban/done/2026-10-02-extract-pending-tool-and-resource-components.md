# Task Tree

- `Inventory all four components, owners, side effects, and regression tests`()
- `Review proposed ports, ownership, and behavioral approval gates`()
- `coroutineScope {`
  - `val pendingTools = async {`
    - `Extract Request User Input component`()
    - `Extract Suggest Subagent Task component`()
  - `}`
  - `val settingsResources = async {`
    - `Extract Hook Settings component`()
    - `Extract MCP Settings component`()
  - `}`
  - `pendingTools.await()`
  - `settingsResources.await()`
- `}`
- `Integrate shared hosts, RPC adapters, and Gradle consumers`()
- `Run component and host regressions through one build coordinator`()
- `Review all four components and record unvalidated platforms`()
- `Record user acceptance and commit the buildable component slice`()

# Details

## Scope and phase

- 用户已验收四组件和此前 Working Directory 的累积成果，并授权提交/推送；
  内层已提交为 `7310a445`，本批进入 done。
  两条独立工作线的 Session 创建也已经用户确认。
- 本批覆盖四个完整交互组件，不只迁移弹窗。相似性用于组织工作，不用于
  把不同业务合并成通用 ViewModel。
- 工作区：外层 `main`，内层 `refactor/spec`；Working Directory 与本批重叠宿主，
  一起作为可构建切片提交。其记录见
  [Working Directory](2026-10-02-extract-working-directory-component.md)。
- 当前会话负责共享宿主、Gradle、文档和构建；Session 457 负责工具组件，
  Session 458 负责设置资源组件，均只改指定文件、不构建或提交。
  用户的其他任务、草稿与共享 IDE 窗口不在本批范围。
- 两条线均已交接实现与组件测试；集中构建、复核和用户验收完成。开始构建前没有运行中的
  Gradle Daemon，统一以 JDK `openjdk-26.0.2` 启动唯一构建 Daemon，
  PID `881440`，后续命令复用该 JVM，`--max-workers=1`。

## Mapping and component responsibilities

每个目标均建立 `app/component/<name>/{spec,impl/viewmodel,impl/view}`，
已新增 12 个 Gradle 项目。`spec` 使用 `kodex.kmp-cli`，
ViewModel 使用 `kodex.kmp-viewmodel`，View 使用 `kodex.kmp-view`；
按现有 convention 保持 target 集合，不自行复制 source-set 层级。

| 工作线 | component | 完整责任 | 子任务 |
| --- | --- | --- | --- |
| 待处理工具 | `request-user-input` | 问题、选项/Other、答案草稿、revision、提交与渲染 | [Request User Input](2026-10-02-extract-request-user-input-component.md) |
| 待处理工具 | `suggest-subagent-task` | 任务列表、批量配置、接受/拒绝及反馈、提交与渲染 | [Suggest Subagent Task](2026-10-02-extract-suggest-subagent-task-component.md) |
| 设置资源 | `hook-settings` | 列表、详情、新增/编辑草稿、类型选择、删除确认 | [Hook Settings](2026-10-02-extract-hook-settings-component.md) |
| 设置资源 | `mcp-settings` | 列表、详情、编辑、删除、启停、认证、重连、Codex 导入 | [MCP Settings](2026-10-02-extract-mcp-settings-component.md) |

- 不迁移整个 Agent、History、Settings、MCP 后端或 Hook 执行器。
- 不新增持久化格式、RPC、业务事务、自动重试或通用资源管理框架。
- Kotlin 包名优先保留；移走原声明而非复制 DTO、保留双份接口或长期转发实现。
  RPC/持久化联合和 serializer 的字段与名称不变。

## Implementation and validation result

- 四组件的 ViewModel 接口、typed dependency/factory、业务 draft/state/effect 与行为 KDoc
  已归入 spec；ViewModel 与完整 renderer 各在自己的 impl topic。Agent/Global Settings
  只暴露稳定 child，旧组件声明、实现、renderer 入口已移除，没有转发副本。
- Suggest 的完整配置菜单归入组件，Working Directory 仍由宿主绑定 exact call；
  三组 docs recording fixtures、Application 配置/视口测试也已切换新接口。
- Hook/MCP adapter 复用 `RpcGlobalEditor` 原写队列，区分 admission 与持久化；
  captured baseline 和已接受的导入 candidates/decisions 在 page hide/close 后仍排空。
  新 RPC 回归覆盖 MCP 写入阻塞时 Hook 不越过队列，以及两者跨关闭排空。
- 原始 MCP 配置/secret Keep 解析与 OAuth 协议留 adapter。组件 URL effect
  在整个 Settings 生命周期消费，失败只取消捕获的准确 job；page switch 不取消 OAuth。
  shared operation failure 只渲染一次；ViewModel state 使用只读 StateFlow wrapper。
- 12 个组件项目及 RPC、Application、Settings 主源码 JVM 编译通过；
  RPC server/client 主源码与 integration-test 的 JVM 测试源码编译通过。

| 定向 JVM 测试 | 数量 | 结果 |
| --- | ---: | --- |
| Request User Input ViewModel / View | 15 / 6 | 全部通过 |
| Suggest Subagent Task ViewModel / View | 18 / 8 | 全部通过 |
| Hook Settings ViewModel / View | 12 / 9 | 全部通过 |
| MCP Settings ViewModel/parser / View | 27 / 24 | 全部通过 |
| Working Directory ViewModel / View | 7 / 6 | 全部通过 |
| Agent、Settings、Application、RPC 宿主与 RPC models | 260 | 全部通过 |
| 合计 | 392 | 0 failure / error / skipped |

- 首轮验证发现了 coroutine test opt-in、docs fixture 旧参数及异步菜单断言时机，
  均已修正；最终全批命令成功，633 actionable tasks，configuration cache stored。
  TestBalloon/JUnit XML 统计为上表；opt-in docs 录制导出没有执行。

在 `Kodex/` 中复现最终验证（Bash brace expansion；复用当前 Daemon JVM）：

```bash
JAVA_HOME=/home/stream/.jdks/openjdk-26.0.2 ./gradlew \
  :app-component-{request-user-input,suggest-subagent-task,hook-settings,mcp-settings}-{spec,impl-viewmodel,impl-view}:jvmTest \
  :app-component-working-directory-impl-{viewmodel,view}:jvmTest \
  :app-{viewmodel-agent,view-agent,viewmodel-settings,view-settings,viewmodel-application,view-application,viewmodel-rpc}:jvmTest \
  :rpc-spec-models:jvmTest :rpc-impl-{server,client}:compileKotlinJvm \
  :integration-test:compileTestKotlinJvm --max-workers=1 --continue
```

- 静态检查：12 项目及引用有效、直接 spec 无 impl/宿主/framework 边，
  reachable project graph 无环、14 个任务相对链接有效、内外层 diff 检查通过。
- 全局 spec 生产依赖闭包仍有既有债务：
  `clean-models -> tool/spec/unified-exec -> utils/shell-client/impl`
  及其 os-environment impl；新组件没有增加直接 impl 边，本批不声称闭包已完全纯化。
- Native、JS/未来 Desktop renderer、CLI smoke 与真实网络/OAuth 集成未运行；
  当前没有可用 IDEA MCP 诊断入口，未进行 IDE sync 验收。本批及此前 Working Directory
  已提交为 `7310a445`，其他用户任务未改动。

## API and dependency graph

- 每个 spec 定义实际使用的 `*ViewModel`、`*Dependencies`、
  `*ViewModelFactory`、state/effect/command 及附着 KDoc。类型名称沿用已有
  名称时直接迁移；实际签名以各 spec 为准，子任务保留对应 handoff 记录。
- ViewModel 工厂接受 typed dependencies 和显式 owner scope；scope 是资源
  所有权输入，不是任意 service locator。renderer 不负责创建 RPC、store 或 manager。
- Agent 暴露两个稳定组件 child；Global Settings 暴露 MCP/Hook 稳定组件 child。
  父级不镜像 child 草稿、对话框或提交状态，不保留两套可写命令 authority。
- 业务草稿、验证、提交阶段和详情/编辑/确认的选择由组件 ViewModel 持有；
  焦点、滚动、hover、布局及输入控件内部 buffer 留 renderer。任何“输入控件状态”
  都不能替代 spec 暴露的业务草稿。
- `view -> component spec`；`viewmodel -> component spec`；
  宿主适配器/assembly 可依赖对应实现。四组件互不依赖对方 impl。
- 工具组件 spec 只依赖 tool/OpenAI/clean-model 等已有 spec 及基础库；
  不依赖整个 Agent contract。设置组件 spec 只依赖 MCP spec、RPC models 等
  值类型及基础库，不依赖 GlobalSettingsViewModel、GlobalRpc 或本地 store。
- 当前 `rpc/spec/contract` 依赖 `app/contract/settings` 的认证摘要。
  移入 `NotificationHook` 等值类型时只依赖 `rpc/spec/models`，避免形成
  `component spec -> rpc contract -> app settings -> component spec` 环。
- 工具两面板共用的选择/文本 renderer helper 可下沉已有 `app/view/components`，
  前提是不携带具体工具状态；不为少量代码建立新的空 spec 或组件间 impl 依赖。
- Settings 对话框沿用当前独立显示语义；组件不能暗中改为全应用独占 popup。
  Settings 页切换与关闭通过明确 child command/close 处理。

## Side effects and lifecycle gates

- 工具提交由绑定 Agent 的端口完成；前端依然负责草稿和用户确认。
  后端 pending 不是组件可写 state。保持已有结果完成与 resume 的职责分离，
  不从通知推导执行许可。
- 当前每种工具由 `RpcAgentPresentation` 对 ToolPending 取 `firstOrNull()`；
  本批保持这个既有单调用投影，不顺带增加多调用队列或丢弃全部其他调用。
  完成后的下一调用仍由现有订阅投影推进。
- 当前工具 submit 在 owner scope 内执行。renderer 等待被取消不等于撤销已接受
  后端命令；component close、owner 终止、pending 更换及迟到结果分别写清。
- MCP 写入 BackendSettings；Hook 写入 CliFrontendSettings。观察状态不成为
  第二份存储，界面成功排队不意味着已经持久化或连接 Healthy。
- 已接受设置写入沿原应用级队列排空；组件关闭拒绝新编辑、停止页面观察及短期
  认证交互，不取消已经接受的写入。队列所需 baseline/preview/依赖须活到排空。
- MCP、Hook 与其他全局编辑继续共享原宿主队列及接受顺序；
  不为每个组件创建各自写队列而改变全局写入顺序。
- 工具面板隐藏或选中 tab 切换只影响 renderer 观察，不关闭 Agent 拥有的 child，
  不丢弃其草稿或取消提交；Settings 页切换与关闭整个 Settings 也分别建模。
- MCP 配置流与运行状态流不是联合原子快照。Secret Keep/Replace/Remove、
  认证 exact attempt、导入 previewId 和字段冲突规则分别保留。
- 未知设置失败沿应用级本地 failure 状态报告；重开仍可见。不得因为拆组件
  重复报告错误、泄露凭据，或把 CancellationException 转成业务失败。

## Parallel execution and shared-file ownership

| 独立工作 | 主工作区 | 集中整合文件 |
| --- | --- | --- |
| 待处理工具线 | 两组 component 代码与迁入的组件测试 | Agent contract、RpcAgentPresentation、AgentRuntimeScreen、Application 配置菜单 |
| 设置资源线 | 两组 component 代码与迁入的组件测试 | GlobalSettingsViewModel、RpcGlobalEditor、RpcMcpSettings、SettingsPopup |
| 整合负责人 | 宿主接线、构建依赖、文档 | shared renderer helpers、RPC/CLI assembly、共同测试夹具 |

- 两条线可以并行起草 spec、实现独立模块及组件测试；每条线内部先完成较小组件，
  再处理复杂组件。本批包含四个目标，复杂状态机不同时堆进一个工作单元。
- 先约定宿主需要的 child/factory/dependency 形态；集中整合文件只由整合负责人
  改写。分支产物不直接同时修改共同宿主或运行同工作树构建。
- 以上是逻辑并发图；如需真实多 Session，先通过工具向用户确认创建，
  每条线提供自包含范围与文件所有权。未获确认不声称已经并行执行。
- 两条线完成模块边界后一起合流；不每做完一个组件就停下来重新选择批次。
  每个组件仍有独立审阅与验证记录。
- 一个构建协调者检查共享 Daemon、JVM、内存和用户活动，再按批调度；
  不以多个 Gradle Daemon 或抢占共享 IDE 增加“并行度”。

## Review gates before and during implementation

- 将已确认迁移约束与候选 API 区分：工厂/port 的具体签名在每组 spec 实施前复核，
  不把文档中的示意自动当作新增 RPC 或业务行为授权。
- 工具代码目前有需定向测试的风险：同 callId 投影去重、提交中换 pending、
  CancellationException 后的状态、revision 耗尽、创建 Sessions 成功但完成工具失败。
  先记录实际行为；若需改语义或修缺陷，列出具体差异单独评审。
- 不承诺跨“创建 Sessions、完成工具、resume”的事务回滚或 exactly-once。
  异常返回/丢失回执不证明副作用未发生，不自动重新创建 Sessions。
- 导入 apply 已排队后切页/dismiss 与 preview 清理的竞态必须测；
  若当前实现与已确认“接受写入排空”约束冲突，阻塞相关子任务并报告，
  不靠取消写入或保留无限期 UI 状态掩盖它。
- 参照 [Frontend boundary](../../checklist/frontend-application-boundary.md)、
  [RPC AgentRuntime](../../checklist/rpc-agent-runtime.md)、
  [RPC Settings](../../checklist/rpc-settings.md)、
  [RPC MCP](../../checklist/rpc-mcp.md)、
  [RPC Authentication](../../checklist/rpc-authentication.md)。
  旧 Hook 控制链不是本批 Settings 的语义；通知 Hook 以 RPC Settings 为准。
  MCP 历史 Replace 的额外 invalidate 不恢复，沿用 RPC MCP 的配置值更新规则。

## Validation and delivery

- 每组件：dependency-only ViewModel 测试、所有 state/effect 分支及 `@throws` KDoc、
  编辑/校验/失败/取消/close 测试、renderer 键盘/焦点/确认与替换迟到回调测试。
- 合流：Agent、Settings、Application、RPC 的 JVM 编译与回归；工具完成事件旧
  serializer 测试、MCP CAS/导入与 OAuth、Hook 本地持久化与队列排空回归。
- 若共享 RPC contract 的传递依赖变化，同时编译 RPC server/client 和 integration
  consumers；公开 wire 声明不变也要验证 generated/serialized 消费方。
- 静态：12 个新增项目及引用、target 集合、spec 无 impl 边/环、旧声明/入口无重复、
  无循环依赖、无 raw secrets 进入展示/日志、文档链接与内外层 `diff --check`。
- 复用本批触及的 Working Directory 回归；不能覆盖或误报此前未验收成果。
- Native/JS 编译和 CLI smoke 以资源/设备可用性安排；未运行必须明确记录。
  不以 JVM 测试替代完整平台验证，不依赖真实 OAuth/网络集成作为单元测试前提。
- 四组件均完成并通过集成检查后一次交付审阅；提交按可构建依赖切片拆分，
  不承诺仅按目录硬拆四份就能独立构建，不提前提交未验收新成果。
