# Task Tree

- [done] 建立全局设置模块
  - [done] 定义 Codex home 与换行键的不可空设置快照
  - [done] 提供纯内存、StateFlow 驱动的原子更新实现
  - [done] 注册 host 模块并覆盖基础状态更新测试

# Details

本轮不接入 TUI 输入框；换行键设置先作为后续 TextInput 的全局配置入口。

已验证：`global-settings` 的 JVM、Linux X64 与 Node.js 测试。
