# Task Tree

- 支持将Kodex Session导出到Codex
  - 规划目标Codex版本和rollout格式兼容边界
  - 定义Session拓扑、事件和元数据的导出映射
  - 定义目标冲突、原子写入、失败恢复和幂等语义
  - 实现显式Session导出流程
  - 验证导出结果可被Codex发现和恢复

# Details

- 状态：未来备忘，当前不启动。
- 当前Session兼容范围对Codex数据保持只读；本任务不阻塞当前只读方案。
- 实现前需用户明确启动规划并审核写入边界。
