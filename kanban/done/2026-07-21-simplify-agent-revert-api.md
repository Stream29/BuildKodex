# Task Tree

- [done] 简化Agent回退API
  - [done] 将`MutableKodexAgentStorage.revert`从接口协议改为extension
  - [done] 将`KodexAgentState.checkout`改名为`revert`
  - [done] 更新实现、调用方和测试

# Details

Storage只有底层索引需要回退原语，组合回退由extension表达。AgentState保留面向调用方的回退协议，但统一使用`revert`命名。
