# Task Tree

- [done] 为会话名称建立持久化设置与生成生命周期
  - [done] 调查Rust Codex的线程名称生成和更新时机
  - [done] 将会话名称加入`KodexAgentSettings`
  - [done] 按确认后的对齐策略接入生成、更新和TUI展示
  - [done] 覆盖存储与会话生命周期测试

# Details

Rust本地实现以首条文本用户消息初始化标题，不额外请求模型生成标题；显式设置覆盖，fork保留设置快照。详细依据见`shared-context/findings/codex-thread-title.md`。

验证通过：`:agent-state-impl:jvmTest`、`:tui-demo:jvmTest`、`:agent-state-impl:linuxX64Test`、`:tui-demo:linuxX64Test`、`:tui-demo:linkDebugExecutableLinuxX64`。
