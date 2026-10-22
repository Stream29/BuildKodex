# Task Tree

- `Trace public Home preparation and application version consumers`()
- `Identify contract dependencies and the unchanged lease lifecycle`()
- `Confirm the two entrypoint migrations with the user`()
- `Implement declaration separation with subagent`()
- `Review defaults KDoc and unchanged Home preparation mechanics`()
- `Run centralized serial Native metadata and regression acceptance`()
- `Record the two API results and remaining gaps`()

# Details

- 用户已确认方案无问题并要求直接派 subagent 迁移。
  Native 目标收敛已派 Session1040，与本任务互斥写入；父 Session 讨论 MCP 下一项。
  [Session1042](file:///home/stream/.kodex/sessions/1042) 已返回编辑回执，
  父 Session 已完成集中验收。
  本任务实际编辑 app/migration 的两个公开入口，不执行真实 Home 数据迁移。
- CurrentKodexApplicationVersion 返回生成构建版本对应的 MigrationVersion；
  CoroutineScope.prepareKodexHome 检查/准备 Home，必要时执行已有数据迁移，
  返回与 owner scope 绑定的 KodexHomeHandle。实际生产调用集中在 app/backend-host；
  前端相关调用是测试 fixture，不能据此要求浏览器执行本地 Home 升级。
- 原两个入口在 cliHostMain，现已补 specMain 声明和 actual。
  Native 四目标收敛后，本模块与 cliHost 层覆盖一致；
  此前“宽模块/窄工厂”的平台障碍不再需要新增模块解决，
  但仍须真实 metadata/Native 验证，不能把目标移除等同于 API 已迁移。
- 最小路线：spec 声明 expect val 和 expect suspend extension；
  原属性与函数标 actual，保留 lazy、内部生成版本常量、数据快照和全部机制。
  默认参数与规范 KDoc 归 spec；SystemCoroutineFileSystem 默认值需要
  utils-kotlinx-io-coroutines/system 的公开 spec 依赖，不能反向依赖实现源码。
  不改函数签名、默认行为、迁移表或版本号，也不按短 inline extension 例外搬清理算法。
- 验收守住：版本/布局拒绝、读写 lease 转换、结构化 owner/取消、
  关闭等待与异常主次、迁移 checkpoint 及无迁移不重写版本。
  复用现有 HomeMigration/Checkpoint/HandleLifecycle 测试和 Native 后端消费者。
- 最终 provider metadata 与128项 Linux Native 测试通过，原 internal 算法和
  public 委托体逐字节一致；版本仍为原 lazy/生成值，新测试确认重复读取 identity。
  真实 CLI metadata/Linux 编译与链接、集成及 RPC server Native 测试源码编译通过，
  四个 Native provider 编译均实际执行并通过，非本机目标未运行测试。
  新调试 CLI 隔离 HOME 下执行 --help 返回0且不写文件；未执行真实用户 Home 升级。
- 并行边界：Session1040 不写 app/migration/** 或版本生成器。
  本任务只写 app/migration/**，不修改全局 convention、版本或后台启动实现；
  子任务不运行 Gradle/编译器、不提交推送，父 Session 集中串行验收。
  修改前1525份文件哈希及模块快照：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/home-api-before/`。
  严格/报告恰好移除两个入口，本批连同 MCP/Search 为29→18项/4模块，
  53个规则样例通过。未提交推送，证据：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/native4-validation-summary.json`。
- 依据：
  - [两个公开入口](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/src/cliHostMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigration.kt#L39)。
  - [已存在的资源生命周期契约](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/src/specMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeHandle.kt#L6)。
  - [后端准备 Home](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/backend-host/src/commonMain/kotlin/io/github/stream29/kodex/backend/host/BackendHost.kt#L103)。
