# Task Tree

- [done] 在 History 中显示相邻 stable elapsed duration
  - [done] 确认线性 duration 语义
  - [done] 确认按需 predecessor 查询
  - [done] 确认 leading/trailing 布局方案
  - [done] 增加通用省略标题尾部布局
  - [done] 增加 History elapsed 读取 contract
  - [done] 接入 stable 与 WorkGroup 标题
  - [done] 覆盖时间边界与折叠测试
  - [done] 验证大历史性能与 release 行为
  - [done] 展示前舍入到最近的毫秒

# Details

- 状态：`done`。保留默认单位组合，并在展示前舍入到最近的毫秒。
- 本次只处理 committed `HistoryItemViewModel`。pending、streaming 和绝对时间展示保持不变。
- 除全局第一个 stable item 外，每项标题显示 ` · +<duration>`。
- 普通 item 的区间是 previous stable timestamp 到自身 timestamp。
- collapsed `WorkGroup` 的区间是 oldest child 之前的 stable timestamp 到 newest child timestamp。
- expanded child 继续使用普通 item 区间；全部 child duration 相加等于 collapsed group duration。
- predecessor 通过 `storage.stable.prevIndex(oldestIndex)` 从现有 full index 按需查询；item 不保存 `previousIndex`、timestamp 或 duration。
- 两端必须存在同 index 的精确 timestamp。缺失、负数、非有限值或 timestamp 读取失败时只省略 duration，不影响 stable 内容。
- duration 在展示前舍入到最近的毫秒，再使用 Kotlin `Duration.toString()` 的紧凑单位组合并添加 `+` 前缀。
- `AgentHistoryViewModel` 增加语义明确的 suspend 读取方法，例如 `elapsedSincePrevious(item)`；实现复用 committed I/O semaphore、timestamp LRU 和 generation 校验。
- 普通 item 在现有异步 stable read 中一并读取 duration；collapsed `WorkGroup` 独立按需读取。LazyColumn 未组合的 item 不触发读取。
- 组件层增加通用 leading/trailing 单行布局。使用 `Row`、`Modifier.weight(1f, fill = false)` 和 `EllipsizedText`，让 leading 使用 trailing 之外的剩余宽度并省略，trailing 紧跟且不被 leading 截断。
- 不修改 Mosaic 布局引擎，不在 History renderer 中手算 terminal cell width。
- stable event 的 duration 通过统一 History header 入口接入 message、reasoning、tool、plan、compaction 和 patch；pending renderer 不消费该值。
- `StoredEventLoadState` 同时持有 event 与可选 duration。timestamp 元数据失败不进入红色 stable payload `Error` fallback。
- revert 或 generation 替换后，旧 duration 读取结果不得发布到新 window。
- 组件测试覆盖宽屏、窄屏、Unicode、resize 和 trailing 优先。
- ViewModel 测试覆盖首项、普通 sparse item、隐藏 state transition、`WorkGroup`、展开加和、缺失 timestamp、负 duration 和 revert。
- Mosaic snapshot 覆盖 message、tool、collapsed/expanded `WorkGroup`、plan、compaction、patch 和读取 fallback。
- 实施后运行相关模块 `linuxX64Test`，再运行 `:app-cli:linkReleaseExecutableLinuxX64`。
- release CLI 使用真实大历史验证首屏、旧端滚动、折叠展开、窄终端截断和日志；比较改动前后的首屏与滚动体感，确认没有可见性能回退。
- `:app-viewmodel-history:jvmTest`、`:app-view-components:jvmTest`、
  `:app-view-patch:jvmTest` 和 `:app-view-history:jvmTest` 通过；对应四个
  `linuxX64Test` 也通过。
- 毫秒舍入改动的 `:app-view-history:jvmTest` 和 `:app-view-history:linuxX64Test` 通过。
- `:app-cli:linkReleaseExecutableLinuxX64 --no-configuration-cache` 在舍入改动后重新通过；
  491 个任务，构建耗时 4 分 50 秒。产物 SHA-256 为
  `fa94d09d8b9cafc1d74998083cf763f71103c9f36ad485f0219ecd2ad62db9ea`。
- release CLI 打开包含 13,388 个 stable event 的隔离真实 session；折叠的 5 个 child
  duration 精确加和为 group duration，展开、收起、旧端滚动、60 列和 30 列布局均正常。
- 三轮交替同机 smoke 中，改动前后 session open 中位数分别为 129.1 ms 和 158.4 ms；
  60 个滚动样本的 p50 分别为 32.0 ms 和 27.8 ms，p95 分别为 43.4 ms 和 49.3 ms。
  区间重叠且没有可见卡顿；隔离日志无 error、exception 或 History read failure。
- 舍入后的 release CLI 在隔离大历史中实际显示 `+18.905s`、`+1m 2.064s`；
  可见 duration 最多三位毫秒小数，隔离日志无错误。
- 临时 session、副本、对照二进制和 tmux server 已清理。
