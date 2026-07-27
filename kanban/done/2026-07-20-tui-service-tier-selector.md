# Task Tree

- [done] 让用户在TUI中选择service tier
  - [done] 核对现有`ServiceTier`模型和Responses API投影
  - [done] 将模型目录的`service_tiers`映射为可选`ServiceTier`
  - [done] 将选择写入会话设置并在请求中生效
  - [done] 在状态栏提供可聚焦的选择入口
  - [done] 覆盖可选值、请求投影和交互测试

# Details

service tier属于会话级请求配置，不能只作为未接入的模型字段存在。

已验证：`openai-models`、`agent-state-impl`和`tui-demo`的JVM/Linux X64测试，以及Linux X64 TUI可执行文件链接。远端macOS工作树未同步当前TUI模块，未使用其陈旧源码验证。
