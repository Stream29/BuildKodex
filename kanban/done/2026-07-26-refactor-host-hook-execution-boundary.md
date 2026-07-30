# Task Tree

- [done] 重构Host Hook执行边界
  - [done] 删除HostHookCommandRunner对象
  - [done] 将进程执行改为ShellClient.runHook扩展
  - [done] 用组合模型替代ResolvedHookHandler
  - [done] 删除无消费者的resolved catalog公开面
  - [done] 让KodexHooks承载协程生命周期
    - [done] 使KodexHooks继承CoroutineScope
    - [done] 将Host Hook资源挂载到父协程树
    - [done] 让scope结束统一回收ShellClient
  - [done] 收紧ShellClient所有权
    - [done] 隐藏ShellClient构造器
    - [done] 提供CoroutineScope.ShellClient安全工厂
    - [done] 为Host Hook与unified exec分别创建专属ShellClient
    - [done] 更新ShellClient与unified exec测试
  - [done] 更新决策、测试并运行跨平台验证

# Details

- `runHook`只接受启动Hook进程所需的精确参数。
- resolution阶段直接过滤禁用handler，不携带已失去用途的来源与排序字段。
- `KodexHooks`是持有运行时任务状态的长生命周期对象，必须纳入宿主协程树。
- `ShellClient`必须由父`CoroutineScope`创建；不同功能边界不得共享实例。
