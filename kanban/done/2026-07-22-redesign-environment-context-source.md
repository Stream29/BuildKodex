# Task Tree

- [done] 重新设计environment上下文来源
  - [done] 盘点当前EnvironmentContext构造和cwd所有权
  - [done] 调研Rust的session、turn、step快照与tool局部workdir行为
  - [done] 定义environment identity、cwd、filesystem authority与generation
  - [done] 定义显式environment更新与tool局部`cd`的边界
  - [done] 确定请求级environment generation
  - [done] 调整实现与测试

# Details

- 用户已授权按任务顺序实施。
- 模型可见的environment内容本身较简单，但其快照是AGENTS.md、skills和dynamic tools选择cwd与filesystem的作用域根。
- 单次tool调用的workdir或shell内部`cd`不回写已选environment。
- 本任务只定义environment真源和快照语义，不承担其他上下文的发现逻辑。
- `AgentEnvironmentGeneration`原子绑定模型可见context和按environment id索引的filesystem authority。
- generation只在宿主显式选择不同environment时递增；日期和时区在读取当前generation时重新采样，但不改变environment identity。
- `AgentEnvironmentGeneration`同时携带模型可见context和按environment id索引的filesystem authority，AGENTS.md、skills与工具在一次请求中可共享同一代环境选择。
- 已通过JVM、Node.js和Linux Native真实文件系统测试。
