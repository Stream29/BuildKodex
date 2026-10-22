# Task Tree

- `Identify backend exports and real RPC consumers`()
- `Separate backend spec ownership from shared protocol ownership`()
- `Confirm declaration shapes and resource invariants with the user`()
- `Implement backend spec actual declarations with subagent`()
- `Review public members and unchanged private mechanisms`()
- `Run centralized serial Native and real consumer acceptance`()
- `Record accepted exports and remaining gaps`()

# Details

- 用户以“处理吧”批准 mcp/backend 的8处导出迁移。
  任务由 subagent 编辑、父 Session 集中串行验收，现已完成。
  [Session1044](file:///home/stream/.kodex/sessions/1044) 已返回最终编辑回执。
- 范围是4个公开类：McpManagerImpl、McpServiceImpl、DefaultMcpOAuthClient、
  PreparedMcpOAuthLogin；前3类各有一个同名 CoroutineScope 工厂；另有
  validateMcpConfigurationUpdate。容器报一次不等于只迁顶层名字，
  各自公开成员、继承接口和公开构造参数也需完整契约。
- 它们服务后端管理、运行连接、OAuth准备/令牌刷新与配置持久化校验；
  rpc/server 实际调用这些工厂/附加操作，后续该模块的导出签名也使用相关类型。
  先补这一层可闭合后续 RPC server 的契约依赖，不需要再次拆分 MCP 项目。
- 目标归属是 mcp/backend 自己的 specMain，不是 mcp/contract。
  共享协议模型与轻量端口仍留原位；不能把管理状态、SDK、I/O、凭据或 PKCE
  交换闭包带入前端共享依赖。Backend 内允许自身 spec 暴露必要后端 API 类型。
- 最小路线：同包 expect 类/操作与创建函数，原机制 actual。不靠 internal 化
  收回已有导出，不复制 DTO 或新增转发层。internal 实现构造器及私有状态
  不因迁移变成公开构造能力；PreparedMcpOAuthLogin 的现有公开构造器则保留
  exchange 回调参数，但不将私有 exchange 属性公开。
  McpServiceImpl 的 CoroutineScope 能力及所有继承抽象成员也需在 metadata 验证。
- 保留配置 CAS/身份校验、客户端 owner 复用与关闭顺序、取消/异常处理、
  prepare 不开回调监听器、PKCE/令牌与 RPC 形状。回调监听、网络和进程算法留实现层，
  不能按“短 inline extension”例外搬整类机制。
- Native 四目标收敛后，原 cliHost 窄平台障碍应随目标对齐解决，
  但不把目标删减当成8项已完成。须复跑 MCP Native 回归及 RPC server/后端消费者，
  不连接真实外部服务或使用用户凭据。
- 并行边界：Session1040 拥有 Native/convention 及必要构建接线，
  Session1042 拥有 app/migration，Session1036 拥有 SafeRw。
  三组均已返回编辑回执；Native/Home 已完成集中验收，
  SafeRw 生产迁移仍有 contracts 阻塞，独立留待确认。
  本子任务只写 mcp/backend 源码/测试，不修改构建脚本或其他模块；
  必要 spec 依赖如不足先报告，由父 Session 在稳定后处理。
  子任务不运行 Gradle/编译器，不提交推送。
- 修改前1526份文件哈希及 MCP backend 快照：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/mcp-api-before/`。
- 父 Session 已将3个 internal 构造器及其资源默认值留回实现层，
  修正继承默认参数重复、actual 默认值和属性显式 public。
  Scope getter 仍委托同一个 scope。四个实现文件经这些声明差异规范化后与快照一致，
  业务/网络/PKCE/关闭算法未变。
- 验收：最终 provider metadata 与31项 Linux Native 测试全通过。
  Prepared 公开边界测试还确认构造期间不提前调用 exchange。
  RPC/backend 消费者及完整 CLI metadata、Linux 编译/调试链接、
  RPC/集成测试源码编译通过；未连接真实 MCP/OAuth 服务。
  严格与报告逐条恰好移除本模块8项；连同 Home/Search，本批29→18项/4模块，
  53个规则样例通过，严格入口因剩余18项仍正确失败。
- 本批合计369项 Native 测试，367通过，2项存储失败在本批迁移前精确工作区快照中
  同名/同类型/同消息复现，不属于 MCP suite；不称作干净 HEAD 复现。
  未运行其他系统/架构测试，未提交推送。证据：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/native4-validation-summary.json`。
- 依据：
  - [管理器与工厂](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/backend/src/cliHostMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt#L57)。
  - [OAuth准备及公开载体](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/backend/src/cliHostMain/kotlin/io/github/stream29/kodex/mcp/impl/McpOAuth.kt#L62)。
  - [后端创建MCP资源](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalState.kt#L72)。
