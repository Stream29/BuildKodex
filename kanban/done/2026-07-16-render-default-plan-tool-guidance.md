# Task Tree

- [done] 为 Default mode 注入 `update_plan` 指引
  - [done] 让 `ModeKind.Default.render()` 渲染 Rust 对齐的 Planning 与 `update_plan` developer instructions
  - [done] 将 collaboration mode 渲染改为非空，并简化请求投影
  - [done] 更新 Default mode 请求投影、renderer 与决策记录
  - [done] 验证 JVM、JS Node、Linux X64 测试

# Details

`update_plan` 的使用策略由非 Plan mode 决定，不由工具 runtime 是否存在决定。Plan mode 保留其禁止调用 `update_plan` 的专属指令。

受影响模块已验证 JVM、JS Node 与 Linux X64。Linux Native 集成套件完整通过；JS Node 的两个真实端点探针在聚合运行时遇到外部 `fetch` 瞬断，分别单独重跑后通过。
