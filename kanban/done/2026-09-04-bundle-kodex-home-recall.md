# Task Tree

- [done] 为 Agent 注入 Session meta 并内置 Kodex Home Skill
  - [done] 调查 environment context 与 Session 构造
  - [done] 调查 Skill 发现与 Home migration
  - [done] 确定目标版本和文件所有权
  - [done] 盘点 Session meta 引起的接口形状变化
  - [done] 收敛为 Storage URI 且不单传 index
  - [done] 接受 URI 切换造成的一次性 identity 变化
  - [done] 审查跨平台、下游与 migration 阻塞点
  - [done] 从 AgentStorage 投影 Session meta
    - [done] 将 Storage contract 的 id 重命名为 uri
    - [done] 让 filesystem storage 生成 URI-shaped file locator
    - [done] 让 in-memory storage 生成 memory URI
    - [done] 让 cache、wrapper 和 consumer 使用 uri
    - [done] 让 AgentState 从既有 storage 读取 uri
    - [done] 从每次请求的 settings snapshot 读取当前 session name
    - [done] 删除当前一对一模型不再需要的 AgentAddress
  - [done] 扩展 environment context
    - [done] 在结构化 prefix contract 中增加 Session meta
    - [done] 在 `<environment_context>` 中渲染 storage URI 和 name
    - [done] 保持 prefix 临时且不进入 history 或 compaction
  - [done] 内嵌 Kodex Home Skill
    - [done] 增加版本冻结的 `kodex-home` Kotlin raw String
    - [done] 将 String 直接编译进四平台 CLI
    - [done] 更新 Skill 内容以匹配当前 Home 与 timeline 布局
  - [done] 增加 `0.3.5` Home migration
    - [done] 注册尚未激活的 `0.3.5` future entry
    - [done] 原子覆盖产品管理的目标 SKILL.md
    - [done] 保留 Skill 目录中的其他未知内容
    - [done] 覆盖重入、冲突和失败清理
  - [done] 更新持久规则与发行检查
    - [done] 更新 Agent context 与 Kodex Home checklists
    - [done] 将版本化 Skill String 纳入 migration 冻结检查
    - [done] 增加发行时的自动安装 smoke test
  - [done] 完成定向与跨平台验证
    - [done] 运行 context、state、Session 和 migration tests
    - [done] 验证四个 native CLI 目标
    - [done] 运行隔离 Home 的升级验证
    - [done] 运行格式与 diff 检查

# Details

## 目标与决策

- 每次普通 Responses 请求的 `<environment_context>` 提供当前 root Session 的 storage
  URI 和 name，使 Agent 能借助 `kodex-home` Skill 精确定位并确认自己的历史。
- `kodex-home` Skill 编译进单个 CLI executable，并由 Kodex Home migration 自动写入实际
  `dataDirectory`，不增加发行归档中的独立文件。
- 用户已确定使用 `0.3.5` future migration。
- 用户已确定 `<kodex-home>/skills/kodex-home/SKILL.md` 由产品管理；migration 可以覆盖
  已有或被修改的该文件。
- 产品只管理该 `SKILL.md` 文件，不删除同目录其他文件。
- 当前源码版本是尚未发布的 `0.3.4`，本机 Home 也已经登记为 `0.3.4`。`0.3.5`
  能确保这些开发期 Home 在后续升级时执行安装。
- 本任务不 bump application version、不发布 release，也不修改本机 `~/.kodex` 或
  `~/.agents/skills/kodex-home`。

## Session meta 数据流

- Session name 的 authority 是该请求使用的 `KodexAgentSettings.threadName` snapshot；
  rename 从下一次普通 Responses 请求开始可见。
- Storage URI 的 authority 是 `KodexAgentStorage.uri`。它是在一个 storage 生命周期内
  不可变的 URI-shaped opaque locator，不承诺符合 RFC URI grammar，也不交给通用 URI
  parser。
- `FileSystemAgentStorage(directory, fileSystem)` 已经使用同一个
  `CoroutineFileSystem.resolve(directory)` 保存规范化绝对 `directory`。它把该路径投影为
  外观与 file URI 一致但不做 URI escaping 的 locator；当前 root Session 的 locator 指向
  `<kodex-home>/sessions/<entry-index>`，不是 Kodex Home 根目录。
- Filesystem locator 的精确外观是：
  - POSIX local path：`file:///home/user/.kodex/sessions/42`
  - Windows drive path：`file:///C:/Users/user/.kodex/sessions/42`
  - Windows UNC path：`file://server/share/sessions/42`
- Formatter 对 POSIX absolute path 添加 `file://`；对 separator-normalized Windows drive
  path 添加 `file:///`；对以 `//` 开头的 normalized UNC path 添加 `file:`。
- Windows path separator 统一为 `/`，但 drive letter 和 UNC authority 保留 resolved path
  的原始大小写；POSIX path 中的 literal backslash 原样保留。
- 空格、Unicode、`#`、`?`、`%` 和其他 path 内容原样保留，不做 percent-encoding 或
  percent-decoding。内部 consumer 始终把完整 locator 当作 opaque String。
- `agent-storage/filesystem` 增加 platform-aware、确定性的内部
  `resolved Path -> URI-shaped locator` formatter；不增加 URI parser dependency。
- `InMemoryKodexAgentStorage` 使用 `memory:<lowercase-hex-token>`。token 继续使用当前对象
  identity hash 的 unsigned lowercase hex 表达，不引入 UUID；它只标识进程内 storage，
  不承诺跨进程稳定性，也不暗示 filesystem location。
- `CachedAgentStorage` 和通用 storage wrapper 必须转发底层 `uri`，避免包装后改变
  identity。
- `KodexAgentState` 已经持有构造时传入的 `storage`，因此其公开 factory 不增加参数。
  私有实现保留仅由 application-wide `contextSettings` 构造的
  `AgentContextPrefixResolver`，并在每次请求调用 `resolve(settings, storage.uri)`。
- Prefix contract 增加结构化
  `AgentSessionMeta(uri, name)`；resolver 组合该次调用传入的 storage URI 和当前 Agent
  settings 中的 name。Renderer 保持现有 `cwd`、`shell`、`current_date`、`timezone`
  顺序，并在 `<environment_context>` 末尾追加：

  ```xml
  <current_session>
    <storage_uri>file:///home/user/.kodex/sessions/42</storage_uri>
    <name>Current task</name>
  </current_session>
  ```

- `<current_session>` 明确表示该节点描述的就是收到当前请求的 Agent 自己所在的 Session；
  `storage_uri` 指向该 Session 自己的 data directory，不是其他历史 Session 或 Kodex Home。
- Prompt XML DSL 负责转义 storage URI 和 Session name，不允许内容突破对应 XML 边界。
- Session index 不再单独传播或渲染。普通 `<kodex-home>/sessions/<index>` 布局通常会让
  index 出现在 locator path 中，但 `CoroutineFileSystem.resolve()` 可能跟随 symlink，
  contract 不保证能从最后一个 path segment 恢复 index。
- Skill 按 Kodex 自己的 locator 格式恢复完整 Session directory，不调用通用 URI parser，
  也不做 percent-decoding。
- `memory:` URI 没有可读取的磁盘历史。Skill 必须识别 scheme 并明确停止 filesystem
  recall，不得把 opaque token 当作路径。
- Filesystem storage URI 在 reopen 时由同一 resolved path 确定；Session fork 后创建的
  storage 自然得到指向新 entry 目录的新 URI。
- Session meta 只属于每次请求重新渲染的 transient prefix。它不新增 AgentStorage
  timeline 或持久化字段，也不进入 history 或 remote compaction input；name 继续只由现有
  settings timeline 持久化。
- `<name>` 始终存在并原样渲染 `threadName`；空字符串不省略，也不从 URI 生成 fallback。

## 接口形状与兼容面

### 重命名的公开 Storage identity

- `Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/KodexAgentStorage.kt:16`
  把 backend-defined `id` 重命名为 URI contract：

  ```kotlin
  public interface KodexAgentStorage {
      public val uri: String
      // Existing timelines remain unchanged.
  }
  ```

- `uri` 仍使用 `String`。Contract 通过 KDoc 约定 URI-shaped scheme、storage 生命周期内
  稳定性和 backend-specific 语义，不增加 runtime validation、`Path` 或 URI parser
  dependency。
- 在 `agent-context/prefix/contract` 新增一次请求的投影模型：

  ```kotlin
  public data class AgentSessionMeta(
      public val uri: String,
      public val name: String,
  )
  ```

- `Kodex/agent-context/prefix/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/contract/AgentContextPrefix.kt:16`
  的 `AgentContextPrefix` 在现有属性末尾增加必填属性：

  ```kotlin
  public data class AgentContextPrefix(
      public val cwd: Path,
      public val shell: Shell,
      public val agentMd: AgentsMdInstructions,
      public val availableSkills: List<AvailableSkill>,
      public val sessionMeta: AgentSessionMeta,
  )
  ```

  新属性追加在最后，保留既有四个属性的顺序和 `componentN` 序号；data class constructor
  和 `copy` 仍发生 source/binary shape 变化，所有手工构造 prefix 的
  renderer/resolver tests 必须补齐。

### 保持 AgentState factory 不变

- `Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt:86`
  的公开 factory 保持原形状：

  ```kotlin
  public suspend fun CoroutineScope.KodexAgentState(
      client: OpenAiClient,
      storage: MutableKodexAgentStorage,
      contextSettings: StateFlow<AgentContextSettings>,
      mcpService: McpService,
  ): KodexAgentState
  ```

- 当前分布在 15 个 Kotlin 文件中的 80 处 `KodexAgentState(` 文本引用都不需要增加
  Session 参数。私有实现已经持有 `storage`，直接读取重命名后的 `uri` 即可。
- `Kodex/agent-context/prefix/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/impl/AgentContextPrefixResolver.kt:22`
  的 constructor 保持原形状，`resolve` 增加该请求的必填 `uri` 参数：

  ```kotlin
  public class AgentContextPrefixResolver(
      contextSettings: StateFlow<AgentContextSettings>,
  ) {
      public suspend fun resolve(
          settings: KodexAgentSettings,
          uri: String,
      ): AgentContextPrefix
  }
  ```

  所有 backend 都必须为每次调用提供 URI-shaped locator，因此不提供 default。AgentState
  生产路径传入
  `storage.uri`；resolver tests 使用显式的 `memory:test` fixture。Resolver 继续从
  `settings` snapshot 读取 name，并从自己的 StateFlow 读取同次解析所需的
  application-wide context。当前五处 resolver constructor call 保持不变；一处 production
  和五处 test `resolve` call 增加 URI argument。

### 改变的 Storage 实现与 consumer

- 当前六个直接实现 `KodexAgentStorage` 的 production/test 类型把 `id` 改为 `uri`。
- 当前 storage identity 有 25 处直接引用，分布在 17 个 Kotlin 文件；这些引用执行机械
  rename 和对应语义更新，仍不影响 80 处 AgentState factory 引用。
- `FileSystemAgentStorage.uri` 返回 resolved `directory` 的 URI-shaped file locator。
- `CachedAgentStorage.uri` 转发创建时捕获的 filesystem URI，并与既有 `id` 一样遵守
  Session scope active check。
- `InMemoryKodexAgentStorage.uri` 返回 `memory:<token>`；`SessionAgentStorage` 和两个
  test wrapper 转发 delegate URI。
- AgentState、runtime composition/decorators、hooks、WebRun、image artifacts、Session
  相关 tests 把 storage identity 读取从 `.id` 改为 `.uri`。OpenAI 自身的
  `KodexAgentSettings.sessionId`、Responses item id、tool call id 和 Unified Exec 的
  live `session_id` 不属于该 rename。
- 直接保存或接收 Storage URI 的 Kotlin property/parameter 统一命名为 `uri`，不使用
  `storageUri`、`sessionUri` 或 `agentUri`。`HookSessionContext.sessionId` 改为 `uri`。
- 该命名规则还覆盖 `toHookSessionContext/toHookTurnContext`、`WebRunToolClient`、
  logging 和 image output helper 的对应参数；不能对所有名为 `sessionId` 或 `id` 的符号
  做无语义批量替换。
- Hook command JSON 顶层字段从 `session_id` 直接改为 `uri`，不保留过渡 alias。
- Structured logging 删除 `session_id/agent_id`，只保留一个 `uri`：
  - `logger.session(uri)` 写入 `scope=session` 与 `uri`。
  - `logger.agent()` 只切换到 `scope=agent` 并继承 URI。
  - `logger.tool(...)` 继续继承 URI。
- OpenAI `SearchRequest.id` 是上游协议字段，名称保持 `id`，值改为 Storage URI。
- 当前 Session 与 Agent 严格一对一；删除 `AgentAddress`，不在 `AgentViewModel` 增加
  Session index 或 URI 替代属性。Session owner 使用自己的 `sessionIndex`，跨层归属用
  ViewModel object identity，异步工作继续由 ViewModel scope、call id 和 revision 隔离。
- Renderer 新增私有 Session meta helper；公开 `AgentContextPrefix.render()` 形状不变。

### 明确保留不变的接口

- `KodexRootSessionRepository.create/createFork/open`、`KodexAgentSession`、
  `KodexSessionEntry` 和 `KodexRootSessionEntry` 不增加属性或参数；filesystem storage
  已经拥有 Session directory。
- `KodexAgentState` contract、`AgentRuntime` contract 和 runtime decorator 接口不变；
  Session meta 只影响普通 Responses request 的 prefix 组装。
- `AgentContextSettings` 不承载 Session meta，因为它是 application-wide context source
  配置并被所有 Session 共享；把 Session URI 放入该 Flow 会导致并行打开的 Session
  相互覆盖 identity。
- `AgentContextSettings` interface、现有 implementer 和
  `KodexAgentDependencies.contextSettings` 的类型保持不变；不增加 Session-specific Flow
  wrapper 或 factory。
- `KodexAgentDependencies` 不承载 Session meta，因为同一个 dependencies 实例被整个
  repository 的多个 Session 共享。
- `KodexAgentSettings` 不新增 storage URI；它会持久化和 fork，identity 放入其中会复制
  出错误或过期值，name 已由现有 `threadName` 表达。
- AgentState 不 downcast `FileSystemAgentStorage`，直接使用 backend 提供的
  `storage.uri`。
- 不改变 timeline、global settings、Session entry 或 Home 的序列化 shape。`0.3.5`
  migration 只负责物化内嵌 Skill，不承担数据 schema 迁移。

### 模块依赖与兼容结论

- Storage contract 仍只公开 `String` identity，不增加依赖；prefix contract 也不新增
  project dependency，不形成 Session/State 循环。
- `KodexAgentStorage.id -> uri`、`AgentContextPrefix` constructor 和
  `AgentContextPrefixResolver.resolve(settings, uri)` 都是 source/binary shape 变化；
  `AgentContextPrefixResolver` constructor 本身不变。
- `HookSessionContext.sessionId -> uri` 会改变公开 property、generated getter、data class
  `copy` parameter 和 named-argument source shape；字段类型和 constructor/component
  顺序不变。
- 删除 `AgentAddress`、`AgentViewModel.address`、Agent ViewModel factory 的 `address`
  parameter 和 Koin arguments 中的 address。Session fork 改用 owning
  `PersistedSessionViewModel.sessionIndex`，Application 的归属检查改用 root Agent object
  identity。
- `WebRunToolClient` constructor、`ImageGenerationTools.outputDirectory`、logging helpers
  和 Hook projection extensions 的参数名改为 `uri`；JVM/Kotlin 函数类型不变，但使用
  named argument 的源码需要更新。
- Logging API 的 `agent(agentId)` 改为无参数 `agent()`，并把结构化 identity 收敛为
  单一 `uri` 字段。
- 当前交付物是 CLI，不把这些模块作为稳定外部 SDK 发布，通过全仓编译迁移 implementer、
  consumer 和 fixtures；`KodexAgentStorage.id`、Hook `session_id` 和 `AgentAddress` 都不
  保留 alias 或 overload。
- 用户已接受 filesystem identity 从 `filesystem:<resolved-path>` 变为具有真实 URI 外观的
  URI-shaped `file:` locator。`toCodexThreadId()` 直接哈希新 String，不增加 legacy seed
  映射，因此既有 filesystem Session 升级后会得到新的 OpenAI `thread_id` 和 wire
  `window_id`；从该版本起，同一 resolved path reopen 再次保持稳定。
- In-memory identity 的字符串值继续是 `memory:<unsigned-lowercase-hex-hash>`，只有 Kotlin
  property rename，不发生对应的值切换。
- Hook payload 改用顶层 `uri`，runtime logger 改用单一 `uri`，WebRun identity 和新生成
  图片的 identity-derived output directory 也会一次性切换。Image artifact 继续把完整
  Storage URI 交给现有 sanitizer，不增加 hash 或 legacy artifact key；既有 generated
  image 文件及 history 中已记录的路径不移动、不删除。
- 该变化不修改或移动 Session directory，不改变 timeline 内容，也不需要 Home schema
  migration；本任务的 `0.3.5` migration 仍只负责内置 Skill。

## Implementation record

- 已完成 Storage URI、Session meta、context 渲染、AgentAddress 清理、Hook/logging identity
  切换和 `0.3.5` Skill migration。
- JVM 定向测试通过：filesystem、in-memory、context prefix、agent state 和 migration。
- `agent-storage-filesystem` 的 Linux x64、Linux arm64、mingwX64 native 编译通过；macOS
  arm64 在 Linux 主机上因既有 Apple cinterop 环境不可用而跳过。
- `app-cli:linkReleaseExecutableLinuxX64` 通过；其他 native CLI link 需要对应平台工具链。
- `git diff --check` 和过期 identity 符号检查通过。
- 全仓没有保留 `AgentAddress` 或 Storage identity 的 `id` alias；OpenAI/Codex/Unified Exec
  自身的 `session_id` 不属于本任务的 Storage URI。
- 面向 Agent 的 prompt contract 是有意新增嵌套 `<current_session>`；既有 cwd、shell、日期、
  时区、AGENTS.md 和 available skills 的相对结构与顺序保持，Skill 只依赖新节点。

## 阻塞审查结论

- 当前没有剩余产品决策阻塞；以下项目是实现阶段必须先通过的技术 gate，不授权任务自动
  进入 executable 或开始实现。
- 跨平台 URI-shaped formatter 是主要实现风险：
  - Formatter 必须按 host 与 resolved path 形状区分 POSIX local、Windows drive 和 UNC，
    分别产生 `file:///...`、`file:///C:/...` 和 `file://server/...` 外观。
  - Windows 输入的 path separator 统一为 `/`；drive letter 和 UNC authority 保留
    resolver 结果的大小写。
  - Windows resolver 当前会移除 `\\?\` namespace prefix；formatter 以移除后的 DOS/UNC
    path 为正常输入，并用测试锁定这一边界。
  - POSIX 文件名中的 literal backslash 必须原样保留，不能被误判为 separator。
  - 空格、Unicode、`#`、`?` 和 `%` 必须原样保留；不得调用 URI encoder、decoder 或
    parser。
  - 纯函数 vector tests 覆盖所有平台形状；Windows x64 还需要实际 resolve 后的 smoke
    test，只有跨平台编译不足以证明 locator 正确。
- Storage identity rename 的 blast radius 已封闭：
  - 当前共有六个直接 implementer，以及 17 个 Kotlin 文件中的 25 处 storage identity
    引用；`AgentAddress` 对应 consumer 直接删除，其余按语义迁移。
  - 只重命名语义上直接承载 Storage URI 的 Kotlin 符号；OpenAI item/tool id、turn id、
    notification id、Unified Exec live session id 和数值 Session index 不变。
  - Hook JSON 从 `session_id` 直接改为 `uri`；logging 从 `session_id/agent_id` 收敛为一个
    `uri`；上游 `SearchRequest.id` 名称不变。
- 当前 Session/Agent 一对一约束允许删除 Agent address：
  - Agent contract、factory 和 Koin arguments 不再传播 address 或 URI。
  - Session owner 自己持有 repository index，跨层归属使用 root Agent object identity。
  - Logging 仍保留 session、agent、tool event scope，但只有 session scope 引入 URI，
    后续 scope 继承它。
- Session URI 不进入共享 `StateFlow<AgentContextSettings>`：
  - Resolver constructor 保持只接收 application-wide StateFlow。
  - 每个 AgentState 在普通 Responses request 的 resolve 调用中传入自己的
    `storage.uri`。
  - 因此多个同时打开的 Session 可以共享 context source 更新，而不会串用 Storage URI。
- 一次性 identity 切换已经接受：
  - 不增加旧 `filesystem:` seed 到新 `file:` URI 的兼容映射。
  - Provider thread/window、Hook 值、logger 值、WebRun 值及新 image output directory
    会切换；timeline 与 Home schema 不迁移。
- Skill 内容直接维护为版本化 Kotlin raw `String`，不增加 resource、generator 或
  `ByteArray` 中间表示。实现必须显式处理 raw String 中的 `$` interpolation 和 `"""`
  delimiter，并固定 indentation 与结尾 LF，避免维护内容和实际落盘内容不一致。
- Home migration 在 application 构造 context resolver 和 Skill catalog 前完成，因此
  首次完成 `0.3.5` 升级的同一进程即可发现内置 Skill，不需要重启。
- Migration action 已运行在现有 Kodex Home exclusive write lease 内；不增加第二套进程间
  锁。实现仍需用 migration 专属 temporary 和 atomic replace 处理单进程取消或崩溃。
- Migration 跟随 Home 内外的 Skill directory symlink，但不保留目标 file symlink；action
  不比较旧内容，每次执行都把目标 atomic replace 为普通文件。Home lease 不约束 symlink
  外部目标的非 Kodex writer；这里只保证本次发布文件完整，外部并发写采用最后一次替换。
- `0.3.5` 是一次性收敛 migration，不是每次启动的 self-heal：
  - 升级时覆盖缺失或被修改的目标文件。
  - 用户在 Home 已登记为 `0.3.5` 后删除或修改该文件，不会由同版本启动自动修复。
  - 后续产品内容更新或再次收敛必须使用更高版本 migration。
- Skill name 仍遵守现有 last-wins discovery：
  - Kodex Home 位于 Agents Home 之后，因此覆盖同名的旧手动 Agents Home Skill。
  - 更晚的 Codex Home、自定义 source 或 project Skill 仍可覆盖内置版本。
  - 不增加特殊优先级或不可覆盖的产品 Skill；用户禁用 Kodex Home source 时也继续不展示。

## 内嵌 Skill

- 在 `app/migration/impl` 的 `v0_3_5` package 中直接增加发布后冻结的
  `KodexHomeSkill.kt`。
- Skill 的唯一 source of truth 是可直接 review 和编辑的 internal Kotlin raw `String`；
  不保留独立 resource `SKILL.md`，也不增加 Gradle code generation。
- String 使用 `trimIndent()` 消除 Kotlin 源码缩进，并显式补一个结尾 LF。正文中的 `$`
  使用 Kotlin template escape；正文不得无处理地包含 raw String 的 `"""` delimiter。
- Migration 用现有 filesystem abstraction 将该 String 以 UTF-8 写入 temporary，再
  atomic replace 到目标 `SKILL.md`。测试直接比较落盘文本和 Kotlin String。
- Skill 内容基于现有 `kodex-home` Skill 更新：
  - 读取 environment context 中的 `current_session.storage_uri`，并把它理解为当前 Agent
    自己所在 Session 的 data directory。
  - 按 Kodex URI-shaped locator 约定把 `file:///...` local path 或
    `file://server/share/...` UNC locator 转回 host path；不调用通用 URI parser，也不做
    percent-decoding。
  - POSIX local locator 去掉 `file://` 后直接得到 `/...`；Windows drive locator 把
    `file:///C:/...` 转成 `C:/...`；UNC locator 转成 `//server/share/...`。
  - 对 `memory:` 或未知 scheme 明确说明没有可读取的 filesystem Session，不猜测路径。
  - 不要求或推断独立 Session index；resolved Session path 经过 symlink 后不保证以数字
    index 结尾。
  - 使用 `current_session.name` 确认当前 Session，并以磁盘最新 settings 为最终历史真源。
  - 不从 Session locator 反推 Kodex Home。Skill 继续说明完整 Home 的 settings、logs、
    credentials 和 generated images；默认 Home 可说明为 `~/.kodex`，自定义全局 Home
    未知时不得猜测路径。
  - 说明当前 `index/work/settings/timestamp/token-count/unstable` 六条 timeline。
  - 说明 `latest.json`、`archive.mark`、`lock.json` 和 `version.json`。
  - 删除已废弃的 `stable/compaction/subagents` 当前布局描述。
  - 保留认证文件保密和 unstable 不得视为已完成历史的要求。
- Kodex Home context source 默认启用，且发现顺序位于 Agents Home 之后，因此同名的旧手动
  Agents Home Skill 无需先删除，内置文件会成为默认可见版本。Codex Home、自定义 source
  或 project 中更晚发现的同名 Skill 仍按既有 last-wins 规则覆盖它；用户显式禁用 Kodex
  Home context source 时继续尊重该设置。

## `0.3.5` migration

- 在 `KodexHomeMigrations` 中追加严格递增的 `0.3.5` entry；当前 `0.3.4` 构建忽略该
  future entry，发行时的独立版本 bump 才激活。
- 新 Home 和任何低于 `0.3.5` 的受支持 Home 都通过既有 migration 选择协议执行安装。
- Migration 目标是传入 `home` 下的 `skills/kodex-home/SKILL.md`，因此自定义
  `dataDirectory` 与默认 `~/.kodex` 使用相同语义。
- 每次执行先清理自己唯一命名的 stale temporary，再创建目标目录、写入完整 temporary，
  最后 atomic move 覆盖 `SKILL.md`。
- 不根据既有内容提前返回；每次 migration action 都 atomic replace 目标，因此目标
  `SKILL.md` 即使原来是 symlink，也稳定收敛为产品管理的普通文件。
- 允许 `skills/` 或 `skills/kodex-home/` 是指向 Home 内外的 directory symlink，并按
  filesystem 的正常路径语义跟随。
- 失败或取消时在不可取消清理中删除本次 temporary，不提前推进 `version.json`。
- 非目录的 `skills/` 或 `skills/kodex-home/`、无法覆盖的目标及其他 malformed
  filesystem 状态明确失败，不删除或猜测修复未知内容。
- Migration 只覆盖目标 `SKILL.md`，保留 `skills/` 下的其他 Skill 和目标目录中的其他文件。
- 发布 `0.3.5` 后，该 migration、Kotlin Skill String、fixture 和重入语义全部冻结；
  后续 Skill 内容升级使用更高目标版本的新 migration。

## 计划修改范围

- Agent storage：
  - `agent-storage/contract`
  - `agent-storage/contract-ext` tests
  - `agent-storage/filesystem`
  - `agent-storage/in-memory`
- Agent context：
  - `agent-context/prefix/contract`
  - `agent-context/prefix/impl`
  - `agent-context/prefix/render`
- Storage URI consumer：
  - `agent-state/impl`
  - `agent-session/filesystem`
  - `agent-session/in-memory`
  - `agent-runtime/impl`
  - `agent-runtime/decorator/turn-hook`
  - `agent-runtime/decorator/compact`
  - `agent-runtime/decorator/tool` tests
  - `hook/contract`
  - `hook/tool-utils`
  - `hook/impl`
  - `utils/logging`
  - `tool/web-run`
  - `tool/image-generation/impl`
  - `integration-test`
  - `app/viewmodel/history` tests
  - 受影响的 test fixtures
- Agent address 清理：
  - `app/contract/agent`
  - `app/viewmodel/agent`
  - `app/viewmodel/session`
  - `app/viewmodel/application`
  - 受影响的 test fixtures
- Skill 与 migration：
  - `app/migration/impl`
  - `.agents/skills/release-kodex/SKILL.md`
- 持久文档：
  - `checklist/agent-state-and-runtime.md`：记录 Storage URI-shaped identity 和 transient
    Session meta。
  - `checklist/cli-session-view-models.md`：删除 Agent address，改用 owner/object identity。
  - `checklist/kodex-home.md`：把 `skills/kodex-home/SKILL.md` 声明为现有
    user-managed skills 规则的唯一产品管理例外，并允许 `0.3.5` migration 写入。
  - `checklist/hooks.md`：把 native Hook input 的 `session_id` 改为 `uri`。
- 不修改：
  - AgentStorage schema 与既有 timeline payload
  - global settings schema
  - 已发布的 `0.3.3` migration
  - 当前机器的 Kodex Home 和手动安装 Skill

## 验证计划

- AgentStorage URI：
  - filesystem storage 对 resolved POSIX、Windows drive 和 UNC paths 分别生成
    `file:///...`、`file:///C:/...` 和 `file://server/...` 外观。
  - Windows separator 统一为 `/`，但 drive letter 与 UNC authority 保留 resolved 结果的
    大小写；UNC 不退化为 local path。
  - 空格、`#`、`?`、`%`、Unicode 和 POSIX literal backslash 原样保留，不进行 URI
    encoding、decoding 或 parsing。
  - in-memory storage 生成对象生命周期内稳定的 `memory:` locator；代表性不同对象的
    token 不同，但不为现有 identity hash 增加全局无碰撞承诺。
  - cached storage、Session wrapper 和通用 wrapper 原样转发 delegate URI，并保留 active
    check。
  - Contract 只通过 KDoc 定义 URI-shaped locator；不增加 runtime scheme/blank validation。
  - 全仓不再存在 `KodexAgentStorage.id` implementer 或 storage `.id` consumer。
  - 直接承载 Storage URI 的 Kotlin property/parameter 名为 `uri`；不存在
    `storageUri/sessionUri/agentUri`，也不误改无关的 `sessionId/id`。
- Prefix contract 与 renderer：
  - 精确渲染 `<current_session>`、`<storage_uri>` 和 `<name>` 的名称、顺序与数值。
  - 节点自身明确表示 URI 属于当前 Agent 所在 Session，而不是其他 Session 或 Kodex Home。
  - 正确转义 Session name 和 storage URI 中的 Prompt XML 特殊字符。
  - `file:` 与 `memory:` URI 都完整渲染，不省略 `<storage_uri>`。
  - 现有 `cwd`、`shell`、`current_date`、`timezone` 顺序不变，`current_session` 在末尾
    追加。
  - 空 `threadName` 仍渲染存在但内容为空的 `<name>`，不省略或生成 fallback。
  - Resolver constructor 仍只接收共享 context settings；每次 `resolve(settings, uri)`
    使用该次显式 URI。
  - 保持日期、时区、cwd、shell、AGENTS.md 和 skills 输出不变。
- AgentState：
  - 每次普通 Responses request 都包含该次 Agent settings snapshot 的 name 和
    `storage.uri`。
  - Session rename 后的下一次请求包含新 name，但 storage URI 不变。
  - 两个 AgentState 共享同一 `StateFlow<AgentContextSettings>` 并交错请求时，各自只渲染
    自己的 `storage.uri`。
  - 自定义 Application `dataDirectory` 下的 Session 投影精确的
    `file:` locator，恢复后的 path 是 `<custom-data-directory>/sessions/<index>`。
  - Session path 经过 symlink resolve 时，locator 指向 canonical data directory；context
    不增加 index，也不假设最后一个 path segment 是数字。
  - transient context 不进入 storage 或 remote compaction。
- Identity 一次性切换：
  - filesystem URI 在同一路径 reopen 时稳定，fork 和不同路径得到不同 URI。
  - `toCodexThreadId()` 直接投影新 URI；fixture 明确验证结果不同于旧
    `filesystem:<path>` seed，不实现 legacy 映射。
  - `HookSessionContext.uri` 只投影到 JSON `uri`，不再输出 `session_id`。
  - `logger.session(uri)` 附加唯一 `uri`；`logger.agent()` 和 tool scope 继承它，不再输出
    `session_id/agent_id`。
  - WebRun 内部使用 `uri` 投影到上游 `SearchRequest.id`。
  - Image output helper 继续清洗完整 URI，不增加 hash 或 legacy key。
  - `AgentAddress`、`AgentViewModel.address` 和 factory address 参数均不存在；Session
    owner 与 object identity 保持现有操作归属。
  - 既有 timeline、generated image 和 history path 不被迁移或删除。
- Agent ViewModel identity 清理：
  - Fork 使用 owning `PersistedSessionViewModel.sessionIndex`，并继续拒绝非 rootAgent
    object 作为 source。
  - Application popup ownership 通过 `PersistedSessionViewModel.rootAgent` object identity
    判断。
  - Request-user-input 的交错 call、revision、auto-resolution 和 ViewModel close tests 不再
    依赖 Agent address，仍保持实例隔离。
- Session repositories：
  - filesystem 的 create、open、reopen、fork 都从各自 storage 暴露准确 URI，不增加
    repository 或 Session factory 参数。
  - in-memory 的 create、open、reopen、fork 都暴露各自 `memory:` URI。
- `0.3.5` migration：
  - 从 `0.3.3`、`0.3.4` 和 unversioned baseline 升级后安装精确内容。
  - 自定义 Home 使用自己的目标路径。
  - 缺失、相同、不同或 symlink 目标文件都通过 unconditional atomic replace 收敛为产品
    regular file。
  - `skills/` 与目标 Skill directory 的 Home 内外 symlink 都按 filesystem 语义跟随。
  - 重复执行、temporary 残留和 atomic move 前中断可重入。
  - 失败不推进版本并清理自己的 temporary。
  - 保留 sibling Skills、目标目录额外文件和未知 Home 数据。
  - 当前 `0.3.4` 下 entry 保持 inactive，`0.3.5` 下按顺序激活。
  - 同一次 application 启动在 migration 完成后能发现该 Skill；同版本后续启动不执行
    self-heal。
- Skill 行为：
  - POSIX、Windows drive 和 UNC locator 都能指导 Agent 得到对应 Session data path。
  - 包含空格、`#`、`?`、`%`、Unicode 或 literal backslash 的 path 不做 percent-decoding。
  - `memory:` 和未知 scheme 不尝试 filesystem recall。
  - 不从 Session locator 向上反推 Kodex Home；完整 Home 指南保留，但自定义全局 Home
    未知时不猜测路径。
- Skill discovery：
  - 同名 Agents Home Skill 被 Kodex Home 内置版本覆盖。
  - 同名 Codex Home、自定义 source 和 project Skill 继续按现有顺序覆盖内置版本。
  - 禁用 Kodex Home context source 时不发现内置版本。
- Kotlin String 与发行：
  - String fixture 校验完整 frontmatter、必须章节、无 CR、无前导空行和唯一结尾 LF。
  - Migration 落盘文本与内嵌 Kotlin String 完全一致。
  - 运行 `agent-storage-{contract,filesystem,in-memory}`、
    `agent-context-prefix-{contract,impl,render}`、`agent-state-impl`、
    `agent-session-{filesystem,in-memory}`、受影响的 runtime/hook/viewmodel modules 和
    `app-migration-impl` 的 `allTests`。
  - 编译 Linux x64、Linux ARM64、macOS ARM64 和 Windows x64 release CLI。
  - 在 Windows x64 上对实际 resolved drive/UNC path 运行 locator smoke test。
  - 在 `0.3.5` 发行检查中，用隔离的 `0.3.4` Home 启动 CLI，验证 Skill 自动生成并能被
    catalog 发现。
  - 运行受影响 Kotlin 编译器检查、`git diff --check` 和临时文件清理检查。
