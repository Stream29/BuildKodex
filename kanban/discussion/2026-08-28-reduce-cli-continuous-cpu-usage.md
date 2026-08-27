# Task Tree

- 降低 Kodex CLI 等待和运行期间的持续 CPU 占用

# Details

- 状态：`discussion`。当前仅记录调查事实，不进入规划或实现。
- 异常实例为 tmux `ACodeSpace:4` 中的 PID `2888416`。约 21.7 小时内累计使用约 4 小时 11 分钟 CPU，平均占用一个逻辑核的 `19.3%`。
- Mosaic 当前无条件以约 1,000 FPS 驱动外部 frame clock：`Kodex/Mosaic/mosaic-runtime/src/commonMain/kotlin/com/jakewharton/mosaic/mosaic.kt:103`。
- 每次 frame 后只等待 1 ms：`Kodex/Mosaic/mosaic-runtime/src/commonMain/kotlin/com/jakewharton/mosaic/mosaic.kt:111`。
- frame listener 会在每次脉冲中检查输入、发送内部 frame，并检查 layout/draw：`Kodex/Mosaic/mosaic-runtime/src/commonMain/kotlin/com/jakewharton/mosaic/mosaic.kt:350`。
- Running Indicator 的可见帧间隔为 100 ms，但通过无限动画消费 Mosaic frame：`Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/RunningIndicator.kt:19`、`:27`。
- 任一已打开 session 运行时都会启用该动画：`Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt:79`。
- 隔离采样中，进程连续 3 秒没有磁盘读取，所属 session 也没有新日志事件，但仍占用一个逻辑核的 `15.7%`，并产生约 875 次 voluntary context switch/秒。
- 进程生命周期平均产生约 905 次 voluntary context switch/秒，与 1 ms frame pump 的唤醒频率吻合。
- 次要放大因素包括同时打开多个大 session、本地 JSON 读取与反序列化、Kotlin/Native GC，以及无界 Native 文件 I/O dispatcher。
- 该进程持有 5 个 session，共约 337,942 个文件；其中 `AutoBlackSoul` 和 `冰雹猜想` 的磁盘占用分别约为 1.1 GiB 和 2.6 GiB。
- Native 文件 I/O 当前使用 `Dispatchers.IO.limitedParallelism(Int.MAX_VALUE)`：`Kodex/utils/kotlinx-io-coroutines/src/nativeMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/PlatformCoroutineFileSystem.kt:9`。
- 后续 discussion 需要确定 frame pacing、Running Indicator 定时方式和大 session I/O 是否拆分处理；当前尚未确定修改路线。
