# Task Tree

- [done] 修复 Codex Home Stop Hook 在 Codex Lite 中不执行
  - [done] 核对 Codex Home 选择、配置解析、trust 与运行时织入链路
  - [done] 使用最新 Linux release binary 复现 Stop Hook 未执行
  - [done] 撤回非必要的原子补偿、哨兵语义与snapshot reader重构
  - [done] 仅保留合法`tokenCount[0] = 0L`初始化及必要测试适配
  - [done] 运行相关测试与构建验证

# Details

- 根因是canonical storage初始化未发布token-count timeline的index 0状态，旧快照读取因没有可见值而取消response flow。
- 最终产品实现只在canonical `initialize()`中增加`tokenCount[0] = 0L`一行。
- `0L`是真实合法token count；现有UI、context-window和其他reader按普通timeline值处理。
- 已撤回额外的跨timeline补偿、in-memory直接构造改动、snapshot reader改写、哨兵语义和竞态测试。
- filesystem session JVM测试、in-memory session测试编译、CLI JVM编译、CLI Linux X64 Kotlin/Native编译和IDE构建检查通过。
