# Task Tree

- `Read frozen rules and inventory all backend projects`()
- `Migrate contracts and consolidate the seven authorized units`()
- `Check source bytes, source sets, dependencies and scoped changes`()
- `Report source completion and hand off integration validation`()
- **`Await repository acceptance of the migration checkpoint`()**

# Details

## 协调者整合回填

- settings/全仓消费者已切换，完整模型、生产图和实际 JVM/JS/Linux main 编译
  已验证。测试与干净基线缺口见 [主任务](2026-10-12-discuss-kmp-spec-impl-source-sets.md#L87)。
- 下文静态计数、旧 ID 交接要求是本线交付时记录，不再表示全仓未整合；
  本线没有自行运行重型验证。保留 executable 等待整仓人工验收。

## 授权与边界

- 用户明确授权全仓迁移执行；本子任务独占后端五域：
  `Kodex/agent-context`、`agent-runtime`、`agent-session`、`agent-state`、
  `agent-storage`，另仅维护本文。
- 主任务：[全仓源集迁移](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-12-discuss-kmp-spec-impl-source-sets.md#L1)。
- 冻结规则：[KMP 源集设计](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/kmp-spec-impl-source-sets.md#L1)。
- 参考：[后端静态审查](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-12-audit-kmp-spec-backend.md#L1)；
  其旧静态基线及“保留旧源集”建议不替代本轮授权和冻结规则。
- 实施 HEAD：`450e3bd7eeef460c7293d6f6fedb55cc1a4242c5`。
  开始已有主 Session 试点，尤其 session filesystem、storage in-memory
  已依赖 `:utils-read-write-mutex`；保留这些修改，不回退。
- 不修改 settings、根、build-logic、其他域消费者或全局 checklist。
  协调者统一旧 ID → 新 ID；本轮暂留跨项目旧 ID，仅删除合并自边。
- 不运行 Gradle、编译、IDE、远程重型操作；不提交、不创建 Session。
  本文留在 executable，源改动完成不等于验收或 done。

## 逐旧项目 → 新项目映射

路径相对 `Kodex/`；ID 为 `:` 加路径各段用 `-` 连接。
“同路径”表示项目/ID 保留，不表示契约仍放 commonMain。
覆盖全部 33 个旧项目、全部 16 个旧 spec；7 个合并单元静态减少 10 个项目。

| 旧项目 | 新项目 | 源集/职责 |
| --- | --- | --- |
| `agent-context/spec/agents-md` | `agent-context/agents-md` | 模型、warning sealed 层次 → specMain |
| `agent-context/impl/agents-md-filesystem` | `agent-context/agents-md` | 加载实现和 commonTest → 同名源集 |
| `agent-context/spec/available-skill` | 同路径 | 共享元数据 → specMain |
| `agent-context/spec/contract` | 同路径 | 设置接口、序列化模型及有效性约束 → specMain |
| `agent-context/spec/prefix` | `agent-context/prefix` | prefix/session 元数据 → specMain |
| `agent-context/impl/prefix` | `agent-context/prefix` | resolver 和 commonTest → 同名源集 |
| `agent-context/impl/prefix-render` | `agent-context/prefix` | renderer 和 commonTest → 同名源集 |
| `agent-context/spec/prompt-dsl` | 同路径 | DSL marker → specMain；builder、转义与构造函数 → commonMain |
| `agent-context/spec/skill` | `agent-context/skill` | catalog/read 接口、模型与 sealed result → specMain |
| `agent-context/impl/skill-filesystem` | `agent-context/skill` | filesystem resolver 和 commonTest → 同名源集 |
| `agent-context/impl/skill-render` | `agent-context/skill` | renderer 和 commonTest → 同名源集 |
| `agent-runtime/spec/contract` | 同路径 | runtime 根契约与异常 → specMain |
| `agent-runtime/impl/composition` | 同路径 | 组合实现 → 原源集 |
| `agent-runtime/spec/decorator/compact` | `agent-runtime/decorator/compact` | decorator 接口、retry 异常 → specMain |
| `agent-runtime/impl/decorator/compact` | `agent-runtime/decorator/compact` | runtime/retry policy、commonTest → 同名源集 |
| `agent-runtime/spec/decorator/tool` | `agent-runtime/decorator/tool` | decorator 接口 → specMain |
| `agent-runtime/impl/decorator/tool` | `agent-runtime/decorator/tool` | runtime/routing、commonTest → 同名源集 |
| `agent-runtime/spec/steer` | `agent-runtime/steer` | 等待/claim 契约 → specMain |
| `agent-runtime/impl/steer` | `agent-runtime/steer` | StateFlow 实现、commonTest → 同名源集 |
| `agent-session/spec/contract` | 同路径 | 根接口、依赖所有权及 close 默认行为 → specMain |
| `agent-session/impl/filesystem` | 同路径 | session/cache/lease 实现 → 原源集 |
| `agent-session/impl/in-memory` | 同路径 | 可生产消费的 session 实现 → 原源集 |
| `agent-session/test` | 同路径 | 可消费的 main fixture → commonMain，不并进 commonTest |
| `agent-state/spec/context-window` | 同路径 | token-budget 计算 helper → commonMain；创建真实 specMain，不造接口 |
| `agent-state/spec/contract` | 同路径 | 状态/异常/同步契约默认行为 → specMain；命令扩展 → commonMain |
| `agent-state/impl/state` | 同路径 | 状态主实现 → 原源集 |
| `agent-state/test` | 同路径 | 可消费的 main fixture → commonMain |
| `agent-storage/spec/clean-models` | 同路径 | 模型、sealed unions、内在 projection/codec → specMain；失败事件转换 helper → commonMain |
| `agent-storage/spec/contract` | `agent-storage/core` | timeline 接口、模型及通用默认操作 → specMain |
| `agent-storage/impl/filesystem` | `agent-storage/core` | filesystem/StorageUri expect → commonMain；四 actual 保持平台源集；测试合入 commonTest |
| `agent-storage/impl/in-memory` | `agent-storage/core` | in-memory/锁实现、commonTest → 同名源集 |
| `agent-storage/spec/contract-ext` | 同路径 | 初始化、projection、compaction/retention 算法 → commonMain；创建真实 specMain，不造接口 |
| `agent-storage/impl/filesystem-layout` | 同路径 | 历史布局实现及测试 → 原源集，不并入 core |

## 分类依据与接线

- 不以旧 spec 目录、公开声明或“有函数体”为唯一依据。
  契约默认行为可以有函数体；算法 helper 不因此自动成为契约。
- prompt-dsl 的 marker 是 DSL 使用契约；builder 和 escape 算法属于实现。
  同文件的 builder、internal constructor/build、private helper 一起留
  commonMain；只抽出独立 marker，保持原 PromptXmlKt 文件门面。
- context-window 与 contract-ext 没有独立的类型/接口声明，只有计算/存储
  编排 helper；不为填充 specMain 改 API。仍 opt in 真实 specMain，
  commonMain → specMain；这两项目的业务 specMain 为空，实现依赖不上移。
- clean-models 的模型履行 `toResponseHistoryItems` 契约，所需
  StableToolHistory/PendingToolHistory/InvalidToolInvocation helper 随模型
  同入 specMain；不能把模型必需的 codec 放下游。
  `PendingToolEventFailure.kt` 是独立的失败转换实现，留 commonMain。
- state 的 `KodexAgentStateExtensions.kt` 调用上述失败转换实现，因此一起
  留 commonMain；state specMain 不反向依赖它。
  历史 `impl` package 的 InvalidTransitionException 仍是契约，移 specMain。
- storage 通用 timeline 默认遍历/append/revert 只调用抽象接口，
  与接口的语义约束同入 specMain；具体 IO、JSON codec、mutex、
  StorageUri expect/actual 不上移。
- 合并项目取插件并集，先应用 `kodex.kmp-spec-impl` 再应用
  `kodex.kmp-host`；保留 serialization/TestBalloon。
  spec 依赖使用 `named("specMain")`；实现/test 依赖保持原层与 api/implementation。
- 同配置重复依赖去重；继承可见的 api 与原 implementation 不借迁移改作用域。
  合并单元内部旧 spec/impl 自边删除；其他旧 ID 暂留给协调者统一重写。
- 所有项目保留 Host convention；core 保留 filesystem 的显式 posix 模板、
  file OptIn 与 import；不裁剪 targets，不改 compilerOptions/interop/测试关联。

## 风险与待真实验证

- 本子线不修改 settings 和全仓消费者 ID；协调者统一整合前，
  工作区中间状态不能直接作为可构建产品或发布图。
- runtime/session/state 根 contract 不并 composition/主实现，避免审查 H1–H3
  的生产闭环。clean-models 所依赖的三种工具独立契约仍由协调者保证。
- 合并项目使原契约消费者获得更大的实现依赖闭包；
  不承诺 spec-only 隔离、发布等价或更快增量编译。
- 待验证：实际源集/compilation/variant、specMain metadata、
  serialization 生成符号、TestBalloon 入口、独立消费者与全部原测试。
- 待验证：JVM/JS Node/Native 编译与运行、StorageUri actual/posix、
  持久化 golden/旧 settings、取消清理与借用服务关闭顺序。
- 待协调者整合后重算生产图和测试图、旧 ID 零残留、目标/编译器设置等价；
  IDEA/性能及其他宿主验收不在本轮静态检查范围。

## 本轮静态检查

- [x] 读取 AGENTS、指定技能、Draft、主 executable、冻结 checklist、后端报告。
- [x] 核对 HEAD、既有主 Session 试点及全部 33 个后端构建脚本。
- [x] 核对全部 16 个旧 spec 的真实 specMain/实现职责落点。
- [x] 核对 Kotlin/API/KDoc/测试字节、文件合并无覆盖及平台源码保持。
- [x] 核对插件并集、依赖层次、合并自边、JVM 门面/同包声明风险。
- [x] 核对只修改五域和本文；无构建/提交/IDE/远程/新 Session。
- [x] 源改动完成并准备交接映射、静态证据与风险。
- [ ] 协调者统一注册/消费者并完成真实验证及验收。

### 已获得的源码检查证据

- 使用 Git HEAD blob 与逐旧路径映射对照，而不是依赖 rename 检测或旧 build 输出。
  原 147 个 Kotlin 文件均有唯一落点；98 个仅移动，字节完全一致。
  原文件除 PromptXml.kt 抽出 3 行 marker 及其 1 行 KDoc 外全部逐字节一致。
  marker 新文件含原 package/KDoc/声明，builder 文件余下内容逐字节一致。
- 现有 148 个 Kotlin 文件：commonMain 37、specMain 62、commonTest 45，
  jsMain/jvmMain/mingwX64Main/posixMain 各 1。
  增加的一个文件是 marker 拆分，不增加/复制 API 声明或业务行为。
- 45 个原 commonTest 文件及四份 actual 全部逐字节保留；
  七个合并单元源码/测试目标路径无覆盖。
- 33 个旧 build 脚本对应 23 个现有脚本，16 个旧 spec 全部 opt in；
  七个合并项目插件并集一致，所有插件顺序 spec-impl 在 Host 前。
- 合并项目没有指向其旧输入或新 ID 的自边；
  main/test 各自同包同名 JVM 顶层文件门面候选检查无冲突。
  这是文本检查，不替代 Kotlin 编译器/metadata 验证。
- core 的 file OptIn/import 前言与原 filesystem 完全一致，
  StorageUri expect 位于 commonMain；四份 actual 均在原名平台源集。
- `git diff --check`（仅五域跟踪文件）通过；未读取旧产物作为新迁移证据。
- 逐 33 个旧 build 脚本解析 main/test 的 api/implementation 声明，
  按清单投影、删除合并自边后，与 23 个新脚本各配置集合一致。
  主 Session mutex 试点替换计入起始工作区；无额外库替换/依赖降级。
- common/spec 主编译输入无重复顶层 class/interface/object 声明，
  spec 文件无显式导入同项目 commonMain 声明；specMain 中未发现
  对本轮下移 helper 的符号引用（含 KDoc）。
  这些文本扫描不能证明 Kotlin 名称解析或跨项目契约职责全部成立。
- 仅后端顶点的字面 main 图按新 owner 投影无环；测试边单独保留。
  未把其他域并行修改、插件生成依赖、resolved variant 合入此证据。
- 新/旧源码、build 脚本和本文的空白检查通过，包括未跟踪的新文件。
  最后 HEAD 仍为 `450e3bd7eeef460c7293d6f6fedb55cc1a4242c5`。
- 本轮仅使用 apply_patch 修改五域产品及本文；外域并行修改未接管。
  无临时文件、进程、锁或设备资源需要清理；旧忽略 build 目录不是本轮产物，
  不删除用户/其他 Session 的缓存。

## 协调者交接

- **后端源码改动完成，静态检查通过；未验收，未 done。**
- settings/全仓 build 引用使用上表路径映射，尤其以下 ID 组应整体重定向：

| 新 ID | 全部旧输入 ID |
| --- | --- |
| `:agent-context-agents-md` | `:agent-context-spec-agents-md`、`:agent-context-impl-agents-md-filesystem` |
| `:agent-context-skill` | `:agent-context-spec-skill`、`:agent-context-impl-skill-filesystem`、`:agent-context-impl-skill-render` |
| `:agent-context-prefix` | `:agent-context-spec-prefix`、`:agent-context-impl-prefix`、`:agent-context-impl-prefix-render` |
| `:agent-runtime-decorator-compact` | `:agent-runtime-spec-decorator-compact`、`:agent-runtime-impl-decorator-compact` |
| `:agent-runtime-decorator-tool` | `:agent-runtime-spec-decorator-tool`、`:agent-runtime-impl-decorator-tool` |
| `:agent-runtime-steer` | `:agent-runtime-spec-steer`、`:agent-runtime-impl-steer` |
| `:agent-storage-core` | `:agent-storage-spec-contract`、`:agent-storage-impl-filesystem`、`:agent-storage-impl-in-memory` |

- 本域非自依赖仍保留旧 ID；并行迁移后的 openai/tool/utils 旧 ID 也需协调者
  统一重写并同配置去重。保留的 9 个 spec 项目、7 个其他实现/fixture
  项目保持原路径/ID；不要把“specMain 空业务的 helper-only 项目”当遗漏契约。
- 全仓只读 import 搜索显示，本轮下移的公开 helper 当前由实现/测试消费，
  未发现它们被已迁 specMain 导入；仍需最终整合后重查跨领域契约职责。
- 建议真实验证至少包含 core 双实现全部原测试、clean-models 序列化/golden、
  根契约 metadata、context DSL、runtime decorator/steer、session/state 清理反例，
  以及 RPC/app/tool 的独立消费者；本子线没有执行这些任务。
- 全局 checklist 路径同步由协调者维护；本轮只在本文记录实际源集落点。
