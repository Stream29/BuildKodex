# Task Tree

- [done] 将session工作目录纳入`KodexAgentSettings`
  - [done] 确定cwd的数据类型与旧数据读取语义
  - [done] 在settings模型与存储测试中加入cwd
  - [done] 让新session以当前工作目录初始化cwd
  - [done] 让context、hooks和skills读取session cwd
  - [done] 让文件与进程工具读取session cwd
  - [done] 验证fork、checkout和冷恢复保留cwd
  - [done] 运行相关跨平台测试

# Details

- cwd是session级持久化状态，不再由CLI应用全局代持。
- cwd使用`Path`建模并以字符串序列化；缺失字段回退到`Path(".")`以读取旧数据。
- 新session必须写入解析后的绝对cwd。
- cwd统一约束命令、补丁、图片、AGENTS.md、skills和hooks的项目范围。
- `AgentEnvironmentSource`负责把持久化cwd解析为运行时环境。
- 工具参数中的`workdir`只覆盖单次调用，默认使用session cwd。
