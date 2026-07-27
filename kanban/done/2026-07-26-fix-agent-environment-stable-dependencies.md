# Task Tree

- [done] 固定Agent environment的稳定宿主依赖
  - [done] 从构造函数移除clock、timeZone与fileSystem
  - [done] 从环境选择状态移除fileSystem
  - [done] 清理上层provider中的无效转传
  - [done] 更新测试
  - [done] 验证受影响模块

# Details

- 默认实现固定使用系统时钟、系统时区与系统协程文件系统；只有工作目录和shell属于可变环境选择。
- environment impl、prefix filesystem与runtime skill的`allTests`通过。
- IntelliJ IDEA检查与构建通过。
