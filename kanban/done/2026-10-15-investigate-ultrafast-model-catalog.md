# Task Tree

- `Compare the live backend catalog with a real authenticated provider response`()
- `Verify the live backend authentication source and identity`()
- `Reproduce catalog loading and authentication switching with the exact released binary`()
- `Identify the responsible code and distinguish evidence from inference`()
- `Remove diagnostic credentials and stop isolated processes`()
- `Implement authentication-driven catalog refresh with mapLatest`()
- `Validate account switching cancellation and failure recovery`()
- `Review the scoped diff and preserve validation evidence`()

# Details

- 用户在调查后授权用 mapLatest 修复。实现限于后端模型目录认证订阅、
  必要装配和回归测试；不重启现用后端、不部署、不提交、不发起模型推理。
- 保留固定 client 的原单参数目录工厂；后端 owned 工厂必须接收认证 store。
  使用 mapLatest 取消旧请求，账号切换/退出清掉原账号远端目录；
  同账号刷新失败保留原快照，新账号失败只保留内置回退。失败不能终止后续观察。
  显式 refresh 同样检查认证快照，防止在途旧请求写回。
- IDEA MCP 连接失败，已尝试识别项目；后续用原生读写和差异检查。
  构建仅在 remote MacBook 的本任务独立目录执行，先探测 JVM 与资源占用。
- 已实现认证订阅、mapLatest 取消、身份变化回内置目录、同账号失败保留、
  自动及显式 refresh 的认证快照检查；后端装配必须传入认证 store。
  新增六项目录回归，既有 RPC OAuth 用例增加登录后目录更新断言。
- Mac 私有验证根：`~/ACodeSpace/local/kodex-model-catalog-870-validation`；
  APFS 克隆现有 source/Gradle/Native 缓存后同步本次源码，不改共享缓存。
  启动前无 Gradle/Kotlin Daemon；明确选择现有 Java 25，使用原设备锁
  `~/ACodeSpace/demo/kodex-gradle-research-445/device-heavy.lock`，
  验证期间保留锁并监控其他 JVM/IDE 接管、内存/swap 与30分钟上限。
  批次 `jvm-native-01` 运行目录 JVM、RPC server JVM、目录 macOS Native。
- 现用后端 PID 3780096，0.4.15，2026-10-15 16:50:49 +08:00 启动。
  `/proc/3780096/exe` 与磁盘发布二进制 SHA-256 相同：
  `ce35d3144dd4143f247c217a7069d1f0cb83c7fc0417297b8f186c661cf50ffd`。
- 只读 GlobalRpc 确认：当前 authSource=kodex、plan=ProMax；
  accountId/email 与 `.kodex/auth.yml` 派生信息相同。另一 Codex
  凭据源属于不同 ProLite 账号。文档及诊断输出不保存账号标识或凭据。
- 当前后端目录的 Astra / Sol 6.1 都只有 priority。
  使用真实 Kodex access token、派生账号头、相同 client_version=0.159.2 /
  originator=codex_cli_rs / User-Agent 直接 GET `/models`，两模型都列出
  priority 与 ultrafast。只改 client_version=0.162.1 或 originator=kodex
  的对照同样有 Ultrafast；不因此修改生产身份或兼容版本。
- 直接请求另一 ProLite 源，两模型都只有 priority。初次逐字段比较未模拟
  ReasoningEffort 的 ultra→max 读取兼容及稳定去重，出现差异；补齐实际
  生产解析规则后，全部九个模型的所有 ModelInfo 字段与顺序完全相同。
  目录内容相同不是启动账号的审计记录，不据此断言历史切换时间。
- 代码目录只在初始化时自动请求一次；认证状态变化、设置 reload 和前端
  getModels 都没有调用目录 refresh。当前没有启动时原始模型响应或目录
  来源时间/账号日志，不能把“启动时远端缺项”当作已确认事实。
- 已用相同发布二进制、独立 HOME/端口与稀疏设置复现。
  临时凭据保留真实 access/id token，只将 refresh token 换成不可用占位值，
  防止诊断旋转用户凭据；不复制 Session、MCP 或 Hook 配置。
  脚本 finally 关闭自己的 RPC、终止自己的隔离进程并删除整个临时 Home。
- 初期两轮隔离超时；随后显式检查认证发现是诊断脚本将 YAML timestamp
  经 PyYAML 重编码为空格格式，导致 Kotlin Instant 加载失败。改成保留
  原始文件文本、只替换 refresh token 后认证正常。也将不够通用的
  gpt-reserve 判据改为内置目录特有 gpt-5.5 消失。早期失败不算产品失败。

## 调查结果（修复前，基线 082bfc66）

| 使用相同发布二进制的隔离场景 | 当前认证 | Astra / Sol 6.1 目录 |
| --- | --- | --- |
| 直接以 Kodex 凭据启动并等待远端目录 | ProMax | priority、ultrafast |
| 直接以 Codex 凭据启动并等待远端目录 | ProLite | priority |
| 隔离 RPC CAS 将 authSource 从 codex 改为 kodex | 已切到 ProMax | 仍只有 priority |
| 切换后再等待 3 秒、12 秒 | ProMax | 仍只有 priority |

- 确认缺陷是后端模型目录与认证生命周期脱节：目录只收到 OpenAiClient，
  初始化请求一次，未观察认证变化；认证 store 独立观察 source 并发布新账号。
  对照账号用量组件已观察 authStore.state，并在账号变化时处理旧值。
- 相关位置（工作区现已包含修复）：
  [认证与目录组装](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L59-L69)；
  [目录接入 client 与认证 store](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalState.kt#L86)；
  [目录认证驱动刷新](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/model-catalog/src/commonMain/kotlin/io/github/stream29/kodex/openai/modelcatalog/OpenAiModelCatalogStoreImpl.kt#L49-L91)；
  [认证源变化后重载](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStore.kt#L60-L67)。
- 同一二进制的 ProMax 冷启动成功，排除当前版本无法解析或跨 RPC 传输
  Ultrafast 的解释；真实后端认证对照排除当前账号选错的解释；
  0.159.2 原始响应有 Ultrafast，排除必须升级兼容版本的解释。
- 现用实例高度符合账号切换后残留旧目录，但没有保存启动目录请求的账号、
  时间与原始响应，不能把历史来源推断写成取证事实。
- 修复边界：在后端将目录绑定当前认证身份，在认证身份变化时刷新并防止
  旧请求覆盖新账号结果；不能仅改前端选项或硬合并内置 Ultrafast。
  调查阶段未修改产品；后续获准实现仍不重启、reload 或 CAS 真实后端。
- 所有隔离进程均在 finally 终止并确认退出，临时 Home（含 access/id token）
  已删除；诊断 Python 脚本在完成后清理。不保存真实凭据、账号标识或原始响应。

## 修复验证

- `jvm-native-01`：54 秒，Gradle 成功；XML 回读确认目录 JVM 15 项、
  RPC server JVM 141 项、目录 macOS Native 15 项，合计171项，
  failures/errors/skipped 均为0。新六项在 JVM 与 Native 各执行一次。
- 覆盖未认证启动/登录、ProLite→ProMax及反向切换、退出、mapLatest取消、
  忽略取消后迟到结果、同账号失败保留、跨账号失败清理、失败后恢复、
  显式 refresh 跨账号拒绝、关闭后不再观察。RPC OAuth 用例确认登录后
  无需重建服务或连接即可观察新模型。
- 最终补齐 nullable 参数/属性 KDoc 后，批次 `final-jvm-native-02`
  三个测试任务均实际执行，30秒成功；XML回读仍为15/141/15项，
  failures/errors/skipped 均为0。两轮均未触发资源守卫。
- 报告、最终源码补丁和清理回执保存在
  [本机验证证据](file:///home/stream/ACodeSpace/local/kodex-model-catalog-870-validation/evidence/)。
  停止本任务私有 Daemon 后删除 Mac 的 source/overlay/Gradle/Native 缓存；
  原设备锁 inode `106252171` 不变。Mac仅保留不含凭据的证据及runner。
- 未改变 RPC 契约、service tier 能力筛选、远端完整替换规则或用户设置；
  未提交、安装、发布或重启现用后端。
