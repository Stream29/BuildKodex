# Task Tree

- [done] 在 History 中显示 turn duration
  - [done] 补全首个 turn marker timestamp
  - [done] 识别 turn marker 与 stable 边界
  - [done] 设计 history 与 active footer 编排
  - [done] 按需计算并缓存 history duration
  - [done] 在 active turn job 下维护秒级 ticker
  - [done] 渲染 Worked for 虚拟分隔行
  - [done] 覆盖 revert、边界与 job 生命周期测试
  - [done] 验证大历史性能与 release 行为

# Details

- 状态：`done`。
- 现有 `HistoryItemViewModel` elapsed duration 保持不变；turn duration 是额外的一级展示。
- `settings[0]` 是首个 turn marker，初始 `turnId` 随 settings 初始化；初始化需要同时写入
  `timestamp[0]`。旧历史缺少该 timestamp 时，以首个 stable item timestamp 兼容。
- 后续 `markNewTurn` 会在 settings timeline 写入新的 `turnId`，并在同一 index 写入
  timestamp。后续 marker 应以 `turnId` change point 识别，不能把普通 settings change
  point 当成新 turn。
- active turn 是当前存在运行中 turn job 的逻辑 turn；history turn 是当前没有运行中
  turn job 的逻辑 turn，包括尚无后继 marker 的最新 turn。
- 有后继 marker 的 history turn 由当前 marker 和下一个 marker 确定范围。结束点是下一个
  marker 之前的最后一个 stable history item。
- 没有后继 marker 的最新 history turn 以当前最后一个 stable history item 为结束点。
- 结束点不按 stable event variant 筛选；reasoning、tool、patch、plan 或 compaction
  都可以成为 turn 的最后一个 stable item。
- history duration 是结束点 timestamp 减去当前 marker timestamp。
- 在结束点后插入只存在于 History 展示层的虚拟 item；不写入 storage，也不进入 model history。
- 虚拟 item 显示为 `---Worked for <duration>---`。
- active turn 使用当前时间减去最新 marker timestamp。每秒刷新一次，ticker 必须是
  active turn job 的 child job，并随 turn job 结束或取消。
- history footer 参与历史线性窗口；active footer 始终位于当前 turn 的可见末尾。
- 有后继 marker 的 history footer 使用 next marker index 作为线性位置；没有后继 marker
  的最新 history/active footer 作为 HistoryViewModel 的独立 tail state，不重建 committed
  window。
- footer 不参与 stable event `read`、item elapsed、revert context target 或 WorkGroup child；
  但 history footer 会作为 folding breaker。
- marker scanner 按已加载 stable window 增量读取 settings change points，并缓存已确认的
  turn boundary；revert 或 generation replacement 时失效。
- 边界与 duration 只通过 index、settings 和 timestamp 元数据计算，不为此解析 stable payload。
- history duration 一旦读取即可缓存；每秒刷新只作用于一个 active turn。
- duration 展示复用现有规则：舍入到最近的毫秒后使用 Kotlin `Duration.toString()`。
- 测试覆盖首个 turn、多个 turn、空区间、timestamp 缺失或倒退、折叠区间、revert、
  active/history 切换、Resume、取消和 ViewModel 关闭。
- 验证：
  - `:app-viewmodel-history:jvmTest`：16 tests passed。
  - `:app-viewmodel-history:linuxX64Test :app-view-history:linuxX64Test`：passed。
  - release CLI binary：`Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`。
  - 使用 release binary 在 120×40 PTY 中实际创建 session；active footer 秒级更新，
    response 后显示 history footer，`Ask the user` 输入请求正常呈现。
- 验证：
  - `:app-viewmodel-history:jvmTest`：16 tests passed。
  - `:app-viewmodel-history:linuxX64Test :app-view-history:linuxX64Test`：passed。
  - release CLI binary：`Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`。
  - 使用 release binary 在 120×40 PTY 中实际创建 session；active footer 秒级更新，
    response 后显示 history footer，`Ask the user` 输入请求正常呈现。
