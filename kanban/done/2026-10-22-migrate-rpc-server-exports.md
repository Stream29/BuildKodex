# Task Tree

- `Inspect the seven remaining server declarations and real host consumers`()
- `Confirm the existing module spec actual migration route`()
- `Freeze scope and acceptance checks after approval`()
- `Implement server declarations with the assigned subagent`()
- `Review unchanged mechanisms and complete public members`()
- `Run centralized metadata Native and host consumer acceptance`()
- `Record accepted exports and remaining independent failures`()

# Details

- 用户已要求“subagent去实现，你验收”，授权本组实际迁移；父Session负责复核，
  并在xiaoxin-ubuntu集中串行验收，子任务不构建/提交推送。
  [Session1052](file:///home/stream/.kodex/sessions/1052)已交付，父Session已验收。
  迁移前17项中，rpc/server占7项：BackendGlobalRpc、
  withBackendGlobalState、BackendGlobalState、BackendMcpOAuthBridge、
  BackendOAuth、withBackendServices、BackendServices。
  容器只报一次不代表只迁类名，所有独立公开成员及继承抽象成员都需完整契约。
- 这些声明装配后端服务、管理设置/MCP与OAuth资源，并把同一服务图注册到RPC连接；
  实际 app/backend-host 调用 withBackendServices。它们不是需要移往前端的wire DTO。
  GlobalRpc等共享协议已有契约，不新增contract/backend项目或重复协议。
- 确认最小路线：同包声明放rpc/server自己的specMain，原算法标actual。
  MCP后端类型已有spec，平台已统一Native四目标；签名/defaults需要的
  app-settings-filesystem、mcp-backend、openai-model-catalog等依赖按需上移
  本模块spec，不把私有I/O/日志等依赖一并上移。
- BackendGlobalRpc、BackendMcpOAuthBridge的公开构造入口保留；
  BackendOAuth的公开次构造器保留，内部注入deadline实现构造器不暴露。
  BackendGlobalState/BackendServices内部资源构造器仍留实现层；
  公开属性、register及操作要有spec。默认参数/KDoc归spec，实际机制不搬入spec。
- 验收重点为准备Home的前置条件、Session仓库延迟创建、借用与自有资源、
  消费者先关闭、取消及主异常/suppressed顺序、OAuth准确attempt绑定和RPC注册。
  不改协议、存储格式或服务准入，不访问真实账户/外部服务。
- 本组唯一写入rpc/server/**，允许必要的本模块spec依赖接线与小测试；
  不改RPC协议/客户端、backend-host、根配置或其他子任务目录。
  CLI与kRPC补丁并行编辑，父Session在三组源码稳定后统一构建。
- SafeRw已独立完成，本组未回退其扩展迁移。
  修改前1529文件哈希及本模块快照：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/final17-before/`。
- 已完成四份spec声明及四份common actual接线，public构造/成员保留，
  internal资源/注入构造器不进入spec。实际机制、字符串与原17份测试在声明修饰、
  KDoc和default迁移归一化后保持；默认参数完整保留在spec。
- xiaoxin-ubuntu：本组142项Native测试与迁移前精确快照相同，全部通过；
  provider/完整CLI metadata、真实Linux消费者编译、debug CLI链接及集成测试源码通过。
  RPC server的Linux Native ABI dump迁移前后逐字一致。
- 本组7项全部消除，三组总计严格/报告17→0。此项不代表其他平台已运行测试、
  全仓所有测试或发布验收；原存储失败与JS锁门禁仍独立保留。
  未提交推送。汇总：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/final17-validation-summary.json`。
- 依据：
  - [服务作用域与装配](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L33)。
  - [OAuth公开与内部构造入口](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendOAuth.kt#L56)。
  - [真实后端消费者](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/backend-host/src/commonMain/kotlin/io/github/stream29/kodex/backend/host/BackendHost.kt#L107)。
