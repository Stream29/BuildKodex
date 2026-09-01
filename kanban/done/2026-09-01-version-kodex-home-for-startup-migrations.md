# Task Tree

- 实现 Kodex Home 版本化与启动迁移
  - [done] 调查现有 Home 与启动流程
    - [done] 确认根目录没有格式版本
    - [done] 确认 Session 没有格式 manifest
    - [done] 定位首个业务数据读取点
    - [done] 核实现有租约与迁移脚本
  - [done] 确定版本和并发协议
    - [done] 直接保存完整应用版本
    - [done] 只登记实际迁移版本
    - [done] 每个目标 release 一个入口
    - [done] 允许 release 前 future 表项
    - [done] 发布后完全冻结 migration
    - [done] 保留多进程并发运行
    - [done] 使用跨进程读写租约
    - [done] 使用原地可重入 migration
    - [done] 不使用 marker 或 journal
  - [done] 建立 filesystem 读写租约
    - [done] 新增独立 host KMP 模块
    - [done] 复用 renewable filesystem lease
    - [done] 实现 guard 和 writer intent
    - [done] 覆盖到期、取消与竞争测试
  - [done] 建立 Home 版本管理
    - [done] 新增 app migration 模块
    - [done] 从 Gradle 生成当前应用版本
    - [done] 建立全局版本迁移表
    - [done] 实现 future entry 版本激活
    - [done] 实现直接 version 存储
    - [done] 实现基线验证和版本登记
    - [done] 实现重入与错误模型
  - [done] 建立 storage directory helpers
    - [done] 新增 agent-storage filesystem-layout 模块
    - [done] 提供无状态 layout helpers
    - [done] 提供 raw record 文件操作
    - [done] 直接复用现有 CoroutineFileSystem
    - [done] 覆盖 layout 与 raw I/O
  - [done] 接入应用启动与关闭
    - [done] 在 logging 和业务读取前准备 Home
    - [done] 让 Application 持有读租约
    - [done] 在完整 shutdown 后释放租约
    - [done] 保留自定义 dataDirectory 测试入口
  - [done] 保持首版为只登记基线
    - [done] 不登记 no-op migration
    - [done] 不改写现有 Session payload
  - [done] 完成验证与维护文档
    - [done] 运行真实临时文件系统测试
    - [done] 运行跨进程读写互斥测试
    - [done] 运行隔离 Home CLI smoke test
    - [done] 接入 release migration gate
    - [done] 运行受影响 Gradle checks
    - [done] 运行编译器检查与 diff 检查

# Details

- 调查与规划已由用户审查通过；本轮开始实施代码，但不迁移本机 `~/.kodex`。
- 已确定的长期规则记录在
  [`checklist/kodex-home.md`](../../checklist/kodex-home.md)。

## 调查结论

- 当前 `~/.kodex` 根目录只有 `auth.yml`、`settings.yml`、`sessions/`、
  `log/` 和 `generated_images/`，没有 version 或 manifest 文件。
- 当前本机有 186 个 root Session；所有 Session 都没有 `manifest.json`。
- `KodexHome` 目前只提供默认路径，不处理 layout：
  `Kodex/utils/kodex-home/src/commonMain/kotlin/io/github/stream29/kodex/utils/kodexhome/KodexHome.kt:6`。
- Home 没有统一 initializer：logging、settings、auth、Session repository 和 generated image
  persistence 分别在首次使用时创建自己的路径。
- 自定义 Application `dataDirectory` 只传给 settings、auth、Session 和 Agent context；file logging
  与 generated image artifacts 仍直接使用默认 `KodexHome`。
- filesystem repository 直接创建 Home 和 `sessions/` 后扫描数字目录，没有版本检查：
  `Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt:422`。
- filesystem AgentStorage 直接绑定六条 timeline，创建时也不写 manifest：
  `Kodex/agent-storage/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemAgentStorage.kt:18`、
  `Kodex/agent-storage/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemAgentStorage.kt:104`。
- `FileSystemAgentStorage` 在构造时绑定当前 `index/work` timeline 集合和当前 serializers；
  `FileSystemIndexVersioned.getUnsafe` 也直接使用当前 serializer decode payload：
  `Kodex/agent-storage/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemAgentStorage.kt:26`、
  `Kodex/agent-storage/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemIndexVersioned.kt:175`。
  因此启动 migration 不能通过当前 AgentStorage API 打开旧 layout 或旧 payload。
- 历史 layout 的六条 timeline 是 `compaction/settings/timestamp/token-count/stable/unstable`；
  当前 layout 是 `index/work/settings/timestamp/token-count/unstable`。Migration 必须向目录
  helpers 显式提供版本布局，不能把当前集合当成全部历史格式。
- 既有 Session 中的 `subagents/` 是遗留数据；当前 filesystem repository 不再创建或读取它，
  普通运行和 migration 必须保留，除非用户明确删除整个 Session。
- 默认 CLI 当前先初始化日志，再打开 Application：
  `Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt:10`。
- Application 首次读取的规范数据是 global settings，之后才构造 auth 和 Session repository：
  `Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt:147`、
  `Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt:216`。
- `settings.yml` 已明确移除独立 schema version，并采用宽松字段兼容：
  `Kodex/app/shared/settings/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/KodexSettingsStore.kt:124`。
  Home 版本不得重新变成 settings schema version。
- 原有 `FileSystemLease` 已提供 renewable heartbeat、正常释放和过期接管，但只支持单一独占 owner；
  最终实现将三种 lease 统一到 `Kodex/utils/filesystem-lease/contract/` 和
  `Kodex/utils/filesystem-lease/impl/`。
- 现有 `ReadWriteMutex` 只在单进程内工作，不能解决多个 CLI 的 filesystem 协调：
  `Kodex/utils/read-write-mutex/src/commonMain/kotlin/io/github/stream29/kodex/utils/ReadWriteMutex.kt:10`。
- 调查时本机同时运行两个 `kodex-cli`，因此不能用一个全生命周期独占 Home 锁取代现有多实例能力。
- 两个现有 Python 脚本都是离线一次性迁移。一个明确声明应用不会调用它，另一个依赖 Linux `/proc`
  检测运行进程；它们不能作为四平台 CLI 的启动迁移框架：
  `Kodex/scripts/migrate-index-metadata-to-settings.py:3`、
  `Kodex/scripts/migrate-compaction-stable-events.py:156`。
- 当前应用版本是 `0.3.2`，由 Gradle convention 和 MCP client 分别写入：
  `Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt:10`、
  `Kodex/buildSrc/src/main/kotlin/kodex.kmp-shared.gradle.kts:12`、
  `Kodex/mcp/impl/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt:228`。
  `app/migration/impl` 必须从模块的 `project.version` 生成当前版本，不再增加第四份手写版本。
- 本轮只读复查时 Home 已有 190 个 root Sessions、1,393,252 个 timeline JSON files，
  总计约 7.5 GB；最大的单个 Session 有 442,476 个 timeline files，另一个 Session
  约 6.0 GB。
- 最大 Session 的 `timestamp`、`work` 和 `unstable` timeline 分别达到约 196K、99K
  和 97K files；migration 文件基建必须按大型单目录设计，不能依赖当前
  `FileSystemAgentStorage` 的业务访问路径。
## 版本模型

- 文件使用 `~/.kodex/version.json`：

  ```json
  "0.3.2"
  ```

- 文件直接保存最后一次成功准备 Home 的完整应用版本，不增加 `formatVersion` envelope。
- Home migration 版本只接受 canonical `major.minor.patch`，并按三个非负整数比较，不按字符串字典序排序。
- 当前应用版本由 Gradle `project.version` 生成给 `app/migration/impl` 使用，不在 migration 源码中手工维护第二份常量。
- 缺失 version 表示最后一个未引入版本机制的 release；当前基线是 `0.3.2`。
- 缺失 version 不直接等同于空 Home：
  - 没有规范数据的新 Home 登记当前应用版本。
  - 已有规范数据的 Home 必须验证每个当前 root Session 的六条 timeline，再执行适用
    migration 并登记当前应用版本。
  - legacy `subagents/` 不属于当前 schema；验证和 migration 必须原样保留。
  - 验证失败时不写 version，直接拒绝启动并报告首个不兼容位置。
- 根 version 是全部 Session 的统一 authority，不恢复早期设计中的 per-Session `manifest.json`。
- 全局迁移表只登记实际改变 Home 数据的目标 release，不为无迁移 release 增加 no-op。
- 从 stored version 升级时执行 `stored < toVersion <= current` 的表项，并按 `MigrationVersion` 顺序运行。
- 即使没有匹配的 migration，也必须在 write lease 下把 version 更新为当前应用版本。
- 未知根文件、用户上下文、legacy `subagents/`、日志和生成产物不参与版本判断。

## Filesystem 租约

- `Kodex/utils/filesystem-lease/contract/` 的 `FileSystemLease` 只组合 `AutoCloseable` 与
  `CoroutineScope`；`Kodex/utils/filesystem-lease/impl/` 在三个文件中提供三种工厂实现。
- `FileSystemLease(...)`、`FileSystemReadLease(...)` 和 `FileSystemWriteLease(...)` 都是显式
  owner `CoroutineScope` 的构造函数式工厂；正常返回即表示取得 lease。
- Heartbeat job 是 lease 和 owner scope 的结构化子任务；`close()` 取消 lease scope，
  需要等待 owner 文件释放时等待 lease 的 `Job` 完成。
- 锁目录由调用方传入；Home 使用 `~/.kodex/.locks/home/`。
- owner 文件采用 `<pid>.read.lock` 和 `<pid>.write.lock`，内容复用 `pid + acquiredAt + expiresAt`
  heartbeat 身份。
- 同一进程在同一锁目录取得多个 read handle 时复用一个 owner heartbeat，并使用进程内引用计数；
  最后一个 handle 关闭后才删除 owner。
- 仅靠 owner 文件存在性检查有 reader/writer 同时通过的竞态，因此增加短期 `guard.lock`：
  - reader 在 guard 内清理 stale owner，确认没有 write intent，再发布 read owner。
  - writer 在 guard 内清理 stale owner，确认没有其他 writer，再先发布 write intent。
  - write intent 发布后新 readers 不得进入；writer 等待已有 readers 全部释放后才获得独占访问。
- API 返回 lifecycle lease handle，而不是伪装成 `kotlinx.coroutines Mutex`。
- Home 普通运行持有 shared read lease；版本登记、推进和迁移持有 write lease。
- 默认启动不无限等待 migration write lease。存在 readers 时返回明确冲突，让用户关闭其他 Kodex 后重试。
- write acquisition 失败或取消时必须等待自己的 write intent 删除后再返回。
- 通用模块使用真实临时目录验证，不引入 mock filesystem。

## 启动顺序

- 默认 CLI 顺序调整为：
  - 打开 `app/migration/impl` Home coordinator。
  - 取得 read lease，并比较 stored version 与当前应用版本。
  - 版本相同时保留 read lease，不扫描业务数据。
  - 缺失或不同时释放 read、取得 write，并重新读取 version。
  - 验证基线，按全局表执行适用 migration，再写入当前应用版本。
  - 释放 write 并重新取得 read lease，确认 version 仍与当前应用版本相同。
  - 初始化 file logging。
  - 打开 global settings、auth、MCP、hooks 和 Session infrastructure。
- write 升级必须重新读取 version，不能依赖升级前快照。
- 不同应用版本不能同时进入业务数据层；较新版本升级时要求其他 Kodex 进程退出。
- `KodexApplication` 保存 `KodexHomeHandle`：
  - 正常 `shutdown()` 在所有业务基础设施关闭后 `closeAndJoin()`。
  - 非 suspend `close()` 发起关闭；heartbeat 到期仍是崩溃兜底。
  - Application 打开失败时也必须释放 handle。
- 默认 Main 可以预先打开 handle 以保证 logging 之前检查；公开的自定义 `dataDirectory`
  Application 入口没有预开 handle 时自行执行同一协议。
- Lower-level repository 仍只处理调用方给定目录，不重复读取根 version。

## 迁移协议

- `Kodex/app/migration/contract/` 定义 `MigrationVersion` 与 migration entry；
  `Kodex/app/migration/impl/` 负责 `KodexHomeHandle`、应用版本、version store、
  baseline validator 和全局 migration table。
- 模块构建从 Gradle `project.version` 生成当前应用版本；release 流程不增加新的手写版本位置。
- 全局表以发生数据变化的目标应用版本为键，直接保存普通 Kotlin suspend migration 方法：

  ```kotlin
  val migrations = listOf(
      Migration(MigrationVersion("0.3.3"), ::migrateIndexedHistory),
  )
  ```

- 示例版本不预定实际 release；实现时使用包含该 migration 的目标应用版本。
- 表中不登记无迁移 release；表项必须使用字面量目标版本，并保持版本唯一、严格递增。
- 允许在产品改动中预先合并高于当前应用版本的 future entry；Home version coordinator 忽略它，release 的独立
  application version bump 将其激活。
- 升级选择满足 `stored < toVersion <= current` 的全部表项。migration 方法只接收 Home path
  和 `CoroutineFileSystem`，以此前适用表项的已提交结构为输入，不要求应用 release 逐版本相邻。
- 同一目标 release 只有一个表项；多个数据变化组合为该方法内部具名、固定顺序的 steps，
  全部完成后才发布一次目标 version。
- Migration 方法只接收执行所需的 Home path 和普通 filesystem dependency；不增加
  `MigrationRunner`、`MigrationExecutionContext`、worker pool 或 progress framework。
- Migration 方法内部直接使用标准 `coroutineScope`、`async` 和 `awaitAll` 拆分有独立数据
  所有权的子任务；任一子任务失败或取消时遵循标准结构化并发传播。
- Future entry 在目标 release 发布前可修改；发布后目标版本、方法、step 顺序和重入语义完全冻结。
- 每个已发布 migration 方法及旧 codec、fixture 和重入逻辑必须永久保留，使新二进制可以从
  任意受支持旧版本直接升级。
- 已发布 migration 的问题只允许在更高应用版本新增 migration 修复，不得修改、删除、重排或
  复用历史表项。
- 每个目标版本使用稳定的独立源码与 fixture 路径；release gate 对比上一发布版本，拒绝既有
  migration、旧 codec 或 fixture 的修改、移动和删除。
- Historical migration 只依赖稳定 filesystem-layout primitives 和该目标版本冻结的私有模型、
  codec，不依赖持续演进的当前业务 schema。
- Migration 直接原地修改数据，不创建 `.migration.json`、journal、staging、backup 或 rollback。
- 每个目标版本 migration 固定顺序：
  - 按该 migration 的 source layout 打开 root Session directories。
  - 根据目标数据状态跳过已完成部分。
  - 按数据依赖使用普通结构化并发处理 Sessions 或 timelines。
  - 通过 filesystem layout 对 records 直接 rename 或 rewrite。
  - 把目标 schema 自身的自然完成特征与 `latest.json` 放在对应数据操作最后写入。
  - 方法正常返回后把 `version.json` 更新到该 migration 的目标应用版本。
- 进程中断时 version 保持前一个版本；下次启动重新调用同一个 migration，由目标探针、幂等
  rewrite 和 resumable rename 尽力继续。
- 不保存 cursor；每个 step 根据当前文件内容区分旧状态、目标状态和歧义状态。Filesystem
  layout 不推断 target-specific schema。
- 快速 probe 只能使用目标 schema 自身已有的字段或 entry，不创建专用 migration state 文件；
  没有廉价自然特征时允许重扫受影响数据。
- Resumable positive index shift 按 index 降序处理，并根据 source/target 文件存在状态区分待移动、
  已移动、冲突和缺失；已移动 entry 不得再次平移。
- Migration 不保证单文件 overwrite 或无法识别的部分 rename 可恢复；探针发现歧义或损坏时拒绝继续。
- 多级升级在同一个 write lease 下逐表项写入 version；失败步骤不得发布目标 version。
- 全部适用表项完成后，把无迁移尾段更新到当前应用版本。
- Baseline validator 和 migration 必须保留 legacy `subagents/` 与未知 Session 子项。
- 已发布 migration 自身无法完成时保持自动升级阻塞，只能由用户另行授权离线修复任务处理；
  不得用修改历史方法绕过。
- 首次实施只验证现有基线并登记该 release 的当前应用版本，不改写本机 Session。
- 首个真实 migrator 应与
  [`2026-08-28-design-indexed-history-timeline.md`](../executable/2026-08-28-design-indexed-history-timeline.md)
  协调：把其中计划的手工 `uv` 脚本迁移为打包在 CLI 中、以目标应用 release 版本登记的
  原地 Kotlin migrator。
  本任务不修改该活跃 executable 文件。

## AgentStorage 文件布局基建

- 新增 `Kodex/agent-storage/filesystem-layout/` host KMP 模块。
- 模块不依赖 `agent-storage-contract`、clean models、OpenAI models 或当前
  `FileSystemAgentStorage`。
- 模块提供 filesystem storage directory 的无状态 helpers：
  - 调用方声明 timeline names。
  - 每条 timeline 对应一个 directory。
  - Numeric records 是 `<non-negative-index>.json`。
  - `latest.json` 是独立 pointer。
  - 未声明的 child paths 不进入模型且不被修改。
- Timeline directory 提供：
  - 将 canonical numeric filenames 解析为排序后的 `IntArray`。
  - 由 index 构造 record path。
  - raw whole-file read/write。
  - record move/delete。
  - latest pointer read/write。
- Layout 不实现 sparse visible-value lookup、append-only 校验、revert、fork、cache 或业务
  projection；这些属于 live `AgentStorage`，不是 migration 文件处理。
- 每个历史 migration 在自己的冻结目录声明 source/target timeline names 和旧 payload codec：
  - 旧 layout 可以声明 `compaction/stable`。
  - 当前 layout 可以声明 `index/work`。
  - Layout 模块本身不随当前 AgentStorage schema 硬编码唯一 timeline set。
- Migration 对不变化的 record 只做 raw move；只对需要变换的 payload 使用 migration 私有
  serializer 或 `JsonElement`。
- 当前 `agent-storage/filesystem` 后续可以复用该模块的 path 和 numeric filename 规则，但
  首次实现不要求同时重构 live storage。
- Migration 的调用形态保持朴素：

  ```kotlin
  private suspend fun migrateIndexedHistory(
      home: Path,
      fileSystem: CoroutineFileSystem,
  ): Unit = coroutineScope {
      val timelineNames = setOf(
          "index",
          "work",
          "settings",
          "timestamp",
          "token-count",
          "unstable",
      )
      val sessions = Path(home, "sessions")
      fileSystem.list(sessions)
          .mapNotNull { entry ->
              val name = entry.name
              name.toIntOrNull()
                  ?.takeIf { index -> index >= 0 && index.toString() == name }
                  ?.let { entry }
          }
          .map { directory ->
              async {
                  requireStorageLayout(directory, timelineNames, fileSystem)
                  migrateSession(directory)
              }
          }
          .awaitAll()
  }
  ```

- 示例只说明普通 suspend function、layout helpers 和标准结构化并发的边界；具体 migration
  可以根据 timeline 依赖改变子任务粒度，不由共享框架规定。

- Layout 直接调用现有 `CoroutineFileSystem` 的 `list`、whole-file read/write、move 和 delete；
  不为 migration 扩展通用 filesystem interface，也不增加 wrapper、batch executor 或 processing scope。
- 所有操作继续是普通 suspend functions；migration 的 child tasks、循环和 CPU decode 使用标准
  结构化并发与取消传播。

## 错误与兼容性

- `stored == current`：持有 read lease 并继续，不扫描 Session。
- `stored < current` 且存在适用表项：取得 write lease 后按表中版本顺序升级。
- `stored < current` 且没有适用表项：不改业务数据，只登记当前应用版本。
- `stored > current`：拒绝启动，禁止旧二进制读取或写入业务数据。
- version 不是 JSON string 或不是合法 `MigrationVersion`：拒绝启动，不自动覆盖。
- 全局表版本重复或乱序：拒绝启动。
- 全局表高于 current 的 future entry 保持 inactive；它不得参与 migration 选择。
- migration 抛出失败：不写目标 version；下次启动重跑同一 migration。
- 重入探针发现冲突、歧义或损坏：拒绝继续，不尝试 rollback 或猜测恢复。
- 版本机制发布前的二进制无法被新协议阻止。首次版本化 release 不改写基线数据；完成后续
  数据迁移后，不支持再用 pre-version 二进制打开同一 Home。

## 计划修改范围

- 新增：
  - `Kodex/utils/filesystem-lease/contract/`
  - `Kodex/utils/filesystem-lease/impl/`
  - `Kodex/agent-storage/filesystem-layout/`
  - `Kodex/app/migration/contract/`
  - `Kodex/app/migration/impl/`
  - 对应 common 和 host filesystem tests
- 修改：
  - `Kodex/app/migration/impl/build.gradle.kts` 的 runtime 版本源码生成
  - `Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt`
  - `Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt`
  - `Kodex/app/cli/build.gradle.kts`
  - `Kodex/app/viewmodel/application/build.gradle.kts`
  - `.agents/skills/release-kodex/SKILL.md` 的 migration activation 与冻结检查
  - Gradle module discovery 所需配置
  - Application lifecycle 和隔离 Home tests
- 暂不修改：
  - `KodexAgentStorage` payload contract
  - global settings YAML schema
  - `auth.yml`
  - 本机 `~/.kodex`
  - 现有活跃 executable 看板文件

## 验证计划

- Filesystem read/write lease：
  - 多 reader 共存。
  - 同进程多个 read handle 正确引用计数。
  - reader 与 writer 互斥。
  - write intent 阻止新 reader。
  - 并发 reader/writer acquisition 无双重成功。
  - owner 正常关闭、取消和 heartbeat 到期清理。
  - malformed heartbeat fail closed。
  - 使用 helper processes 验证真正的跨进程竞争，不用同进程 scope 冒充不同进程。
- `app/migration/contract` 与 `app/migration/impl`：
  - 新 Home 初始化。
  - 当前应用版本由 Gradle `project.version` 生成且一致。
  - 合法 unversioned Home 登记当前应用版本且 payload 不变。
  - legacy `subagents/` 和未知文件在登记及 migration 后 byte-for-byte 保留。
  - 非法 unversioned Home 不写 version。
  - equal、older、newer、non-string 和 malformed `MigrationVersion`。
  - 无 migration、单 migration 和跨多个非连续目标版本升级。
  - migration 表的重复、乱序和 future entry inactive 校验。
  - current version 提升到 future target 后自动激活。
  - 同一目标版本多 step 顺序固定。
  - 在 rename、rewrite 和 Session boundary 中断后重跑并尽力完成。
  - 部分状态无法识别时明确失败。
- `agent-storage/filesystem-layout`：
  - 不依赖 `KodexAgentStorage` 或当前 clean models。
  - 分别打开 `compaction/stable` 与 `index/work` layout fixtures。
  - 大型 timeline numeric filenames 正确转换为排序 `IntArray`。
  - 未声明与 malformed child paths 保持不变。
  - raw read/write、move、delete 和 latest pointer operations。
- Application：
  - Home 检查先于 logging 和 settings/auth/session 读取。
  - 两个 Application read handles 可共存。
  - 活跃 Application 阻止 migration writer。
  - 打开失败、`shutdown()` 和 `close()` 都释放 lease。
- 使用真实临时 Home，不读取或写入本机 `~/.kodex`。
- 按 Gradle skill 复用已运行的 daemon JVM，运行新增模块 `allTests`、受影响 Application/CLI tests
  和最终 checks。
- 构建隔离 Home 的 Linux x64 CLI smoke test；实现首个 migrator 时再覆盖 macOS ARM64、
  Linux ARM64 和 Windows x64 release executable。
- Release gate 列出本次新激活 entry，并拒绝修改、移动或删除上一 release 已发布的 migration
  source、旧 codec 或 fixture。
- 对修改的 Kotlin 文件运行编译器检查和 `git diff --check`。
