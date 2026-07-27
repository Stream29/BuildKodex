# Task Tree

- [done] 移除Host Hook warning数据通道
  - [done] 删除解码模型与catalog中的warnings
  - [done] 简化非法Hook的跳过逻辑
  - [done] 更新测试并运行相关跨平台验证

# Details

- 当前不保留字符串warning；未来通过KMP logging在问题发生处记录。
- 非法或不支持的Hook继续被跳过。
