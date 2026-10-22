# Task Tree

- `Freeze independent backend modules and frontend call boundaries`()
- `Split logging libraries and redirect backend consumers`()
- `Keep TUI file diagnostics internal and remove frontend scope dependencies`()
- `Validate model isolation metadata JVM Native and scoped tests serially`()
- `Record remaining lint findings and preserve unrelated work`()

# Details

- 上游：[导出 API 迁移](2026-10-20-audit-exported-api-spec-boundaries.md)。
  用户确认 `utils/logging/backend-scope`、`backend-file` 两个后端模块，
  无相互生产依赖；logging 为分组目录，原父项目硬切换删除。
- 用户进一步选择同时收紧前端依赖。app/rpc、Session Settings ViewModel、
  CLI 异常日志改用普通 KotlinLogging；TUI 不调用后端文件日志初始化。
  CLI 同时有 backend 子命令和 TUI 入口，不把二者共用一个二进制误判为前端调用后端日志。
- 保持能力覆盖：scope 保留原 Shared 目标，file 保留原 JVM/四桌面 Native；
  技术可移植性不授权前端使用 backend-scope。不新增目标、stub、第三个公共日志模块或转发层。
- 后端公开入口/包名/默认值不变，公开声明放各自 specMain，机制留 commonMain。
  两库各有独立测试；原 scope 到文件的组合回归进入 integration-test 的显式筛选测试，
  不运行真实模型集成测试。
- TUI 在 CLI 内保留 internal appender 与初始化，直接复用 KotlinLogging/Kermit/系统 IO，
  保持原格式、滚动参数、Tui-pid 命名及非活跃进程日志清理规则；
  不通过 backend-file、backend-scope 或新的公共客户端日志库实现。
- 产品 HEAD `d212c43cf`；原文件 hash 和必要源码快照在
  `~/ACodeSpace/demo/kodex-detekt-998/logging-before/`。
  保留已有迁移、用户改动和正常共享进程；不暂存/提交/推送。
- 集中单 worker、复用空闲 JDK25 daemon；验证新模块无相互依赖、
  前端应用/RPC/VM 无直接后端日志库依赖或 scope 调用、CLI 仅后端启动分支调用后端初始化。
  编译两个库 metadata/JVM 与真实消费者，补 scope JS/file Linux，
  运行定向测试及严格/报告 lint；已在基线复现的旧失败单列。
- 既有混合 MCP 模块仍被 app/application、app/rpc 为其契约使用，并带有 backend-scope 的
  传递依赖；这是上游保留的 MCP 拆分边界，不在本项擅自重构。
  本轮不得将“前端不直接依赖/调用后端日志库”写成“前端传递闭包已完全没有后端代码”。

## 实施与验收

- 已删除原 `:utils-logging` 项目，注册两个新项目；
  实际 KMP 模型确认父项目不存在，两个库无相互生产依赖。
  file 只有系统 IO 项目依赖；scope 无其他项目依赖，共同第三方 KotlinLogging 不构成两库相互依赖。
- scope 的 spec、实现、测试与迁移前源码字节相同；
  file 除 expect/actual、默认值归属和规范 KDoc 外，实际机制不变。
  文件独立测试改用普通事件 payload，原 scope→文件组合回归保留在筛选集成测试中。
- app/application、app/rpc、Session Settings ViewModel、CLI 不再直接依赖后端日志库；
  三处前端 `.global()` 移除。TUI 分支调用内部 initializeTuiLogging，
  后端子命令仍通过后端宿主初始化；没有让 TUI 调用 backend-file。
- TUI 直接绑定上游 KotlinLogging/Kermit，未新造公共日志模块；
  原分级、marker/payload 格式、异常记录、10 MiB/5 文件滚动参数、Tui-pid 命名和
  八组非活跃 PID 日志清理保留。清理代码除注释外与原流程一致，
  writer 构造仍先于进程级配置变更；回归验证当前进程、旧格式及无关文件不被清理。
- 两库与相关前端/后端消费者 metadata、JVM 编译通过；
  scope JS/Wasm、file Linux Native 测试和 CLI Linux klib 编译通过。
  JVM 定向测试 412 项：407 通过、5 失败、0 错误/跳过；
  scope 4、file 1、TUI 1、显式筛选组合 1 均通过，Linux file 另 1 项通过。
- 五项失败与此前干净 d212c43cf 基线结果匹配：Application 的四个旧版本断言、
  BackendHost 的一个重连断言；测试源码 hash 未变。
  本次没有重新导出基线或修改旧断言，消费者组合命令实际退出 1。
  集成测试只执行 backendLoggingCompositionTest 一项，不执行真实模型用例。
- 53 个 lint 规则样例重跑通过，严格/报告结果一致；
  43→42 项/13 模块，只消除 initializeLogging 发现，无新增声明问题。
  CLI main 仅因 import 删除发生报告行号变化，仍是待讨论入口。
  严格命令实际退出 1，未加 baseline/suppress。
- 快照中除 21 个修改/删除的旧文件和 11 个新增文件外，1487 份产品文件 hash 不变；
  未改协议/版本/lint 机制，未暂存/提交/推送。

## 回执与限制

- 日志、XML 和总回执在 `~/ACodeSpace/demo/kodex-detekt-998/logging-*`；
  总回执 `logging-validation-summary.json` 保留模型、依赖、测试、基线对照、
  源码保持性和 lint 差异。源集探针已存入汇总，临时 init 文件已删除；
  旧父项目的 build 输出和空 src 目录已清理。
- 模型和实际验证均复用同一 JDK25 daemon、单 worker，无资源阈值中断或新 daemon。
  IDEA MCP 初始化失败，未声称 IDE Sync 验收；未操作用户窗口或停止正常共享进程。
  JVM/Linux 之外未做运行验证，不能将编译通过等同于全部宿主已运行。
- 缓存命中强制 lint 全扫两次为 4.526/4.185 s，其中分析 2.680/2.573 s；
  无变化缓存命中为 1.579 s，分析 UP-TO-DATE。
  首次独立全扫含配置为 16.163 s；本次共享重型编译环境未做隔离 A/B。
  只读检查 JIT 仍启用、code cache 未满，观察到 powersave 调频策略；
  没有更改调频或 daemon 参数，不据此断定耗时变化原因。
- 本项完成日志库拆分与前端直接调用/依赖收紧，不表示既有混合 MCP 等传递闭包已隔离；
  该债务继续由上游跟踪。保留正常共享 daemon，无遗留验证 worker。
- 文档已做删减、变更含义和全文一致性复核；有界辅助文档复审超时且未返回意见，
  不计为通过，记录在 `logging-document-review.json`。
