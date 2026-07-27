# Task Tree

- [done] 将demo转为正式CLI入口
  - [done] 移除模块和界面中的demo语义
  - [done] 明确应用启动、依赖装配和资源所有权
  - [done] 更新运行入口及入口级测试

# Details

`cli/app`现使用正式的应用、会话、状态和界面命名。`CodexLiteApplication`负责依赖装配，并按session、model catalog、client、Koin容器的顺序释放资源；初始化中途失败也会释放已创建资源。

Linux X64与macOS ARM64上的真实会话测试和调试可执行文件链接均已通过，IDE检查未发现错误。JVM测试仍受Mosaic既有的JDK22 jextract绑定生成问题影响。
