# Task Tree

- [done] 简化Host Hook输出处理
  - [done] 收缩命令原始结果模型
  - [done] 删除Parser与Interpreter的重复中间模型
  - [done] 将事件输出直接投影为Hook contract结果
  - [done] 将additionalContextLimit传递给输出落盘逻辑
  - [done] 清理死代码并更新Hooks决策
  - [done] 运行相关跨平台测试与集成测试

# Details

- 保留stdout与stderr分离；结构化输出来自stdout，退出码2的理由来自stderr。
- `null`退出码统一表示启动失败、超时、输出不完整或无法取得退出状态。
- 输出落盘保护独立于Hook wire解析。
