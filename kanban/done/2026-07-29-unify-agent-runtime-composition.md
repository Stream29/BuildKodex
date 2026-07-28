# Task Tree

- [done] 统一Agent runtime composition
  - [done] 合并master与subagent runtime工厂
  - [done] 移除两种session后端的runtime类型分支
  - [done] 运行受影响构建与测试

# Details

- master与subagent保留独立的State、Storage、工具实例和协程生命周期。
- 根lease与根AgentPathResolver仍由Session层管理。
