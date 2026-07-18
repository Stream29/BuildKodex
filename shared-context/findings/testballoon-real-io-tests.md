# TestBalloon 真实 I/O 测试

- 默认 session 使用顺序执行和虚拟时间 `TestScope`。
- `Concurrent` 会并发执行子项，并主动关闭 `TestScope` 以规避线程饥饿。
- 进程、网络和文件系统等真实 I/O 测试应使用 `TestCompartment.RealTime`：顺序执行且关闭 `TestScope`。
- `asParameterForEach` 为每个测试创建独立 fixture，可安全用于并发测试。
- TeamCity 报告器为保持深度优先事件顺序会延迟转发测试起始事件；缺少 `testStarted` 不能证明测试 action 尚未执行。
