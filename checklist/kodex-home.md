# Kodex Home

## 根目录边界

- `KodexHome` 只定义默认进程路径 `$HOME/.kodex`；不得在该常量中执行目录创建、扫描或迁移。
- `KodexApplication.openDefault()` 使用 `KodexHome` 作为结构化应用数据根。
- `KodexApplication.open(dataDirectory = ...)` 的 `dataDirectory` 只替换该实例的版本元数据、全局设置、私有认证、Session、`AGENTS.md` 和 skills 根。
- 进程级 file logging 和 generated image artifacts 始终使用默认 `KodexHome`，不随自定义 `dataDirectory` 改变。
- 外部 Codex Home 是认证和显式 MCP 导入的只读数据源，不属于 Kodex Home；具体边界遵循 [Codex CLI Storage 兼容性](codex-cli-storage.md)。
- 不预先创建完整 Home skeleton；每个 owner 只在首次需要时创建自己负责的根级文件或目录。
- 不设置全局清理器遍历并重写整个 Home；每个 owner 只维护自己声明的路径和临时命名空间。

## 顶层所有权

- `version.json` 和 `.locks/home/` 归 `app/migration` 所有。
- `settings.yml` 归 `app/shared/settings/filesystem` 所有；设置语义遵循[全局设置](global-settings.md)。
- `auth.yml` 归 `app/shared/auth/filesystem` 所有。
- `sessions/` 归 filesystem Session repository 所有。
- `log/` 归 file logging 所有。
- `generated_images/` 归 generated image artifact persistence 所有。
- `AGENTS.md` 和 `skills/` 是用户管理的只读输入；Kodex 不得自动创建、重写或删除。
- 其他未知根级文件和目录是用户或未来版本数据；普通运行和 migration 必须原样保留。
- 每个 owner 的 sibling temporary、staging、backup 和 lock 名称必须可唯一识别，且不得与用户可配置名称冲突。
- 一个 owner 不得清理、修复或迁移另一个 owner 的路径。

## 按需生成

- 默认 CLI 启动 file logging 时只创建 `KodexHome/log/`；日志初始化失败必须在 Application 打开前报告并终止默认 CLI。
- 打开缺失的 `settings.yml` 时只发布当前 defaults，不创建 Home 或设置文件；首次设置更新才创建父目录并写入完整 snapshot。
- 打开缺失的 `auth.yml` 时发布 credentials unavailable，不创建 Home 或认证文件；Kodex 登录成功或 token refresh 时才写入，logout 只删除 `auth.yml`。
- Application 构造本身不得为了展示初始 New Session 而创建 `sessions/`；首次创建 Session、打开 persisted Session 或刷新 Session catalog 时，repository 才创建 Home 和 `sessions/`。
- 首次成功持久化生成图片时才创建 `KodexHome/generated_images/<sanitized-session-id>/`。
- 读取 `AGENTS.md` 或发现 skills 时允许 Home 和对应路径不存在；读取逻辑不得借机创建目录。
- `app/migration` 只按版本协议创建自己的 version 和 lock 路径，不得为了版本登记提前创建其他 owner 的文件或目录。

## 全局文件维护

- `settings.yml` 和 `auth.yml` 必须通过同目录唯一 temporary 文件加原子替换发布；正常完成、失败或协程取消都必须清理本次 temporary。
- 同一进程内的 settings read-modify-write 和 auth 更新分别串行化。
- 不为 `settings.yml` 或 `auth.yml` 增加跨进程资源锁；多个 Kodex 进程同时更新时只保证每个已发布文件完整，最终采用最后一次原子替换的 snapshot。
- owner 在下次访问前必须识别并恢复或清理自己因进程中断残留的 temporary；不得根据通用 `.tmp` 后缀删除未知文件。
- `settings.yml` 缺失字段使用当前 defaults，未知字段忽略且读取不重写；已知字段非法或 YAML 损坏时拒绝打开设置，不得覆盖原文件。
- `auth.yml` 缺失、无法读取或无法解码时发布准确的 unavailable 状态，不得自动覆盖；只有显式登录、refresh 或 logout 可以改变文件。
- `log/` 使用 rolling files，单文件达到 10 MiB 时滚动并最多保留 5 个文件；Home migration 不处理日志内容。
- generated image artifact 写入失败不得让 image generation tool result 整体失败；成功路径必须写入 Session history。
- generated image artifacts 不随 Session archive、Session delete、版本登记或 migration 自动删除；只有用户明确请求的 artifact 操作可以删除。

## Session 根布局

- `sessions/` 的非隐藏直接子项必须是非负十进制 index 命名的目录；隐藏子项留给 owner 内部状态。
- 发现非隐藏非数字子项、负数名称或非目录 entry 时必须报告 layout 错误，不得静默忽略或删除。
- 新 Session 使用当前磁盘和 repository snapshot 中最小缺失的非负 index，并通过 `mustCreate` 目录原子保留该 index。
- 创建失败或初始化失败时必须在不可取消清理中删除本次保留的完整 Session 目录，不得留下会在下次扫描中被识别为有效 Session 的半成品。
- 每个当前 Session entry 直接包含 `index/`、`work/`、`settings/`、`timestamp/`、`token-count/` 和 `unstable/` 六条 sparse timeline。
- 每条 timeline 的编号 `<index>.json` 文件是持久化真源；`latest.json` 只是可从编号文件重建的 tail pointer。
- 新建空 Session 必须创建全部六条 timeline，并把每条 `latest.json` 初始化为 `-1`。
- timeline 只把非负十进制 `<index>.json` 识别为数据；owner temporary、revert 状态和未知文件不得被当作 change point。
- timeline append、pointer 更新和 revert 必须保持编号文件为最终真源；进程中断后由 timeline owner 恢复或清理自己的 write/revert 残留。
- 完整 Session 打开期间持有该 entry 的 renewable `lock.json` 独占租约；关闭 Session scope 后释放，进程崩溃后允许按 heartbeat 到期接管。
- Catalog 的无锁读取、临时 lease 和 `latest.json` 修复遵循 [CLI Session 与 Agent ViewModel 边界](cli-session-view-models.md)。
- `archive.mark` 的存在是 root Session archived 状态真源；archive 创建空 marker，unarchive 删除 marker。
- 删除 Session 时先关闭本进程 handle，再取得该 entry 的独占租约；用户明确删除整个 Session 后可以递归删除 entry 内全部当前、未知和 legacy 数据。
- Fork 必须先保留新 index，再复制六条 timeline 的已提交 raw prefix；失败时删除完整 target，且不得修改 source。
- 既有 `subagents/` 是当前 runtime 不再读取或创建的 legacy 数据；除用户明确删除整个 Session 外，普通运行、版本登记和 migration 必须原样保留。
- 清理 legacy `subagents/` 只能通过单独授权、具有严格 preflight 和恢复路径的显式任务执行。

## Home 版本

- 根目录 `version.json` 是结构化 Kodex Home 的唯一版本真源，内容直接保存最后一次成功准备 Home 的 Kodex 完整应用版本字符串，例如 `"0.3.2"`；不得增加 `formatVersion` envelope。
- 应用版本使用严格 SemVer 比较，不得按字符串字典序排序。
- 当前版本必须由 Gradle `project.version` 生成给 `app/migration` 使用，不得在 migration 源码中维护另一个可漂移的当前版本常量。
- 缺失 `version.json` 表示最后一个未引入版本机制的 release；应用必须验证当前 root Session 布局，再按全局迁移表升级并写入当前应用版本。
- 基线验证只把当前六条 root timeline 视为规范结构；legacy `subagents/`、用户输入、日志、artifacts 和未知文件只验证不会被改动，不把它们升级为当前 schema。
- `app/migration` 是 Home 版本检查和数据迁移的唯一应用模块。
- `app/migration` 保存按目标应用版本排序的全局迁移表；表中只登记实际需要数据迁移的 release，不为无迁移 release 添加 no-op。
- 从 stored version 升级时，只执行满足 `stored < migration.version <= current` 的表项，并严格按 SemVer 顺序执行。
- 每个历史 migration 方法必须保留，供跨多个 release 直接升级。
- 没有匹配 migration 的版本升级仍必须在独占租约下写入当前应用版本。
- `settings.yml` 继续使用无版本宽松 YAML；兼容的设置字段增删不增加 migration 表项。

## Migration Registry 管理

- `app/migration` 中的全局 registry 是全部自动 Home migration 的唯一代码真源；不得在 release 文档或第二张表中复制 authority。
- 每个 registry entry 使用字面量目标应用版本和一个 migration 方法；目标版本必须是合法 SemVer，并在表内唯一且严格递增。
- Registry 允许预先登记高于当前应用版本的 future entry；Home version coordinator 必须忽略 `targetVersion > currentVersion` 的 entry。
- Release 的独立 application version bump 是 future entry 的唯一激活开关；bump 后自动纳入 `stored < targetVersion <= currentVersion` 的选择结果。
- 无持久化数据变化的 release 不登记 entry；仅把 `version.json` 推进到当前应用版本。
- 同一个目标 release 只允许一个 registry entry；该 release 的多个数据变化必须在一个 migration 方法内组合为具名、固定顺序的 steps。
- 一个 target migration 的全部 steps 完成后才能写入该目标版本；中间 step 不得提前更新 `version.json`。
- 多 step migration 不保存 step journal；每个 step 必须根据当前数据判断已完成、待执行或无法继续，并使重复执行尽量收敛到目标结构。
- 每个 migration 接收实际 stored version，并以此前所有适用 entry 已提交后的结构为输入；不得假设应用 release 逐版本相邻。
- Migration 必须只使用本地 filesystem、确定性转换和已打包代码；不得依赖网络、交互输入、当前时间语义或外部 Codex 数据。
- 持久化 schema 的不兼容变化必须在同一产品任务中增加目标 release 的 migration、fixture 和验证；兼容变化不得为了留记录而增加 no-op。
- Future entry 在目标 release 发布前可以修改或删除；目标 release 一旦发布，该 entry 的目标版本、方法、step 顺序和重入语义全部冻结。
- 已发布 entry 不得删除、重排、重用目标版本或修改实现；后续发现的数据问题只能由更高目标版本的新 migration 修复。
- 如果已发布 migration 自身无法完成，自动升级保持阻塞；只能通过用户另行明确授权的离线修复任务处理，不得回写或替换历史 migration。
- 每个已发布 migration 及其读取旧格式所需的 codec、fixture 和重入逻辑必须长期保留。
- 每个目标版本的 migration、私有旧 codec 和 fixture 必须放在稳定的独立源码路径中，避免普通重构触碰已发布历史。
- Historical migration 只能依赖稳定的 filesystem-layout primitives 和该目标版本冻结的私有模型、codec；不得依赖持续演进的当前业务 schema。
- Release 验证必须与上一已发布版本比较所有既有目标版本的 migration source、旧 codec 和 fixture；任何修改、移动或删除都阻塞 release。
- Release 验证必须列出相对上一 release 新激活的 registry entries，并在打包前运行它们的 fixture、跨多版本升级、中断后重跑和性能 tests。
- Release 不得发布目标版本已激活但缺少 migration fixture、重入测试或目标结构检查的构建。

## AgentStorage 文件布局基建

- 每个 registry entry 直接保存普通 `suspend` migration 函数；不得为 migration 增加 runner、execution context、worker pool、progress framework 或自定义 task abstraction。
- Migration 使用标准 `coroutineScope`、`async`、`awaitAll` 和取消传播组织自己的子任务；并行粒度与执行顺序由该 migration 的数据依赖决定。
- 新增独立 `agent-storage/filesystem-layout` 模块，在不依赖 `KodexAgentStorage`、当前 clean models 或当前业务 codec 的情况下建模 filesystem AgentStorage 目录。
- Layout 模块只建模一个 Session storage directory、其中具名的 sparse timeline directories、numeric `<index>.json` records 和 `latest.json`；不实现 AgentStorage 的 visible-value、append、revert、fork 或业务投影语义。
- Timeline layout 提供 numeric record indexes、record path、raw whole-file read/write、move、delete 和 latest pointer 操作；payload 类型与转换规则由对应版本 migration 定义。
- Numeric record 枚举只接受 canonical non-negative decimal `<index>.json`，使用 primitive `IntArray` 保存 indexes，按需由 index 构造 record path；不得为大型 timeline 长期保留逐 record 业务对象或 decoded payload。
- 不在 layout 模块硬编码唯一的当前 timeline 集合；每个历史 migration 在自己的稳定源码目录声明 source 和 target timeline names，支持 `compaction/stable`、`index/work` 等不同布局。
- 未被 source 或 target layout 声明的 Session 子项必须保留；layout 打开和操作不得自动清理未知、legacy 或 owner temporary 路径。
- Historical migration 使用自己冻结的最小 codec 或 `JsonElement` 解码需要转换的 records；未变化 payload 直接 move 或按 raw bytes 处理。
- `agent-storage/filesystem` 可以逐步复用 layout 模块的路径和 numeric-record 规则，但 migration 不得通过当前 `FileSystemAgentStorage` 打开旧数据。
- `utils/kotlinx-io-coroutines` 提供 layout 所需的通用高性能文件能力；不得在 layout 模块复制一套平台 filesystem backend。
- `CoroutineFileSystem` 的 directory-name 枚举应允许平台实现避免为每个 child 构造完整 `Path`；普通 `list` API 保持可用。
- `CoroutineFileSystem` 的 whole-file `readBytes` 和 `writeBytes` 必须允许 blocking 平台在一次 I/O dispatcher 进入中完成整个文件操作；不得对每个 64 KiB segment 重复切换 dispatcher。
- 长目录枚举和 whole-file chunk loop 必须在同一次 dispatcher 进入内周期性检查取消；directory-name 枚举、whole-file I/O 和 move/delete 都保持普通 suspend API，不增加持久任务状态。
- 使用真实临时 filesystem 验证不同历史 layout、numeric record 解析、raw I/O、rename 和取消；使用接近实际大型 Session 的目录 benchmark 记录扫描、rename、read/write 和总吞吐，不设置易抖动的固定 CI 时间阈值。

## Home 租约与启动

- 普通 Kodex 进程在首次读取设置、认证或 Session 前取得结构化 Home 的共享读租约，并持有到应用基础设施全部关闭。
- 只有与 `version.json` 相同应用版本的多个进程可以同时进入结构化业务数据层。
- Home 初始化、版本推进和数据迁移必须取得独占写租约。
- 跨进程读写租约使用独立 filesystem 模块，基于 renewable heartbeat 文件、短期 acquisition guard 和 `<pid>.<read|write>.lock` owner 文件实现。
- reader 必须在 guard 内清理 stale owner、确认没有 write intent，再发布 read owner。
- writer 必须在 guard 内清理 stale owner、确认没有其他 writer、先发布 write intent，再等待既有 readers 退出。
- 同一进程对同一锁目录的多个 read handle 共享一个 owner heartbeat 并使用进程内引用计数；最后一个 handle 关闭后才释放。
- 正常关闭或协程取消必须删除 owner；进程崩溃后由 heartbeat 到期接管清理。
- 无法解析的 owner 必须 fail closed 并报告路径，不得猜测为 stale。
- Home 读租约表示进程使用当前结构，不表示 settings、auth 或 Session 文件只读；普通领域写入继续遵循各自协议。
- 每次启动都读取 `version.json`；应用版本相同时不扫描全部 Session。
- stored version 高于当前应用版本、version 不是 JSON string 或版本不是合法 SemVer 时，必须在读取结构化业务数据前拒绝启动。

## Migration

- migration 只改动自己声明的当前结构化数据，不得改动 `log/`、`generated_images/`、`AGENTS.md`、skills、legacy `subagents/` 或未知路径。
- Migration 直接原地修改目标文件和目录，不建立 staging、backup、journal、全局 marker、回滚或跨文件原子发布协议。
- Migration 基建不提供 transaction 或持久 cursor；“继续”表示下次重新调用同一普通 suspend 方法，并根据当前目标数据跳过已完成工作。
- 每个 migration 必须尽量可重入：重复执行时先用目标数据特征跳过已完成单元，再继续仍处于可识别中间状态的单元。
- 每个 step 必须定义旧状态、目标状态和歧义状态；filesystem layout 只执行调用方指定的 raw 文件操作，不猜测 migration-specific schema。
- 每个 migration 应把目标 schema 自身已有的自然完成特征和 `latest.json` 等总结性元数据放在对应数据步骤最后写入，使重跑可以廉价识别完整单元；不得为此创建专用 migration state 文件。
- 如果目标 schema 没有廉价的自然完成特征，重跑允许重新扫描受影响数据，不得改用隐式 marker 冒充业务数据。
- 单文件 overwrite、rename 序列或任意 step 中断都允许留下损坏或无法识别的状态；migration 不承诺自动恢复。
- 中断时 `version.json` 保持旧版本；下次启动重新选择并从头调用同一个 migration 方法，由该方法的重入逻辑尽力继续。
- 如果重入探针发现冲突、歧义或损坏，migration 立即失败并阻止业务数据打开，不猜测或静默跳过。
- 每个实际 migration 方法正常返回后直接把 `version.json` 写到该表项的目标版本。
- 全部实际 migration 完成后，再把无迁移尾段写到当前应用版本。
- 首次引入版本机制的 release 不得同时改写基线数据；只验证 Home 并登记该 release 的应用版本。
- 版本机制只能约束支持该协议的二进制；完成后续数据迁移后不得再用首次引入前的二进制打开同一个 Home。

## 验证

- Home 测试必须使用隔离临时目录，不得读取、写入或迁移本机 `~/.kodex`。
- 自定义 `dataDirectory` 测试必须验证结构化数据迁移到自定义根，同时明确验证 logging 和 generated image artifacts 仍使用默认 `KodexHome`。
- 验证空 Home 不会一次性生成全部顶层 entry，每个 owner 只在对应首次使用点创建自己的路径。
- 验证 settings、auth、Session 和 timeline owner 各自的故障协议；migration 单独验证在代表性 operation boundary 中断后重跑能尽力继续，且无法识别时明确失败。
- 验证普通启动和 migration 保留用户根级内容、legacy `subagents/` 和未知文件。
- 验证显式 Session delete 会等待 entry lease，并只删除目标 Session 及其关联 entry 内容，不删除 generated image artifacts。
- 验证 migration 不能通过当前 `FileSystemAgentStorage` 或当前业务 codec 打开旧 layout。
- 使用大型 directory fixture 对 directory-name scan、numeric index materialization、raw rename 和 whole-file read/write 记录吞吐与峰值内存。
