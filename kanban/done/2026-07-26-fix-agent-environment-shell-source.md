# Task Tree

- [done] 重构Agent environment上下文
  - [done] 将shell加入全局设置及持久化投影
  - [done] 删除长期存活的environment source与generation
  - [done] 将filesystem prefix provider重构为全局共享的AgentContextPrefixProviderImpl
  - [done] 让provider消费StateFlow<Shell>并接合全局与agent信息
  - [done] 分离cwd、shell与日期时区的渲染
  - [done] 清理应用层与模块依赖
  - [done] 更新测试
  - [done] 验证受影响模块

# Details

- `cwd`来自当前session settings，`shell`来自可热更新的全局设置。
- 日期与时区在每次渲染时直接从宿主读取。
- 环境提示对齐Rust侧，注入shell逻辑名称而非可执行文件路径。
- JVM、Node、Linux与macOS实机相关测试通过，IDE构建通过。
- Windows VM未运行，未执行Windows实机测试。
- Mosaic原生任务仍无法保存configuration cache，但不影响本轮构建成功。
- CLI JVM测试另有一个既有session repository故障注入断言失败，与本任务无关。
