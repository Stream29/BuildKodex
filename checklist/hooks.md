# Hooks

- 产品语义以 [RPC notification-only 约定](rpc-settings.md#frontend-notification-only)为准：
  后端发布业务事件，前端本地执行通知命令；不提供控制型 Hook。
- `hook/spec/notification` 承接真实 `NotificationHook` / `NotificationHookType`，
  契约与 serialization API 依赖位于该独立项目的 `specMain`，
  `commonMain` 按 [KMP 源集设计](kmp-spec-impl-source-sets.md) 依赖它。
  保留原 FQCN 与序列化形状；配置值仍由 `CliFrontendSettings.hooks` 持有。
- `hook/impl/notification` 承接唯一 `collectNotificationHooks` 执行路径，
  Application 生命周期建立一个消费者，不随 Settings、tab 或借用 Session view 开关。
- Hook Settings 组件负责编辑与写入队列 admission，不执行命令；
  前端文件实现负责持久化，不另建 Hook 配置仓库或第二套状态。
- Application 的本地 unhandled operation reporter 复用此执行器和既有
  Unhandled error 选择项；输入为 `{"type":"unhandled_error","message":...}`。
  取消不报告，不构造 Session 标识或后端 Agent Stop，详见
  [本地错误输入](rpc-settings.md#前端本地-unhandled-error)。
- Agent/Composer 的前端操作错误也使用该出口，不显示在输入框上方；
  Composer 仍保留失败草稿状态，非错误提示仍可显示。
- 已发布 UnhandledError Stop 的运行失败通过 `AgentRunFailureReported`
  返回失败回执，前端不重复通知；取消、拒绝和传输错误不冒充该类型。
- 每条事件读取一次有序配置快照，匹配命令按序串行运行；
  使用前端默认 Shell、启动 cwd、Notification JSON stdin 和固定 10 秒预算。
- 失败只记前端诊断；取消传播并清理当前进程，不延长关闭以排空事件。
  输出不能阻止工具、注入上下文、恢复执行或作为后端回执。
- Runtime/Session 不再注入 `KodexHooks`、NoOp 或 Turn/Tool/Compaction 控制端口。
  已退役协议不为历史测试保留；正常执行、pending、取消与 compaction 断言继续覆盖。
- 已发布文件迁移的历史 Hook 解析仍被冻结：不自动转换或执行旧命令。
  历史设计见已完成任务，不在当前 SOP 保留不可调用的控制协议。
