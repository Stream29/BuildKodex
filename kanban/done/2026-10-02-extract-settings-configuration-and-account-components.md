# Task Tree

- `Inventory six components, state owners, RPC consumers, and baseline tests`()
- `Specify ports, ownership, compatibility, and review gates`()
- `Review the six-component plan and authorize implementation`()
- `Freeze shared DTO ownership, port signatures, and serializer fixtures`()
- `Assign exclusive component and host files to implementation work units`()
- `coroutineScope {`
  - `val configuration = async {`
    - `awaitAll(`
      - `async { Extract Context Source Settings component() },`
      - `async { Extract Session Title Settings component() },`
      - `async { Extract Application Preferences component() },`
    - `)`
  - `}`
  - `val account = async {`
    - `Extract Authentication Settings component`()
    - `Extract Account Usage component`()
  - `}`
  - `val reset = async {`
    - `Extract Usage Reset component`()
  - `}`
  - `configuration.await()`
  - `account.await()`
  - `reset.await()`
- `}`
- `Integrate shared hosts, adapters, RPC dependencies, and Gradle consumers`()
- `Run component, host, serializer, and downstream checks through one coordinator`()
- `Review the complete batch and record unvalidated targets`()
- `Record user acceptance and archive the complete batch`()

# Details

## Scope and authorization

- 用户已审阅六组件计划并明确授权实施；任务进入 executable。
  用户已验收本批实现，无异议；三条实现 Session 已经用户单独确认。
  本批主任务和六份子任务归档到 done；用户随后明确授权提交和推送。
- 工作区：外层 `main`，内层 `refactor/spec`，内层基线 `7310a445`。
  前一批已验收、提交并推送，记录见
  [四组件批次](../done/2026-10-02-extract-pending-tool-and-resource-components.md)。
- 配置编辑是一个家族；账号与用量是另一个家族。相似性用于划分工作单元，
  不把六个业务合并为通用 Settings ViewModel。
- 每个组件采用 `app/component/<name>/{spec,impl/viewmodel,impl/view}`，
  已新增 18 个实际使用的项目；不新增 `core`、空 spec 或重复协议模型。
- 不迁移整个 Settings、后端标题生成器、认证 store 或 usage store；
  不改变 RPC 方法/载荷、设置文件格式或两种凭据来源的维护规则。
- 用户其他 discussion/executable/done 任务、shared-context 文件和 Draft 不在范围内。

## Mapping and complete component ownership

| 家族/工作单元 | component | 完整迁移责任 | 子任务 |
| --- | --- | --- | --- |
| 配置/独立 | `context-source-settings` | 内置开关、自定义列表、新增草稿/校验、启停和删除 | [Context Source](2026-10-02-extract-context-source-settings-component.md) |
| 配置/独立 | `session-title-settings` | 自动标题开关、模型/推理强度、菜单、默认与不可用展示 | [Session Title](2026-10-02-extract-session-title-settings-component.md) |
| 配置/独立 | `application-preferences` | General 页左右宽度与换行/提交键，以及菜单和状态 | [Preferences](2026-10-02-extract-application-preferences-component.md) |
| 账号/同工作单元 | `authentication-settings` | 来源选择、摘要、登录入口、注销确认/进行中/失败 | [Authentication](2026-10-02-extract-authentication-settings-component.md) |
| 账号/同工作单元 | `account-usage` | 用量分支、fallback、刷新、重置入口和呈现 | [Account Usage](2026-10-02-extract-account-usage-component.md) |
| 账号/独立复杂单元 | `usage-reset` | 券列表、准确选择/确认、消费结果/失败和生命周期 | [Usage Reset](2026-10-02-extract-usage-reset-component.md) |

- 原入口集中在 `app/view/settings/.../SettingsPopup.kt`、
  `ContextSourcesSettings.kt`、`CodexAccountUsageSettings.kt`。
  模型/命令主要来自 `app/contract/settings/.../GlobalSettingsViewModel.kt`、
  `RpcGlobalEditor.kt`、`RpcUsageReset.kt`；这些共享文件只由整合负责人修改。
- 六组 factory 均接受 typed `*Dependencies` 和明确 owner scope。
  spec 声明 ViewModel、state/effect、全部命令、依赖行为和附着 KDoc，
  包括准入/副作用顺序、失败/取消/关闭、渲染分支及实际公开的 `@throws`。
- child 持有业务 draft、验证、确认与操作状态；renderer 只持有菜单锚点、
  focus/scroll/widget buffer。父级不镜像这些状态或保留第二套编辑命令。
- spec / ViewModel / View 分别使用 `kodex.kmp-cli` /
  `kodex.kmp-viewmodel` / `kodex.kmp-view`，保持 conventional target 集合。

## Proposed dependency and integration design

- `GlobalSettingsViewModel` 发布六个稳定 child，并保留现有 MCP/Hook child；
  只移除已迁走字段/命令，不重写 Session/New Session 的设置范围。
- 配置端口隐藏 BackendSettings/CliFrontendSettings、raw CAS/MutableStateFlow
  和存储句柄。整合 adapter 仍使用原 `RpcGlobalEditor` 应用队列及字段 baseline；
  queue admission 不是保存回执。宽度变化不进入持久化队列。
- Account Usage 通过依赖端口发出 `requestReset` 意图，由宿主连接 Usage Reset。
  Usage Reset 可依赖 Account Usage spec 的现有用量 DTO；
  Account Usage spec 不反向持有 reset 类型，避免互依赖环。
- Authentication 通过 typed login-opening port 请求已有 OpenAI Login；
  真正 OAuth、loopback/browser、凭据提交仍在原组件/adapter，不能复制登录状态机。
  来源变更与 reset dismissal 的跨组件协调放宿主，不让组件直接调用其他 impl。
- `SettingsAuthenticationState`、`SettingsAccountUsageState` 已是 GlobalRpc wire DTO。
  已分别迁入 authentication/account-usage spec，保持原 FQCN、嵌套类型、
  serializer descriptor、字段、可空值与默认值；不复制 UI/wire 两份模型。
  `rpc/spec/contract` 改为直接依赖这两个 spec，而非整个 `app/contract/settings`；
  六个 component spec 均不得依赖 RPC contract 或 Global Settings contract。
- `SessionTitleSettings`、`NewLineKey`、`SubmitKey`、设置文件值类型留原共享所有者；
  新 spec 可暂用 `app/shared/settings/contract`，明确记录其既有混合依赖债务，
  不借本批扩大到整个 shared settings 根迁移。
- 标题默认模型通过 adapter 的 `defaultModel` 输入复用现有常量；
  新 spec/VM 不依赖后端标题生成器，也不在组件里复制默认模型字符串。
- 纯 renderer 样式沿已有 `app/view/components`；不得反向依赖 `app/view/settings`。
  公共文本/日期 helper 只在确有多方消费者时保留在公共 renderer 层。

## Baseline behavior and review gates

| 已核对的事实 | 计划约束或待评审决策 |
| --- | --- |
| 侧栏宽度属于 RpcGlobalSettings 临时 StateFlow；输入键保存前端文件 | 不把宽度写盘，不让换行/提交键成为两个可独立写入的状态 |
| null 标题模型使用现有默认常量；选项是目录顺序加当前有效模型 | 保持 fallback/顺序，不随目录变化改写配置；默认常量通过 adapter 提供 |
| Context Source 的 `~` 比较使用前端 home，最终路径留后端解释 | 先记录当前差异；不把迁移变成后端路径解析/RPC 扩展 |
| OpenAI 页首次/每次重新进入会刷新 usage | 保留导航触发；不在新 VM 构造里额外重复刷新，按钮禁用不等于所有命令已有去重 |
| 注销目前在确认命令开始时捕获 source；文案固定称 Kodex/Codex 不受影响 | 文案需符合两种来源的现行 SOP；改为打开时绑定 source 属于行为差异，须复核而非暗改 |
| 内部 reset `confirm(expected)` 校验对象身份，父接口却重新读取最新 confirming | 提案把原 exact-confirmation 能力完整暴露给 renderer；先测试迟到回调差异，再冻结签名 |
| 无券明细时 show 产生 PreparationFailed；界面仍说后端会自动选券 | 修正过时提示，不增加自动选券、prepare/attempt RPC 或隐含消费 |
| expiry=null 时现有确认界面省略日期 | 按已确认 SOP 明确显示未知；不将其描述为永不过期 |
| Preparing 只有声明和 renderer 分支，当前生产路径不发布 | 静态核对后评审移除这个非 wire 的遗留分支；不为了填状态而新建准备 RPC |
| 页面离开调用 reset.dismiss，消费进行中 dismiss 无效；close 取消前端 owner | 区分隐藏、关闭、调用等待取消与后端已接受副作用；未知结果不自动重放 |

- 本批主要是边界迁移。以上文案/SOP 纠正、exact callback 和未使用状态处置
  必须在实现前列出测试及行为差异；不把“与旧代码一致”写成与现行 SOP 冲突的 spec。
- 账号摘要、authSource、usage 是独立流，不承诺原子组合或确认期间的账号锁定。
  保持后端按消费准入时当前账号校验券，前端不新增账号参数或幂等身份。
- 所有确定消费结果完整呈现；普通调用失败刷新用量但不证明未消费，
  不自动 retry/revoke/清空旧凭据，不承诺 exactly-once。

## Parallel work and shared-file ownership

- 逻辑上最多五个独立工作单元：三个配置组件、账号两组件、复杂 reset；
  简单组件可并行，reset 单元不叠加另一个复杂状态机。
- 上方 Task Tree 是执行规划，不代表已创建 Session。实际并行 Session 的数量
  与文件权限在启动时经用户确认；一个整合负责人、一个构建协调者。
- 独立工作单元拥有各自 component 树和专有测试；只拟定既有共享声明的迁入
  内容，不同时从共享文件删除。整合负责人统一删除旧声明、接入 typed adapters
  并分配原测试用例，避免复制 suite 或同时改同一文件。
- 集中所有权：Global Settings contract、SettingsPopup/导航、
  RpcGlobalEditor/RpcSettings/RPC contract/build 文件、Application 接线、
  共用 test fixture、共享 codec/serializer 检查与 checklist。
- 先固定必要端口与 DTO dependency 方向，再写各自实现；共享宿主在交接后合流。
  不通过多个 Daemon、同时构建变动中的工作树或操控用户 IDE 增加并行度。

## Implementation checkpoint

- 用户已确认三条并行 Session：配置三组件、认证/用量两组件、重置单组件。
  三条线只改各自新组件树、测试及对应子任务；现会话独占所有既有共享文件、
  旧声明清理、适配器、宿主 Gradle 与集中验证。
- DTO 继续使用原 `io.github.stream29.kodex.app.settings.contract` 包名；
  认证、用量重定位保留全部原 wire 形状，以现有 RPC value 测试及固定 fixture 校验。
- 注销仍在确认命令开始时捕获 live source；对话框展示当前 source，
  不实施尚未单独确认的 open-time target binding。登录目标仍在宿主创建 Login 时绑定。
- Reset 的 exact confirmation 能力已直接传给 renderer；移除无生产发布者的
  非 wire `Preparing`。新增迟到/等值伪造确认测试，未知 expiry 明确显示，
  取消或关闭只约束本地状态，不承诺撤销后端副作用。
- 配置字段写入维持原应用队列及各字段 baseline，宽度仍是运行期状态；
  UI disabled 不自动新增程序命令去重或业务锁。关闭组件不撤销已接受队列写入。
- Global Settings 仅暴露稳定 child、共享 failure 与 Login 宿主 intent，
  不再复制组件状态、命令或 reset controller。SettingsPopup 只保留导航/viewport、
  dropdown handles 和 overlay 接线；完整交互属于各组件。

## Validation and acceptance

- 六组件分别提供 dependency-only VM 测试和完整 renderer 测试，
  覆盖 state/effect/command、验证、失败、取消、closed/late callback 与键盘操作。
- 集中验证 Settings/Application/RPC、前一批 MCP/Hook/Working Directory/工具
  回归，queue 顺序、关闭排空、前端宽度不落盘与输入键 codec。
- 认证/用量 DTO 重定位前后保存 serializer descriptor/JSON fixtures；
  运行 RPC contract values 测试，编译 server/client/generated consumers 和
  integration 测试源码，不改 frozen wire。
- 重点测试无明细/空列表/未知计数区别、旧账号 fallback、重置二次确认、
  迟到确认、确定结果后刷新失败、未知结果仅刷新一次、取消与重复命令。
- 静态核对 18 项目、旧声明移除、生产/test 依赖方向分别检查、无新增环、
  KDoc 链接/throws、文档链接和内外层 diff。已有 spec 闭包债务另记，不假称全仓纯化。
- 构建前重查 Daemon/JVM/用户活动，复用可用 JVM 并集中运行；只记录实际通过的
  目标。Native/JS/Desktop/CLI、IDE sync、真实网络登录未验证时明确列为剩余项。
- 六组件全部合流、统一交付审阅，用户已验收并另行授权提交、推送。
- 规范入口：[Frontend boundary](../../checklist/frontend-application-boundary.md)、
  [RPC Settings](../../checklist/rpc-settings.md)、
  [RPC Authentication](../../checklist/rpc-authentication.md)、
  [RPC Account Usage](../../checklist/rpc-account-usage.md)。

## Validation results and acceptance checkpoint

- 最终集中构建通过：复用 JDK 26 Gradle Daemon，单 worker、关闭 parallel；
  18 个新项目、Settings/RPC/Application 生产源码和 Integration 测试源码编译通过。
- JVM XML 结果共 **851 项，0 failure/error/skipped**：

| 验证组 | 测试数 |
| --- | ---: |
| 六个新组件 VM/renderer | 175 |
| 前批工具、MCP/Hook、Working Directory、Login 组件 | 150 |
| Settings/Application renderer 与 ViewModel、RPC 前端适配 | 225 |
| RPC client/server | 164 |
| 共享设置 contract/filesystem 与标题生成 | 76 |
| RPC contract，包括固定认证/用量 wire fixtures | 61 |

- 新组件分项：Context Source 16、Session Title 10、Preferences 11、
  Authentication 27、Account Usage 46、Usage Reset 65。六个 spec 的测试任务
  `NO-SOURCE` 不计为测试用例。
- 主测试命令使用 `--max-workers=1 -Dorg.gradle.parallel=false --continue`；
  对六组件 `spec/impl-viewmodel/impl-view` 的 `jvmTest`，以及前批六组件
  `impl-viewmodel/impl-view` 的 `jvmTest` 集中执行，再执行：
  `rpc-spec-contract`、`app-view-settings`、`app-viewmodel-settings`、
  `app-viewmodel-rpc`、`app-viewmodel-application`、`app-view-application`、
  `rpc-impl-client`、`rpc-impl-server`、`app-shared-settings-contract`、
  `app-shared-settings-filesystem`、`app-shared-session-title` 的 `jvmTest`，
  以及 `integration-test:compileTestKotlinJvm`。
- 发现并修正旧调用遗漏、菜单 padded-label 测试匹配和异步输入等待；
  RPC frontend 测试 fixture 改为串行 owner dispatcher，遵守组件命令约定，
  避免初始 observation 与 UI 命令在 Default 多线程上争用。未改 MCP 生产实现。
- 首轮既有 Application renderer-cancellation 用例曾出现
  `IllegalStateException: RpcClient was cancelled`，单独重跑及最终显式
  `--rerun` 均通过。生产 RPC teardown 未修改；这是已记录的偶发验证风险，
  不宣称本批修复了所有 RPC 关闭竞态。
- 18 项目路径/plugin、spec 直接依赖、单一 DTO 声明、旧 API 清理、
  KDoc/文档链接及内外层 diff 检查通过。允许的 shared-settings contract
  仍有既有 shell-client impl 闭包债务，本批没有把它描述成全仓纯 spec。
- Native/JS/Desktop/CLI、IDE sync 和真实 OAuth/provider 测试未运行；
  用户已明确验收通过，七份任务归档到 done；
  内层实现提交为 `05a472e9`，外层设计记录和子模块指针分别提交，
  并按用户授权依次推送内层 `refactor/spec` 与外层 `main`。
