# Task Tree

- [done] 实现 `clock.curr_time`
  - [done] 定义 namespace JSON tool 与输出 DTO
  - [done] 注入可测试的时间来源
  - [done] 覆盖跨平台序列化与运行行为

# Details

本任务仅实现当前 UTC 时间查询，不包含需要新输入中断语义的 `clock.sleep`。

JVM、Node JS、Linux x64 已运行 common 测试；Linux ARM64 与 MinGW 测试源集已编译。macOS 实机验证未执行，因为当前 `macbook` 主机名无法解析。
