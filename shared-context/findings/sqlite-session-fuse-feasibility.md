# SQLite Session FUSE可行性

## 原型范围

- 原型位于`Kodex/experiments/sqlite-session-fuse/`。
- SQLite用一个递归`agents`表和六个timeline表保存Session规范数据。
- `fuse_nodes` SQL view将表投影成当前`~/.kodex/sessions`路径结构。
- `latest.json`由每张timeline表的`MAX(state_index)`生成。
- `archive.mark`由根Agent元数据生成。
- `lock.json`和`.kodex-*`临时文件属于运行时协议，不进入规范数据。
- FUSE只读，并可挂载任何满足节点列契约的SQL view。

## 已验证结果

- 真实内核FUSE挂载成功，无需修改Kodex实现。
- 挂载期间提交SQL `INSERT`后，新记录文件和新的`latest.json`可见。
- 挂载期间提交SQL `UPDATE`后，已读取路径返回新内容；挂载禁用属性、目录项和数据缓存。
- 从一个现有Kodex Session导入88个timeline记录后，挂载结果与原始JSON逐字节一致。
- 嵌套Subagent、空timeline的`latest.json = -1`和归档标记均能正确投影。
- SQLite事务可以原子修改多条timeline，语义强于当前文件后端的操作级补偿。

## 可行性边界

- 数据库到只读文件树的映射可行，代码量和语义复杂度都可控。
- 把当前Kodex未经修改地放在可写FUSE之上并不简单。适配层必须复现临时文件、原子rename/replace、revert暂存、租约、归档、删除和缓存失效协议。
- 当前通用flat view优先证明SQL可配置性。SQLite对单路径查询仍会扫描compound union，不适合作为大型Session库的性能实现。
- SQLite 3.46.1下的10万条synthetic `stable`记录热查询中，flat view精确路径查询中位数约69.7 ms，主键直查约0.002 ms；该结果用于确认查询形态问题，不作为跨机器性能指标。
- 可扩展实现需要索引化的物化路径目录，或按路径解析后执行有索引的参数化SQL；不能假设任意flat SQL view都会自动获得文件系统级查找性能。
- 原型使用当前机器可直接运行的fusepy与libfuse 2。生产实现应独立选择和验证维护中的FUSE 3绑定。

## 方向判断

- SQLite作为`KodexAgentStorage`和`KodexSessionRepository`的直接规范后端是可行且值得继续验证的方向。
- FUSE更适合作为只读兼容、检查、导出或调试视图，而不是Kodex访问SQLite的主写入路径。
- SQL view适合配置可见路径、过滤timeline和生成派生文件；高频主路径仍应使用有索引的专用查询。
