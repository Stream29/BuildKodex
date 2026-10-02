# Task Tree

- `Inventory the five components, all entrypoints, and baseline tests`()
- `Specify typed ports, state/rendering contracts, and ownership differences`()
- `Review the complete batch and authorize implementation`()
- `Freeze shared declarations and exclusive file ownership`()
- `coroutineScope {`
  - `val configuration = async {`
    - `awaitAll(`
      - `async { Extract Session Settings component() },`
      - `async { Extract New Session Defaults component() },`
      - `async { Extract Runtime Configuration component() },`
    - `)`
  - `}`
  - `val browsing = async {`
    - `Complete Session Catalog component`()
    - `Extract History Index component`()
  - `}`
  - `configuration.await()`
  - `browsing.await()`
- `}`
- `Integrate shared hosts, adapters, declarations, and Gradle consumers`()
- `Run component and downstream regression checks through one coordinator`()
- `Record user acceptance before committing or pushing`

# Details

## Scope and authorization

- 用户已审阅五组件计划并明确授权实施，任务进入 executable；
  已确认四条执行 Session，本批已完成并通过用户验收。
- 内层 `refactor/spec` 基线 `05a472e9`，外层 `main` 基线 `5831f81`。
  前批已验收并推送，见 [六组件批次](../done/2026-10-02-extract-settings-configuration-and-account-components.md)。
- 一份主计划、五份子计划；同批交付不意味着合并五个领域 ViewModel。
  四个已确认的工作单元及其文件所有权见 Implementation checkpoint。
- 不迁移完整 History、Composer、Shell 终端、运行/停止/压缩或整个
  Application/Agent 生命周期；不改 RPC、存储格式或供应商请求协议。

## Mapping and complete ownership

| 家族 | component | 完整责任 | 子任务 |
| --- | --- | --- | --- |
| 会话配置 | `session-settings` | 当前 Session/Draft 的配置、身份区、准确 Rename/Working Directory child 接线 | [Session Settings](2026-10-03-extract-session-settings-component.md) |
| 会话配置 | `new-session-defaults` | 持久化新会话默认值、四字段选择器与全部菜单 | [New Session Defaults](2026-10-03-extract-new-session-defaults-component.md) |
| 会话配置 | `runtime-configuration` | Agent/Draft 状态栏快速配置按钮、组合模型菜单与 Questions 菜单 | [Runtime Configuration](2026-10-03-extract-runtime-configuration-component.md) |
| 目录/索引浏览 | `session-catalog` | 完成已有 spec/VM 的 renderer、菜单、删除 child 与打开/关闭接线 | [Session Catalog](2026-10-03-complete-session-catalog-component.md) |
| 目录/索引浏览 | `history-index` | 稀疏索引扫描/投影、行读取、hover、时间戳、Check out 及两侧栏入口 | [History Index](2026-10-03-extract-history-index-component.md) |

- 统一树：`app/component/<name>/{spec,impl/viewmodel,impl/view}`。
  四个新组件各三个项目，Catalog 复用原 spec/VM，只新增 view：
  预计新增 13 个项目，目标树合计 15 个；不制造空 spec/core。
- spec 持有实际使用的接口、typed dependency/factory、state/effect/command
  和规范 KDoc。逐一写明返回值、顺序、异常/`@throws`、取消/关闭、
  状态分支怎样渲染；impl KDoc 可解释实现但不成为另一事实来源。
- 原公共 FQCN 能保留则保留，避免随目录迁移无关改名；旧共享声明只迁移，
  不复制第二份。已有 Rename、Working Directory、Title、Delete 组件复用。
- 插件分别使用 `kodex.kmp-cli`、`kodex.kmp-viewmodel`、`kodex.kmp-view`。
  spec 不依赖 RPC/旧父 contract/renderer/具体 I/O，VM 不依赖 view；
  实际借用的领域契约可以依赖，既有混合闭包债务必须单独记录。

## Baseline differences and review gates

| 已核对事实 | 迁移约束 |
| --- | --- |
| 当前 Session source 绑定准确 Agent binding 或准确 Draft，revision 失效后不可重定向 | delayed command 不能重新解析当前选中对象；模型目录变动不伪造目标 revision |
| 公开 Session factory 使用 cancel-on-close，本地 CAS retry 被取消 | 不照搬 Global/defaults 的“关闭后排空”策略；后端已接受动作不保证回滚 |
| 默认值编辑拥有应用级队列，关闭弹窗仍排空已接受写入 | 保留原队列/字段 baseline，不为了统一结构合并成 Global Settings 的同一个队列 |
| Settings 的 reasoning 是固定列表、tier 是 enum；Runtime 菜单按所选模型能力分层 | 不把两个菜单统一成一种可选值规则，也不自动纠正已存设置 |
| Runtime 一次提交 model/effort/tier，Questions 单独更新 | 保留组合原子性；renderer 当前调用等待受 composition scope 取消，需保持 caller cancellation |
| Catalog 构造无 I/O，打开 renderer 时 refresh；mutate 后 reload 失败不撤销 mutation | 不重复初始化读取、不自动重放 fork/delete；加载失败不伪装成功空列表 |
| Catalog 仍有 renderer-local deleteTarget | 将业务目标/准确 Delete child 放组件所有权，菜单 anchor/坐标仍只在 renderer |
| Index 以 cacheNonce/generation 和稀疏 indexes 识别目标，读取前后校验 | 不用位置或相同数字 index 替代身份；迟到结果不能恢复旧行/hover |
| Index 的 Check out 仅调用完整 History 的 scroll-to-storage-index | 不顺带增加 revert/fork 或搬走 Agent 的 destructive confirmation |

- Catalog 的既有 spec 将补全而非复制为另一个 controller；
  保留 snapshot 联合及命令失败传播，明确新增打开/关闭端口与 Delete child
  生命周期。所需 local interaction token 不是 backend request/account 协议。
- Index 当前 row/hover 的异步 Loading/Ready/Failed 在 renderer 内；
  拟移到组件 row/detail handle 或状态机，spec 定义 acquire/release、准确目标
  和迟到抑制。像素 anchor、左右定位、滚动、focus 和 hover grace 留 renderer。
- 两侧栏复用准确 Agent 的 index authority，但 renderer interaction 实例
  不得互相串目标；不要用完整 Agent/Application 当组件 service locator。
- 普通失败、取消、closed no-op/异常沿基线区分；发现行为缺陷另列评审，
  不把新重试、全局锁、写入排空或不可编辑限制藏在结构迁移里。

## Work lines and shared-file ownership

- 配置线可分为三个独立 component 树；浏览线 Catalog 和 Index 的算法/
  生命周期较复杂，各自作为完整工作单元，不同时承担多个复杂根模块。
- 组件负责人只改各自新树、专有测试与子任务交接；整合负责人独占既有
  shared contracts、SettingsPopup、SessionTreeCliScreen、SessionSidebar、
  RuntimeStatusBar、AgentRuntimeScreen、Application/RPC/Agent/Draft 接线、
  既有测试 fixture、全仓 Gradle 消费方及 checklist。
- 先固定父级稳定 child、单一声明归属与 dependency port，再并行实现；
  不由多个 Session 同时删共享定义或改同一宿主。业务 child 的创建与关闭
  必须在具体父级所有权里明确，不能为每个 renderer frame 重建。
- 纯样式可放现有 `app/view/components`；抽共享选择器必须有实际消费者，
  不制造一个含业务默认值/菜单能力规则的通用“configuration core”。
- 一个构建协调者，显式复用可用 Daemon JVM、单 worker；不通过多个
  Daemon、争用用户 IDE/设备或同时构建变动中的树增加并行度。

## Validation and delivery

- 五组件分别提供 fake-dependency VM 测试与完整 renderer 测试；
  Catalog 复用现有 spec/VM 测试并增加 UI/interaction；Index 保留扫描算法测试，
  增加 row/detail 生命周期与两侧栏宿主测试。
- 集中跑 Settings/RPC/Application/Agent/Draft、前批组件回归：
  Session close cancels retry、defaults close drains、组合更新不丢无关字段、
  exact rename/cwd/delete、Catalog mutation-success/reload-failure、
  Index cacheNonce 更换及 delayed callback/scroll routing。
- 编译 RPC server/client、Integration 测试源码及直接消费方；
  Native/JS/CLI/IDE/真实网络若未验证必须明确记录，不能用 JVM 替代。
- 检查 15 个组件项目及 conventional plugin、单一声明、旧宿主业务代码
  清理、无反向依赖、KDoc/链接与内外层 diff。独立测试不替代宿主接线证明。
- 五组件设计已获用户审阅与实施授权；实现后统一验收，
  提交/推送仍需独立授权。
- 规范：[Frontend boundary](../../checklist/frontend-application-boundary.md)、
  [Module boundaries](../../checklist/spec-impl-module-boundaries.md)、
  [RPC Settings](../../checklist/rpc-settings.md)、
  [RPC Timeline](../../checklist/rpc-timeline.md)、
  [RPC Session](../../checklist/rpc-session.md)。

## Implementation checkpoint

- 用户已确认四条实现 Session：Session/Defaults、Runtime Configuration、
  Catalog、History Index。各线独占对应 component 树；现会话独占所有
  既有 shared declarations、宿主、adapter、测试迁移、Gradle 和集中构建。
- 当前未发现可用 IDEA 诊断工具，不操作用户的研究用 IDE/窗口。
  构建前重新核对 JVM 与共享工作树活动。
- History/Index 共用的 completed-input/suggestion read-only row、有限宽度
  wrapping 与 elapsed rounding 移到现有 `app/view/components`，保留 FQCN
  和单一投影，不让新 Index view 反向依赖完整 History UI。
- 五组件实现已交接；新项目已接入 Agent/Draft、Settings、Application 与
  两侧 Index 宿主。15 项目及直接宿主 JVM 主源码、RPC server/client 与
  Integration 测试源码已编译通过；最终集中回归通过，见下面的验收记录。
  其他用户任务与共享 IDE 未触碰。

## Central validation and acceptance checkpoint

- 本批完成 13 个新增项目、两个已有 Catalog 项目的扩展：五组件合计 15 个
  spec/ViewModel/View 项目；已移除旧共享声明与宿主内对应业务 renderer。
- 统一命令选择 60 个组件与 16 个宿主 `jvmTest` 目标，并编译 RPC
  server/client、Integration 测试源码；最终 Gradle `BUILD SUCCESSFUL`。
  排除 `NO-SOURCE` 目标的遗留报告后，55 个有测试的项目共 **975 项通过，
  0 失败、0 跳过**。缓存有效的先前通过结果与本轮重跑结果共同纳入统计。
- 五目标组件共 155 项测试：

  | 组件 | spec/VM/view 合计 |
  | --- | ---: |
  | Session Settings | 25 |
  | New Session Defaults | 14 |
  | Runtime Configuration | 19 |
  | Session Catalog | 52 |
  | History Index | 45 |

- 宿主证明包括准确 Agent/Draft 的原子三字段更新与无关字段保持、
  Session cancel-on-close/defaults accepted-write drain、VM-owned Rename/CWD、
  Catalog 打开后准确关闭/旧 popup 替换、Delete false/reload failure、
  Index 原 nonce、稀疏存储扫描、两侧栏及 Check out 仅滚动而不改后端。
- 整合修正：
  - Catalog 成功导航后的终止 dismissal 移出其可取消等待，避免 child
    自关闭把成功导航误报为取消。
  - 通用菜单默认在 action 前 dismiss；Catalog 准入需保留准确 menu handle。
    新增默认不变的 `dismissOnClick=false` 选项，由 Catalog 先校验/消费自己的
    handle 再关闭；共用菜单与完整目录流程均有回归。
  - Runtime renderer 按准确 child 设 key；替换时结束旧观察、菜单和调用等待，
    不关闭借用 child，也不把旧命令发给新目标。
- 验证修正而非产品改动：异步测试等待实际结果/帧、菜单标签忽略对齐空格、
  coroutine debug recovery 检查原始 cause；真实存储扫描使用 real-time 测试。
  History 离线录制 fixture 可显式使用同一个单线程 frontend dispatcher，
  后端仍在 Default，避免虚拟时间跑过真实 I/O 或并发 Compose layout；
  不跳过录制测试，不重试 fork/consume 等业务动作。
- 首轮既有 Home cancellation 清理用例曾出现 `RpcClient was cancelled`；
  后续定向重跑及最终完整命令通过。不将一次通过视作穷尽该既有竞态。
- 静态复核：15 个 conventional plugin、唯一 spec 声明、组件 spec/VM
  无 renderer 反向依赖、旧状态 owner 清理、66 个本地文档链接、两仓库
  `diff --check` 通过。
- 仍是分阶段图：Defaults 借用 `app-shared-settings-contract`，其现存
  `utils-shell-client-impl` 依赖未在本批解决；Index 借用 Storage spec，
  经 clean-models/Unified Exec 仍有同类 shell 混合闭包。没有宣称全仓 spec-only。
- 未运行 Native/JS、真实网络 Integration 测试、CLI smoke 或 IDE sync；
  JVM 与 Integration 源码编译不替代这些验证。
- 共享 Daemon 使用其现有 JDK 26.0.2，单 worker；其他任务、IDE 和用户文件
  未改。用户已验收；内层代码已提交为 `721ac0fe`，主任务及五份子任务随后
  归档到 done，外层记录与子模块指针分批提交并推送。
