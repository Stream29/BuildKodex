# Task Tree

- [done] 移除Hook专属观测协议
  - [done] 删除`hook:contract`中的运行事件、状态、输出条目与sink
  - [done] 简化Host Hook命令执行、解析和provider接口
  - [done] 移除仅验证专属观测事件的测试
  - [done] 更新Hooks决策与统一观测规划中的当前基线
  - [done] 运行Hook相关测试与编译验证

# Details

- Hook业务request/result与Stop continuation使用的run id继续保留。
- Hook运行状态未来纳入统一UI与Runtime观测系统，不建立Hook专属公共事件通道。
- JVM、JS/Node与Linux Native Hook测试通过；相关Runtime JVM测试与integration-test JVM编译通过。
- IDEA对本次代码文件的检查与定向构建通过。
- CLI Gradle编译仍被Mosaic既有的JDK 22 JNI绑定缺失阻塞，与本任务修改无关。
