# Task Tree

- `Inspect all eighteen remaining declarations and their consumers`()
- `Probe Native entrypoints metadata contracts and the pinned patch ABI`()
- `Confirm the SafeRw compatibility versus implementation placement choice`()
- `Confirm implementation of the other seventeen migrations`()
- `Accept the three assigned migration groups centrally`()

# Details

- 用户要求一次性看完全部剩余点。复核时18项/4模块，隔离小样不算迁移完成；
  随后授权SafeRw改扩展并验收至17项/3模块。用户现已要求其余17项由subagent实施、
  父Session已完成验收，严格与报告17→0；不将此结果扩大为全仓所有测试/发布通过。
  独立存储测试失败不计入这些结构lint数量。

## RPC server — 7项

- [完整声明与职责](2026-10-22-migrate-rpc-server-exports.md)：5个后端类型、
  withBackendGlobalState/withBackendServices两个作用域入口。
  在原rpc/server自身spec声明、原机制actual；保留公开构造器、继承/独立成员、
  默认值、资源关闭顺序和准确OAuth操作绑定，不另拆协议项目。
- MCP签名依赖和Native目标已补齐，未发现新的架构选择；
  实施仍须metadata/Native及真实backend-host消费者验收。

## kRPC Native utils补丁 — 9项

- 保留原项目，不因上游包名带internal就把其公开符号改internal：
  预编译kRPC组件实际依赖这些符号。
- 2个opt-in注解完整移spec，保留message/level/元注解和Apache来源；
  RpcInternalConcurrentHashMap接口、嵌套Entry及已有expect工厂移spec。
  接口默认set行为保留，不机械将契约模型/接口expect化。
- getDeferred、operator set、getOrNull及hex转换声明expect/原体actual；
  它们当前不是inline，不为了短函数例外改变inline属性。
  私有字符转换和deferred完成/失败语义不改。
- RpcInternalThreadLocal保留公开无参构造和get/set/remove，spec expect类、
  common actual状态机制。internal currentThreadId按编译可见性接线，
  不随意收回原Native公开actual符号。
- 锁内复制entries/keys/values的补丁仍留实现；atomicfu仍为实现依赖。
  coroutines用于签名时须spec可见。
- 已读取固定0.10.3上游与当前补丁的Linux KLIB：唯一名均
  `org.jetbrains.kotlinx:utils`，同一2.4.20工具的signature-v2 ABI dump逐字一致。
  这是迁移前基线，不代表迁移后已兼容。实施后必须再次对照ABI/元注解，
  并验证原预编译kRPC消费者、取消/释放、map快照和Native CLI真实链接。
  不改替换坐标、KLIB唯一名、依赖版本或重新引入原utils KLIB。

## CLI main — 1项

- 在app/cli spec补同FQN的expect main(Array<String>)，原mosaicMain入口actual；
  保持convention现有entryPoint，不新增转发入口，不internal化绕过lint。
- 同版2.4.20独立Native小样已链接并运行，参数回显正常：
  入口本身不是必须加lint豁免的理由。
  实施后仍需真实CLI链接、隔离HOME的help/错误命令与退出码回归；
  小样不代表真实CLI已迁移验收。

## SafeRw — 1项，已确认改为扩展

- 现有类有两个suspend inline成员、EXACTLY_ONCE契约，以及value/rwMutex引用。
  用户允许的是短声明式inline extension，不自动覆盖本类/成员/状态。
- 2.4.20隔离结果：无体expect与actual中contract可编译，Native消费者初始化检查通过；
  但消费公共spec metadata失败，即使增加经refines编译的common actual metadata仍失败。
  同一消费者面对未拆分普通类的公共metadata则通过。
  直接在expect成员写contract body报告EXPECTED_DECLARATION_WITH_BODY。
  因而不能只看Native成功就宣称contract无损。
- 用户确认只将readSession/writeSession改成spec inline扩展，
  类/构造入口spec声明，状态actual留common；接受成员变扩展的API符号、
  导入及显式泛型实参变化，不采用整类例外或旧成员转发。
  [执行与验收已完成](../done/2026-10-22-migrate-safe-rw-contract-api.md)，在xiaoxin-ubuntu验证，
  该检查点严格/报告17项/3模块。spec-only与spec+common消费者均保留val初始化保证，
  这次取舍没有扩大整类例外；完整scope与测试限制见回执。
- 标准库2.4.20的AutoCloseable.use使用被抑制诊断的expect占位body表达contract；
  该先例不是本项目获准采用同样技巧的依据。本路线不建议用占位error、
  压匹配诊断或删除contract换取清零。
  [同版官方源代码](https://raw.githubusercontent.com/JetBrains/kotlin/v2.4.20/libraries/stdlib/src/kotlin/AutoCloseable.kt)。

## 验证与范围

- 17项已按现规则完成：RPC server、[kRPC补丁](2026-10-22-migrate-krpc-patch-exports.md)
  与[CLI入口](2026-10-22-migrate-cli-entrypoint-spec.md)互斥目录并行；
  父Session在xiaoxin-ubuntu集中串行验收。SafeRw不回退，不新增模块或通用框架。
  Native165项、metadata/真实CLI链接与smoke通过，补丁Native4及RPC server Linux ABI不变；
  规则回归53项成功结果缓存命中，严格/报告0项。
- 原存储2项失败及portable JS锁门禁是独立质量债务，不以结构lint清零宣称已修复。
  未提交推送，汇总：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/final17-validation-summary.json`。
- 复核小样在仓库外、单进程串行、固定JDK25，未启动Gradle、
  操作IDE或改变用户进程/全局配置。源码样例、命令、诊断与ABI基线留证据，
  临时可执行文件/klib/编译metadata目录清理。
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/remaining-api-review-evidence/summary.json`。
