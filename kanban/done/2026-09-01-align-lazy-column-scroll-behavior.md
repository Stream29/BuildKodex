# Task Tree

- 对齐 LazyColumn 的滚动消费行为
  - [done] 复现 item 粒度相关的滚轮速度差异
  - [done] 对照 Compose LazyList 的滚动与预取机制
  - [done] 确定同步重测与 pending delta 路线
  - [done] 制定具体修改与验证计划
  - [done] 实现滚动状态与同步重测
  - [done] 改造按行计算的测量窗口
  - [done] 增加快速路径与生命周期保护
  - [done] 扩展 Mosaic 预组合与预测量能力
  - [done] 实现方向感知的空闲预取
  - [done] 补齐行为、集成与性能测试
  - [done] 更新滚动行为文档

# Details

- 当前阶段只确认实施计划，不修改生产代码。
- 目标是对齐 Compose 的滚动消费语义，而不是完整移植像素滚动、fling、动画或 nested
  scroll。
- 单个滚轮事件仍请求三个终端行。只要尚未到达真实数据边界，item 的拆分粒度不得改变
  实际滚动行数。

- 当前问题
  - Mosaic 在每帧布局前排空当前输入。多个滚轮事件可能在一次新布局发生前连续到达。
  - `LazyListState.scrollBy` 只能在旧 `LazyMeasuredWindow` 内移动，并把窗口边缘误当成
    数据边缘。
  - 当前向前缓存主要按 item 数量扩展。细碎 item 对应的缓存行数较少，因此同一批输入
    更早得到零消费；大 item 天然提供更多已测量行。
  - 单纯增大 overscan 只能掩盖问题，仍会受 item 粒度和 burst 大小影响。

- Compose 对齐原则
  - `LazyListState` 保存尚未布局消费的整数行 delta。
  - 当前布局结果能完整覆盖请求时走快速路径，不触发完整重测。
  - 请求跨出当前测量缓存时，立即同步重测；measure 遍历必要 item 并返回真实消费量。
  - 只有 provider 的真实首尾边界可以留下未消费 delta。
  - Prefetch 在空闲预算内提前组合、测量滚动方向上的下一个 item；它优化跨缓存延迟，
    但不参与滚动消费量和边界判断。

- 阶段一：滚动状态与重测连接
  - 在 `LazyListState` 增加 pending scroll delta 和本次 measure 的消费结果。
  - 为 `LazyColumn` 绑定 Mosaic `Remeasurement`；绑定不受
    `userScrollEnabled` 影响，后者只控制用户输入。
  - 处理 attach、detach、state 替换和重复绑定，避免失效节点被旧 state 持有。
  - `scrollBy` 先尝试快速路径；无法完整消费时累积 pending delta，并同步调用
    `forceRemeasure()`。
  - 空数据、显式位置请求、异常和 detach 必须确定性清理 pending 状态。
  - 保持现有交互约定：按实际消费量发出滚动 interaction；真实边界的零消费允许事件
    继续冒泡。

- 阶段二：measure 消费 pending delta
  - 将 pending delta 输入 `LazyColumn` 的 measure 流程，由 measure 计算新 anchor、
    visible items、缓存窗口和实际消费行数。
  - 正向和反向布局都按视觉行遍历任意数量的可变高度 item。
  - 覆盖零高度 item、超高 item、尾部回填、部分边界消费、空列表和数据缩减。
  - 保留 stable key 恢复、尺寸变化和 reverse layout 语义。
  - 测量期间不得触发 History 存储读取；当前 materialized provider 的首尾仍是本次
    measure 的真实边界。

- 阶段三：行数化缓存和快速路径
  - 将前后缓存统一改为按终端行计算，初始目标为视口两侧各一个 viewport。
  - 缓存是性能窗口，不是滚动边界；跨缓存必须同步扩展测量。
  - 实现 Compose `copyWithScrollDeltaWithoutRemeasure` 同类快速路径：请求和新视口都在
    现有几何范围内时，只更新 anchor、layout info 和 placement 所需状态。
  - 完整同步重测次数应与跨缓存次数相关，而不是与滚轮事件数相关。
  - `bringIntoView` 保持同步定位，但要避免 state 已触发重测后再次重复重测。

- 阶段四：Mosaic 预组合与预测量原语
  - 为 `SubcomposeLayoutState` 增加公开的 `precompose`，返回可取消的预组合 slot handle。
  - handle 支持按正常 item constraints 执行 `premeasure`，并暴露安全、幂等的 `dispose`。
  - 预组合 slot 不参与当前布局；后续同 key `subcompose` 必须直接接管该 slot，保留已完成的
    composition 和 measurement。
  - 未命中的预组合 slot 在取消、节点 detach、state 替换或 provider 失效时释放，并遵守
    现有 content type 复用和 slot retention 规则。
  - 更新 Mosaic JVM、Native API dump，并测试预组合接管、预测量复用、取消、重复 key、
    remembered state、effects 和节点生命周期。

- 阶段五：LazyColumn 空闲预取
  - 增加可取消的 prefetch scheduler。它只能在当前输入、布局和绘制完成后执行 UI 线程
    工作，并在新输入或下一帧工作到来时优先让出。
  - 将 precompose 和 premeasure 拆成两个可取消阶段；每个阶段开始前主动让出 UI 线程，
    使同一批输入和当前布局优先完成。
  - 默认策略与 Compose LazyList 一致：根据最近的非零滚动方向，预取有效缓存窗口之外
    相邻的一个 item。
  - visible items 或滚动方向改变时重新评估目标；方向反转、provider/key 变化、真实边界、
    state 替换和 detach 时取消陈旧请求。
  - 命中预取后，正式 measure 必须直接接管已组合、已测量 slot，不得再次组合或测量。
  - reverse layout 使用视觉滚动方向推导目标，不得把数据 index 的增减方向直接当成视觉
    方向。
  - 预取不得触发 History 存储读取，不得改变 visible layout info、anchor 或滚动消费量。
  - 第一版只实现单列表、单相邻 item 预取；nested prefetch、自适应多 item 预取和可暂停的
    增量 composition 不在本任务范围内。

- 阶段六：回归测试
  - 在六行视口连续注入十个滚轮事件。普通布局和 reverse layout 下，单行 item 与单个
    百行 item 都应在非边界位置消费三十行。
  - 覆盖两个方向、混合高度、零高度、超高 item、真实边界的部分消费和零消费冒泡。
  - 验证单次滚轮仍为三行，PageUp、PageDown 和程序化 `scrollBy` 不回退。
  - 验证快速路径不会为每个事件完整重测，跨缓存时会在 `scrollBy` 返回前完成重测。
  - 验证 stable key、增删重排、视口缩放、尾部回填及 state 生命周期。
  - 验证 focus `bringIntoView`、History reverse layout、follow-latest 和按需分页。
  - 使用可控 scheduler 验证相邻 item 的预组合、预测量、正式接管和无重复工作。
  - 验证方向反转、快速连续滚动、provider 变化和 detach 会取消陈旧预取。
  - 验证有输入或布局待处理时不会执行预取，预取异常不会破坏主布局。

- 阶段七：性能与真实终端验收
  - 使用一万项的细碎、混合和大块内容，覆盖六、三十和六十行视口以及普通和反向布局。
  - 分别测试一、十和一百个事件的 burst，记录滚动位移、测量 item 数和完整重测次数。
  - 记录 precompose、premeasure 的调度数、取消数、命中率和正式 measure 的重复工作数。
  - 非真实边界处，消费行数必须等于请求行数，且不得因 item 分组不同而变化。
  - 组合和测量规模应受经过的行数及最终缓存限制，不得退化为全列表测量。
  - 稳定方向滚动并有足够空闲时间时，下一次跨缓存应优先命中已预测量 slot。
  - 持续高频输入时允许预取被取消或跳过，但输入、布局和绘制不得等待预取。
  - 运行 JVM 测试、Linux Native 测试与 release 构建，并在真实 PTY/TTY 中人工复核。

- 预计修改范围
  - `Kodex/Mosaic/mosaic-runtime/.../SubcomposeLayout.kt`
  - `Kodex/Mosaic/mosaic-runtime/...` 下的 prefetch scheduler、测试和 API dump
  - `Kodex/app/contract/lazy-list/.../LazyListState.kt`
  - `Kodex/app/contract/lazy-list/.../LazyColumn.kt`
  - `Kodex/app/contract/lazy-list/.../LazyColumnBringIntoViewModifier.kt`
  - `Kodex/app/contract/lazy-list/...` 下的 prefetch strategy 和 lifecycle 实现
  - `Kodex/app/view/components/.../LazyColumnTest.kt`
  - `Kodex/app/view/history/...` 下的 History 集成测试
  - `checklist/tui-interaction-components.md`

- 完成条件
  - 滚动距离不再依赖 item 粒度。
  - 连续输入跨出旧测量窗口时不再错误返回零消费。
  - 真正到达数据边界时仍准确报告部分或零消费。
  - 大列表保持惰性、有限测量，并有测试防止性能回退。
  - 空闲时能够提前组合和测量下一目标 item，正式布局可以无重复工作地接管。
  - 高频输入优先级高于预取，陈旧预取可以及时取消且没有状态或 effect 泄漏。
  - 真实终端中本次复现的滚轮快慢差异消失。

- 实施结果
  - 同帧十个滚轮事件在普通和 reverse layout 下均通过细碎单行项与百行大项的三十行
    位移回归测试。
  - Mosaic precompose handle 已覆盖预测量复用、正式接管、幂等取消和 effect 释放。
  - LazyColumn 预取已覆盖缓存外相邻项的预组合、预测量及方向反转取消。
  - JVM 与 Linux Native 的 Mosaic runtime、LazyColumn 组件测试及 Mosaic API check
    通过。
  - Linux x64 release executable 构建通过，并在 120×30 PTY 中启动和正常退出。
  - History JVM 测试编译被已有未提交改动阻塞：
    `ReplaceableHistoryModel`尚未实现新增的`requestScrollToStorageIndex`。
