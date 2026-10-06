# Task Tree

- `Verify live settings values and legacy references`()
- `Map original value declarations to existing functional specs`()
- `Specify two actual store contracts and filesystem implementation`()
- `List legacy deletion and test transfer`()
- `Define wire, file, and host acceptance`()
- `Receive explicit implementation start`()
- `Cut over original values and actual filesystem stores`()
- `Publish exact API and test handoff`()
- `Receive centralized build and acceptance evidence`()
- `Receive user acceptance`()

# Details

## Scope

- 用户验收通过，随主任务归档 done；内层本地提交 `206266c8`，未推送。
- 父任务：[行为重新归属](2026-10-06-reassign-shared-application-behavior.md)。
- 基线 `e89c5094`；本子计划负责配置纯值、两侧文件存储、旧合并设置退役。
  后台认证/自动命名和 Hook 执行分别由其他工作单元负责。
- 不重新设计字段、文件格式、CAS 或关闭语义；不执行 Home migration/真实文件清理。

## Real value mapping

| 原声明，保持 FQCN | 目标 spec | 消费者 |
| --- | --- | --- |
| KodexAuthSource | `app/component/authentication-settings/spec` | Authentication VM、BackendSettings、backend auth、文件 codec |
| KodexNewSessionSettings | `app/component/new-session-defaults/spec` | defaults VM、BackendSettings、draft 创建、后端 defaults codec |
| SessionTitleSettings、DefaultSessionTitleModel | `app/component/session-title-settings/spec` | Title Settings、BackendSettings、后端自动命名；RpcConfigurationSettingsDependencies 只引用纯 default |
| SidebarContent、SidebarSettings、DefaultSidebarWidthColumns、MinimumSidebarWidthColumns、NewLineKey、SubmitKey | `app/component/application-preferences/spec` | preferences、Application SidebarSettingsViewModel、CliFrontendSettings、renderer |

- 上述模型从 KodexGlobalSettings.kt/SessionTitleGenerator.kt 拆出真实原声明，
  不通过 typealias、兼容 getter 或新 model→old model converter 维持两个 API。
- SidebarSettings 当前确有生产消费者，不是只属于旧持久文件：
  [sidebar spec](../../Kodex/app/contract/application/src/commonMain/kotlin/io/github/stream29/kodex/app/application/contract/SidebarSettingsViewModel.kt)、
  [真实投影](../../Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/SidebarSettingsViewModel.kt)。
  保留 constructor validation/default；实际启动宽度仍 applicationWidth/4，
  不恢复持久化宽度，不把 28 当新初始化策略。
- 原两来源 enum serializer、NewLineKey/SubmitKey 配对、model/default 值原样保留。
  文档按当前两来源可续期回存修正，不恢复旧 Codex 只读说明。
- 四个 spec 移除 `app-shared-settings-contract` 依赖，维持只有 pure spec/library 依赖。
  RPC models 增加这些 spec 和 Hook 模型 spec 的直接边，不指向 viewmodel/view。
- BackendSettings/CliFrontendSettings/CliSidebarSettings 保持原 RPC models，
  不建立可写 GlobalSettings 合并投影。

## Actual persistence contracts

- 新项目 `app/settings/spec/persistence` 与 `app/settings/impl/filesystem`，
  使用现行 convention；spec 只依赖 RPC models、coroutines 等真实签名所需依赖。
- 将原 `BackendSettingsStore`、`CliFrontendSettingsStore` 名称直接变为实际接口：
  - `settings: StateFlow<原模型>`、`settingsPath: Path`。
  - 原 `reload()`、`update(transform)`；后端另有原 `compareAndSet(expect, update, validate)`。
  - 原默认 validate 参数、返回值、异常与取消语义随操作迁入 spec KDoc。
- 私有/内部文件实现实现这两个接口；原 `openBackendSettings/openCliFrontendSettings`
  唯一工厂保留 FQCN，返回原名接口。真实 BackendGlobalState/RpcGlobalSettings/Application
  消费它们，不留 public concrete old class 或转发工厂维持平行类型。
- 工厂打开时 reload、missing-file 不生成文件、等值/stale CAS 行为、持久后发布、
  transform/validate 的锁内执行、原失败传播保持不变。
- 测试允许直接构造内部实现注入故障；迁移内部构造调用不能改变生产工厂行为。
- 同一 filesystem impl 项目保留两 store 与原 SplitSettingsFiles/Persistence；
  这是共用文件机制，不是合并业务域、共用 state/mutex 或前后端服务定位器。
- Kaml 私有文件 DTO/codec 留实现，RPC serializer 不随文件 snake_case 变化。
  前端不读后端文件，后端不读前端偏好；错误文件不被 defaults 掩盖。

## Delete and transfer

- 删除旧 KodexGlobalSettings、KodexGlobalSettingsStore、InMemoryKodexGlobalSettings、
  KodexSettingsStore/openKodexSettings 与旧 combined-file codec。
  其生产主张已由两侧 store 取代；live Responses probe 的引用由协调者直接接管。
- 旧 InMemoryKodexGlobalSettingsTest、旧 loader 原格式测试不应通过保留旧模型而续命：
  - 有效值模型 validation/键配对断言移到对应 spec 测试。
  - 两来源认证断言移到后端真实认证测试。
  - 已退役旧 combined store 行为测试删除。
  - file default/权限/失败的通用断言适配到真实 split store。
- BackendSettingsCasTest、SplitSettingsStoreTest 直接迁入新 filesystem impl；
  混合 suite 可分为 backend/frontend，只保留同一原子写入机制。
- KodexSettingsPermissionsTest 的有效私有文件权限断言适配到两侧 split-file；
  基线只有 combined-store 用例，不能删除整套测试掩盖私有文件权限证据。
- 原六个 app/shared 项目由协调者集中删除，当前子单元不跨线清理 auth/title/notification。
- 冻结 `app/migration/impl/.../v0_4_7` 与更早 versions 原声明/fixture 不动，
  migration 自己保留的历史模型不计入当前运行 API 重复。

## Acceptance and handoff

- 原 BackendSettings/RPC 及文件 golden 比对：serializer descriptor、字段/default/null、
  enum 值、unknown key、稀疏 defaults、YAML 非法/显式 null、Shell 缺省由实际 backend
  装配提供，spec 不执行 host discovery。
- Store 真实签名消费者测试：两个 open 返回唯一接口类型，所有正常 reload/update/CAS
  与故障测试使用它们；伪 spec 无生产消费者判为失败。
- 隔离 Homes 验证两文件互不覆盖、backend stale/equal/validate CAS、原子替换、
  permissions、取消和 cleanup suppressed；不用不同 mutex 假装跨进程 CAS。
- 四个 value spec `jvmTest`/主源码、`:rpc-spec-models:jvmTest`、
  `:app-settings-impl-filesystem:jvmTest`；不存在测试源码只记编译。
- `:app-migration-impl:jvmTest` 必跑；已发布 migration 生产 codec/算法和输入 fixture diff 必须为零。
  验收 harness 不保留已删除的合并 loader：以独立、显式的真实目标值断言迁移结果，
  不修改历史输入、不从被测 codec 生成期望值，不削弱凭据/默认值/中断恢复断言。
- JVM 原 Settings/Application/RPC renderer/VM 回归由协调者集中运行；
  输入键配对、Sidebars 初始化/宽度瞬态、Title default 展示保持原 UI。
- 后端线依赖本单元 pure specs/store 接口；交接包含各模型的原 FQCN、唯一接口签名、
  公共工厂、内部故障注入入口、codec/测试迁移清单。
- 不运行 Gradle/共享 IDE 的独立 worker 只写分配代码和交接；集中构建资源归协调者。

## Readiness

- 静态使用与项目依赖已核对，工程方案就绪；没有新增产品语义或未决外部依赖。
- 用户明确启动，已进入 executable。若执行时发现旧合并 store 的新生产消费者，暂停删除并补齐直接接管；
  不临时恢复旧 API、复制 DTO 或修改已发布 migration。

## Integration handoff

- 配置线已核验 `e89c5094b3ccd4f69ce4dbab8e8f30d1d8cb5df5`；以下 API 与路径为本次实施交接，
  **implementation ready**，集中构建/运行验证 **PENDING**。
- `:app-settings-spec-persistence` → `app/settings/spec/persistence`：
  `kodex.kmp-cli`；main API 依赖 `:rpc-spec-models`、coroutines core、kotlinx io core。
- 两个唯一接口均在 `io.github.stream29.kodex.cli.settings`：
  ```kotlin
  interface BackendSettingsStore {
      val settings: StateFlow<BackendSettings>
      val settingsPath: Path
      suspend fun reload(): BackendSettings
      suspend fun update(transform: (BackendSettings) -> BackendSettings): BackendSettings
      suspend fun compareAndSet(
          expect: BackendSettings,
          update: BackendSettings,
          validate: (BackendSettings, BackendSettings) -> Unit = { _, _ -> },
      ): Boolean
  }
  interface CliFrontendSettingsStore {
      val settings: StateFlow<CliFrontendSettings>
      val settingsPath: Path
      suspend fun reload(): CliFrontendSettings
      suspend fun update(transform: (CliFrontendSettings) -> CliFrontendSettings): CliFrontendSettings
  }
  ```
- `:app-settings-impl-filesystem` → `app/settings/impl/filesystem`：
  `kodex.kmp-cli` + serialization plugin；main API 依赖 persistence spec、
  `:utils-kotlinx-io-coroutines-impl`，implementation 依赖 Kaml 和 serialization core。
  同包保留原两个工厂（唯一实际入口，不转发旧工厂）：
  ```kotlin
  suspend fun openBackendSettings(
      settingsDirectory: Path,
      defaults: BackendSettings,
      fileSystem: CoroutineFileSystem = SystemCoroutineFileSystem,
  ): BackendSettingsStore
  suspend fun openCliFrontendSettings(
      settingsDirectory: Path,
      defaults: CliFrontendSettings = CliFrontendSettings(),
      fileSystem: CoroutineFileSystem = SystemCoroutineFileSystem,
  ): CliFrontendSettingsStore
  ```
- 工厂 reload 后返回；internal 实现/故障注入构造（同包）：
  `FileSystemBackendSettingsStore(fileSystem: CoroutineFileSystem, settingsDirectory: Path, defaults: BackendSettings)`、
  `FileSystemCliFrontendSettingsStore(fileSystem: CoroutineFileSystem, settingsDirectory: Path, defaults: CliFrontendSettings)`。
  原 `SplitSettingsFiles.kt` / `SplitSettingsPersistence.kt` 同包移动到此 impl，机制不拆分。
- 原 FQCN `io.github.stream29.kodex.cli.settings.*` 的真实值声明移入四个 component spec：
  auth → `KodexAuthSource.kt`；new-session → `KodexNewSessionSettings.kt`；
  title → `SessionTitleSettings.kt`；preferences → `ApplicationPreferencesValues.kt`。
  各文件仍在 `src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/`。
  `io.github.stream29.kodex.cli.sessiontitle.DefaultSessionTitleModel` 位于 title spec 的
  `src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/DefaultSessionTitleModel.kt`；
  旧 title 文件不由本线改动，后端线负责删除其旧声明。
- 四个 component spec build 本线移除旧 settings-contract 边并添加 serialization plugin/core；
  不添加 rpc/models 边。协调者负责 RPC models 对四 spec 的直接依赖、registry、
  所有其他旧 Gradle 文件及宿主/consumer 接线。
- 测试迁移：`BackendSettingsCasTest.kt` / `SplitSettingsStoreTest.kt` → 新 impl commonTest；
  原权限 suite → 新 impl jvmTest，转为两侧私有文件断言。
  combined-only suite 退役；有效 sidebar/key/value 断言移入四个 spec，
  增补实际接口工厂静态类型与 serializer/golden 证据。

### Implementation checkpoint

- 本线源代码与测试迁移完成；仅修改分配树、四个 pure-spec builds 和此子任务交接。
  原 settings 两个 build 文件仍留给协调者删除；没有改动旧 title 源码或冻结 migration。
- 两个接口的 KDoc 覆盖 defaults、锁内操作、stale/equal CAS、验证、异常、
  lost completion、可能已持久化及 cleanup 主异常优先语义；两工厂 reload 后返回唯一接口。
- 静态对比：`SplitSettingsFiles.kt` / `SplitSettingsPersistence.kt` 与基线逐字节一致；
  两个 store 的执行代码只改 interface/override、实现名与默认参数归属，操作体未变。
- 七个原值声明及两个宽度常量的执行/序列化声明与基线一致，仅修改过时 KDoc；
  非 migration 的 app/rpc commonMain 中七个值类型与两个 store 类型均为单一定义。
- 原 CAS suite 的 12 个和 split suite 的 18 个测试声明全部保留（包含循环生成用例，
  不是执行数量）。增补 CAS validate 的 skip/exact-current/失败传播与 lost completion、
  两侧 cleanup-only failure、工厂静态类型及 internal 构造测试。
- 新 `SettingsCompatibilityEvidenceTest.kt` 在 filesystem commonTest 承接有效旧断言：
  Codex config 不回退、MCP 清空、ultra→max、缺省 question mode、canonical update、
  context 不创建文件与 snake_case file golden。sidebar 最小宽度/默认值/键配对、
  原 descriptor 与 JSON golden 位于对应四 spec 的 commonTest。
- jvmTest 保留原权限 suite 名，覆盖 backend/frontend 的临时文件与最终文件
  OWNER_READ/OWNER_WRITE，以及本次临时文件清理。平台不支持 POSIX 时沿用跳过权限断言。
- 四 spec 的旧 settings-contract 边已移除，未引入 rpc/models 边；测试增加
  serialization-json 依赖。原 RPC `SettingsModelsTest.kt` / `NotificationHookModelsTest.kt`
  仍由协调者集中运行，不改其原协议证据。
- scoped diff/新文件 whitespace 检查通过；冻结 migration 源码/fixtures 当前 diff 为零。
  这些都是静态证据，没有执行 Gradle、测试、IDE、commit/push、分支或真实 Home 操作。
- 当前 `kodex.kmp-cli` convention 仅配置 JVM 与 Native targets，不配置 JS；
  集中验证须使用真实支持目标，不能把不存在的 `compileKotlinJs` 计为验证通过。

### Central validation

- 四个 pure spec 共 10 项、filesystem 61 项、RPC models 31 项、migration 59 项 JVM
  测试全部通过；实际接口返回类型、原 descriptor/golden、权限、CAS 和故障断言保留。
- 新 persistence spec/impl Linux x64 主源码编译通过；CLI/Integration JVM 编译通过。
- 协调者将 migration 验收 harness 的旧 loader 调用替换为独立目标值期望；
  生产 migration 与输入 fixture diff 为零，不通过保留旧模型维持测试。
- 完整宿主与平台范围见[统一验收证据](2026-10-06-reassign-shared-application-behavior.md#central-acceptance-evidence)。
  用户已验收，内层本地提交 `206266c8`，未推送。
