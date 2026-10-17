# Task Tree

- `Freeze the authorized scope and current product inputs`()
- `Remove unused wiring, factory trampolines and duplicate helpers`()
- `Narrow the two validated recovery boundaries and add regression tests`()
- `Validate metadata, consumers and selected JVM and Native tests on MacBook`()
- `Review API documentation, preserve evidence and release owned resources`()

# Details

## Authorization and scope

- 用户在[主实验与独立复核结论](2026-10-17-verify-ablation-audit-findings-878.md)
  交付后要求落实有价值的修改。本轮实施 A1–A5、D1–D3，
  加上独立复核已核实的 `tool/core` 两项重复 `specMain api` 声明。
- 默认保留现有顶层创建函数、依赖 ports、ViewModel 契约和原 owner/关闭语义；
  删除九组无其他仓内消费者的 Factory 接口/默认单例，不保留别名或转发兼容层。
  其规范性创建/生命周期 KDoc 合并回所属 `specMain` ViewModel 契约，
  不把它们直接丢弃或仅留在 commonMain。
- Composer 复用既有 composer-view 依赖，公开两个确有跨模块消费者的纯 renderer
  helper 并声明边界/异常 KDoc；不新建模块、Controller 或第二份状态。
- D1 只合并重复异常分支。D2/D3 明确保留 IOException 等普通失败的 typed
  failure/warning，传播取消和 Error；正式回归覆盖正反故障，而非套用全仓替换。
- 不纳入 SHA-256 共享、旧 settings adapter 搬迁、SafeRw 替换、Patch 模型压平、
  Skill resolver 扩展或其余静态待审项。保留已反证的 owner/generation/取消分类检查。
- 不改变 RPC/持久化形状、冻结 migration/codec/fixture、平台覆盖、依赖破环边界；
  不修复基线已有 Composer host 超时来制造全绿。
- 实施阶段不提交、发布或重启真实后端；后续用户单独授权提交推送，见文末。
  保留并行默认堆 6g、Ktor 记录及旧审查任务阶段。

## Baseline and validation gates

- 产品 HEAD：`adbe5215867d0af283feb5d69a6d2b0b42035273`。
  初始产品改动仅 `gradle.properties` 默认堆 4g→6g，不属于本轮。
- 本机 IDEA 已检测为打开 BuildKodex；当前未暴露 IDEA MCP 工具。
  不操作其窗口、同步或构建；代码修改使用原生文件工具。
- Gradle 全部在 remote MacBook 私有副本执行；使用既有共享设备锁、
  明确 JVM、4g/2g 堆与单 worker 预算、内存/swap/时间及其他使用者接管守卫。
- 再跑未修改基线的代表性 JVM 编译/测试，记录原 Composer 40000 ms 超时。
  候选验证 140 个 metadata、全仓生产/测试编译和所改模块的实际测试；
  对既有失败逐项比较，不跳过新增故障或篡改断言。
- 验证九组件、Composer/Agent renderer、Path Picker、AGENTS、AgentState、
  Tool 和读写锁；在 Mac Native 复验适用的组件/故障测试并编译 CLI。
  不把 compile/metadata 当作真实 IDE、交互 TUI 或其他 OS 的运行验收。
- 收尾核验主工作区并行输入、私有副本与候选输入一致，停自有 JVM，
  释放设备锁并清理不再需要的临时源码/缓存。

## Progress

- 本轮本地证据根：
  `file:///home/stream/ACodeSpace/demo/kodex-cleanup-878/`。
- 实验补丁仅作已验证变换的参照；实际交付补齐契约 KDoc、
  公开 helper 文档及正式测试，不直接照搬删除文档的实验补丁。
- A1–A5 与 tool/core 去重已实施；九组 Factory 无剩余源码引用，
  创建规则已合并回九个 specMain ViewModel KDoc，普通创建入口和原构造不变。
  Composer 只保留一份共享 helper，并增加公开边界的输入回归。
- 旧 CLI logging helper 和唯一 CLI 测试移除后，一并去掉其直接 migration
  依赖及空测试模块的 TestBalloon opt-in；后端真实 migration 接线和 KMP targets 不变。
- D1–D3 已实施并补齐异常 KDoc，新增九项正式文件失败/取消/Error 回归，
  覆盖 Path Picker 验证/列举两处恢复与 AGENTS 读取。先将相同测试覆盖到未改生产
  基线确认红灯，再验证正式候选，不能以“新测试只在新代码通过”替代回归证明。
- 新一轮未改生产基线：86 个代表性 JVM 用例，仅原 Composer host 用例
  40000 ms 超时。加入相同新测试的旧代码共 22 用例，精确复现四项
  吞 Error/独立取消的预期失败，普通 I/O 与既有取消用例仍通过。
- 首轮候选 282 个 JVM 用例中 281 通过，新增 13 项（九项故障与四项
  Composer 边界）均通过；原 Composer 超时仍同一用例/异常/消息。
  完整测试编译另保留 integration-test 的旧四参数 open 调用三条错误，
  已在未改基线专项复现；CLI 清尾的两行最终差异也已单独重验。

## Final validation and delivery

- 最终源码快照共 1300 个输入，与主工作区逐项一致。本轮改变 45 个输入，
  包括删除四份失效源码/测试、新增两份正式回归测试；默认堆 6g 不计入本轮变化。
- 九个私有 ViewModel 类及其后续实现体与基线逐字一致。119 份受保护输入
  （RPC/模型、冻结 Home migration、读写锁及 owner/generation 等检查）保持原字节。
- 最终 CLI 清尾后再次验证：
  - 140 个 `allMetadataJar` 和全仓 JVM 生产编译通过。
  - 全仓 JVM 测试编译仅 integration-test 保留旧四参数 open 的三条错误；
    未改基线专项复现完全相同的文件/行号/诊断，不为清理顺带改旧测试。
  - 16 个实际 JVM 测试任务强制执行，282 用例中 281 通过；
    仅原 Composer host 40000 ms 超时与基线用例/异常/消息完全相同。
    空 CLI 测试任务为 NO-SOURCE，不算通过用例。
  - 14 个 Mac Native 测试任务强制执行，193 用例全部通过。
    相同九项故障测试覆盖到未改生产的 Native 基线上时，同样精确复现四项预期失败。
  - AGENTS、AgentState、Tool Core 三个适用 Host 模块的 JS 生产与测试编译通过；
    未把编译记作 Node/浏览器运行验收。
  - Mac Native CLI 编译、debug 链接通过；新二进制在隔离 HOME/USERPROFILE 下
    `--help` 退出 0，无 stderr、不复制凭据、不启动或连接真实后端。
- 共九次远端 Gradle 调用，无资源守卫中止，最大 swap 0 MiB，
  最低可回收内存 8213.9 MiB。最终测试动作未用 UP-TO-DATE/FROM-CACHE 代替执行；
  不根据热缓存耗时宣称性能提升。
- 167 份原始 XML/日志/回执下载后 SHA-256 全部一致；
  [机器汇总](file:///home/stream/ACodeSpace/demo/kodex-cleanup-878/summary.json#L1)、
  [源代码边界核对](file:///home/stream/ACodeSpace/demo/kodex-cleanup-878/source-review.json#L1)、
  [证据索引](file:///home/stream/ACodeSpace/demo/kodex-cleanup-878/README.md#L1)。
- Mac 候选输入核对通过，旧代码测试覆盖已撤回，基线 1302 个输入恢复一致；
  三个自有 JVM 已停、无残留。设备锁已释放未删除，私有源码和克隆缓存已清理，
  仅保留证据/复现脚本；本机传输归档已删除。
- 本轮只归档这份实施记录，原审查/迁移任务及并行 Ktor 记录的正文和阶段未修改。
  实施交付时未提交、推送、发布或重启现用后端。真实 IDE、交互 TUI、其他 OS 和外部二进制
  API 兼容未重新验收，不宣称整仓全绿。

## Authorized submission

- 用户验收交付说明后要求“提交推送”；产品已分两笔推送 `origin/main`：
  `cf5de93d` 清理已验证的冗余结构，`2f260d94` 修正文件恢复边界并加入故障回归。
- 暂存内容逐文件匹配已验证候选；默认堆 6g 不包含在提交中。
  外层仅纳入本实施记录、独立复核完成记录及产品子模块指针，
  不提交并行 Ktor 讨论或原审查活动卡片，不改变其他任务阶段。
