# Task Tree

- `Reconcile confirmed design and isolate the authorized feature branch`()
- `Implement and validate the backend lifetime foundation`()
- `Implement the exclusive instance lock and connection discovery`()
- `Implement stable management transport and independent backend entry`()
- `Merge settings and migrate notification ownership`()
- `Connect TUI bootstrap and same-instance subscription recovery`()
- `Validate native CLI scenarios and application regressions`()
- `Review implementation boundaries against confirmed decisions`()
- `Present boundary findings for user review`()
- `Converge the authorized boundary findings`()
- `Finalize convergence evidence and resource cleanup`()
- `Review redundancy through isolated ablation experiments`()
- `Apply the three authorized ablations and validate their composition`()
- `Review abstractions and failure behavior after composition`()
- `Present confirmed post-composition findings for user review`()
- `Clarify recovery liveness and root-error semantics`()
- `Await implementation direction for the clarified mainline changes`()
- `Implement and validate the confirmed lifecycle renewal and diagnostics fixes`()
- `Present the corrected preview for user acceptance`()
- `Commit and push the three semantic feature batches`()
- `Merge and push the verified product main`()
- `Synchronize project guidance and the product gitlink`()

# Details

## 上游与后续状态 — 2026-10-19 / Session945

- 用户明确本卡是[Codex 对齐](../discussion/2026-09-30-align-kodex-with-codex-rs.md)的已完成下游，现有交付范围与证据不变。
- [Daemon 协程生命周期治理](2026-10-14-discuss-daemon-coroutine-lifecycle.md)也已按用户确认结项；历史待研究细节不再构成活跃阻塞，未来遇到具体问题再优化。

## 后续发版 — 2026-10-15

- 用户另行授权后，`082bfc66` 已发布为
  [v0.4.15](https://github.com/Stream29/Kodex/releases/tag/v0.4.15)；
  四平台 Release、迁移/租约门槛、原生冒烟及安装清理见
  [正式发行记录](2026-10-15-release-kodex-0-4-15.md)。
  下方合入阶段“未发布”是当时状态，不覆盖这次独立发版结果。

## 合入完成 — 2026-10-15

- 用户确认合入方向并授权“分批提交推送再合”；三批签名提交已逐批推送：
  `b4271b22` 生命周期/本地传输基础，`dffd636a` 产品接线与统一设置，
  `0f625c44` 宿主、恢复、迁移及失败呈现回归。
- 产品 `main` 已签名合并并推送为 `082bfc66bff44e74343c98879d75227036257bb4`；
  合并树与 feature 最终树完全一致，1,311份源码哈希匹配 lifecycle-04。
  不以拆分提交替代最终组合验收：沿用下节20组、1,025次显式重跑及两平台实操证据。
- 本期 Daemon 实施完成，归档本卡；当时全局协程研究另由
  [生命周期研究卡](2026-10-14-discuss-daemon-coroutine-lifecycle.md)跟踪（现已结项），
  不将候选 Session/MCP 治理或未来 Tool as CLI 算入交付。
- 本次只提交、推送与合入；没有 tag、GitHub Release、安装服务或替换用户正在运行的程序。
  下方“不提交/未实施”等表述保留各阶段的历史授权边界，以本节最终状态为准。

## 问题2/3/4实施授权 — 2026-10-15

- 用户明确选择实施2、3、4；范围是核心服务图退出联动、前端独立定时renew、
  具体失败原因及异常链。问题1代理行为继续接受，不实现全局协程框架、
  未确认的Session/MCP故障策略或未来工具功能。
- 本机修改现有feature worktree，重型验证仍只在Mac。初步接线：
  核心退出关闭准入并触发既有stop；续命Job归前端，不归单次transport恢复；
  具体原因附加到已有私有实例记录，读取时匹配实例，不能据记录判存活。
  没有新建错误服务或第二套生命周期状态。
- 已补局部取消/普通失败释放锁与保留原因、父取消不误报、实例原因隔离、
  恢复gate超过TTL仍持续续命的回归。最终 lifecycle-04 快照通过
  lifecycle-final-02：20组、1,025次显式重跑，失败/错误/跳过均为0，
  约167秒；Mac/Linux原生链接和另外两平台完整CLI编译通过。
- 两平台 native-demo-14 各12项、native-lifetime-07 各4项、真实TCP
  native-reconnect-06 各5项帧核对通过；native-failure-02 各10项通过，
  包括前台/自动启动显示真实错误并释放锁，以及实例原因在TUI可见。
  运行期TUI检查注入末态记录，不冒称真实核心运行期崩溃。
- 早期跨模块可空属性编译失败及故障脚本假定错误包含路径的失败均保留；
  后者实际已经显示OS具体原因，修正脚本后在同一最终二进制上通过。
- 本机Linux/Mac和远端Mac隔离预览已更新并核对哈希，原预览Home保留，
  原生launcher查询均为not_running。当前[交付与证据](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-evidence-787/final/lifecycle-01/README.md)
  替代此前“仅澄清/未实施”的状态；此前审查记录按对应快照保留。
- 最终源码/预览哈希、59处交付文档链接与303份本轮证据哈希核对通过；
  Mac五个演示Home、私有配置缓存密钥/运行时及本机scratch已清理，
  无本轮进程或暂存凭据，swap为0，共享锁inode保持并释放。
- 不恢复已删三项消融，不改日志回收/旧store，不提交、推送或发布。

## Session847 调研交付接收 — 2026-10-15

- 用户通知847已完成；主线已读取其最终回复、用户讨论与现状矩阵。
  交付是调研与最小治理方向，不是代码修复。原产品仍逐文件匹配
  simplified-02 的1,311文件，没有新增修复后验证或改动预览。
- 问题2不缺父Job：work已经是结构化子任务；缺的是核心服务图独立结束
  到宿主停止的业务联动。核心图不能结束后留下只等stop的空宿主。
  收尾仍先停止准入、取消并等待资源，网络/Home结束后最后释放实例锁。
- 普通命令、连接、标题等允许独立结束的任务不套用该向上联动规则；
  不全局替换SupervisorJob，不建立Job注册表或第二套活跃状态。
- 完整Session/MCP树仍有未确认边界，且没有统一的“Session后台状态同步
  协程”。需点名实际投影/缓存维护任务后再讨论，不把这些问题变成修复
  宿主原问题的前置条件。将跨scope业务结构关系沉淀到checklist是后续要求，
  不能把候选树整体当作已批准规范。
- 主线待实施仍是2、3、4：核心图退出联动；前端独立定时renew；
  CLI/TUI显示具体原因且保留异常链、不新增脱敏层。问题1保持接受。
  当前只接收交付，未将本次通知视为新的实施、提交或发布授权。

## 审查反馈与分工 — 2026-10-15

- 用户接受问题 1 的环境代理行为：未来连接可能需要代理，不强制绕过代理，
  不将此项继续作为修复阻塞；本期仍不提前实现远程部署。
- 问题 2 的生命周期行为不可接受。用户已接受创建
  [Session847：Daemon 协程生命周期治理调研](file:///home/stream/.kodex/sessions/847)，
  由其接管既有生命周期讨论卡，具体调研并直接与用户讨论；不先实施全局框架。
  主线不同时修改该讨论卡，产品源码/公共设计与验证资源需协调。
- 问题 3（恢复期间续命）和 4（根错误因果链/呈现）留在主线逐项澄清。
  当前不根据审查结论自行选择语义或改代码。
- 问题 3 已确认：前端独立定时发送 renew，不以 Session 恢复或业务状态
  作为发送门槛；不是额外增加探活/健康检查机制。后端仍按实际收到且接受的
  renew 更新 TTL，前端结束停止发送，已进入 Stopping 的后端不能被复活。
  现有 /heartbeat 实际就是续命入口，名称不作为新协议或本轮改名要求。
- 问题 4 已确认：CLI/TUI 应显示具体失败原因，本次原因呈现不要求脱敏。
  保留原始异常因果链，不新增专门脱敏层；不把“显示原因”扩大为必须默认
  打印整段堆栈，也不顺带改写其他领域的既有错误处理。具体回传接线按最小
  实现确定，不在澄清阶段提前新建通用错误框架。尚未开始实施问题 3/4。

## 组合后抽象与行为复审 — 2026-10-15

- 用户要求再次核验抽象或行为错误；审查 simplified-02 组合源码，
  不重新提出已撤回的流式副作用重放、未来工具续命/CLI 环境问题，
  也不以缺少 deadline 本身判错。
- 生产源码与 1,311 文件快照匹配。只读追踪外加隔离故障观察测试，
  当前没有修改产品代码、预览或引入全局协程框架。
- 以下保留审查发现；用户的接受/分工以上节为准，
  不把问题 1 继续作为整改项。前三项为新确认，最后一项为既有问题复核：
  - P1：当前 Linux 原生 IPC 在 http_proxy 存在且没有回环 bypass 时走代理，
    连同有效 Authorization 一起发送；直连状态仍 Ready。只用本机假代理和
    临时密码验证，未对外发送/记录凭据。旧消融前二进制同样复现，不是
    默认引擎消融才引入的问题；不据此声称已有远程控制能力。
  - P1：startup work 独立抛出 CancellationException 后宿主仍 Active、
    Starting 且持锁。Starting 不参与 idle expiry，stop 信号又未完成，
    不能自动收尾。JVM/Native 故障观察确认，测试最后显式取消并核对释放。
  - P2：真实 TCP 重连建立健康 transport 后，将恢复回调阻塞在 gate，
    TTL=3 秒的后端在回调结束前正常超时退出。心跳在回调之后才启动，
    把业务恢复耗时错误地耦合到连接续命；不是重放副作用或修改 TTL 规则。
  - P2：根异常被无原始 cause 链的通用初始化失败替换，catch 点未记录，
    与前次生命周期记录一致；JVM/Native 均以 marker 复核。
- 隔离 `behavior-review-03` 实际重跑 JVM 11 / Native 8 项，其中五次
  用于确认缺陷现象；全部通过只表示成功复现，不是修复后回归。
  `behavior-review-01` 是观察测试自身类型推断编译错误，已在副本显式
  使用 Throwable 修正，01/02/03 原记录均保留。
- [分级报告与最小复现证据](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-evidence-787/final/behavior-review-01/README.md#L1-L92)
  已保留；原产品/预览仍为 simplified-02，未顺手修复或扩大设计。
- 本机/远端审查副本、临时 Home、私有配置密钥已清理，构建进程已结束，
  共享设备锁原 inode 106252171 保持并释放；生产快照再次核对一致。

## 三项消融正式采纳 — 2026-10-15

- 用户明确授权将上一轮三个消融项清除；本轮将 A/B/C 一起应用到
  `feat/backend-daemon`，不处理日志回收、旧 split store 或未来工具功能。
- 改前 boundary-03 与记录匹配；改后 16 个受影响文件的内容/删除状态
  与三份已验证 patch 一致。生产代码减少 93 行、测试增加 10 行，
  合计减少 83 行，删除 8 个平台配置文件。
- 用户本次同时接受 HTTP 客户端使用已有依赖驱动的默认引擎选择；
  各平台引擎依赖、管理/业务模块隔离、插件、认证及超时预算保留。
- 生命周期只观察 phase，首次停止原因仍由原 stop Deferred 保留；
  四阶段、停止接受、TTL、取消及管理 JSON 形状未改。
- 组合版本已独立验证；此前三个独立实验不充当组合验收。
  本机改代码，重量级 Gradle 仅在 remote MacBook，保留原资源守卫和锁。
- 首次组合验证 `simplified-final-01` 暴露设置页旧测试仍注入本地 store，
  共修正两个测试文件的六处调用，补三条真实后端 Hook 归属断言；
  没有加回生产兼容接口或删测试。三项加必要测试适配后生产减 93 行、
  测试增 13 行，总减 80 行。
- 校正证据口径：旧收集器会遍历已存在 XML，历史 986 汇总含当轮未请求
  的设置 VM 等组，不能作为当前组合通过依据。本轮改为校验成功构建回执、
  明确请求且实际执行的测试任务及新 XML 时间；测试用 task-specific
  `--rerun` 重跑，不把历史报告混入。失败记录保留。
- `source-manifest-simplified-02.sha256` 的 1,311 份源码在 Mac 验证前后匹配；
  `simplified-final-02` 通过，约 193 秒、20 组明确实际重跑测试任务，
  1,014 次用例报告，failures/errors/skipped 均为 0。不是全仓测试声明。
- Mac/Linux 完整原生 CLI 链接、Linux arm64/Windows 编译通过；两平台
  `native-demo-13` 各 12 项、`native-lifetime-06` 各 4 项实操通过。
  `native-reconnect-05` 的真实断连/恢复帧各 5 项核对通过，草稿仍可编辑。
- 本机 Linux/Mac 与远端 Mac 隔离预览均已替换为组合二进制，
  哈希与验收产物匹配；两种原生 launcher 查询均为 not_running，
  未改预览 Home 内容。源码仍未提交，main 未混入功能，不合入或发布。
- Mac 演示 Home、暂存凭据/配置缓存密钥与私有运行时已清理，swap 为 0，
  共享锁 inode 106252171 保持并释放；本机 scratch、演示密码记录已删除。
- [当前交付与验证证据](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-evidence-787/final/simplified-01/README.md#L1-L55)
  取代独立消融 checkpoint 的未采纳状态；日志、旧 split store 不动。

## 冗余设计与消融审查 — 2026-10-14

- 以下保留上一轮审查当时的实验状态，正式采纳进度以上节为准。
- 用户要求最后复审本次变更的冗余设计，检验能否通过消融降低复杂度；
  本轮先在隔离副本做对照，不直接改动已验证的 feature worktree 或预览。
- 完整统计包含未跟踪文件：96 个变化文件，生产净增 1,812 行、
  测试/fixture 净增 979 行、构建净增 185 行，均包含注释和空行。
  总净增 2,976 行；仅看 tracked diff 会漏掉 39 个新增文件。
- 首个候选：UnifiedSettingsStore 的 YAML AST → 自写 serializer →
  YAML 文本 → 原解码器 AST 往返。消融改为复用原解码器的节点入口，
  不改默认值、校验、文件形状、CAS 或冻结迁移。
- 第二候选：per-process logging 的附加历史扫描/裁剪，与必须的独立日志
  文件命名分开评估。删除会失去历史自动回收，不能将普通启动测试通过
  冒充“所有日志行为等价”。
- 实际隔离实验另验证了 HTTP 工厂与生命周期重复状态；没有把上述日志
  策略直接删除，也没有改变前后端边界。
- 管理客户端/业务客户端隔离、独占锁、同实例重连和旧配置迁移仍是本期
  已确认功能；不为减少文件数合并职责或删掉相应验证。
- 本机当前有 BuildKodex IDEA 进程；实验在未打开的私有源码副本进行，
  不切换用户项目、不在 IDE/本机启动重量级 Gradle。Mac 构建继续使用
  原设备独占锁与资源守卫；无提交、推送、合入或发布。
- 三个独立副本验证完成：
  - A：删去设置 AST/text 往返与 SettingsNodeWriter，生产净减 22 行；
    333 次定向执行、Mac/Linux 原生多 TUI 实操通过。
  - B：删去 8 个 HTTP expect/actual 配置文件，共减 60 行；
    388 次定向执行、Mac/Linux 原生实操和真实 TCP 重连帧核对通过。
    但默认引擎选择替代显式选择是策略取舍，不能只凭测试通过称完全等价。
  - C：删除 BackendLifecycleState 中重复的 stopReason 表示，
    只保留 phase StateFlow 与原 stop Deferred，生产减 11 行、测试增 10 行。
    16 项生命周期测试保留；原 DTO 构造器校验换成实际阶段订阅验证。
    46 次定向执行和两平台原生实操通过，管理 JSON 形状未改。
- 合计 767 次执行有模块重复，不是全仓回归；三次构建均无资源守卫触发。
  B/C 另通过 Linux arm64/Windows 编译，没有声称这两平台运行通过。
  三个 patch 只单独验证，组合 revision 尚未验证；生产候选共减 93 行，
  约为本批生产净增的 5%，不能据此夸大为已经能删除上千行。
- 另有 223 行旧 split store/接口仍在生产源集，仅发现测试消费者。
  这是原有代码，不算本批新增；移除需保留迁移测试能力，不靠删测试减少行数。
- 当前架构层次的保留理由：管理/业务模块隔离、通用 transport 两端复用、
  Native/JVM 平台桥接、独占 guard/私有 sidecar 及迁移重入保护各有实际职责。
  不为消减文件数重新混合 server/client，也不新造通用配置框架替换删除项。
- [实验报告、patch、XML 与原始终端证据](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-evidence-787/final/ablation-01/README.md#L1-L96)已保留；
  各 patch 在未修改 feature worktree 上通过 apply --check。
  feature 与预览仍为 boundary-03，本轮没有直接采纳消融或提交发布。
- Mac 三个实验源码/构建副本、四个演示 Home、暂存配置缓存密钥已清理；
  本机实验副本/二进制/私有 Home 已删除。公共证据不含 connection/password
  记录；原源码、预览及共享设备锁 inode 保留。

## 本轮收敛授权 — 2026-10-14

- 用户已授权先收敛本轮审查发现，再单独调研 Daemon 下全局协程生命周期。
  本批仅处理生产/测试隔离、TUI/管理通道分层、停止准入、重连错误分类及
  设置投影/Hook 输入耦合；不先建立全局 Job 管理框架。
- 本机仍在 `feat/backend-daemon` 隔离 worktree 开发；当前可见 IDEA 是
  Ctrip thin client，不控制窗口或切换用户项目。重量级 Gradle 继续仅在
  remote MacBook 的私有副本运行。无提交、推送、发布或真实 Home 操作授权。
- 保留真实运行与定向测试，不留生产层的旧测试兼容选择器来维持通过。
  本轮完成后回写剩余生命周期研究入口；新的全局方案另行讨论/授权实施。
- 本机实现已拆开管理客户端、业务客户端、后端服务端与共用 WS transport；
  旧组合根移到 test-support，前端设置不再接受文件 store 替代。
  新增后端关闭准入、坏 Session 恢复隔离、失联不误判及活跃连接上的恢复
  回调异常不重试测试。Hook 改为列表快照 getter，统一设置使用库内投影。
- `boundary-jvm-01` 已在 Mac 通过：完整 CLI JVM 编译、六组定向回归及
  View 测试源码编译。新模块 DSL/明确 API 类型的前序编译失败证据保留。
  后续 Native 构建、跨目标编译及实际 TUI 验证见下方当前源码证据，
  不复用旧快照的原生验收结论。
- 当前源码 `source-manifest-boundary-03.sha256` 已通过
  `boundary-final-01`：Mac 3m22s、1,982 tasks，355 executed /
  1,627 up-to-date；JVM/Mac Native 定向回归、两种 CLI 链接和
  Linux arm64/Windows x64 编译通过。前序 Native 新增准入测试错误地
  依赖未知 RPC 失败的具体 Exception 类型；已按既有跨线错误契约改成
  验证失败及四次真实门禁拒绝，保留文件未变断言，未放宽生产门禁。
- Mac/Linux `native-demo-09` 各 12 项，`native-lifetime-05` 各 4 项
  真实实操通过；`native-reconnect-03` 已重新采集，实际终端帧核对
  同实例重连、横幅变化和草稿保留/继续编辑。无模型付费调用。
- 本批的既有问题已落地收敛：测试宿主只在 test-support；设置由后台 RPC
  唯一接入；TUI 不再引用管理请求；停止门禁进入后端服务图；恢复失败
  不再进入传输重试；去掉新手写 StateFlow 和伪前端 Hook DTO。
  原问题分析保留为历史记录，不表示这些项仍待整改。
- 后续[全局协程生命周期调研](2026-10-14-discuss-daemon-coroutine-lifecycle.md)
  已完成首轮 owner 映射和静态风险核对；调研记录不提前授权全局 owner/Job 改造。
- 收敛后汇总 18 组模块/目标、986 次定向执行，failures/errors/skipped
  均为 0；该统计不宣称全仓测试全绿。当前 Mac/Linux 隔离预览已替换为
  本轮二进制，并保留原始记录、终端帧验证及文件/二进制哈希。
- 本轮代码与定向运行收敛已完成。Mac 的 3 个新增演示 Home、私有 Gradle
  运行时/配置缓存/密钥已清理，无本轮进程或暂存凭据；本机 scratch/
  密码记录也按证据核对后删除。源码和后续可复用缓存保留。
- 功能仍在未提交的 `feat/backend-daemon` worktree，main 未混入修改，
  不自动提交、推送或发版。本卡继续保留 executable 等待用户效果验收；
  后续讨论卡负责生命周期调研，不把本批收敛当成全局生命周期问题已解决。

## 代码边界审查 — 2026-10-14

- 用户要求审查是否为实现目标引入越界设计。本轮只读产品源码与差异，
  未改产品代码，未运行 Gradle、模型调用或占用远端设备。
  以下保留审查当时的发现；其整改结果以上节当前收敛记录为准。
  原测试/原生实操结果是运行证据，不是当时这些边界已成立的证明。
- **明确边界倒退：测试兼容路径留在生产层。**
  `app/impl/application` commonMain 仍公开 `withKodexApplication`，
  消费已准备的 Home、装配 backend services 与 in-memory RPC，并新增
  后端 Hook owner 以适配旧测试。这条旧组合根不经过 Daemon 实例锁，
  生产依赖仍包含 migration/server/in-memory/OpenAI client。
  `RpcGlobalSettings.open` 又允许用非空 `FrontendSettingsAccess` 替换
  后端 RPC 设置投影；旧文件 store 被接入此接口，多项测试继续断言
  `settings.frontend.cli.yml` 独立持久化。默认 CLI 没走这两条路径，
  但“测试用”的文字没有把替代生产路径隔离到 test support。
- **明确边界倒退：TUI 与管理通道没有按确认分离。**
  `withLocalRpcClient` 在建连/重连中调用 `/control/status`；
  同文件公开 status/stop/reload helper，application 又 API 依赖整个
  local client/server 模块。与此前确认的“TUI 不依赖或调用管理 RPC”
  不符。启动引导的连接检查属于已允许的例外，不能扩大到运行中的 TUI。
  未发现 TUI 自动调用 stop/restart；问题是职责依赖，不是同用户身份鉴别。
- **实现缺口：Stopping 尚未成为既有业务调用的共同准入边界。**
  `LocalBackendServer` 只在新 WS 建连时检查 Ready，既有连接直接借用
  `BackendServices`；`requestStop` 改状态后，宿主先等接受应答边界，
  再取消业务图。此窗口内 Session host 只检查自己的 owner 是否活跃，
  不检查 Daemon Stopping，仍可能接受新任务/写入。需在后端业务准入
  处协调，而不是仅靠断连接或 UI 禁用。此为静态调用链结论，未新增竞态实测。
- **实现缺口：重连层将业务恢复失败降级为普通断线。**
  `prepareReconnection` 执行 Session 激活；若其抛出非连接类异常，
  `withLocalRpcClient` 的宽泛 catch 仍关闭连接并每 500 ms 重试。
  一个损坏 Session 可能阻止健康连接发布，并把真实错误表现成持续重连。
  应区分连接失败与 Session/业务失败，不重写已有错误/失活边界。
- **抽象收敛项，不直接等同于权限越界：**
  `UnifiedSettingsStore` 使用 InternalCoroutinesApi 手写
  `SettingsProjection.collect/replayCache`，应审查能否用既有投影与显式
  owner 解决；后端 Hook 为复用执行器构造带默认 UI 偏好的
  `CliFrontendSettings`，执行端实际只需 Hook 列表。这些是实现便捷导致
  的额外框架/跨用途耦合，不是已证实的第二份可写真源。
- **未判为越界：** loopback 单实例、独立拉起、管理 HTTP 与业务 WS、
  原实例订阅恢复、统一设置与追加冻结迁移均支撑已确认目标。
  未发现启用远程连接、安装系统服务、角色凭据系统或 TUI 自动重启后端。
  独立进程日志与有界预算也是必要支撑；8 组保留/10 MiB 截断是实现
  选择，不应写作用户亲自确认的数值政策。0.4.15 仍只是未发布 checkpoint。
- 不以本轮审查授权直接整改、提交或合入。当前状态为功能实操通过、
  架构边界待收敛，保留 executable。

## 完整实现与效果验收 — 2026-10-14

- 本节是最新状态；下文首批 fixture 与各轮审查是历史检查点，不能据其
  “尚未接入”措辞判断当前功能缺失。
- `feat/backend-daemon` 已接通完整本地链路：稳定内核独占锁、私有端口/
  每次启动随机密码记录、loopback HTTP 管理与 WS/kRPC 业务连接、
  脱离 TUI 的后端进程、start/status/stop/reload、续命/空闲退出、
  统一配置迁移、后端业务 Hook 与前端错误 Hook、多 TUI 和原实例重连。
- 原实例短暂 TCP 断线测试使用真实代理断开 socket，再恢复连接；
  实际前端装配的页签、Agent/Composer 对象、文本和光标均保留。
  不重放结果未知的命令，不自动接管重启后的新后端。
- 配置以运行中的统一快照保存；外部编辑显式 reload，校验失败保留
  运行状态。0.4.15 新迁移追加在 registry，冻结旧 entry/codec 未修改；
  旧 Hook 规则按执行归属合并，Ultrafast 与带前导零的字符串保留。
- 原模块/Native 下载阻塞已解决：直接获取并校验官方同版本分发包，
  传到 Mac 私有工具链；未在本机运行 Gradle，也未降级 Kotlin。
- 最终源码冻结为 `source-manifest-complete-07.sha256`，本机与 Mac
  验证前后匹配。所有重量级 Gradle 在 Mac，包含完整 Mac arm64 和
  Linux x64 Debug CLI 链接；Linux arm64 与 Windows x64 完整 CLI 编译。
- `final-acceptance-01` 成功：1m48s，1,900 tasks，311 executed /
  1,589 up-to-date。应用装配、宿主 JVM/Mac Native 回归均通过。
  前序成功记录另覆盖生命周期、迁移、统一设置、RPC 服务/适配、Hook
  与设置组件；不将这组定向验收宣称为整个仓库所有测试全绿。
- 收尾补上随机凭据/信号初始化的实例锁 finally 保护，并按锁定库的实际
  `-<index>.log` 后缀修正轮转日志清理。新增活跃 PID、legacy/foreign
  名称保护测试。`final-acceptance-03` 成功：1m47s，1,866 tasks，
  301 executed / 1,565 up-to-date；最终预览已重新链接和实操验证。
- 追加应用层回归发现七处旧拆分配置/Hook 所有权预期，已更新真实
  fixture 与断言；保留原清理、持久化、通知去重和错误恢复检查。
  另在静态复核中发现 fixture 新 Hook 子作用域不返回，主动停止唯一
  自有验证并修正为显式 owner/finally 清理；失败/停止证据保留，
  不冒称用户占用资源或删掉测试。最终两目标应用层回归已通过。
- 业务 Hook 测试在所有客户端断开后触发离线模型故障，实际 shell
  在 Home 写入一次 marker；默认业务 consumer 在 Ready 发布前完成
  订阅。已接受 turn 不随连接等待取消，TTL 由 turn 继续更新。
- 原生演示及最终公共证据位于
  `file:///home/stream/ACodeSpace/local/kodex-backend-daemon-evidence-787/`；
  [预览说明](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-evidence-787/README.md#L3)
  可直接使用本机 `preview-linux/run-preview` 或 Mac 私有目录的
  `preview-mac/run-preview`，脚本只使用自己的空白隔离 Home。
- 最终 Mac/Linux `native-demo-08` 各通过 12 项真实三 TUI 演示；
  `native-lifetime-04` 各通过 4 项前台 start、独占拒绝、TTL 自动退出和
  释放锁实操。后端结束提示在旧 TUI 真实画面中可见，不被屏幕高度挤掉。
- 两平台另完成 `native-reconnect-01`：实际断开 TCP 并暂停代理，
  原生 TUI 显示重连状态，恢复后横幅撤下、原草稿保留且可继续输入。
  后端 instance/PID 未改变；公共 `verification.json` 核对真实终端帧。
- [XML 汇总](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-evidence-787/mac-results-complete/final/test-summary.json)
  共 16 组模块/目标、744 次执行，failures/errors/skipped 均为 0。
  两个 Debug 预览二进制的跨设备哈希已核对；公共证据全量哈希复核通过。
- Mac 无本轮运行进程、临时 Gradle 凭据或设备锁占用；累计清理 13 个
  演示 Home、主动停止批次留下的 2 个精确临时测试 Home、私有 Gradle
  运行时、配置缓存及其私有加密密钥和重复下载包。本机演示 scratch/
  密码记录也已删除；当前试用脚本 status 均为 not_running，共享锁已复核释放。
  保留功能源码、可复用缓存、公开证据及试用二进制，不安装系统服务。
- 本卡保留 executable 等待用户实际预览；不是仍缺基础原语。
  功能代码未提交/推送/合入 main，0.4.15 只为本功能分支的协议/迁移
  checkpoint，不是公开发版。main `3223a52d` 保持干净并与 origin 一致。

## 实施授权与当前批次 — 2026-10-14

- 用户追加要求：做到完整可运行并展示实际效果后再汇报，不再以基础模块或
  单元测试检查点结束。继续本机实施、Mac 重型验证，直到独立后端、
  管理命令、多 TUI、配置迁移及真实 CLI 场景验收；不据此扩大到远程连接、
  付费 live 模型、公开发布或占用用户真实会话。
- 用户授权创建 `feat/backend-daemon`，本机开发，重量级 Gradle 仅在
  `remote MacBook` 上运行。本卡进入 executable；以下历史审查中的
  “未授权实施”只描述当时状态，不限制本次明确批准的开发。
- 产品仓库 `Kodex/` 原工作区存在其他 Session 的 OpenAI 超时改动，
  不切换、暂存、重置或打包这些改动。已在同一仓库创建隔离 worktree：
  `file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787`，
  分支 `feat/backend-daemon`，实际创建基线 `3223a52d`。
  创建时 main 正好推进；该已提交超时修复是基线的一部分，不是本任务修改。
- 开工时本机 IDEA 为无关的 Ctrip 远程 thin client，随后用户打开外层
  BuildKodex/main；本轮未控制这些窗口或将功能 worktree 切入用户 IDE。
  remote MacBook 预检无活跃 IDEA/Gradle/Kotlin 进程，24 GiB 内存、约
  104 GiB 可用。远端使用 Session787 私有源码副本和缓存，不运行真实 Home。
- 不继承早先已经执行完毕的 commit/push 或 release 授权；本次先开发验证，
  不自动发布，也不把其他工作区改动纳入结果。用户随后明确授权处理产品
  main 的遗漏提交/推送并 rebase；检查时其改动已提交并推送为 `3223a52d`。
  本轮 fetch、push 和远端哈希确认一致，未生成空/重复提交；
  功能 worktree 执行 `rebase --autostash main` 后全部恢复，无遗留 stash。
  此授权不作为未来功能分支或外层工作区自动提交/发布的许可。
- 按依赖顺序实施。历史首批先落实独立、无业务/RPC transport 依赖的后端
  生命周期原语：Starting/Ready/Stopping/Stopped，成功续命、默认 5 分钟
  TTL 与无限 TTL，手动停止/空闲退出的不可逆准入及资源清理完成的边界。
  使用当前 specMain/commonMain 约定和确定性并发/时钟测试。
- 后续已接入独占锁、稳定控制协议、平台后台拉起、设置合并与迁移、
  后端 Hook、TUI 重连；完整验收以上节为准，不以首批原语冒充交付。
- 本机只做文本/静态检查；本轮所有 Gradle 任务放 Mac。检测可用 Gradle
  Daemon JVM 并显式指定，若已有可用实例则必须复用；新私有实例不改用户
  全局设置、GC 政策，不抢占其他设备任务。

## 历史首批实现检查点 — 2026-10-14

- 本机 worktree 已新增 `app/impl/backend-daemon`，应用原
  `kodex.kmp-spec-impl` / `kodex.kmp-cli` / `kodex.kmp-tests` 约定，
  模块已登记到 settings 的 app/impl 树。产品 main 未混入本轮修改。
- 契约与状态在 `src/specMain`，实现和创建/监视函数在 `src/commonMain`；
  该模块不依赖业务模型或 RPC transport，也不伪称拥有后端进程/资源图。
- 已实现单调时钟 TTL、默认 5 分钟/无限 TTL、成功续期、不可逆停止、
  停止接受与清理完成的区别、等待者取消隔离、已取消调用方的准入检查。
  `finishStop` 必须由真实宿主在清理后调用；接受 stop 的网络应答与宿主取消
  时序仍由后续控制 transport/宿主接线处理。
- 共 16 项确定性测试，含 Default dispatcher 的真正并发 stop/renew/expiry，
  虚拟时钟到期、TTL 配置不隐式续期、取消观察者/调用方与非法状态。
- **原项目与 Native 验证未通过门槛**：Mac 首轮请求
  `:app-impl-backend-daemon:jvmTest` 和 `macosArm64Test`，停在冷缓存
  `kotlin-native-prebuilt-macos-aarch64-2.4.20` 下载/重试，30 分钟守卫触发
  （exit 143），未进入本模块编译/测试；不是测试失败，也不是编译通过。
- **JVM 逻辑已获限定验证**：Mac JVM-only fixture 直接编译当前生产源及
  测试，沿用产品 catalog 的 Kotlin 2.4.20 / coroutines / TestBalloon、
  JDK25、`commonMain.dependsOn(specMain)` 层级与 allWarningsAsErrors，
  16 tests / 0 failures / 0 errors / 0 skipped；首轮约 56 秒，
  修正 factory KDoc 与 fixture 创建 DSL 后，以 `--rerun-tasks` 再次实际
  编译和测试，最终 Gradle exit 0、约 10 秒，5 tasks 全部 executed。
  该 fixture 不替代原项目 Gradle 装配、完整 CLI、其他目标或 Native 验收。
  首轮 fixture 创建委托弃用提示已在最终 fixture 修正；既有 internal KGP
  property 警告保留，没有据此修改原 KMP convention 或降级锁定工具链。
- 三个源码快照均核对 1,284 份文件；第二轮加入取消准入修正，第三轮仅修正
  factory KDoc（没有后台任务启动，而非声称不存在 Deferred/Job 对象）。
  最终验证前后都与本机 v3 hash manifest 一致；两次 JVM jar 哈希相同。
  第一轮 15-test 快照未得到测试结果，不能混作跨目标成功。
- Mac 使用既有 `device-heavy.lock` 的同一 inode，以非阻塞独占方式取得，
  没有占用 GUI。显式选用 GraalVM JDK25；预检无可复用 daemon，测试后
  private Gradle/Kotlin 进程已停止、共享锁释放、凭据暂存删除。
  全程最大 swap 使用为 0；首轮可用内存估计最低约 10.89 GiB，
  JVM fixture 最低约 11.15 GiB，没有触发内存/swap守卫。
- 公共证据与 fixture/JVM jar 已同步本机：
  `file:///home/stream/ACodeSpace/local/kodex-backend-daemon-evidence-787/mac-results/`。
  JVM jar SHA-256 为
  `6b69e1d3c6890ce5800ffcaef159465b951bc35b7945a892a144e846112282f7`。
  私有源码与构建缓存留在 Mac 的 Session787 validation 目录供后续
  已授权开发复用；失效下载 scratch 与失败配置缓存已删除，没有活动任务。
- 首批仍未完成跨目标门槛，Task Tree 保持本批执行中。真实独占锁、监听
  端口、管理命令、后台拉起、设置合并迁移、通知和 TUI 接线尚未交付；
  不归档、不提交功能分支、不推送产品功能或外层文档。

## 授权与分工 — 2026-10-13 / Session 787

- 用户要求说明书由独立 subagent 接手，主线与用户讨论后端 Daemon 化设计。
- [Session799：产品说明书收尾](file:///home/stream/.kodex/sessions/799)
  已由用户接受并创建，接管现有说明书总卡、用户协作和工作台三卡；
  用户将在该 Session 直接协作。创建不表示说明书已完成。
- 本卡归主线 Session787；起初仅设计与只读核对，现按上节授权进入实施。
  不修改说明书任务，不占用共享 GUI、不安装服务、不沿用已执行完的推送授权。
- [Tool as CLI](../discussion/2026-10-03-discuss-integrate-tool-cli-into-kodex.md#L27)
  仍等待独立 Daemon 模型与前后端边界获接受，不因本卡开始而进入实现。

## 已核对的现状

- [CLI 入口](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L10)
  先准备 Home，再在 Application 作用域中运行 renderer。
- [当前组合根](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L74)
  嵌套 `withBackendServices`、`withInMemoryRpc` 和前端；已存在业务 RPC
  边界，但不是前端可断开、后端独立存活的进程模型。
- [现行宿主生命周期](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/rpc-architecture.md#L33)
  规定 renderer 结束后清理两侧；后续独立部署方向要求前端退出不直接终止
  已接受任务。Daemon 设计必须明确替代哪些宿主规则，不能只换 transport。
- [Session 保活](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/rpc-session.md#L102)
  已有 runningTurn 自保活；Session TTL、前端连接和 Daemon 存续不是同一层。
- [前端设置与通知](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/rpc-settings.md#L65)
  目前 Hooks 在前端执行，通知无 replay；后台存活不自动获得无人值守 Hook
  或通知补播能力。重连后业务状态恢复与通知事件补发必须分开讨论。

## 已确认的约束与当前方向

- 用户在评估认证端口及远程加密后明确收敛：第一期只考虑本地连接，
  支持同机同用户的多个前端；跨设备连接不纳入本期。
- 用户当前思路是 backend Daemon 暴露可连接、带简单认证的端口；
  按此方向实施，协议及认证细节在技术方案中明确，不扩展远程范围。
- 本期仅监听 loopback，本机连接启用认证；先前讨论的可配置远程监听
  不作为本期功能。远程 TLS、隧道与证书配置一并延期，不继续作为本期问题询问。
- 用户进一步明确：每个 OS 用户只有一个 Home，固定为 `~/.kodex`，
  因而每个用户只有一个 backend Daemon。不是系统所有用户共用一个实例。
- 用户确认后端始终在 `~/.kodex` 下工作：后端进程工作目录固定为 Home，
  不继承最先启动它的 TUI 的项目目录；后端业务通知 Hook 也使用该目录。
  这不将既有 Agent 显式配置的项目/工具工作目录改成 Home。
- 不设计生产环境的多 Home 路由、Home 切换或可配置 Home 入口；
  单实例身份以当前 OS 用户及其固定 Home 为边界，不按工作目录/项目各起实例。
- 直接运行 `kodex` 时自动连接已有 Daemon；没有实例时自动启动后再连接。
  该确认不授权发现连不上就抢占旧实例、强杀后端、升级替换或重放业务命令。
  自动启动由进入 TUI 前的引导层负责，不赋予 TUI 生命周期管理能力；
  不将该规则自动扩展到所有管理 CLI 命令。
- 网络 transport、管理协议细节及必要业务 RPC 变更仍待设计；
  已确认的命令与停止语义按下文执行，不重复作为开放选项。
- 用户要求后续设计问题逐项通过 ask human 确认，每次只问一个问题。
- 用户进一步要求沿用此前已确认的前后端分离决策，不把已定归属重新提问。
  审查先对账 checklist、组件契约和既有交付记录；实现缺口由技术方案修正，
  只有现有约束无法推导的新行为取舍才提交用户确认。

## 单实例锁与连接发现 — 用户指定方向

- 后端运行期间持有文件系统锁，避免多个后端冲突；锁记录同时包含端口号
  和连接密码，本机前端检查锁状态并读取连接信息，不另设独立的发现机制。
- 用户确认每次后端启动随机生成新密码，不跨重启保留；
  本机前端从锁记录取得端口和本次密码。
- 用户确认本地端口由系统在后端启动时自动分配；后端实际绑定 loopback
  端口后将其发布到锁记录，前端读取连接，不要求固定端口。
- 实现需区分锁所有权、连接信息已发布和服务可连接，不能仅凭锁文件存在
  判定后端可用，也不能因连接失败直接删锁或启动第二个后端。
  含密码的记录须限制为当前用户可读写。
- 当前 Home 迁移实现使用共享运行期租约与独占迁移租约；不能直接把现有
  Home 租约当作已经满足新 Daemon 的独占单实例要求。具体锁实现待设计。

## 后端实例与前端存续 — 用户确认

- 后端重启而旧前端仍运行时，旧前端应进入错误状态，不能透明接入新后端；
  用户要求前端也重新建立。具体错误展示与重新建立的入口仍待细化。
- 正常存活的前端持续为后端续命，不因业务空闲自行退出；手动重启可打破
  此存续关系。这是正常生命周期约束，不是后端不会崩溃的保证。
- 用户确认前端保活以成功收到续命为准：TUI 进程即使仍存在，只要没有
  成功续期且没有 runningTurn 续期，仍按 TTL 超时退出。后端不另以进程
  存在否决超时；暂停或失联的旧 TUI 恢复时仍受同实例重连规则约束。
- 用户明确后端自续命的所有者是 `runningTurn Job`，不依赖前端存活；
  等待用户回答/审批不续命，不能因任务在用户意义上“尚未完成”而另加
  保活来源。没有 runningTurn 和前端续命时，仍按最后续期的 TTL 退出。
  这里的等待指宿主交互等待，不把运行 Job 内的普通协程挂起当作停止续命。
- Daemon TTL 默认 5 分钟，不再收到任何续命后超时自动退出；TTL 可配置，
  `-1` 表示永不因 TTL 超时退出，不阻止手动停止或重启。
  续命频率及具体配置入口尚未确定。
  Daemon TTL 与现有 Session TTL 分属不同生命周期，不能直接混用。
- 用户确认同一后端实例短暂断线时，前端显示断线状态并自动尝试恢复连接；
  恢复后重新读取状态、建立订阅，不自动重发结果未知的操作。
  若发现后端已换实例则报错，不自动转接新实例。
- 用户明确 opened Session tabs 是纯前端本地状态；后端只有 active Agent
  与 running Agent 概念，不维护哪些页签已打开。连接恢复只重新绑定前端
  仍需访问的后端资源，不从后端获取或重建“已打开页签列表”。
- 续命协议的具体时序、实例身份校验与重试时序尚待设计；
  退出判定与新连接/新任务到来的竞态需在实现前明确。

## 两类前端与版本边界 — 用户确认

- 只交付一个 `kodex` 可执行文件，通过不同入口选择运行角色：
  `kodex` 为 TUI，`kodex backend start` 为 backend，
  `kodex <other command>` 为管理/命令行 CLI。后端仍是独立进程，
  不因共用可执行文件而与 TUI 共用生命周期。
- 管理 CLI 前端用于执行 `kodex backend stop` 等命令；它仅使用非常受限的
  管理 RPC 子集，设计目标是基本不受产品版本变化影响。具体命令集及
  控制协议兼容规则仍待设计，不将该目标解释为任何版本都无条件兼容。
- TUI 前端是现有交互界面，使用受版本兼容性约束的业务 RPC；
  它没有直接管理后端生命周期的权限。前述续命属于保活，不授予停止、
  重启或替换后端的管理权。
- 用户明确这里是接口/模块职责隔离：TUI 不依赖或调用管理 RPC，
  管理 CLI 负责启停；不额外建立同一 OS 用户下的客户端真实身份鉴别、
  管理角色凭据或安全身份系统。连接密码仍用于本地端口认证。
- 用户确认 TUI 按业务协议版本判断兼容，不要求主应用版本或构建完全相同。
  业务协议版本与配置文件版本采用同类思路，是主应用版本的离散子集，
  不另设脱离主应用版本体系的编号。多个主应用版本可以沿用同一协议版本；
  协议版本更新条件尚待细化。
- 用户确认第一期 TUI 与后端的业务协议版本必须相同，不同则拒绝业务连接；
  不维护跨协议版本适配或兼容范围。主应用版本不同本身不构成拒绝理由。
- 新 TUI 与已运行后端版本不兼容时，报错并保留后端；
  不向 TUI 提供确认后直接重启后端的流程，不影响既有客户端和运行任务。
- 管理 RPC 的协议、认证接入及服务端准入仍待明确；
  TUI 与管理 CLI 的代码/接口依赖须分开，不仅仅隐藏界面按钮，
  但不把职责隔离升级为客户端身份安全隔离。
- 用户最终确认后端命令统一为 `kodex backend <action>`，去掉 `daemon`
  层级：`start` 运行后端，`stop` 提交停止请求，`status` 查询状态。
- `kodex backend start` 沿用已确认的当前进程运行语义，不自行后台化
  后返回。自动拉起时由启动方负责后台化；手动运行可用于调试。
- 用户确认保留普通 `kodex` 的启动引导：没有后端时，引导层在后台拉起
  `kodex backend start`，等待就绪后再进入 TUI；已有后端则连接并校验兼容。
  TUI 本身只连接和保活，不负责停止、重启或替换后端。
- 用户确认新启动遇到正在停止且仍持锁、无法服务的后端时，直接启动失败；
  不等待它退出后接续启动新实例，不夺锁、不强杀。这属于已有后端无法服务，
  不能当成“没有后端”进入自动拉起分支。
- 额外后台启动参数或 `restart` 入口不属于已确认范围；不作为本期阻塞，
  不由命令层级调整自动引入。

## 管理停止与协作式取消 — 用户确认

- 后端提供一组管理 RPC；管理 CLI 调用后，由后端自己负责执行 Kotlin
  协程的协作式取消。停止不是由 CLI 直接杀进程，也不是等待 Agent 自然完成。
- 取消和资源释放沿用[协程资源生命周期](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/coroutine-resource-lifecycle.md#L3)：
  由后端资源所有者组织取消及清理，不让单次管理 RPC 的连接拥有后端资源。
- 用户确认管理 CLI 在后端接受停止请求后即返回，不等待清理及进程退出。
  命令成功只代表请求已被接受，不能显示为已经退出。
- 后端负责请求接受后的取消与清理，不因管理 CLI 返回或断开连接而撤销。
  接受应答与 RPC 服务关闭的时序需细化；若没有收到接受确认，不能仅凭
  RPC 断线宣称请求成功或后端已经退出，也不推导出超时自动强杀权限。
- 实际退出状态的查询方式及取消不及时响应时如何报告仍待设计。

## 本地认证端口方案 — 待细化

- 候选传输为 kRPC over WebSocket；[官方 Ktor 集成](https://kotlin.github.io/kotlinx-rpc/krpc-ktor.html)
  已提供该模式，但文档版本为 0.10.4，项目锁定 kRPC 0.10.3 / Ktor 3.5.0，
  尚不能据此承诺所有 Native 目标可直接使用。
- 简单认证候选细节：每次启动的随机密码采用高熵令牌，同机客户端从仅当前
  用户可读的锁记录取得；认证通过后才创建业务
  RPC 连接。令牌不放 URL、日志或命令行参数，不复用模型供应商凭据。
  不引入账号或用户角色系统；管理 CLI 与 TUI 按上节确认做接口职责隔离。
  不重启后端的密码重置入口不属于已确认范围，不作为本期阻塞。
- 连接密码保护的能力包括业务 RPC 的工具执行，不是只读访问；
  不能据此给 TUI 暴露管理 RPC。最小防护仍包括认证超时、
  连接/消息上限与失败节流。

## 前端设置所有权 — 用户确认

- 用户明确前端设置受后端管理，TUI 从后端订阅，不由各 TUI 独立持有
  可写文件真源。修改应提交后端，再由订阅传播到其他 TUI。
- 用户随后提出将现有两份设置合成一份，按后端统一管理的单文件方向继续
  设计，替代此前保留 `settings.backend.yml` 与 `settings.frontend.cli.yml`
  两文件的目标。用户确认同一文件保留 `backend` 与 `frontend` 分区，
  两类 Hook 各归对应分区；最终文件名、具体字段及 RPC 投影仍待设计。
  文件合并不自动意味着前端与后端执行职责或所有设置 RPC 合并。
- 前端错误 Hook 仍在 TUI 执行；其配置随前端设置从后端取得。
- 用户明确 TUI 连接不上后端即视为启动失败，不进入一个离线可用的 TUI，
  不以“先用默认配置或旧缓存进入界面”替代成功连接。
  尚未取得 Hook 配置的启动失败不通过另增缓存来补救。
- 启动失败与成功启动后的连接中断分开处理；后者仍按已确认规则尝试恢复
  同一后端实例，不因本条决定取消运行期自动重连。
- 设置 RPC 的读取/订阅/写入契约与并发校验、迁移时序需单独审阅；
  当前 `rpc-settings.md` 仍描述已交付的前端直接读写文件，尚未改写或实施。
- 用户确认手动编辑设置 YAML 后，通过显式管理 CLI 重载命令让后端
  加载修改，不要求重启后端，也不新增自动文件监听。
  命令的具体名称及重载范围/失败语义仍待技术方案明确。
  TUI 内修改继续提交后端并经订阅传播，不能要求每次 UI 修改后手动重载。
- 用户确认保存时以运行中设置为准，可覆盖尚未重载的外部文件修改，
  不因检测到外部编辑而拒绝保存，也不自动合并；此规则包括 TUI 设置保存
  与后台 MCP 凭据回存。手动修改只有经成功重载接受后才进入运行中设置，
  在此之前存在被覆盖的风险。

## 通知重新设计 — 执行归属已确认

- 用户重新讨论后确认：Agent 业务通知 Hook 由后端直接统一执行，
  不再依赖 TUI 在线，也不由多个 TUI 分别执行同一业务通知。
  此决定替代此前“各 TUI 分别执行”的选择。
- 用户确认 TUI 自身的错误和警告保留前端本地错误 Hook，
  不转交后端执行；这与后端 Agent 业务错误通知分开。
  不因保留本地错误 Hook 而让 TUI 重复执行后端业务通知。
- 两类 Hook 可配置不同命令；设置文件合并后仍区分后端业务通知和前端
  本地错误通知的执行归属，不把合并文件解释为合并 Hook 执行器。
- 用户已确认自动迁移并保留现有通知命令；新目标改为合并设置文件，
  同时按用途拆分 Hook 规则：原本覆盖 Agent 错误与 TUI 本地错误的规则，
  分别保留在两类配置中。保留适用规则的名称、顺序和命令，
  不在迁移期间执行命令；具体字段位置随单文件结构确定。
- 现有前端执行 Hook、无订阅者不保留通知的规则仍是已交付行为，
  本卡记录新的后端通知设计，尚不改写 `rpc-settings.md` 或产品实现。
- 自动迁移的具体版本及恢复校验留待实施方案，不能改写已发布的历史迁移。
  新增合并迁移应承接历史拆分结果，而非删除历史拆分 entry；
  跨版本直升、已有目标与中途失败重跑均须验证。
- 用户确认后端进程及业务通知 Hook 的 cwd 固定为 `~/.kodex`；
  原来相对于 TUI 启动目录的脚本命令可能需要用户调整，不将迁移字符串
  冒充执行环境完全兼容。前端本地错误 Hook 不受此后端 cwd 决定改动。
- 后端 Hook 的 shell、环境变量及资源生命周期仍待细化。
  后端发布业务事件与后端执行通知 Hook 是不同职责；
  移动执行位置本身不承诺桌面通知可达、事件持久化或恰好一次投递。

## 远程评估留档 — 本期延期

- 曾评估单用户认证端口的跨设备使用，但未做网络原型或跨平台验证；
  用户最终选择第一期只做本地，不实施远程监听、远端凭据配置或 TLS。
- 后续若重启远程设计，需重新处理传输加密与服务端身份验证、远端路径浏览
  及连接配置；当前本地目录选择器不能直接当作远端文件系统浏览器。
  本次讨论不授权新增目录浏览 RPC、文件同步或本地工具转发。

## 后续技术方案检查项

- 单实例：发现、当前用户鉴权、并发首次启动与遗留端点的安全处理；
  区分尚在启动、无实例、活实例不可连接和版本不兼容。
- 分离管理 CLI 的稳定受限 RPC 与 TUI 的业务 RPC，按上节确认划分生命周期
  权限及兼容性检查；明确拒绝不兼容业务连接时仍可使用的管理能力。
- Daemon 续命与退出的时序、主动关闭、运行中升级与后台崩溃处理；
  前端存活及 runningTurn Job 续命，交互等待不续命，
  TTL 超时退出且支持 `-1`，按上节用户确认执行。
- 同一后端实例自动重连的实现：实例身份、重试时序、订阅恢复及状态重新读取；
  不自动重发结果未知的操作，不能静默切换后端实例。
- 多前端：并发编辑、问题回答/建议审批、后端管理前端设置的订阅和写入契约；
  业务通知 Hook 已确认后端执行，其余通知边界按上节继续设计。
- 环境边界：工作目录浏览、工具执行路径、认证和 MCP 生命周期分别归谁；
  本期同机部署不因远程评估而改变现有目录浏览契约。
- Daemon 崩溃或重启后的恢复保证另行确定；不能由前端断连后继续运行
  推导出任意任务都可跨后端进程重启续跑。
- 固定生产 Home 下的测试隔离方式单独设计，不把测试 fixture 的路径参数
  自动当成可配置生产 Home，也不在真实 `~/.kodex` 做破坏性验收。

## 设计门槛

- 优先复用已审定业务 RPC/领域所有权；必要的握手、管理接口或业务契约调整
  单列审阅，不把“设计 Daemon”视为所有冻结契约已解除。
- IPC、服务管理集成及尚未确认的命令/参数仍待设计；单可执行文件和三类
  入口按上述用户确认执行，不重新推导独立后端安装包。
- 确认后的通用设计再写入相应 checklist；当前不改写现行单 CLI 的已交付规则。

## 阻塞点复核 — 2026-10-13 / Kodex 27897e20

- 本轮只读审查当前产品与已确认设计，未构建、未改产品代码。
  将用户行为取舍与技术验收门槛分开；不重复询问已有结论或现有契约能承接的细节。

### 本轮人工决策阻塞 — 均已解除

- **CLI/TUI 权限含义 — 已解除**：用户确认是接口/模块职责隔离，
  不额外鉴别同用户下客户端程序的真实身份，不引入角色凭据体系；
  TUI 不依赖或调用管理 RPC，后端端口仍按已定密码认证。
- **前端存活的判据 — 已解除**：用户确认以成功续期为准，
  TUI 进程存在不阻止 TTL 超时；无前端及 runningTurn 续期则照常退出。

### 由技术方案解决，不转交用户选择实现细节

- 下列六项仍是实施前的设计/验证门槛；人工决策解除不表示技术已验证，
  不自动授权编码、构建或真实 Home 的运行实验。
- **管理协议不能被业务启动门槛挡住**：
  [当前入口](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L13)
  在进入应用前执行 Home migration；
  [服务注册](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L108)
  仍是统一业务服务集合。需将命令分流放在业务初始化之前，管理 stop/status
  不依赖 TUI 业务协议匹配或本客户端能否迁移当前 Home。
  管理协议稳定性还须覆盖其编解码/握手依赖，不能仅靠 DTO 字段不变作保证。
- **现有租约不等于已完成单实例锁**：
  [租约扫描](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseSupport.kt#L82)
  按到期时间清理记录，默认 owner 租期为 30 秒；
  [租约契约](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/src/specMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLease.kt#L7)
  明确租约失效不自动取消父 owner。新锁必须证明并发启动、旧进程恢复、
  锁丢失及清理末期都不出现两个可工作的后端。单实例锁、Home 迁移租约和
  Daemon 空闲 TTL 是三件不同的事；锁记录发布需避免半写、凭据泄露及实例误认。
- **连接故障隔离与重建**：
  [内存 transport](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/rpc/inmemory/WithInMemoryRpc.kt#L37)
  当前端点死亡会结束共同宿主，不能直接搬到多客户端后端。
  必须把单连接清理与后端服务图分离，重连要换掉旧代理、订阅与投影；
  一条慢连接或异常连接不能拖垮其他客户端，不重放结果未知的写入。
- **前端设置与迁移所有权**：
  [当前组合根](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L104)
  仍直接打开前端文件 store。应按新决定移到后端，并优先沿用
  [现有 Get/Flow 与 CAS](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/rpc-state.md#L5)
  及字段冲突规则，不再要求用户重选并发编辑策略。Hook 自动拆分需验证
  中途失败重跑、旧客户端隔离和新协议版本门槛，不提前改写已发布 migration。
- **工作目录与 Hook 所有权**：后端固定 Home 不得丢失 TUI 启动时的项目目录；
  现有新 Session 草稿显式使用前端 cwd，拆进程后仍需保留该语义。
  业务 Hook 由后端持有，不经 TUI 通知订阅转发；前端仅保留本地错误 Hook，
  防止迁移后双执行。桌面通知所需环境、子进程退出清理需验证，不承诺迁移
  命令字符串就等价于原执行环境，也不凭 Hook 在运行额外增加续命来源。
- **跨平台落地验收**：自动后台启动、文件权限/锁、网络 transport、
  stop 接受应答与关闭顺序、认证及版本握手都需要目标平台验证；
  当前读取代码不构成这些能力已通过验收。具体平台实现与参数放入技术方案，
  不扩展到远程连接、服务自动安装或真实 Home 的破坏性测试。

## 第二轮阻塞复核 — 启动与关闭交界

- 仍为 Kodex `27897e20` 的只读审查，不把上一轮已解除的问题重新列为未决。

### 行为审查

- **新启动遇到 Stopping — 已解除**：用户明确这是持锁后端无法服务的
  非法启动边界，新 `kodex` 直接启动失败，不等待旧实例退出后接续启动。
  不夺锁、不强杀、不让心跳撤销停止；旧 TUI 不自动接入新实例的规则不变。
- **手动编辑设置文件的生效方式 — 已确认显式重载**：用户选择管理 CLI
  重载命令，无需重启后端、不自动监听文件。具体协议及失败处理仍需设计；
  不将该选择视为设置发布及所有运行资源能自动事务回滚的承诺。

### 新增技术缺口

- **自动拉起不能直接复用受管子进程的生命周期**：
  [ProcessClient 契约](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/spec/src/specMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClient.kt#L66)
  规定 owner 关闭会终止其子进程，原始标准流也由该 owner 持有。
  后端引导必须有独立启动/交接边界，证明启动方结束、取消、终端关闭或
  输出管道无人读取不会连带终止或阻塞已经就绪的后端；不能用泄漏 Job/句柄
  绕过现有资源清理契约。进程创建成功不等于后端认证及业务初始化就绪。
- **日志不能未经核对就由多个进程共同轮转**：
  [日志配置](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/logging/impl/src/fileLoggingMain/kotlin/io/github/stream29/kodex/utils/logging/ApplicationLogging.kt#L29)
  当前统一使用 `log/Kodex` 名称。多 TUI、管理 CLI 与后端需明确独立写入/
  轮转所有权以及总保留预算；现有代码不构成跨进程轮转安全的证明。
  启动失败仍需有诊断出口，不把尚未连接成功的后端当作唯一日志通道。
- **关闭是不可逆阶段，不是普通断线**：接受 stop 后需关闭业务准入，
  心跳不能使其重新 Ready；单实例锁需覆盖后端资源释放，不在停止 RPC
  返回时提前解锁。status 应能区分可确认的停止中、未运行和无法确认状态，
  不能仅凭端口连不上报告未运行。取消或资源清理不响应时仍不自动强杀。
- **后台通知消费者的启动顺序**：
  [现有执行器](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/notification/src/commonMain/kotlin/io/github/stream29/kodex/cli/notification/NotificationHooks.kt#L44)
  原来由 TUI 持有。迁移后需在接受 Agent 业务前建立后端消费者，
  并解除 TUI 的业务通知执行链；否则无 replay 的事件源可能在启动交接时
  丢掉通知，或新旧路径同时执行。保留前端本地错误输入，不混成 Agent 事件。
- **配置损坏与磁盘变更不可混同订阅同步**：现有设置 store 只在打开、
  显式 reload 或写入路径读文件，不能由 TUI 订阅推导出磁盘文件自动热加载。
  初始非法 YAML 仍按现有规则失败，不擅自覆盖成默认值或增加修复型后台。
  新增显式重载需串行协调两类设置的解析、校验、运行副作用及订阅发布，
  并明确部分失败的结果。当前 store 的 CAS 路径也会重读磁盘，不能简单
  暴露 reload 就声称所有外部编辑都只会在重载时生效；需统一文件与内存
  真源的边界，防止重载绕过 MCP 等既有更新校验。

## 第三轮阻塞复核 — 显式重载与运行状态

- 只读核对 Kodex `27897e20`，不改产品实现；重载命令已获设计确认，
  本轮不重新讨论是否需要该命令。

### 用户调整与后续设计

- **两文件部分接受问题已被新方向替代**：用户提出两份设置合为一份，
  并确认保留 `backend` / `frontend` 分区，不再询问哪份文件先接受。
  后续明确单文件重载接受边界；
  不能把文件合并当作用户选择了先前问题中的任何失败策略。
- **外部编辑与后端保存冲突 — 已确认运行中设置优先**：用户选择后端按
  运行中设置保存，允许覆盖尚未重载的手动修改；不增加外部编辑冲突拒绝
  或三方合并机制。TUI 与后台 token 回存都遵循此规则。
- 运行资源更新不因单文件而自动获得事务回滚能力；仍需在发布配置前校验，
  不覆盖用户磁盘文件以伪造成功。

### 新增技术发现

- [当前 store.reload](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/FileSystemBackendSettingsStore.kt#L33)
  读完立即发布新状态；它不包含
  [业务设置写入边界](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalState.kt#L111)
  的 MCP 配置变化校验与 manager 协调。新命令不能直接暴露 store.reload，
  必须在发布前检查候选配置，再由明确的接受边界应用。
- [MCP 配置校验](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpConfigurationValidation.kt#L12)
  禁止把已有凭据直接移植到不同 server 身份。重载不能成为绕过此规则的
  旁路；协议应区分配置未被接受，与配置接受后外部服务暂不可连接。
- 后台 MCP token 保存也使用设置 store 的读改写路径，不只是 TUI 会修改
  文件。按用户确认，保存应从后端当前运行快照生成，不再沿用每次写入前
  读磁盘并吸收外部修改的行为；外部文件通过显式重载进入运行状态。
  重载、TUI 写入与后台 token 回存仍须串行协调，不能用较旧的内存快照
  覆盖后端自己已经接受的新值。
- [MCP manager 提交](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt#L331)
  会处理删除 server 时的登录取消和认证状态变化；重载应走同等业务协调，
  不能只刷新订阅视图却留下旧登录流程。不能因配置更新取消所有 Agent，
  也不新增全局停机式重载。
- 重载范围是应用设置（现按单文件方向设计），不自动扩展为重新执行 Home migration、
  重置 Daemon 密码或模型登录。既有供应商认证维护有独立文件和续期路径，
  不因新增设置重载而合并所有凭据生命周期。
- Agent 业务错误 Hook 移到后端后仍需保留既有错误归因：运行失败的
  `AgentRunFailureReported` 回执不再触发前端本地 Hook；普通连接错误仍归
  前端。删除业务通知订阅不能误删这条区分，也不能把 Hook 执行失败再次
  转成 Agent 业务错误产生递归通知。

## 第四轮阻塞复核 — 连接恢复与本地交互

- 只读核对 Kodex `27897e20`；自动恢复同一实例、不重放未知结果操作、
  新实例报错等已确认规则不重新询问。

### 前端状态归属 — 沿用既定设计，撤回重复提问

- **已打开页签 — 用户明确为纯前端状态**：opened Session tabs 不属于
  后端；后端只维护 active/running Agent，不增加 tab RPC 或后端页签目录。
  前端页签的存续与 RPC binding 的重建分离，不能将后端 Agent 活跃列表
  当作当前 TUI 已打开的页签列表。
- **本地草稿归属已定，不再让用户重选**：
  [RPC 业务边界](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/rpc-architecture.md#L23)
  已明确前端保留导航、弹窗、编辑草稿与展示状态；
  [Composer 已交付记录](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-03-extract-composer-component.md#L36)
  明确 text/cursor/revision 由唯一 Composer 持有。此前把“允许清空草稿”
  当成减少重连实现工作的选项不当，现撤回。
- 网络 binding 重建不等于用户关闭前端 owner；技术方案应保持纯前端状态
  的所有权，不通过重建整套界面丢弃未发送输入，也不复制第二份草稿真源。
  对已失效业务目标的确认框、缓存和命令仍按既有准确 owner/nonce 规则失效，
  不能为“保留 UI”复活旧操作。保留本地输入不等于重发结果未知的提交。
- 本轮查出的是独立连接生命周期接入的技术缺口，不将其声称为既有重连
  测试已通过，也不因新增 transport 重新设计前后端职责。

### 技术门槛与可复用规则

- [RpcServices](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcServices.kt#L15)
  持有从特定 client 创建的固定代理；
  [RpcSessionViews](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionViews.kt#L108)
  仅自动重试 SessionNotActive，其他错误进入 Failed，恢复时会关闭并替换
  AgentViewModel。重连协调不能只换底层 socket，也不能让旧代理继续接收操作。
- [Composer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/ComposerViewModel.kt#L37)
  持有输入文本、修订号和提交状态。连接失效时应区分本地输入与后端状态，
  阻断无有效 binding 的写入，不建立断线操作队列；已发出的命令保留结果未知
  语义，不能靠重新读取相似历史推断某次提交恰好成功或失败。
- Daemon 默认 5 分钟 TTL 与 Session 现有 60 秒 TTL 不同：即便恢复到
  原 Daemon，Session owner 也可能已失活并重建。应复用现有 Session 保活、
  cacheNonce 校验及迟到结果隔离，不因实例相同就复用旧历史缓存或旧确认框。
  缺失 Session 不得自动创建替身；页签恢复不自动 resume Agent。
- [OAuth 前端桥](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcOAuth.kt#L33)
  将 listener 与 exact attempt 放在前端作用域，闭包使用当前 GlobalRpc。
  重连不能只保留浏览器窗口而遗漏旧代理/监听器的清理；既有登录失败后
  手动重试、不重放 callback、有限取消和后端 attempt 过期规则继续有效，
  不新增登录迁移到其他前端或后台无条件续命。
- 成功建连还需初始化必需设置与状态；不能先用伪造默认值把 TUI 标为
  Ready，再让初始化失败落成永久陈旧界面。运行期断线需有独立可见状态，
  不依赖 stateIn 将上游错误自动广播给 UI。

## 第五轮阻塞复核 — 旧客户端、迁移与版本隔离

- 只读核对当前 Kodex `daa15ea7`；其他 Session 的产品/文档改动未触碰。
  本轮没有新增必须由用户决定的行为，以下从已有规则推导为实施前门槛。
  额外 restart、独立密码重置等未经要求的扩展不再挂作“待确认阻塞”。

- **新单实例锁不能独自隔离旧 CLI**：
  [现有 Home 准备](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigration.kt#L84)
  在无需迁移时只持共享租约；旧 CLI 不认识新 Daemon 锁。
  配置合并必须以更高版本 migration 激活，并沿用独占迁移要求：
  仍有旧 Home owner 时拒绝迁移，要求先正常关闭旧程序，不强杀或偷改设置。
  迁移完成后，旧版按 stored Home version 高于自身版本拒绝打开。
  这些入口检查不替代前述租约过期/暂停进程的单实例安全验证。
- **合并迁移不能照抄拆分迁移的缺失规则**：
  [历史 0.4.7 拆分](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/v0_4_7/MigrateToV0_4_7.kt#L14)
  拒绝“旧源不存在且只剩一个目标”的迁移歧义；
  但[正常加载](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/SplitSettingsPersistence.kt#L15)
  允许已迁移 Home 的任一设置文件缺失并采用默认值。
  新合并 entry 必须接受这种合法起点；目标已经发布而旧源只删除一部分的
  重入场景需另行核对，不能将缺失旧源对应的有效目标分区误重置为默认值。
  目标文件名即使复用历史名称，也须按 Home registry 顺序解码，不按文件名
  猜格式或重跑冻结的历史拆分。验收覆盖旧单文件直升、两源/单源/无源、
  目标冲突和删除中断后的重跑。
- **三种版本不是同一个值**：应用发布版本、业务协议版本、Home migration
  checkpoint 分别校验。TUI 业务准入只要求已确认的协议版本相同，
  不额外用本地可执行版本与磁盘 checkpoint 相等作门槛；磁盘准备和迁移
  由实际运行后端处理。管理命令更不能因业务协议不同失去 stop/status 能力。
- **稳定管理协议不能携带整份业务模型作为依赖**：
  [当前 RPC contract 模块](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/build.gradle.kts#L13)
  依赖 settings、MCP、Session 等业务模型。管理契约应保持小而独立，
  显式重载请求无需提交客户端解析的设置对象，由后端按自身 schema 读取；
  管理结果避免捆绑当前业务异常全集或暴露配置/凭据原文。
  底层传输/编解码兼容仍需版本矩阵验证，不用“方法少”代替兼容证据。

### 审查收敛

- 当前剩余项是技术方案与验收矩阵，不是让用户再次决定前后端归属。
- 后续应把已列门槛收敛为启动/关闭状态机、最小管理契约、配置迁移与
  重载提交顺序、连接恢复资源图及平台测试；不为实现难度重新提出丢草稿、
  忽略未知结果或绕过版本/锁检查的产品降级。
