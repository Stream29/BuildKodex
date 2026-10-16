# Task Tree

- `Record the live hang and debugger evidence`()
- **`Discuss safe recovery and the signal callback boundary`()**
- `Define isolated reproduction and regression checks`()
- `Wait for authorization before planning or implementation`()

# Details

## 状态与授权

- 状态：Discussion。用户要求先记录现场；尚未选择救援方式或修复方案。
- 来源：Session 851；用户发现隔壁 tmux 中执行 Dify PR 43842 复审的
  Kodex 卡死，随后授权用 sudo 附加 GDB 抓取线程栈。
- 已执行只读检查、两次 GDB 取证并立即 detach；最后核验 `TracerPid=0`。
  未强制返回栈帧、修改寄存器或内存、解锁、发送恢复按键、终止或重启进程。
- 本卡不授权继续附加调试器、原地救援、重启、产品改动、提交或发布。
  后续接管前先核验 PID、启动时间、可执行文件和 Session 归属，
  不把记录的 PID 当作永久有效标识。

## 现场位置

- 设备：当前 Linux 主机；工作目录
  `file:///home/stream/ACodeSpace/push/BuildDify`。
- tmux：`ACodeSpace:14.0`；TTY：`/dev/pts/12`；
  Kodex PID/TID：`199849`；父 shell PID：`352676`。
- 可执行文件：
  `file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`。
- 取证时该进程持有 Session 839、843、850 的锁；当前复审为
  [Session 850](file:///home/stream/.kodex/sessions/850)。
  干预该进程可能影响三个 Session，不仅是当前页签。
- 持久保留的本地取证文件：
  [mutex 归属、完整主线程栈和各线程短栈](file:///home/stream/ACodeSpace/local/kodex-hang-851-20261015/worker-mutex-main-stack.txt#L1-L369)。
  这是本机证据文件，不是仓库内附件；后续移机时需一并携带。

## 已确认机制

- **主线程在 SIGWINCH 回调中重入 Worker 调度锁，形成自死锁。**
- mutex 地址：`0x1abb8f30`；GDB 读取的 `__owner=199849`、
  `__kind=0`；同一主线程正在 `pthread_mutex_lock` 等待这把锁。
  [锁归属及等待位置](file:///home/stream/ACodeSpace/local/kodex-hang-851-20261015/worker-mutex-main-stack.txt#L5-L13)。
- 被信号打断的位置为 `Worker::putJob`，下层调用来自
  `Worker.executeAfter → EventLoopImplBase.enqueue` 和 UI frame 调度。
  信号栈包含 `mosaic_tty_sigwinch_handler → onResize`；
  回调沿 StateFlow、Compose snapshot 和 channel 通知，再次进入
  `Worker.executeAfter → pthread_mutex_lock`。
  [完整重入链](file:///home/stream/ACodeSpace/local/kodex-hang-851-20261015/worker-mutex-main-stack.txt#L9-L53)。
- 大量其他工作线程也在等待 `0x1abb8f30`；因此不只是渲染停止，
  调度、请求完成处理及维护工作均可能受阻。
  [其他线程等待同一锁](file:///home/stream/ACodeSpace/local/kodex-hang-851-20261015/worker-mutex-main-stack.txt#L91-L123)。
- 现场足以确认这次自死锁，不是仅凭 `futex_do_wait` 推测。
  没有证据把根因归为 GC 卡顿、PR 测试阻塞或单次模型请求缓慢。

## 时间线与观察边界

- 以下本地时间均为 2026-10-15、UTC+08:00。
- 12:26:29：Session 850 最后一条测试命令已返回，耗时约 0.54 秒、
  exit code 1；原因为环境缺少 `pytest_env`，不是测试仍在运行。
  返回结果保存在
  [work/89.json](file:///home/stream/.kodex/sessions/850/work/89.json#L1)。
- 12:26:29.472：随后发起下一次模型请求，没有对应完成记录。
  [运行日志](file:///home/stream/.kodex/log/Kodex.log#L6328-L6331)；
  此日志会轮转，稳定现场依据以保留的 GDB 文件及本节摘录为准。
- Session 843、850 的锁分别最后写于 12:26:31、12:26:32；
  同进程 Session 839 的锁最后写于 12:57:33，租约截止 12:58:03。
  **不能据此宣称整个进程从 12:26 起立即全部冻结。**
- 13:08 只读检查：画面固定为 `Starting response…`、
  `Worked for 33m 35s`；tmux 不在复制模式。
  主线程两次间隔约 10 秒的 `schedstat` 执行时间、调度次数完全相同；
  进程仍存活，不是已退出后残留的终端画面。
- 约 13:15 的第二次 GDB 取证保存了 mutex owner 和完整主线程重入栈，
  随后 detach。此后没有实施救援；本卡不保证后续读取时现场仍原样存活。

## 未决问题

- SIGWINCH 的具体来源未追溯，可能的窗口/布局/客户端尺寸变化
  不能写成已确认的用户操作；故障属于正常信号路径的重入安全问题。
- 尚未做隔离复现、源码级修复或修复验证。
- 取证时工作区 `Kodex` 为 main、HEAD
  `3223a52d9ab0561e35446fa96c14342a87f363c0`；
  当前版本表列 Kotlin 2.4.20、coroutines 1.11.0、
  Mosaic `0.19.0-SNAPSHOT-kodex.78f94c4c2e13`。
  这些是工作区信息，不是已验证的运行中二进制构建来源；
  复现前应核对实际产物与 Mosaic fork。
- 应核查 Mosaic Native TTY 的信号处理与回调交接边界。
  候选方向是信号处理仅做异步信号安全通知，由普通执行上下文处理
  resize、状态更新和调度；尚未确认具体 API 或实现。

## 救援讨论边界

- **重启后续接**：Session 850 已落盘至最后工具结果；
  当前请求的未落盘内容不能保证恢复。尚未执行停止或重启。
- **原地救援**：讨论过用 GDB 跳出本次 resize 重入路径的可能性，
  但未验证其安全性，不能记录为可直接执行的恢复步骤。
- 不直接调用 `pthread_mutex_unlock`：被打断的原路径仍认为自己持锁，
  人工解锁可能引入并发修改或二次解锁。
- 强制返回/跳转前必须核验回调已取得的其他锁、协程与 channel 状态、
  清理责任及恢复后的一致性；保存栈不等于已有安全救援方案。
- 不因租约已过期就在其他进程同时打开这些 Session；
  恢复旧进程前需重新确认所有权，避免双写。

## 后续验证议题

- 使用隔离 Home、假模型/请求和独立 PTY，避免对真实会话或共享 tmux
  反复注入 SIGWINCH。
- 同时覆盖协程投递、Compose frame、后台请求完成与重复 resize；
  核验 UI、后台调度及租约维护继续推进。
- 核查信号处理路径不会直接进入 Kotlin 状态更新、锁或协程调度；
  验证尺寸通知合并、唤醒、关闭竞态和资源释放。
- 决定是否需要 Mosaic fork 修复，再进入 planning；
  当前 Daemon/协程生命周期讨论不自动包含或授权本项实现。
