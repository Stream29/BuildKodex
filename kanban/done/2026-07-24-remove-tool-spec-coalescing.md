# Task Tree

- [done] 移除请求工具列表的静默合并
  - [done] 确认合并逻辑及调用边界
  - [done] 删除 `coalesceToolSpecs` 及其键函数
  - [done] 验证 Agent State 工具投影测试

# Details

请求工具列表由已知的唯一工具声明构成；同名声明不应在该边界静默覆盖。
