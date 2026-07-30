# Task Tree

- [done] 固定绕过Host Hook逐条信任校验
  - [done] 确认信任状态与Hook启用、匹配、执行链的边界
  - [done] 将Codex Hook文件解码与matcher编译收敛到`openai:codex-cli-storage`
  - [done] 让`cli:settings`只组合已解码配置层
  - [done] 让Host Hook catalog只接收类型化配置
  - [done] 删除哈希计算、信任判定与可配置的bypass开关
  - 保留`hooks.state.<key>.enabled`和managed Hook语义
  - [done] 更新配置与执行测试
  - [done] 更新集成测试
  - [done] 更新Hooks技术决策
  - [done] 运行相关格式化、编译与测试

# Details

- Kodex固定执行已启用且匹配的Host Hook，不实现Codex逐条`trusted_hash`审批。
- Hook来源配置因此被视为宿主已经授权执行的配置。
