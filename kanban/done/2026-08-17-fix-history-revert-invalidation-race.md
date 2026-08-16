# Task Tree

- [done] 修复 History revert 全量失效竞态
  - [done] 确认 sequence、count 与 generation 的发布顺序
  - [done] 定义原子 committed window contract
  - [done] 恢复原子的全量 invalidation 语义
  - [done] 覆盖渲染期间 revert 的回归测试
  - [done] 运行相关测试并构建 release
  - [done] 使用真实 session 验证 revert

# Details

- 状态：`done`。原子窗口修复、自动化验证与真实 session 实测均已完成。
- 崩溃时 LazyColumn 仍测量旧 item index，但 `peek()` 已读取到空 committed sequence。
- 修复不得用越界占位项掩盖竞态；revert 必须使旧 committed window 整体失效。
- 将 generation、size 与索引访问合并到单个不可变线性窗口快照中。
- LazyColumn 的 count、key、contentType 与 item 闭包必须捕获同一个窗口实例。
- destructive replacement 发布新 generation 的空窗口，再在同一 generation 发布重载窗口。
- 已发布旧窗口必须继续保持可索引；其 viewport demand 在窗口过期后不再生效。
- JVM 与 Linux Native 相关模块测试均通过。
- IDEA 增量构建通过，仅有既有 native/cinterop 警告。
- `:app-cli:linkReleaseExecutableLinuxX64 --no-configuration-cache` 通过。
- 本地试用包：`Kodex/out/kodex-0.2.5-local-linux-x64.tar.gz`。
- 隔离复制并脱敏的真实 session 已从 299 个 committed item 回退到 6 个；TUI 保持运行，日志无异常。
- 手工验证使用的临时 HOME 与 tmux session 已删除。
