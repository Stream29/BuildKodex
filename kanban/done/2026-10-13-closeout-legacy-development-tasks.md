# Task Tree

- `Confirm the five requested groups and scoped closeout authorization`()
- `Reconcile existing acceptance evidence and resource ownership`()
- `Finish the previously approved twenty-four-cache cleanup`()
- `Archive the fourteen scoped tasks and update inbound links`()
- `Verify documents, retained evidence and released resources`()

# Details

## 授权与范围 — Session 787 / 2026-10-13

- 用户在获知剩余门槛后，明确要求工具链升级验收、Gradle 开发体验、
  旧 OOM/KGP 实验、旧领域重构验收、缓存清理收尾并标记完成。
- 本次关闭这五组的 14 张原 executable 卡；说明书三卡、planning 的
  备选/延期研究、discussion 新功能及 Draft 均不推进；planning 仅修父任务
  入链与对应状态注释。不进行产品修改、构建、
  在线模型调用、提交、推送或发布。
- 已验证的实施按证据关闭；否决的 KGP 维护与未选择的实验按停止研究关闭；
  未执行的原验收矩阵作为阶段结项的剩余范围保留，不改成测试通过。
  看板完成表示用户授权的阶段结项，不表示所有版本、平台、故障矩阵全绿。
- 缓存清理沿用 Session 773 的 24 个获准精确路径，不删五个完整实验根。
  保留源码、私有堆、报告、原回执、wrapper、发布物、共享锁与 R36/769 资源。
  串行持原设备锁，复核所有活跃进程/open-file 使用，删除不使用 sudo。

## 对账依据

- [工具链卡](2026-10-08-validate-kodex-toolchain-compatibility.md#L1)：
  `450e3bd7` 根升级及原 188 次代表性执行、模型/标准 wrapper/JS lock；
  原 Linux Release 资源中止不改成成功。
- [本轮 Mac 证据](file:///home/stream/ACodeSpace/demo/kodex-closeout-787/README.md#L58)：
  冻结 `bbbc9525` / 对照 `450e3bd7`，同一批准的私有 Gradle 8 GiB 预算。
  完整导入、四次手动 Sync、四个内部 specMain 声明导航、metadata、
  arm64 Release/PTY、112 次原生测试与 18 个增量/重置阶段通过。
  三样本暖 smart 中位数 72.423 → 60.043 秒，不是 empty-cache 或通用提速。
- 原 4 GiB OOM、Xiaoxin 守卫停止、外部 common `Json` 导航、
  source-mode/重开/首次认证/其余宿主矩阵不由本轮内部导航代替。
  Mac 结果也不作为每个旧 Gradle 候选的独立因果 A/B。
- [原核心 E2E](../done/2026-10-07-accept-sealed-cli-core-e2e.md#actual-native-results)
  与 [宿主错误出口重验](../done/2026-10-07-restore-cli-unhandled-error-hook.md#actual-native-recheck)
  已完成；原 `7b23c481` 封存产物保持，不用新版 smoke 冒充旧版本完整矩阵。
- 原四个基线运行时失败、两个编译缺口、B2/U、全部 browser/平台/故障覆盖
  及已发布外部二进制兼容不在完成声明内。未来如要补验须另行立范围。
- 本轮对账时产品 HEAD `2b16ae50` 包含其他任务的 warning/hook 改动；本轮不改产品，
  不把 `bbbc9525` 证据升级成后续 HEAD 的全量验收。

## 任务清单与结项类型

| 组 | 数量 | 文件/处理 |
| --- | ---: | --- |
| 工具链升级验收 | 1 | `2026-10-08-validate-kodex-toolchain-compatibility.md`；已采用升级，接受明示边界 |
| Gradle 开发体验 | 3 | `2026-10-02-rescue-gradle-development-experience.md`、`2026-10-02-plan-gradle-model-resource-optimization.md`、`2026-10-04-plan-kodex-gradle-optimization.md`；已落地批次及测量阶段结项 |
| 旧 OOM/KGP 链 | 7 | `2026-10-08-repair-full-model-idea-import-oom.md` 及 metadata/cinterop/idle/V3/receive/minimum 六卡；不采用实验维护路线，停止本轮研究 |
| 旧领域/E2E | 2 | `2026-09-28-plan-domain-spec-impl-boundaries.md`、`2026-10-07-seal-repair-baseline-and-build-e2e-cli.md`；已验收交付及核心 E2E 结项 |
| 缓存清理 | 1 | `2026-10-12-retire-unused-xiaoxin-dev-directories-773.md`；执行并核对原 24 路径 |

## 执行与守卫记录

- 已只读确认 Xiaoxin 无活跃 Java/Gradle/Kotlin/IDE/Xvfb、共享锁无观察到的持有者；
  预检不作为资源预约；实际执行非阻塞取锁并重复检查。
- 原 Session 773 第一次四路径清理回执与脚本存在，本次使用新的审计目录，
  不覆盖 `kodex-disk-cleanup-773`。
- 原工具脚本两次预检安全停止：先是自身共享锁 FD 被误判，后是原生工具链
  内部硬链接。修正仅限已验证 PID/birth 的自身锁 FD；39 个多链接 inode
  的全部别名清点均在原 24 路径内，外部别名仍拒绝，不扩大删除范围。
- 首次执行在第一个获准缓存内被守卫中止，当时尚无整目录完成回执；
  保留原停止回执/脚本/日志和清理前保留内容快照。续跑修正只豁免同一
  PID/birth 在获准缓存内的目录 FD，不豁免 retained root、普通文件、
  cwd/exe/maps 或其他 PID；用自有、正常退出的只读 fixture 验证隔离。
- 续跑先比较原保留快照、公有证据哈希与保护路径身份通过，不覆盖原失败证据。
- [x] 精确范围、保留内容、进程和链接预检通过。
- [x] 原 24 路径全部处理完成，实际回收与保留内容核对。
- [x] 14 卡归档，入链、历史与当前状态一致。
- [x] 本轮临时运行时/资源清理，证据哈希与文件检查通过。

## 缓存执行通过

- [第二批最终回执](file:///home/stream/ACodeSpace/demo/kodex-closeout-787/legacy-closeout/receipt.json#L1)：
  24/24 目录删除且全部缺席，3,931,647 条保留元信息前后一致。
  两份原始元信息快照 hash 相同，公有证据 hash 与保护路径身份核对通过。
- 本批原缓存占用 173.84 GiB；执行期实际可用增量 172.70 GiB，
  包括审计快照占用与同期文件系统写入，不等同于缓存估算。
  原 Session773 四路径回执不覆盖；源码、原始失败、私有堆、报告、
  wrapper、发布物、R36、769、用户缓存和锁均保留。
- 13 张阶段卡保留历史树和原失败结果，以新的结项树替代旧悬停状态；
  清理卡按实际完成关闭。所有已移动卡的阶段入链已更新，历史行号相应平移。

## 证据保存与最终复核

- [清理证据索引](file:///home/stream/ACodeSpace/demo/kodex-closeout-787/legacy-closeout/README.md#L28)：
  两份元信息原字节无损保留在 gzip，远端/本机逐文件 hash 与解压 hash 相符；
  原停止/完成回执字节一致。冗余 raw 与本轮复制 uv/cache 清理，原锁可重取并释放。
- [最终资源回执](file:///home/stream/ACodeSpace/demo/kodex-closeout-787/legacy-closeout/audit-copy/runtime-cleanup.json#L1)：
  Xiaoxin 可用 205.33 GiB，相对本批预检净增加 173.47 GiB，
  包括审计与同期文件系统活动；不覆盖执行回执的 172.70 GiB。
- 14 张原卡及 1 张协调卡阶段唯一；执行队列仅剩说明书三卡。
  Draft、说明书与产品工作区未改，planning/discussion 未推进。
  原运行/编译缺口与被冻结 revision 的边界保留，不新增全绿结论。
- [看板复核](file:///home/stream/ACodeSpace/demo/kodex-closeout-787/legacy-closeout/board-verification.json#L1)
  通过：153 个 scoped 入链、行号/标题锚点、阶段唯一性、无当前悬停语句、
  空白与保留队列检查；24 份远端/本机审计及脚本字节 hash 相符。
