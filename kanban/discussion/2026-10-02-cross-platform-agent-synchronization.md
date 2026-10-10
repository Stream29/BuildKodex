# Task Tree

- **`Clarify agent synchronization scope when work resumes`()**
- `Review existing runtime and cross-platform resource ownership abstractions`()
- `Compare file locks and named result completion signals`()
- `Define lifecycle, failure handling, and validation requirements`()
- `Confirm the design before promoting to planning`()

# Details

- 状态：未来有时间再推进；当前仅记录到讨论看板，不启动设计或实现。
- 用户需求：agent 有时需要等待另一个 agent 的结果，希望有跨平台、类似 `flock` 的同步能力。
- 候选方向（尚未确认）：
  - 跨平台互斥：Linux / macOS 使用 `flock()`，Windows 使用 `LockFileEx()`，对上层提供统一接口。
  - 结果等待：使用具名完成信号或 Future，以唯一任务 ticket 表达成功、失败和结果位置；锁仅用于保护共享状态。
- 设计时需要区分：
  - 互斥访问共享资源与等待任务完成不是同一种语义。
  - 锁释放不代表结果成功：生产者可能尚未启动，也可能异常退出。
  - 跨平台本机同步与跨设备协调是不同范围，后者需另行确定协调方式。
- 待确认：
  - 首期提供文件锁、结果等待，还是两者都提供？
  - 能力通过 agent 工具、CLI、运行时 API 中的哪些入口暴露？
  - 谁持有锁和等待状态，作用域是进程、Session、任务还是服务？
  - 超时、取消、生产者退出、服务重启和结果保留如何处理？
- 后续验证要求：
  - Linux、macOS、Windows 上的并发互斥、异常退出释放及资源清理。
  - 生产者与等待者的两种启动顺序、多等待者、失败传播和 ticket 隔离。
  - 超时与取消能够终止等待，且不意外取消生产者。
- 技术参考：
  - [Linux flock(2)](https://man7.org/linux/man-pages/man2/flock.2.html)
  - [Apple flock(2)](https://developer.apple.com/library/archive/documentation/System/Conceptual/ManPages_iPhoneOS/man2/flock.2.html)
  - [Windows LockFileEx](https://learn.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-lockfileex)
