# Task Tree

- [done] 重新设计skills上下文来源
  - [done] 盘点当前skill roots、metadata discovery、正文和resource读取路径
  - [done] 调研Rust的host、executor和orchestrator skills生命周期
  - [done] 区分available skills catalog、显式选择的SKILL.md正文与resources
  - [done] 定义cwd、environment、roots和source authority的快照语义
  - [done] 定义用户或agent增删改skill时的缓存失效与生效边界
  - [done] 定义已选SKILL.md正文在tool continuation、compaction和resume中的冻结语义
  - [done] 提出独立SkillsService与SkillSelectionRuntime方案
  - [done] 调整catalog generation、authority和资源读取实现
  - [done] 实现显式skill选择与原子持久化
  - [done] 补齐跨平台真实文件测试和runtime测试

# Details

- available skills catalog是动态前缀metadata，不等于用户已选skill的完整正文。
- 已选SKILL.md正文属于turn输入，读取后需要持久化，不应在tool continuation中因文件变化而替换。
- catalog entry必须保留读取该skill的source authority，不能假定所有path都属于同一本地filesystem。
- `AgentContextPrefixProvider`不再同时承担`SkillResourceProvider`职责。
- 每个不可变skills generation同时保留catalog、warnings和skill到filesystem authority的映射；刷新只替换后续轮次使用的generation。
- catalog按canonical skill path而不是name去重，同名不同路径条目保留，并按Repo、User、System、Admin及name、path排序。
- 新用户轮次开始前刷新catalog。显式选择在该轮append user message时解析，读取到的SKILL.md正文与用户消息原子持久化，后续continuation和compaction只读取history。
- catalog与runtime的真实文件测试已通过JVM、Node.js和Linux Native。
