# Task Tree

- `Confirm inline extension permission and freeze the SafeRw scope`()
- `Delegate API preserving migration or report a concrete blocker`()
- `Review the declaration shape and contracts from a consuming module`()
- `Run centralized serial metadata platform and lint validation`()
- `Confirm how to preserve contracts without a whole class exception`()
- `Implement spec inline extensions and common actual state`()
- `Validate metadata Native and real consumers on xiaoxin ubuntu`()
- `Record the scoped API compatibility change and acceptance`()

# Details

- 用户要求 subagent 执行当前 SafeRw 迁移，父 Session 同时讨论下一项。
  [Session1036](file:///home/stream/.kodex/sessions/1036) 已返回阻塞回执，
  保留所有生产代码，新增真实消费者回归；不能将其记为迁移完成。
  仅处理 utils/read-write-mutex 的 SafeRw 公开类、构造入口和两个会话 API；
  初始要求保留包名、泛型协变、成员调用形状、inline/suspend、EXACTLY_ONCE、
  挂起及非局部返回语义，不能删除 contract 使编译通过。
- 用户统一的例外是[非常短、近声明式的 inline extension](../../checklist/kmp-spec-impl-source-sets.md#short-declarative-inline-extensions)，
  不以显式 contract 为条件。SafeRw 当前是成员方法；
  原任务不擅自变更成 extension、移动整类及状态，或放宽此规则。
  若现有成员 API 无法在保留可见 contract 的同时无损分离，
  返回精确阻塞和最小待确认选择，不交付假通过。
- 子任务只写 utils/read-write-mutex 与必要的一个
  agent-session/filesystem 消费端测试文件。ReadWriteMutex 底层算法、
  既有断言、构建脚本、目标及其他迁移不改；不加桥接项目或 lint 豁免。
- 子任务仅编辑和静态复核，不执行 Gradle/编译器或提交推送；
  父 Session 待编辑稳定后集中串行验收。
  基线1524份文件哈希及模块快照：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/safe-rw-before/`。
- 已验收：原读写锁15项 Linux Native 测试，以及真实消费者新7项，
  覆盖 contract 初始化分析、挂起/非局部返回、异常/取消后释放锁。
  生产 SafeRw 源码与快照相同，当前成员实现的跨模块 contract 确实可用；
  这些通过不证明 expect 无体声明可表达同样的显式 contract。
  用户后续明确不再支持 JVM 产品，遵循四 Native 目标；子任务仍不改目标配置，
  JVM 构建工具与产品目标分开，旧 JVM 测试成功不能替代 Native 验收。
  该检查点为全仓18项/4模块，本类占1项。
  当时待确认的是成员显式 contract 的声明形状，不是读写锁算法；
  不将整类/状态或普通成员默认为短 inline extension 例外。
  测试与源码保护证据：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/native4-validation-summary.json`。
- 全部剩余点复核时补充了同版编译器隔离对照：expect/actual成员仅在actual保留
  contract时，Native消费通过，但公共metadata消费（包括refines后的common actual
  metadata）仍不能证明val初始化；原普通类公共metadata对照通过。
  expect成员直接写body则被拒绝。问题具体在公共契约消费，不是笼统“Native不能编译”。
  [小样与取舍](2026-10-22-review-final-spec-api-gaps.md)本身不构成例外批准。
- 用户随后以“行，那就这么改”明确批准只将两个会话操作改为spec inline扩展，
  类/构造入口spec声明、value/rwMutex状态common actual。此决定覆盖旧成员形状
  保留要求：接受成员符号变扩展、跨包import与显式泛型实参变化，
  不增加旧成员兼容转发或把整个类/锁算法搬spec。
  读/写视图、泛型协变、契约、挂起/非局部返回和释放机制必须保留。
- 只修改本模块SafeRw和必要调用者；原测试断言和底层锁算法不改。
  本轮1528份产品文件哈希及4份源码快照：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/safe-rw-extension-before/`。
  用户指定在xiaoxin-ubuntu验收，本机不启动Gradle；远端独立快照不覆盖已有工作区。
- 已完成：spec expected类型、构造入口及PublishedApi internal访问声明，
  common actual只保留value/rwMutex存储与原初始化。两个原成员的contract和锁内调用语句迁为
  spec inline扩展；未搬ReadWriteMutex算法、增加框架/转发层或修改构建配置。
  生产消费者只加2个import，消费测试仅加import和将显式<Unit>改为<_, Unit>，
  原断言、锁释放/取消及非局部返回机制不改。
- xiaoxin-ubuntu独立快照1529文件在构建前后哈希均匹配；
  provider及真实消费模块allMetadataJar通过。
  用真实新产物单独编译的spec-only和spec+common消费证明均通过，
  验证挂起读/写后外部val的确定初始化，不再仅依赖Native actual信息。
- Linux锁测试15项、真实消费端SafeRw契约7项、Wasm测试15项全部通过；
  整个Linux存储suite116项中仍有2项与既有精确基线同名/同类型/同消息失败，
  原失败测试源SHA也相同，未修改存储算法/断言。
  Native4的provider编译、真实CLI Linux编译及集成测试源码编译通过，
  JS/Wasm主代码、测试源码与测试可执行产物编译通过；非本机Native未运行测试。
- JS运行测试被kotlinStoreYarnLock的锁文件不匹配阻塞；未升级源码锁、
  排除校验任务或假称运行成功，另有[锁文件讨论记录](../discussion/2026-10-22-review-portable-js-lock-validation.md)。
  初次全量离线解析也因远端缺少CLI Ktor缓存失败，随后用远端已有凭据配置正常解析，
  未复制凭据或改变依赖版本/仓库策略。本轮未在本机构建。
- 严格/报告18→17项/3模块，逐条只移除SafeRw，53规则样例通过。
  全量命令仍因上述存储测试、JS锁和剩余17项严格lint非零，不称整仓全绿。
  1525份范围外旧产品文件不变，仅3份旧文件修改、1份spec新增；
  未提交推送。汇总：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/safe-rw-extension-evidence/summary.json`。
