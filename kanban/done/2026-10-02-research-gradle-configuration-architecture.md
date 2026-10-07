# Task Tree

- `Read parent task and preserve current module design constraints`()
- `Inventory configuration and platform-model costs`()
- `Classify build logic and plugin isolation blockers`()
- `Design compatible developer target profiles`()
- `Compare mature Gradle optimizations and Declarative Gradle`()
- `Acquire the shared heavy-operation lock for isolated prototypes`()
- `try {`
  - `Prototype prioritized changes outside the active checkout`()
  - `Validate semantics and measure configuration effects`()
  - `Provide genuine Sync scenarios to the sibling session`()
- `} finally {`
  - `Stop owned resources and release the lock`()
- `}`
- `Specify a staged implementation and validation plan`()
- `Report recommendations without changing production code`()
- `Complete coordinator review of the research evidence`()

# Details

## Ownership and authorization

- Parent：[本期 Gradle 开发体验](../executable/2026-10-02-rescue-gradle-development-experience.md)。
- 状态：Session **455** 已完成研究交付，协调 Session **445** 已核对最终模型/源字节/测试数据，并结合 Session454 的真实 Sync 联测完成研究验收与归档（2026-10-02）；不以此授权生产迁移。
- Checkpoint：静态盘点与官方版本核对开始；不启动本机 Gradle/IDE，不修改活动代码。远端仅在共享锁内运行自有隔离原型；输出使用 `configuration-architecture/` 子目录，原报告/基线只读。
- 静态 checkpoint：冻结 Git 对象含 130 个应用 convention 的模块构建脚本；settings 还创建容器项目，故 142 个子项目不等于 142 个 KMP 模块。原 HTML 的 `uniqueProblemCount=2047`，50,158 是总记录，原约 747 是另一个近似归并口径；将按根因重新归类，不解释成独立缺陷数。
- 版本 checkpoint：官方 current 已为 Gradle 9.8.0；9.5.1 文档下载证实使用 `org.gradle.unsafe.isolated-projects`，9.3 起 tooling model cache 因正确性问题禁用。不得套用新版开关或宣称 9.5.1 Sync model cache 收益。
- 原型 checkpoint（02:56–02:58，Xiaoxin，共享 flock）：project-local kRPC convention 的完整 `help` 与模型盘点通过；Isolated Projects 仍失败（50,015 总记录），不能宣称已解决插件阻塞。JVM profile 初次遇到 buildSrc helper 为 `internal`、叶脚本不可访问的构建逻辑可见性错误；保留失败日志并改为公开构建逻辑 helper 后续验证。自有 Daemon 120579 已停止、锁已释放；等待下一次原子领取，未抢占新持有者。
- 联测场景已供协调会话读取：[SYNC-SCENARIOS.md](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/configuration-architecture/SYNC-SCENARIOS.md#L1)。只定义实际 IDEA 对照和源码导航验收，不包含真实 Sync 改进声明。
- 静态阅读可与两个兄弟任务并行；本机只读，禁止在正在开发的 Kodex 工作树启动 Gradle、Sync 或改文件。
- 隔离原型目录：Xiaoxin `~/ACodeSpace/demo/kodex-gradle-research-445/configuration-architecture/`。
- 实验源码从已提交 archive/独立副本创建，不覆盖本期 source/binary/results；重型操作遵循父任务共享锁。
- 仅维护本任务和自己实验输出，不变更用户其他任务、规范或生产构建配置。

## Final checkpoint and handoff — Session 455

- 报告：[configuration-architecture/REPORT.md](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/configuration-architecture/REPORT.md#L1)。原 Session 445 的父目录报告、基线与结果未覆写。
- 原型、control 和 DCL 小原型位于 Xiaoxin 同名 `configuration-architecture/` 子目录；本机同目录保留报告、脚本、最终构建逻辑镜像、原始 results/ 与两个新 IP HTML。
- 最终统计在 KGP hierarchy finalization 后采集，纠正早期 afterProject 快照偏小的问题；不用早期 2,010 source sets / 29,090 configurations 冒充最终数量。
- 保持 142 个子项目、130 个 KMP 模块；local/full 与原配置代理的最终 target、source-set 祖先/目录和配置/任务数量逐项目等价：880 targets（含 metadata）、3,292 source sets、44,370 configurations、26,540 注册 task 名。
- JVM profile 为 260 targets（含 metadata）、550 source sets、9,410 configurations、9,231 注册 task 名；130 项目的 JVM main/test 祖先图与 full 完全一致。所有 894 个产品源文件与冻结 Git 对象 SHA-256 一致。
- 最终移除早期手工补 dependsOn 调用，再运行四组测试，69 个 JVM 测试通过；没有为性能合并/移除 spec/impl 或收紧任何 api 导出。
- 同一 Daemon 交替热态 help：full 三次 4.87/4.22/3.79s、中位 4.22s；JVM 三次 2.13/2.06/2.09s、中位 2.09s。只表示配置代理；未测真实 Sync/索引/RSS，不外推为其提速。
- kRPC dependencyInsight 确认原 `0.10.3` 仍替换为本地 patch。生产当前路径只读核对为 `:rpc-impl-krpc-utils-patch`，不可复制 main 的旧路径。
- IP 原报告 50,158 总记录 / 2,047 自带去重行；local/full 为 50,015 / 2,046，JVM 为 11,240 / 1,572，均失败。主要剩余为 TestBalloon、KGP JS/Wasm/root 状态和 kRPC compiler 属性访问；这些计数不是独立缺陷数。
- DCL 独立小原型通过：Gradle 9.6.0、ecosystem 固定 0.126.0、KGP 2.4.20、JVM 25；有实际 spec/impl、serialization 和接口行为测试。首次 JVM software-type/task 不匹配失败保留，最终 `common+jvm` KMP 编译与 1 个 JSON/契约测试通过；不说明主仓库 Native/插件/IDE 生态已可迁。
- 输出包含优先级、收益/风险/版本/成本/rollback、当前 spec/impl 分支的可执行生产批次与验收矩阵，均待单独审阅授权。
- 清理：所有重型调用通过共享 flock；每轮停止自己的 Gradle Daemon 后释放。未运行 IDEA、后台采样、Native 编译或改网络/设备；未停止兄弟或用户进程。
- 未验证：全仓 JVM、主仓库 Native/JS/Wasm/iOS、RPC round-trip、CLI/真实 IDEA 源码导航与 Sync。原型是无 fork 配置代理，需要与发布/Sync 两个任务的完整 binary 场景联测，不冒充可生产主工程。

## Canonical constraints

- 读取 [JVM toolchain](../../checklist/jvm-toolchain.md) 和 [spec/impl boundaries](../../checklist/spec-impl-module-boundaries.md)。
- 用户正在 [spec/impl 迁移](../executable/2026-09-28-plan-domain-spec-impl-boundaries.md)；每个 implementation 项目需要对应有意义的 spec 项目，不能以合并或空壳 spec 换性能。
- 保留公共 ABI、KDoc、RPC/持久化形状、source-set 语义和实际发布 target 覆盖。
- 正常 JVM toolchain 为 25；Desktop 若涉及编译/运行需遵循 JBR 25 约束。
- 历史 [依赖审计](../done/2026-07-26-audit-gradle-dependencies.md)已有 ABI 驱动的 api/implementation 调整；不能机械把所有 api 降为 implementation。
- 冻结研究基线为 main `3008a6de9f178cda7cd63123a0261d75013f772d`；本机活动分支可能项目更多，差异需只读标记，不能混入不同版本的性能对照。

## Starting evidence

- [根构建报告](file:///home/stream/ACodeSpace/demo/kodex-gradle-research-445/REPORT.md#L55)：移除 fork 后仍有 142 个主项目。
- 各 KMP convention 声明 JVM、linuxX64、linuxArm64、macosArm64、mingwX64，单模块命令也可能触发完整初始化/模型准备。
- Isolated Projects 9.5.1 探针失败：50,158 次记录，约 747 个近似去重项；不是同等数量的独立缺陷。
- 已见阻塞：根脚本跨子项目访问 configurations、TestBalloon `1.0.1-K2.4.0` 动态父属性查找。完整日志和 HTML 位于本期报告目录。
- 普通 configuration cache、build cache、IDE model import 和 Isolated Projects 分开评估；cache-hit help 不等于 Sync cache。
- 本期 CLI help 数据只能作为初始配置基线；与 Sync 子任务共享验证场景，最终收益不凭假设宣称。

## Required research and prototypes

- 盘点 settings 扫描、buildSrc/预编译 convention、重复插件/仓库、eager resolution、跨项目配置、target/cinterop、api 暴露与实际模型数量。
- 为 Isolated Projects 报告按根因归类，定位项目代码与插件来源；验证当前版本能力和版本升级的兼容性，不套用最新版开关到旧版本。
- 评估把 root subprojects 配置迁到 project-local convention，减少 mutable 跨项目访问；不规避真实契约依赖。
- 评估开发 profile：JVM-only 或 host-focused 与完整 CI/release profile 分离，保持 mosaicMain/Native hierarchy、expect/actual 和 sources attachment 正确。
- 评估各 convention 的插件/任务按需应用，配置 avoidance、并行策略和适当 Daemon/compiler 内存；不得以过量并发与更大堆掩盖模型成本。
- Declarative Gradle 依官方最新 status/roadmap、KMP/plugin/software-type 支持评估；如可行做独立小型原型，不迁移活动主工程。
- 每个提案记录收益、代码/维护成本、IDE/Gradle/Kotlin version constraints、撤销方法和验证矩阵。
- 至少完成优先方案的隔离原型和语义检查；重型 benchmark 持共享锁，实际 Sync 效果与 Sync 任务联测或明确尚待该场景完成。

## Output and completion

- 输出按优先级排序的整改路线：fork 发布之外的瓶颈、开发 profile、project-local build logic、插件适配与长期 DSL 选择。
- 分清可验证低风险改动、需要用户设计决策的改动和无法完成的平台/插件阻塞。
- 给出针对当前 spec/impl 设计的可执行实施计划、验收任务与影响范围，不只列泛化最佳实践。
- 原型/指标保存在自己目录并链接到本任务；最终生产实现需要用户审阅和单独授权。
- 所有本任务重型资源停止后释放锁，再交接给协调会话。
