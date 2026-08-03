# Task Tree

- [done] 按最新 skill 迁移看板状态工作流
  - [done] 加载最新 skill 与模板
  - [done] 与用户确认全量迁移范围
  - [done] 同步看板维护规则与状态模板
  - [done] 将 live active tasks 分类迁移到新状态目录
  - [done] 更新 AGENTS 加载路径与任务入站链接
  - [done] 校验目录、模板、任务格式与链接完整性

# Details

- 用户确认全量迁移现有看板。
- 新状态顺序为 `discussion/` → `planning/` → `executable/` → `done/`。
- 迁移按任务当前成熟度分类，不改变任务内容或授权状态。
- 两个已被用户从工作树删除、但仍保留 staged add 的旧状态文件不在 live task 集合中，本任务不恢复或改写它们。
- 不创建 Git commit。
