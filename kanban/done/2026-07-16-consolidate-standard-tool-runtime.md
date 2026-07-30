# Task Tree

- [done] 收敛普通工具 runtime
  - [done] 建立 `agent-runtime:tool` 的 `KodexToolRuntime`
  - [done] 由 `ToolSpec` 推导工具路由并拒绝歧义配置
  - [done] 保留 `PlanRuntime` 的 plan 原子写入路径
- [done] 移除重复 runtime 模块
  - [done] 删除 apply-patch、view-image、image-generation runtime 模块
  - [done] 更新依赖与组合调用点
- [done] 验证通用工具 runtime
  - [done] 覆盖 plain、custom 和 namespace 工具调用
  - [done] 运行 JVM、JS Node、Linux Native 测试

# Details

`KodexToolRuntime`持有delegate和本地Tool列表，只完成能由这些ToolSpec路由到的pending调用。`update_plan`仍需要与plan timeline同一事务写入，因此不并入通用路径。

验证通过：JVM、JS Node、Linux Native 实测；macOS Arm64 交叉编译。
