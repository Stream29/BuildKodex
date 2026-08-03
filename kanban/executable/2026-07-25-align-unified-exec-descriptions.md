# Task Tree

- 对齐`exec_command`与`write_stdin`描述
  - [done] 对比Rust的平台和功能开关
  - [done] [持久化session工作目录](../done/2026-07-25-persist-session-working-directory.md)
  - [done] 与用户确认动态spec的接口形状
  - [done] 固定使用login shell并移除模型参数
  - 校验pipe与PTY两条执行链路
    - [done] 校验JVM与Linux Native
    - [done] 校验Linux与macOS Node.js
    - [done] 校验macOS Native
    - [done] 校验Windows Native编译链路
    - 校验Windows Native
  - [done] 对齐平台相关描述
  - [done] 增加精确描述测试
  - [done] 运行可用宿主的相关模块测试

# Details

- Rust的`exec_command`描述依赖宿主平台。
- Kotlin固定暴露`shell`和`tty`，不暴露environment与permission参数。
- Rust生产配置默认允许login shell，省略`login`时默认使用login shell；测试用默认handler会禁用该能力。
- Kotlin固定使用login shell，不再向模型暴露`login`参数。
- Node.js使用带全平台预编译产物的`node-pty 1.2.0-beta.12`，不启用npm lifecycle scripts。
- Rust与Kotlin均声明`workdir`默认使用turn/session cwd。
- Rust在Windows上为`exec_command`追加专门的安全提示，Kotlin当前spec是全平台静态值。
- Kotlin的`shell`字段描述会动态写入宿主平台与默认shell路径；Rust使用固定的通用描述。
- 保留`tty`模型参数，并验证pipe与真实PTY实现。
- Windows VM当前关闭；Windows Native实机测试尚未运行。
