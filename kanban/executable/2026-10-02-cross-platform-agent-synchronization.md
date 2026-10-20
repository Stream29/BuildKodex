# Task Tree

- `核对参数入口历史投影与测试框架`()
- `按用户决定制定一次性旧历史迁移`()
- `确认实施授权及 0.5.2 激活版本`()
- `coroutineScope {`
  - `实施 P1 版本化旧历史迁移`()
  - `实施 P2 严格参数结构与十分钟上限`()
  - `实施 P3 长等待与真实进程回归`()
- `}`
- `实施 P4 集成验证与交付检查`()
- **`等待用户验收源码与未测平台范围`()**

# Details

## 目标、阶段与授权

- 2026-10-20 / Session989：用户已审阅计划并授权指挥 subagent 完成实施；随后明确同意迁移目标、应用源码版本及业务协议 checkpoint 使用 `0.5.2`。只改源码并做隔离验证，不提交、发布、安装、重启后端或迁移真实 Home。
- 目标与参数规则以 [Shell Client 等待参数调整](../../checklist/shell-client.md#等待参数调整)为准：复用系统锁，两个工具的 `yield_time_ms` 均为模型调用必填，显式等待上限均为 10 分钟。
- 用户要求旧历史一次迁移，保持新数据结构干净；本次用版本化转换替代此前“保留 DTO 默认值及额外缺参检查”的方案，未授权也未执行真实数据迁移。
- 保留原文件名作为任务身份，但不再规划统一锁抽象、Future、任务 ticket、跨 Agent 结果同步或跨设备协调；不新增数据库、云端、subagent 交付前置。
- P1–P3 按不重叠的文件所有权并行委派，统一集成后串行验证；不得并行修改共享 DTO/投影或启动构建。P4 由主代理集成负责，所有批次通过前不宣称整体交付。
- 子任务已交接并归档：[P1 历史迁移](../done/2026-10-20-terminal-wait-history-migration.md)、[P2 参数契约](../done/2026-10-20-terminal-wait-contract.md)、[P3 等待回归](../done/2026-10-20-terminal-wait-regression.md)。主任务保留 executable，等待用户验收，不将未测平台当作已通过。

## 交付结果与验证

- 源码实现完成：必填参数、三条等待路径的十分钟上限、严格 DTO、历史补参迁移、展开参数显示及 `0.5.2` 源码/业务协议激活一同交付。没有新锁系统、旧格式运行期分支或生产等待算法重写。
- MacBook JVM 共 674 例，673 通过、1 个既有失败；Linux JVM 新增真实 smoke 5 例全部通过，其中包含持锁时等待、释放后同次调用完成，以及取消等待者不影响 holder 的 2 例 flock。

| MacBook JVM 验证组 | 用例数 | 结果 |
|---|---:|---|
| Home migration | 128 | 全通过，新转换/当前模型集成 35 例；沿用原有历史测试排除规则 |
| clean-models / AgentState | 30 / 67 | 全通过 |
| unified-exec | 40 | 全通过，含虚拟十分钟边界及真实管道 |
| tool / compact runtime | 12 / 34 | 全通过 |
| history view / viewmodel / sidebar | 96 / 62 / 24 | 全通过 |
| app RPC | 107 | 全通过 |
| backend daemon / 协议 / Session host | 17 / 2 / 22 | 全通过，两个过滤 suite 单独执行且非零 |
| RPC models | 33 | 32 通过；设置字段断言在原始 HEAD 同样失败，shell 模型 3 例通过 |

- `:app-cli:compileKotlinJvm`、`:app-view:compileTestKotlinJvm` 通过；不是 Native 产物或实际 CLI 验收。迁移未触及旧版本冻结源码/fixtures，未运行真实 Home 迁移或十分钟 sleep。
- **未验证**：Mac Native 的联网依赖下载持续阻塞，取消后离线确认缺少 `org.jetbrains.kotlin:kotlin-native-prebuilt:2.4.20`；JS 离线确认缺少 Kotlin `2.4.20`、kotlinx-io `0.9.0`、serialization `1.11.0` 的 klib。Native/JS 未完成编译或测试；Windows 无实机证据，平台描述与归一化仅有参数化 JVM 单测。
- 主任务等待用户验收上述实现及验证边界；没有提交、推送、发布、安装、重启用户后端或操作真实 Home。后续发布须补未测目标，不因本卡源码版本已激活而默认具备发布条件。

## 核对基线与设计落点

- 调查阶段 `Mosaic` 曾有其他会话改动；本轮实施前重新核对产品 HEAD 为 `3bf31daab6ce`、主工作树干净，未覆盖他人文件，也不将源码核对当作运行二进制验证。
- 当前 `readOutput` 已等待 exit 或 yield deadline，首批输出不提前结束窗口；历史修复 `8e5d9a6e5` 保留，不重写底层进程或输出收集算法。
- 实施前上限是 exec/非空 write 30 秒、空 poll 300 秒；exec 默认 10 秒，write DTO 默认 250 毫秒、空 poll 实际调整为 5 秒。
- 新模型调用由 AgentState 的 `toPendingToolEvent` 从原始 JSON 投影，再由 typed handler 执行；pending/stable 历史复用相同 DTO。去掉 DTO 默认值后，既有解码失败路径即可拒绝缺参，无需另加原始 JSON presence gate。
- 实施前历史投影会省略等于默认值的字段；yield 改为无默认值的必需属性后自然编码，不再需要 `@EncodeDefault` 兼容注解。
- 调查时 Home registry 最后目标为 `0.4.15`，应用为已发布的 `0.5.1`、业务协议为 `0.5.0`；本次新增 `0.5.2`，不回填已发布版本。`ShellSessionState.arguments` 同样使用该 DTO，需协调新旧 client/backend 的业务兼容门禁。
- **纠正前轮测试判断**：`TestConfig.testScope(isEnabled = false)` 关闭的是虚拟时间 TestScope，不是禁用测试；不做“先启用测试”的无效修改。该配置中的 `timeout = 30.seconds` 不能当作关闭 TestScope 后的真实 I/O 超时保护，新增 smoke 应显式设置外层预算。

## 本轮实施方案

### 1. 一次迁移后只读新结构

- 新 DTO 的 `yieldTimeMillis: Long` 无默认值，保留字段名及位置；不保留旧格式 decoder、`@EncodeDefault` 或专用 presence helper。缺参、null、坏 JSON 沿现有 `PendingInvalidToolCall` 路径失败，保留调用身份/原始参数，不执行命令或发送 stdin。
- 历史缺失值仅在版本化 migration 中补齐：

  | 记录中的 action | 缺少 `arguments.yield_time_ms` 时补值 |
  |---|---:|
  | `exec_command` | `10000` |
  | `write_stdin`，不论 chars 是否为空 | `250` |

- 这是恢复旧 DTO 的请求值，不是回填实际耗时；不填新的十分钟上限，也不把空 poll 的请求值改成归一化后的 `5000`。已有可表达为 Long 的数值原样保留，包括历史失败调用的负值或超界值，不在迁移中 clamp。
- 仅处理所有当前 root Session（含 archived）中 `work/<index>.json` 的 `type=command_execution_tool_event`，以及 `unstable/<index>.json` 数组内 `type=command_execution` 的 `action.arguments`；先核对冻结的 action 分支形状，不递归搜索任意 JSON 中的同名字段。
- 不修改 `latest.json`、记录索引、timestamp、输出、调用身份、其他 timeline 或无关事件。legacy `subagents/`、未知内容及 invalid/raw 调用的原始参数原样保留；这些不是可自动补参的 typed command 历史。
- 新 migration 使用自己版本目录中的冻结 JSON 规则/字面量，复用 filesystem-layout primitives，不依赖当前 DTO、当前 FileSystemAgentStorage 或运行期 codec。只有缺字段才改写；已完整记录不重写，已迁移与未迁移混合时可重跑。
- 识别为目标 command 但 action/arguments 形状歧义、显式 null、非 Long 值或损坏 JSON 时报告文件路径并阻止打开，不猜测或跳过；取消原样传播。遵循现有原地迁移语义，不新增 backup/staging/journal/跨文件事务；失败保留旧 checkpoint，损坏状态不承诺自动恢复。

### 2. 迁移激活与启动门禁

- 在既有 `KodexHomeMigrations` 增加目标版本 `M = 0.5.2` 的 entry，方法、冻结规则与 fixtures 使用独立 `v0_5_2/` 路径；用户已确认该源码激活版本，不涉及发布。
- migration 在业务 store 打开前由 Home coordinator 持独占租约执行；全部目标记录处理完成后才推进 `version.json`，无需迁移的后续启动不再扫描 Session。历史 registry entries、codec 和 fixtures 均保持不变。
- 严格 DTO、迁移 entry、目标应用源码版本 `0.5.2` 激活必须在同一交付中生效；不能让新 reader 在 M 尚未激活时打开旧 Home。源码激活已获授权，真实 Home 不操作。
- 共享 DTO 的 requiredness 也改变 RPC 消费约束：实施前确认对应业务协议 checkpoint 的调整，拒绝未满足新结构的旧 client/backend 组合；沿用现有版本握手，不新增 RPC 方法或运行期补参。管理协议不随此变更，不重启用户在用的后端。

### 3. 等待范围与生命周期保持

- 保留两个已有最大值常量的名称，把值都改为 `600_000L`；exec、非空 write、空 poll 三条归一化路径全部覆盖。
- 不改变既有下限：非 Windows exec 与非空 write 为 250 毫秒，Windows exec 为 10000 毫秒，空 poll 为 5000 毫秒。负值仍失败，非负但超出有效范围继续按现有方式 clamp，不新增一套超界异常政策。
- 更新 required 列表、平台描述和 KDoc：明确 agent 每次按预计耗时填写，不提示省略参数，也不要求一律填最大值；写入 stdin 与空 poll 均适用。
- 命令完成就返回；yield 到期只返回 session 和当前输出，不杀进程。外部取消仍沿现有 `cancelSession` 路径关闭并注销本次命令，不引入“仅取消等待但保留进程”的新模式。
- 长工具等待仍属于当前 runningTurn，依赖既有 Session/Daemon 周期保活；不添加资源保活真源或放大 RPC HTTP timeout。前端断线、关闭 tab 与显式取消保持既有区别。

## 分批改动与通过条件

以下路径相对 `Kodex/`；P1/P2 必须作为同一次迁移/reader 切换交付，P3/P4 补验证与激活检查。每批先写失败用例，再改最小实现。

### P1：版本化历史转换与冻结 fixtures

- M 与业务 checkpoint 已确认均为 `0.5.2`；`app/migration/.../KodexHomeMigrations.kt` 新增唯一递增 entry，`cliHostMain/.../v0_5_2/` 新增 migration，`cliHostTest/.../v0_5_2/` 新增测试。
- 准备独立合成 fixtures：work 两种 action、unstable 多事件数组、空数组、缺失/已有 yield、已迁移混合、显式 null/坏值、损坏记录；旧 fixture 不用新 DTO serializer 生成。
- 测试只补目标字段、已有值和未知字段保留、无关文件字节不变、`latest.json`/timestamp/index 不变、archived Session 同样处理、legacy subagents 不遍历。
- 覆盖重跑无额外改写、处理部分文件后失败再调用、取消传播、坏数据阻止打开且 checkpoint 不前进；不伪造中断损坏后的自动恢复保证。
- 本批通过：raw JSON 达到统一目标形状；空 Home/空 timeline 可处理，不扩大扫描范围，不依赖当前业务模型。

### P2：严格 DTO、schema 与等待上限

- `tool/unified-exec/contract/.../UnifiedExecModels.kt`：移除两个 yield 默认值及只供其使用的旧等待默认常量；DTO 普通解码缺参即失败，无兼容注解。迁移历史默认值只以字面量留在新 migration 的冻结目录。
- 全仓检索 `ExecCommandArguments`/`WriteStdinArguments` 构造点，逐个补显式等待值，覆盖 runtime、RPC、history、sidebar 和 docs recording 等 fixture；不修改已发布 migration/fixture，不机械替换无关工具。
- `agent-state/core/.../PendingToolEventProjectionTest.kt`：验证现有解码失败路径自然拒绝 exec/write 缺参、null、坏值并保留原始调用；显式 `10000`/`250`/`600000` 往返成功，不新增投影生产分支。
- `agent-storage/clean-models` 测试从 P1 迁移后的样本验证当前 reader、pending 恢复及 FunctionCall 往返；未迁移缺参记录由严格 reader 拒绝，不再测试运行期自动补参。
- `tool/unified-exec/contract/.../UnifiedExecLimits.kt`：两个上限均为 `600_000L`，保留原常量名与下限。
- `tool/unified-exec/.../UnifiedExecSchemas.kt`：两个 `yield_time_ms` 均 `required = true`；描述覆盖空/非空 stdin，不扩大其他参数或 `strict` 设置。
- `ExecCommandShellDescription.kt`：更新 POSIX/Windows 描述的有效范围，移除默认/omit 提示；Windows 其他 shell 安全指引保留。
- `UnifiedExecToolClient.kt`：复用原 wait 算法；检查三条路径确实消费新上限。为直接测试，允许将 write 归一化拆成与 exec 对称的 internal 毫秒 helper，不公开 API 或新增生产可注入时钟。
- `UnifiedExecToolsTest.kt`：校验 required 列表与精确描述；参数化覆盖 exec 各平台、write 空/非空的 `0`、下限、`30000`、`300000`、`600000`、`600001`、`Long.MAX_VALUE`；负值调用保持失败。
- `app/component/history/view/.../BuiltInToolEventView.kt`：去掉按旧默认值隐藏 yield 的条件和相关 import，展开参数始终显示已记录的显式 yield；只改参数行，沿用现有展示布局与状态语义。
- 本批通过：`600000` 原样生效、不被旧 30/300 秒上限截断；所有下限及 clamp 行为不变，描述与实际一致。

### P3：长等待、提前结束与真实锁等待回归

- 在 unified-exec 新增独立等待测试文件，沿 internal 构造器注入测试用 `ShellClient`/`ProcessSession`，配合可控 exit Deferred、stdout 缓冲和 stdin Channel；不改生产进程接口。
- `tool/unified-exec/build.gradle.kts` 的 commonTest 增加已有 catalog 的 `kotlinx.coroutines.test`；测试显式使用单一 `runTest` 调度器，避免嵌套 TestBalloon 的虚拟 TestScope。
- fake 的 client/session 使用独立子 Job 与同一测试调度器，close 不得取消整个测试；每例 finally 释放 fake scopes。虚拟时间覆盖 exec、空 poll、非空 write：到 `599999` 毫秒仍挂起，`600000` 到期返回 session；在 1 秒或 40 秒/310 秒完成则直接返回 exit，40/310 秒的样本只需一次对应工具调用、不产生中间 yield。
- stdout 分批到达但 exit 未完成时不提前返回；yield 后的后续调用只领取剩余输出，最终退出码不丢失、session 最终被移除；取消关闭/注销命令，普通 yield 不关闭。
- 补少量明确的 RealTime 管道 smoke：快速 `echo` 配 10 分钟窗口应迅速完成、分批输出等待至 exit、短窗口到期后可续等；每例用 `withTimeout(30.seconds)` 包住操作并在 finally 清理，复用原真实 I/O 回归，不全局禁用测试或把超时一律放到 10 分钟。
- Linux 锁 smoke 使用临时私有目录与受控锁持有者，先以就绪信号确认锁已取得：等待者发出 `flock` 包裹命令并显式填长窗口，确认持有期间不执行，释放后同次调用拿到 exit；另测取消等待者不终止持有者。不访问共享锁或真实 Home；无 `flock` 时明确记未验证，Windows/macOS 不假定存在同一 CLI。
- 本批通过：十分钟边界靠虚拟时间验证，不运行十分钟 sleep；真实 smoke 有独立外层预算、清理 finally 与无残留检查。

### P4：runtime、保活、跨目标与收尾

- `app/migration` 在版本冻结目录外新增当前模型集成测试，cliHostTest 按需补 clean-models/JSON codec 测试依赖：从旧版本逐 entry 升到 M 后用新 reader 读 work/unstable，恢复 pending；覆盖跨多个历史版本直接升级、future entry 未激活不运行、已到 M 启动不扫描、独占租约冲突与升级失败 checkpoint。P1 的版本 fixture/转换测试保持独立，不绑定当前模型。
- reader 切换与版本激活成组检查：确认 M entry 在目标应用版本可选中，普通 store 只在迁移成功后打开；版本/发布操作需明确授权，不把 future registry entry 已登记当作已完成生产激活。
- `rpc/models/.../ShellSessionModelsTest.kt` 去掉“缺参仍使用默认值”的旧断言；验证 required args 往返及缺参失败。`BackendEndpoint.kt`/`BackendProtocolCompatibilityTest.kt` 按确认后的 checkpoint 检查新旧组合的握手拒绝，不停止或替换旧后端。
- `agent-runtime/decorator/tool/.../KodexToolRuntimeTest.kt`：合法 raw `write_stdin` fixture 补 yield；新增两种缺参调用不进入普通 handler、产生 `StableInvalidToolCall` 并继续 runtime 的检查。
- `rpc/server/.../BackendSessionHostTest.kt`：沿既有 runningTurn 周期续期测试推进到 600 秒量级，验证无前端续期/静默等待期间不被 TTL 卸载；mock 受控挂起工作，不调用真实模型，不改生产续期政策。
- history 展开参数测试同步检查旧默认数值也显示 yield；其余受 DTO 构造变化影响的模块先编译、再运行相应用例，不借编译需要变更功能。
- 按下述命令分组验证，记录实际通过/失败/跳过数量、失败是否基线已有、未测平台和产物来源；不能把编译等同于真实平台运行。
- 所有 scoped 测试与 smoke 通过后，更新 checklist 的待实施状态与本卡结果，向用户交接验收范围和未测项；后续归档不替代必要验收，不自动提交或发布。

## 验证命令与资源门槛

- 实施开始前加载 IDE/Gradle 与相应设备技能，检查运行中的 Gradle Daemon JVM、设备资源与他人工作树；复用其明确 JVM，没有可复用 Daemon 就报告构建阻塞，不另起竞争环境。
- 以下为实施验证命令模板；在 `Kodex/` 执行，`<daemon-java-home>` 必须替换为当时已核验的 JVM。执行前清除或解释继承的 `TESTBALLOON_INCLUDE_PATTERNS`/`TESTBALLOON_EXCLUDE_PATTERNS`，防止过滤导致零用例假通过。

```bash
# P1：迁移及 checkpoint/失败重跑
./gradlew -Dorg.gradle.java.home=<daemon-java-home> :app-migration:jvmTest

# P2：新 reader / 新工具参数
./gradlew -Dorg.gradle.java.home=<daemon-java-home> \
  :agent-storage-clean-models:jvmTest :agent-state-core:jvmTest

# P2 / P3：工具参数与等待行为
./gradlew -Dorg.gradle.java.home=<daemon-java-home> :tool-unified-exec:jvmTest

# P4：共享模型与参数展示，不带协议 suite 过滤
./gradlew -Dorg.gradle.java.home=<daemon-java-home> \
  :rpc-models:jvmTest :app-component-history-view:jvmTest
./gradlew -Dorg.gradle.java.home=<daemon-java-home> \
  :app-backend-host:jvmTest --tests '*backendProtocolCompatibilityTest*'

# P4：runtime 与 host 分次运行，避免 host 过滤排除 runtime 用例
./gradlew -Dorg.gradle.java.home=<daemon-java-home> \
  :agent-runtime-decorator-tool:jvmTest
./gradlew -Dorg.gradle.java.home=<daemon-java-home> \
  :rpc-server:jvmTest --tests '*backendSessionHostTest*'
```

- 带 `--tests` 的目标单独运行，协议过滤不与 rpc-models/history 组合，host 过滤不与 runtime 组合；检查每组 XML 的实际 suite/用例数量，不能只看 Gradle exit code。发现无关基线失败先记录，不顺带修复。
- 在可用 Linux 主机补相关模块 `linuxX64Test` 与 `:app-cli:compileKotlinLinuxX64`；macOS/Windows/JS 的描述与归一化先参数化单测，Native/JS 编译和运行按实际可用目标补证据，不因本卡接管设备或修改全局工具链。
- 不设置 10 分钟真实时钟测试预算作为全套默认；资源不可用的目标列为未验证，由用户决定后续验收，不默认为通过。

## 执行记录

- P1/P2/P3 分别委派到 agy conversation `dc64e3b7-d22d-4480-a506-65cb3cc0a2ac`、`94138b21-9cc8-4017-9bdf-933929a5c48d`、`679a8306-d344-49d9-abd0-c5b447fa31ee`。首轮模型额度耗尽后切换 `gemini-4-argon-high` 续做同一会话，不能将工具退出视为实现完成。
- 本机资源调查时仅约 1.5 GiB 可用内存，不启动本地构建。用户另行明确允许在无现存 Daemon 的 MacBook 建立临时独立验证目录及受限 Daemon，这是本任务构建资源门槛的授权例外。
- 远端目录 `~/ACodeSpace/local/kodex-terminal-wait-989/source/` 为无 Git 的源码快照，不修改用户现有 checkout；单 worker、禁止并行，JVM 使用已缓存的 JBR 25，heap 4 GiB。初始共享 registry 的 Daemon 后被其他项目接用，不接管或终止该资源；后改为任务私有 `GRADLE_USER_HOME`/Daemon registry，仅复用标准依赖缓存，并使用 in-process Kotlin 编译，后续验证复用私有 Daemon。
- 远端 `help` 配置检查通过（54 秒）；这不是产品测试或编译通过。最终验证前已重新同步实现快照。
- 主代理接管最终审阅与集成：修正两个测试清理 receiver、迁移对未知字段的过度拒绝，以及把可被标准 Long decoder 接受的数字字符串误列为失败样本的测试假设；为等待回归补有界且保留原始错误的清理。已核对 shell 字符串转义本身正确，未做不必要改写。
- `rpc-models` 全组 33 例中 1 个未改动的 `settingsModelsTest` 失败：旧断言未包含现存 `hooks`、`daemonTtlSeconds`。本次 shell 模型 3 例通过；原始 HEAD 隔离测试同样 4 例中 1 例失败，错误一致，不顺带修改设置任务。
- Linux 使用 MacBook 导出的同一 JVM classpath、512 MiB 独立 JVM 运行新增真实 smoke，避免本机 Gradle；首轮取消等待者用例暴露测试未等待异步 close 清理的假设。按现有 `ProcessSession` 契约增加 owner Job join 后再释放持锁者，重跑 5 例全通过（3.46 秒），不改变生产取消语义。
- 交接前以 checksum 核对所有改动/新增源码与已验证远端快照内容一致，确认生成的应用版本为 `0.5.2`。文档只读审阅 agy `ef142900-1fec-41e9-91fa-b6e07bb46ff8` 返回 PASS，随后对状态、历史验收范围及资源记录的小幅调整由主代理复核。
- 私有 Daemon `61481` 已通过私有 registry 停止；远端源码/基线/导出/日志/私有 Gradle Home、构建占位目录及本机测试导出/临时报告已清理，未发现本任务的 flock holder/waiter 或临时锁目录残留。用户的共享 Daemon、原 checkout 和标准依赖缓存未清理或接管。
- 文档收尾检查：60 个本地链接/源码行号有效，无新链接错误、旧 phase 引用或同名跨阶段重复卡；1 个原有 Daemon 已移除 worktree 链接在文档 HEAD 中亦存在，未越界修复。两仓 whitespace 检查通过，Draft 未改。

## 源码入口与边界参考

- [新调用投影](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/core/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/tool/PendingToolEventProjection.kt#L51-L76)；[既有 invalid 完成路径](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/decorator/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/KodexToolRuntimeImpl.kt#L48-L60)。
- [严格等待参数 DTO](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/unified-exec/contract/src/specMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecModels.kt#L27-L62)；[迁移历史默认值](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/src/cliHostMain/kotlin/io/github/stream29/kodex/app/migration/v0_5_2/MigrateToV0_5_2.kt#L263-L264)；[pending 投影 Json](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/clean-models/src/specMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/unstable/PendingToolHistory.kt#L10-L12)；[stable 投影 Json](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/clean-models/src/specMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableToolHistory.kt#L12)。
- [Home 迁移协议与冻结规则](../../checklist/kodex-home.md)、[当前 registry](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/src/cliHostMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigrations.kt#L10-L40)、[历史原地转换示例](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/src/cliHostMain/kotlin/io/github/stream29/kodex/app/migration/v0_4_5/MigrateToV0_4_5.kt#L19-L64)。
- [ShellSessionState 共享参数](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/models/src/specMain/kotlin/io/github/stream29/kodex/rpc/models/ShellSessionState.kt#L15)、[当前业务协议 checkpoint](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/backend-daemon/src/specMain/kotlin/io/github/stream29/kodex/backend/BackendEndpoint.kt#L6)、[历史展开参数](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/BuiltInToolEventView.kt#L52-L64)。
- [等待实现与参数归一化](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolClient.kt#L228-L294)；[真实 I/O TestScope 配置](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/unified-exec/src/commonTest/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolsTest.kt#L58-L60)。
- [运行 turn 与取消清理的保活回归](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHostTest.kt#L324-L368)。
- [Tool handler 约束](../../checklist/tool-handler-decisions.md)、[RPC 契约冻结](../../checklist/rpc-architecture.md#rpc-契约冻结)、[Session 保活](../../checklist/rpc-session.md#保活与回收)、[协程资源生命周期](../../checklist/coroutine-resource-lifecycle.md)。
