# Task Tree

- [done] 实现Agent历史checkout语义
  - [done] 为`MutableKodexAgentStorage`定义跨全部timeline的原子后缀回退原语
  - [done] 在内存存储中实现该原语并保留索引`0`的初始化快照
  - [done] 在`KodexAgentState`中暴露语义化的checkout操作
  - [done] checkout后从storage尾部重新推导并发布`latestIndex`与状态值
  - [done] 限制checkout目标为已完成turn的快照，并拒绝进行中的请求、压缩或工具批次
  - [done] 覆盖history、settings、compaction、timestamp和tokenCount共同回退的测试

# Details

现有单条`MutableIndexVersioned.revert(untilExclusive)`是底层能力，但现有
`transaction`只支持追加失败后的回退，不能包装删除既有后缀的checkout。新的
storage级原语必须保证五条timeline全成或全不成，不能让取消或失败留下半截状态。

对展示中最后保留的快照索引`i`，checkout边界为`i + 1`。这与fork使用的
exclusive boundary保持一致。TUI不能直接修改storage，否则`KodexAgentState`
缓存的`StateFlow`会失效。
