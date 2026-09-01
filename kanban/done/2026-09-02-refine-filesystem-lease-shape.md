# Task Tree

- [done] 收敛 filesystem lease 形状
  - [done] 领取 Draft 评语
  - [done] 确认公共 contract 与实现布局
  - [done] 重写三个 lease 工厂与实现
    - [done] contract 仅组合资源与作用域
    - [done] 普通独占 lease 独立实现
    - [done] 共享读 lease 独立实现
    - [done] 独占写 lease 独立实现
    - [done] heartbeat 支持逻辑独立实现
  - [done] 迁移调用方与生命周期测试
  - [done] 完成跨平台验证

# Details

- `FileSystemLease` 只组合 `AutoCloseable` 与 `CoroutineScope`，不声明额外成员。
- 工厂函数成功返回即表示对应 lease 已获取。
- 普通独占、共享读、独占写三种 lease 分别放在三个实现文件。
- 三种实现共用的 heartbeat 与文件操作逻辑放在独立共享文件。
- 三种 lease 均由绑定 owner `CoroutineScope` 的工厂函数创建。
- 工厂命名遵循项目现有构造函数式工厂模式：`CoroutineScope.FileSystemLease(...)`、
  `CoroutineScope.FileSystemReadLease(...)`、`CoroutineScope.FileSystemWriteLease(...)`。
- `close()` 取消 lease 自身作用域；需要等待释放的调用方通过 lease 的 `Job` 等待完成。
- 保留 `<pid>.read.lock` 的进程内共享；每次读 lease 工厂调用返回独立子作用域，
  关闭最后一个 handle 时才释放共享 heartbeat。
- 验证 contract、impl、Session repository、Home migration 的 JVM tests，并编译 Linux targets。
- `utils-filesystem-lease-impl:allTests` 通过；JVM、JS 和 Linux x64 tests 均执行，
  macOS arm64 与 Windows x64 tests 在 Linux host 上完成编译。
- Session repository 的两个 lease lifecycle/互斥集成测试通过。
- 完整 Session repository suite 另有存储语义断言失败：
  `creates an uninitialized root storage` 预期 latest index `1`，实际为 `0`；
  与 lease acquisition 和释放路径分开记录，未在本任务中改动该断言。
