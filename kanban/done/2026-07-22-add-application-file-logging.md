# Task Tree

- [done] 建立统一应用日志初始化
  - [done] 建立独立的`utils/logging` Kotlin Multiplatform模块
  - [done] 引入`kotlin-logging`与`kermit-io`的显式依赖
  - [done] 实现从`kotlin-logging` Direct `Appender`到`RollingFileLogWriter`的事件适配
  - [done] 提供封装全部全局配置的`initializeLogging()`入口
  - [done] 关闭`kotlin-logging`启动提示并在所有目标上使用`DirectLoggerFactory`
  - [done] 创建`$CODEX_HOME/log`并将日志写入`Kodex.log`
  - [done] 使用`kermit-io`配置按大小滚动与旧文件保留，不自行实现滚动细节
  - [done] 在主应用启动且任何可能产生日志的组件初始化前调用`initializeLogging()`
  - [done] 覆盖日志级别、事件字段映射、滚动配置和初始化顺序测试
  - [done] 验证正常启动与MCP transport日志不会写入stdout
  - [done] 运行相关JVM与Native测试、编译和链接检查

# Details

- 状态：已完成实现与Linux、macOS验证。
- 日志门面继续使用`kotlin-logging`；文件写入与滚动使用`co.touchlab:kermit-io:2.1.0`的`RollingFileLogWriter`。
- 目标日志路径为`$CODEX_HOME/log/Kodex.log`。
- 滚动采用按文件大小和最大保留文件数的策略；不增加时间滚动或压缩。
- `kermit-io` 2.1.0在I/O或轮转失败路径可能调用`println`，这是已知上游限制；本任务不复制或重新实现其滚动算法。
- IntelliJ IDEA 2026.2当前正在打开本项目。
