# Task Tree

- `Trace real UI, History paging and Home ownership boundaries`()
- `Receive scope and engineering-plan approval`()
- `Receive implementation authorization`()
- `Execute isolated History, UI-foundation and Home work lines`()
- `Integrate retained View state and all consumers`()
- `Validate original UI, paging, Home lifetime and frozen migrations`()
- `Run the current isolated CLI`()
- `Independently reaccept the batch under conditional commit authorization`()
- `Record scoped acceptance and archive the completed work lines`()

# Details

## Authorization and baseline

- 用户选择两组完整收口，评审方案后明确“直接去执行推进”。
  本批直接进入 executable。随后用户授权独立验收，通过则提交并规划下一批；
  本轮不包含推送或修改其他用户任务。
- `Kodex/refactor/spec` 基线为 `8297d38873e654ce2fce3c78bde905db6f491017`，
  内层工作区干净；外层其他用户文件、Draft 与 shared-context 不动。
- 约束：[spec/impl](../../checklist/spec-impl-module-boundaries.md)、
  [frontend](../../checklist/frontend-application-boundary.md)、
  [Home](../../checklist/kodex-home.md)、
  [资源](../../checklist/coroutine-resource-lifecycle.md)。

## Hard-cutover mapping

| 原项目/职责 | 唯一接管方 |
| --- | --- |
| `app/contract/lazy-list` 全部 UI 声明、实现与测试 | 现有 `app/view/components`；旧 build/source 删除 |
| `app/view/components` | 保留原公共 UI 项目，无新的 LazyList VM/Controller/spec 壳 |
| `app/view/patch` 全部呈现、纯 diff 与测试 | 现有 `app/component/history/impl/view`；FQCN 不变 |
| `app/migration/contract` | `app/migration/spec`，原版本/entry/真实资源与异常契约 |
| `app/migration/impl` | 保留；当前启动 API 接管新 spec，历史 migration 路径/内容不动 |

- 五个范围项目收敛为三个；History 复用已有项目，总 build 入口预期 208→206。
- History 的唯一 `AgentHistoryViewModel`、window、item 和算法身份不变。
  去掉 Compose/Mosaic/LazyList 对 spec 与 ViewModel 的耦合，不增加平行 VM 或 resolver。
- 原公共 UI helper 保持唯一；已被 History/Index 等真实共同使用的只读呈现
  不为目录纯净而复制或制造循环依赖。

## History boundary

- VM 保留 bounded paging、nonce/generation、payload lazy lifetime、pending/streaming、
  seek 数据加载和 portable follow-latest intent；View 持有测量、焦点、滚动位置。
- View 上报捕获 window 与准确 visible items，供原保留 chunk 算法使用；
  stale/released window 不覆盖当前报告。不上报终端行偏移或任意 framework key。
- `followsLatest` 成为普通 StateFlow。真实 pointer/keyboard scroll 更新跟随意图，
  focus/programmatic scroll 不被当作用户退出跟随。
- VM 只发布标准滚动意图；View 在准确 generation/target 上执行，重复同目标
  请求、未 mount、迟到 acknowledgment 和 close/replacement 均有明确语义。
  不用会丢未挂载导航的无缓冲广播，不建立自动重放 journal。
- 根 View 保留按准确 History VM keyed 的 renderer-only 状态，tab 切换不重置位置；
  关闭/替换清理。该缓存不持有另一份业务 window，不成为 Application registry。
- View 内容变化直接触发本地跟随；删除 VM 的纯布局 `notifyContentChanged`。

## Ownership

- [History 线](2026-10-07-decouple-history-renderer-state.md)：History spec/VM/renderer、
  专属 tests；不碰新增 Patch 文件及宿主/Gradle。
- [UI 线](2026-10-07-merge-lazy-list-and-patch-views.md)：LazyList/公共 components
  与 Patch 源码/测试合并；不碰 History 主 renderer、VM 或宿主。
- [Home 线](2026-10-07-cutover-home-migration-spec.md)：新 spec、非冻结启动实现/tests。
- 协调者：所有宿主 API、保留 View state、Gradle 退役/retarget、CLI、Docs
  回归、公共规范与最终证据；并行线不构建、不操作 IDE、不提交。

## Frozen Home guard and exclusions

- 已发布 migration 的 `v*` source、私有 codec、fixture 和路径保持基线字节不变；
  registry entry/version、应用版本及 frozen skills 不变。不新增 migration entry。
- 移动非冻结版本/entry 声明，不把历史 migration 改用当前业务 store/schema。
- Home 保留真实 `prepareKodexHome` 工厂、唯一 handle 和当前生成版本源，
  不新增 manager/runner/version-provider。
- 窄屏 sidebar、State compact 覆盖、工具 wire/retry 政策与原压力 timeout
  根因不混入目录迁移；新发现先复现，改变行为另列评审。

## Acceptance gates

- History：window initialization/older/newer/nonce、准确 visible retention、稳定锚点、
  transient prefix、重复 Check out、tab 切换、未 mount effect、stale ack、
  follow-latest/input source、payload release 与关闭。
- UI：原 scroll/focus/paging、Patch disclosure/200-line pages/完整数据与窄宽输出；
  原 Docs assertions/资源保留，不用手工 VM 调用替代真实 renderer 输入。
- Home：空/当前/旧/未来版本、损坏文件、跨版本、取消、primary/suppressed 清理、
  重复 close、lease release/restart；冻结源与 fixture 路径及内容逐文件比较。
- 构建：实际项目依赖图无环/缺失，旧入口/声明唯一退出，History spec/VM
  无 renderer 主依赖；受影响 JVM 回归、CLI/Integration 编译、Native 主源码。
- 实际隔离 CLI：导航/滚动/Settings、物化与重开、Home 启动/退出；
  无账号无法实操的 live/provider 场景单列，不计为绿色。
- 共享资源：构建前检测并复用实际 Daemon JVM，单 worker；用户接管即停，
  不操作其他 IDE 项目。临时 Home/PTY/日志及自建闲置资源最终清理。

## Execution checkpoint

- 三线主体实施与唯一生产消费者接管完成；原三个 build 入口删除，现有构建入口为
  206（原 208），静态主依赖图无环/缺失；History spec/VM 无 renderer imports。
- 全部 16 个冻结 Home 文件（含 registry）在原路径与基线逐字节一致。
  23 个 LazyList/Patch 迁移文件内容不变，不调整原滚动或 Patch 呈现算法。
- 中央验证已完成 Home/通用 UI/History 与下游 JVM 回归、CLI/Integration JVM 编译，
  以及本轮八个受影响项目的 Linux x64 主源码编译。最终汇总在实际 CLI 后固定。
- 首轮 History 的真实滚动输入竞态修正为 View 的同步、mount-scoped 输入分类；
  不保留 no-replay collector 的窗口切换间隙，也不使用 Unconfined 调度补丁。
- 复验的“重挂漂移一行”消融结果：无 VM 的原 LazyList 准确恢复；带可聚焦行的
  standalone 列表重挂触发已有 FocusRelocation。History 完整宿主用例保留外部
  tab/composer 焦点，准确输出/anchor/offset 全部断言通过；未改框架算法。
- 启用原 Patch 全管线 probe 后发现旧 helper 返回 disclosure release 的较早绘制帧，
  与原 Patch renderer-input suites 的双帧等待不一致。仅对齐输入 fixture，
  保留全部规模/行数/数据断言；显式启用 probe、单次采样重跑已通过。
- 实际隔离 CLI 已完成初步建 Home、无账号物化和追加可见历史，正常退出码 0；
  最终源码重新链接后在同一隔离 Home 重开：目录打开 Session、真实鼠标滚动、
  tab round-trip、回到 latest、Settings 当前会话/Hook 页与退出通过。
  scrolled-before/after tab 的完整纯文本屏幕相等；两个退出码为 0，
  Home/Session lease 全清，22 条接受的 user message 保留，版本为原 `0.4.8`。

## Central validation evidence

| JVM target | 实际 XML cases |
| --- | ---: |
| `app-view-components` | 165 |
| `app-migration-spec` | 2 |
| `app-migration-impl` | 70 |
| `app-component-history-impl-viewmodel` | 42 |
| `app-component-history-impl-view` | 71 |
| `app-component-agent-impl-view` | 22 |
| `app-impl-view`（含原 Docs/recording 与 root 生命周期） | 54 |
| `app-impl-application` | 43 |
| `app-impl-session` | 13 |
| `app-impl-rpc` | 73 |
| `app-component-history-index-impl-view` | 29 |
| `app-component-session-sidebar-impl-view` | 24 |
| `app-component-session-tab-bar-impl-view` | 3 |
| `integration-test` 显式启用、单次 Patch 全管线 probe | 1 |
| **合计（含 up-to-date 结果）** | **612** |

- 最终矩阵无 failure/error/skipped；probe 的最后一次使用启用环境变量，
  不是未启用时短路返回的“绿色”。单次采样不声称统计性能改善。
- CLI/Integration JVM 编译、八个受影响项目 Linux x64 主源码和最终 native CLI
  链接通过。保留原 Scroll/Patch 测试、原 Docs assertions/资源，不用手工 VM
  调用冒充 renderer 输入。
- 两个新增 LazyList 消融用例分别验证无 VM 的准确 partial-row 恢复，以及
  standalone 可聚焦列表的原有 FocusRelocation；未改 LazyList 原文件字节。
- 未验证：Mac/Windows actual、Desktop renderer、live OpenAI/OAuth/MCP/长运行
  provider、真实崩溃重启或跨设备性能。隔离 CLI 没有使用用户 Home/凭据。
- 最终 206 个构建入口/主依赖图、22 个当前文档链接/行号、16 个冻结 Home
  文件及 23 个移动 UI 文件内容、内外层 diff 检查通过。
- 隔离 Home/PTY/控制脚本、传输/诊断日志、冻结检查清单与三个旧生成目录
  已清理；唯一自建 Daemon 在无客户端且空闲后正常停止。用户 Home/IDE 与
  原有其他任务/文件未操作，标准构建产物和测试报告保留。
- 实施阶段未提交/推送；下节记录后续独立验收与本地提交授权。

## Independent scoped acceptance — 2026-10-07

- 按用户“自己验收，通过就提交”的条件授权，重新核对真实 spec→VM→renderer→
  根消费者链路、准确 effect 消费、viewport 保留及 Home 实际工厂/资源实现。
  本批范围通过；不把已记录的基线故障作为迁移已修复的保证。
- 完整 JVM/CLI/Integration 编译矩阵重新通过（含 up-to-date 结果）。
  另外对 components、Home impl、History VM/View、Root View 五个 `jvmTest`
  显式 `--rerun`：402 项实际重新执行，无 failure/error/skipped。
  Patch 全管线 probe 显式启用并 `--rerun`，单次采样 1 项通过；不作统计性能结论。
- 新建隔离 Home 实际运行当前 native CLI：历史迁移启动、草稿呈现、真实鼠标打开
  Settings、Escape 关闭及 Ctrl+C 退出通过；退出码 0，版本 `0.4.8`，无遗留锁文件。
  本次短 smoke 不替代前节真实历史滚动/tab round-trip 的完整证据。
- 重算 206 个实际项目：主图无缺失/环，spec 主依赖无 impl/旧混合项目边；
  16 个冻结 Home 文件原路径/字节及 23 个原 UI 移动文件字节再核对通过。
- 四份实施任务归档 done。后续本地提交按可构建硬迁移与补充回归拆分，
  验收文档和子模块指针分别提交；不推送。下一批先规划收尾验收和有证据的
  生命周期/状态一致性问题，不制造无职责的 spec 项目。
- 内层本地提交：`fac1bdb6`（Home 真实契约）、`ec9bf1c4`（UI 收口与 History
  滚动边界）、`6b7129fa`（独立回归与基线故障证据）。内层工作区干净。

## Preserved baseline risks

- Home 隔离测试已复现 lease owner 发布后、返回 handle 前的取消缺口，可遗留
  read lock；背景 cleanup 失败也不由 `Job.join()` 返回或附加为 suppressed。
  本批没有改变获取/清理协议，不把这些 characterization 测试当作协议修复。
  具体证据见 [Home 子任务](2026-10-07-cutover-home-migration-spec.md)。
- State 压缩覆盖并发 settings、窄屏双侧栏与压力超时的历史风险仍独立跟踪。
  不因本轮结构迁移/新测试通过宣称全面消除。
