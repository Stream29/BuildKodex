# Task Tree

- [done] 修复协程文件资源生命周期
  - [done] 复核泄漏与资源所有权
    - [done] 确认EOF文件描述符持续未关闭
    - [done] 确认取消阻止底层close执行
    - [done] 确认资源取得阶段存在丢失窗口
  - [done] 固化通用生命周期规则
    - [done] 新增canonical coroutine resource checklist
    - [done] 从Native blocking IO checklist链接规则
  - [done] 收敛文件资源API
    - [done] 用scoped source/sink操作替换裸handle返回
    - [done] 保留raw IO类型供非文件系统边界使用
    - [done] 迁移全部文件系统调用方
  - [done] 实现跨平台取消安全清理
    - [done] 保证blocking平台在IO dispatcher内完成完整资源边界
    - [done] 保证Node文件handle取得后立即进入受管边界
    - [done] 让可挂起close在取消后仍实际执行
    - [done] 保留原始失败与suppressed cleanup failure
  - [done] 增加资源生命周期回归测试
    - [done] 覆盖取得资源时取消
    - [done] 覆盖读取和写入期间取消
    - [done] 覆盖正常结束与清理失败
    - [done] 验证owner完成时真实handle已释放
  - [done] 验证历史侧栏泄漏修复
    - [done] 以确定性取消测试覆盖快速滚动触发的行加载取消路径
    - [done] 重复检查目标JSON文件描述符
    - [done] 确认启动读取完成后描述符归零
  - [done] 运行格式化、静态检查与跨平台测试

# Details

- 当前状态：`done`。Kodex Home migration、资源生命周期修复和跨平台验证均已完成。
- `checklist/coroutine-resource-lifecycle.md`是本任务的canonical资源所有权规则。
- migration最终没有扩展`CoroutineFileSystem`；开始本任务时Kodex工作树干净。

## 实施结果

- 2026-09-02发现实现文件被还原；经用户确认后已重新落地并复验。
- `CoroutineFileSystem`仅提供scoped `useSource/useSink`，仓库内调用方已经迁移。
- blocking与MinGW在同一IO dispatcher边界内完成open、operation和close。
- Node等待不可取消的`fs.open`交接完成，使新handle先进入`use/finally`，再恢复调用方取消。
- `CoroutineCloseable.use`在有限的`NonCancellable`收尾中执行close，并保留suppressed failure语义。
- 新增JVM、Linux Native和Node真实文件描述符测试，以及通用取消和清理失败测试。

## 验证结果

- 2026-09-02重新落地后，核心跨平台测试、三个受影响模块的`check`和Linux CLI编译再次通过。
- `utils-kotlinx-io-coroutines:check`通过；JVM、Node和Linux Native测试通过。
- Linux ARM64、macOS ARM64和MinGW目标编译通过。
- MinGW测试程序已在Windows VM运行通过，包括真实文件handle取消清理。
- 三个受影响filesystem模块的`check`通过。
- 真实Linux CLI启动时短暂打开10个`sessions/*/index/*.json`描述符，读取完成后连续检查均为0。
- 2026-09-02 migration后的UI一度显示0个session，无法执行History快速滚动；最终以确定性取消测试、真实handle测试及两个长时间运行的0.3.3 Linux实例中session JSON描述符持续为0完成收口。

## 阻塞审查

- **活动任务冲突**：`kanban/done/2026-09-01-version-kodex-home-for-startup-migrations.md`
  已完成并移动到`kanban/done/`；其最终实现没有修改通用filesystem API，活动任务冲突已经解除。
- **API决策已封闭**：移除filesystem裸handle API，只保留scoped`useSource/useSink`；迁移仓库内全部调用方。
- **Node交接方案已落地**：在`withContext(NonCancellable)`中等待不可取消的open完成，
  建立`try/finally`后恢复调用方取消检查，并在`NonCancellable`中等待close完成。
- **取消测试已确定化**：Node使用`CoroutineStart.UNDISPATCHED`固定取消在Promise交接阶段；
  blocking平台在block、EOF和close闸门处取消，不使用概率性循环。
- **清理失败测试受测试替身规则约束**：真实文件handle无法稳定制造close failure；
  已获准使用最小`CoroutineCloseable`测试实现，不模拟filesystem。
- **平台验证结果**：Node与Linux Native已在当前主机运行；MinGW已在Windows VM运行；
  macOS完成目标编译，共享blocking实现的运行时资源释放由JVM与Linux Native覆盖。
- **close-on-exec验收边界**：本任务排除POSIX descriptor继承，因此父进程泄漏与子进程继承必须分开验收；
  若要求所有后代进程均不持有目标descriptor，必须纳入本任务或建立独立后续任务。

## 已确认问题

- Linux Native进程持续持有34个已经读到EOF的
  `~/.kodex/sessions/132/index/*.json`描述符。
- 另一个进程除EOF描述符外还出现持续存在的offset 0描述符，符合资源打开完成但取消交接丢失handle的路径。
- `CoroutineRawIo.use`会进入`finally`，但其中的挂起`close()`继承已取消Job。
- blocking与MinGW实现的`close()`再调用可取消的`withContext(IoDispatcher)`，底层close可能完全不执行。
- blocking、MinGW和Node实现都可能在资源取得与返回调用方之间失去新handle。
- History Index row的`LaunchedEffect`取消属于正常frontend生命周期；错误位于filesystem资源边界，不通过延长或隐藏frontend composition规避。

## 所有权决策

- 单次文件读取或写入的owner是当前operation coroutine，不是Session、Agent或Application scope。
- operation正常返回、失败或取消前必须完成source/sink清理；owner Job完成即证明其临时handle已经释放。
- Session关闭只通过结构化取消触发并等待operation清理，不直接维护全局文件handle registry。
- `NonCancellable`只覆盖有限的close、release和失败补偿，不覆盖正常读取、写入或长期后台工作。

## API路线

- 在`CoroutineFileSystem`提供scoped文件资源操作，例如：

```kotlin
public suspend fun <R> useSource(
    path: Path,
    block: suspend (CoroutineRawSource) -> R,
): R

public suspend fun <R> useSink(
    path: Path,
    append: Boolean = false,
    mustCreate: Boolean = false,
    block: suspend (CoroutineRawSink) -> R,
): R
```

- 移除或收窄直接返回`CoroutineRawSource`和`CoroutineRawSink`的filesystem API，避免调用方在未建立cleanup前取得或丢失handle。
- `readBytes`、`readString`、`writeBytes`和`writeString`统一建立在scoped API上。
- `FileSystemIndexVersioned.copyRangeRawTo`在嵌套source/sink scoped边界内完成copy和flush。
- `FileSystemAgentsMd`及测试中的直接source/sink调用迁移到同一边界。
- `CoroutineRawSource`、`CoroutineRawSink`和通用`use`继续保留，供MCP stdio等非filesystem资源使用。

## 平台实现

- JVM、Linux与macOS blocking实现一次进入`IoDispatcher`后完成open、block、flush和close，避免从dispatcher返回新打开的资源。
- MinGW在同一scoped操作中持有并关闭Win32 handle。
- Node在可取消的Promise交接前建立handle所有权；取消不能丢弃已经由`fs.open`创建的`FileHandle`。
- 通用`CoroutineCloseable.use`在`NonCancellable`中执行close，并维持现有原始异常与suppressed close异常语义。
- close必须幂等，且不能先标记closed再跳过实际平台释放。
- 本任务不以延长资源生命周期、依赖GC/finalizer或等待进程退出作为释放手段。

## 测试与验证

- 在`utils-kotlinx-io-coroutines`增加真实文件系统取消测试，不只验证方法返回值。
- Linux测试通过`/proc/self/fd`按临时文件路径确认source已释放，不使用总FD数量作为唯一断言。
- JVM与Linux Native都覆盖取消发生在open交接、read返回、EOF和close收尾的路径。
- 写入测试覆盖普通sink与`mustCreate` exclusive sink，并确认取消后文件handle可重新打开或删除。
- 通用close测试验证取消后底层release仍执行，cleanup failure不覆盖原始failure。
- Node测试验证取消后`FileHandle`关闭；MinGW验证handle关闭并至少完成编译检查。
- 运行受影响模块的JVM与Linux Native测试，并编译Node、MinGW和macOS目标。
- 启动真实Linux CLI，快速滚动History Index后多次检查对应进程的
  `sessions/*/index/*.json`描述符，确认不再累积。

## 范围边界

- 本任务修复coroutine ownership和取消清理。
- POSIX文件描述符的close-on-exec属于独立的进程继承加固；本任务不依赖它掩盖父进程泄漏，也不为此替换整个kotlinx-io文件实现。
