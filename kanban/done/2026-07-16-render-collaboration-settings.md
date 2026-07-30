# Task Tree

- [done] 将 Plan mode 投影到正常模型请求
  - [done] 对齐 Rust 协作模式与当前 Kotlin settings 的边界
  - [done] 移除旧 collaboration contract 和 DeveloperInstructions 路径
  - [done] 为 ModeKind.Plan 添加 Rust 对齐的提示词渲染
  - [done] 在 KodexAgentStateImpl 中注入 Plan mode 上下文
  - [done] 更新测试、模块依赖和设计记录
  - [done] 验证受影响模块

# Details

`UpdatePlanArgs` 和 `ThreadGoal` 没有模型提示词投影，继续只作为 settings 状态。内置 Plan mode 使用 Rust 的固定 developer instructions；Default mode 默认不注入 collaboration block。投影只进入普通 Responses 请求，不写入 history，也不参与 remote compaction。

验证已通过 JVM、Linux X64 和 JS Node。远端 MacBook 工作树尚未同步本次新增模块，未在陈旧源码上运行测试。
