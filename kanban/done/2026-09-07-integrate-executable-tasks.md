# Task Tree

- [done] 顺序执行当前四项任务并完成集成验证
  - [done] 盘点任务、依赖与用户执行授权
  - [done] 确定顺序与联合验收边界
  - [done] [显式 Medium](../done/2026-09-07-send-explicit-medium-reasoning-effort.md)
  - [done] [稳定缓存键](../done/2026-09-07-add-stable-prompt-cache-key.md)
  - [done] [封闭 stable 层级](../done/2026-09-07-seal-stable-event-hierarchy.md)
  - [done] [建议历史展示](../done/2026-09-07-render-completed-session-suggestions.md)
  - [done] 完成联合验证、文档与工作树核验

# Details

## 执行结果与验证限制（已完成）

- 四项任务均已实现并完成各自定向验收；按既定顺序实施，sealed 与主历史展示
  联合编译，没有用空分支或运行时未知事件兜底过渡。
- 最终相关 JVM 报告：models 40、定向 client 40、AgentState 34、
  定向 filesystem Session 1、clean-models 30、filesystem storage 16、
  session-title 10、定向 WebRun 1、history ViewModel 30、agent ViewModel 11、
  定向 history View 12、application View 77 项，均无失败。
- 本机 linuxX64Test：models 40、clean-models 30、AgentState 34 项通过；
  CLI JVM 编译和 Linux release 链接通过。模型协议由假客户端与 loopback
  HTTP 验证；UI 由 Mosaic 测试验证，不等同于用户 tmux 中的人工操作验收。
- 编译期实验：隔离源码基线成功，新增 index tool/work tool/非工具事件均
  使实际分派编译失败；旧 index/work JSON、retained 投影和嵌套命令结果兼容测试通过。
- **全量限制**：history View 全量运行曾有 9 项展开交互断言失败
  （CleanEventViewTest 7 项、StreamingRequestResponseViewTest 2 项）；
  未声称这些用例通过，未扩展为另一项交互修复。
- **全量限制**：filesystem Session 全量运行两次停滞后终止；本次新增的
  reopen/fork/cache 用例定向运行通过。不能将定向通过表述为全量通过。
- 上述两项限制已在后续用户授权的
  [失败测试与清理死锁修复](2026-09-07-fix-history-tests-and-repair-lease.md)
  中解决：history View 46 项、filesystem Session 24 项全量及两轮强制重跑通过；
  此处保留原批次验证记录，不将后续结果混作当时验收。
- 修正一个既有 history invalidation 测试替身遗漏的
  `requestScrollToStorageIndex` 接口实现，相关定向用例通过。
- 原生 CLI 不处理 `--help` 参数，非 TTY 启动尝试已终止；不计为终端 UI 验收。
  未在用户运行中的 tmux 上重复批准建议或提交模型请求。
- 已更新相关 checklist 和任务入站链接；保留原有未提交 UI 改动及其他会话
  的任务内容。没有 Git 提交、发布或真实模型收益测试。
- Gradle 使用本机现有 JVM `/home/stream/.jdks/openjdk-26.0.2`；
  native 构建报告现有 configuration-cache 不兼容告警，构建本身成功。

- 用户于 2026-09-07 授权将两项历史相关任务推进 executable，连同原有
  executable 任务一起确定顺序并全部实施。此授权解除原有两任务的暂缓执行限制。
- 当前范围恰为四项；模板不是任务，不接管其他新出现的任务。
- 顺序：Medium 序列化 → 默认缓存键 → stable sealed 迁移 → 已完成建议展示。
  前两项独立且改动小，先验证请求协议；后两项共享事件分派，联合完成编译验收。
- 先完成前两项离线与本机 HTTP 验证；sealed 修改前固定旧序列化 golden，
  再迁移层级、补齐渲染，最后做兼容、完备性、历史集成与 native 检查。
- 不新增并行调度或测试基础设施；复用本机 Gradle Daemon JVM，不切设备。
- 保留既有未提交 UI 改动；不创建 Git 提交或发布，不调用付费真实后端。
- 统筹任务仅引用子任务，不复制子任务树；全部子任务完成后才能完成本任务。
