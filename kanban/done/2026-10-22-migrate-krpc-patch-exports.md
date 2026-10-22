# Task Tree

- `Freeze the pinned upstream ABI and patch source snapshot`()
- `Implement spec declarations with the assigned subagent`()
- `Review unchanged lock snapshots and annotation metadata`()
- `Compare Native ABI and validate real precompiled consumers`()
- `Record scoped acceptance and remaining failures`()

# Details

- 用户已授权剩余17项由subagent实施、父Session验收；本组占9项。
  [Session1053](file:///home/stream/.kodex/sessions/1053)已交付，父Session已验收。
  只有rpc/krpc-utils-patch/**可写，允许本模块必要spec依赖接线；
  子任务不运行任何Gradle/编译器、不提交推送、不改convention/坐标/版本。
- 按[完整复核](2026-10-22-review-final-spec-api-gaps.md)迁移：
  两个opt-in注解、map接口/嵌套Entry和已有expect工厂进spec；
  deferred三函数和hex函数expect/actual，ThreadLocal类expect/actual，
  保留公开构造及成员、注解、默认值、泛型、原Apache来源。
- 不因包名internal降低公开可见性，不改变inline属性或引入转发层。
  锁内entries/keys/values快照、线程/worker身份和异常/完成语义不改。
  currentThreadId内部expect及既有Native actual符号须保持兼容。
- 固定0.10.3上游与当前补丁的Linux ABI dump逐字一致，唯一名
  org.jetbrains.kotlinx:utils；基线SHA-256：
  `212f08244e096a064211a7595cfdcd2410e3b4f9fa59e5cfd8417cbf7f0de1c7`。
  不能把迁移前一致当作迁移后验收；父Session需重比ABI/元注解，
  编译和运行真实预编译kRPC消费端、取消/释放与快照回归并链接Native CLI。
- 产品目标为Native4，保持Gradle/detekt的JVM工具；不恢复JVM/Node产品。
  与RPC server、CLI互斥目录编辑，不能碰已完成SafeRw或共享任务文档。
  基线源码：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/final17-before/`。
  ABI证据：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/remaining-api-review-evidence/`。
- 新增3项ThreadLocal/deferred/hex边界回归，原map实现与断言不变。
  注解/接口迁移为整文件原样移动；其余普通函数体仅加actual，
  ThreadLocal状态不搬spec；父Session补充既有模式的窄beta警告标记，
  未压匹配错误或修改lint。
- xiaoxin-ubuntu实际执行Native4 provider编译；4个目标ABI均与迁前/固定上游
  Linux基线逐字相同，SHA均为上述值。两个RequiresOptIn消息/级别以及8处
  InternalRpcApi marker计数/值与原metadata一致。
  迁后4项补丁测试及真实预编译kRPC消费的19项in-memory RPC测试全部通过，
  全部162项迁前测试身份保留；加3项新回归后合计165项Native全部通过。
- 实际CLI debug链接和隔离命令检查通过，9项全部消除；严格/报告三组17→0。
  53个规则回归结果缓存命中且全部成功；未改变规则、框架或编译检查。
  未运行其他Native架构测试，不发布release，未提交推送。汇总：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/final17-validation-summary.json`。
