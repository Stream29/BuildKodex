# Task Tree

- [done] 移除 Home Skill `.system` 特例
  - [done] 从 Home roots 删除 `skills/.system` 发现
  - [done] 保留四层与项目 Skill 路径
  - [done] 更新测试和持久化约定
  - [done] 运行定向测试与 diff 校验

# Details

- 用户明确只移除 `.system` 特例。
- 状态：`done`。两个 Home 只发现直属 `skills/`，不再单独发现 `skills/.system/`。
- 路线已确认：只删除 Home root 列表中的 System source，并令现有四层测试断言 `.system` skill 不被发现。
- 最小文件集：Skill filesystem resolver、其测试、相关 durable checklist 与此前的 completed task record。
- 不删除 `SkillScope.System` contract，也不调整其现有排序。
- 不改变四层顺序、Git/cwd 的 `skills/` 与 `.agents/skills/`，或其他 catalog 行为。
- 本任务修正已完成的
  [`align-agent-context-discovery-layers`](../done/2026-08-21-align-agent-context-discovery-layers.md)
  中超出用户要求的 `.system` 保留/扩展。
- `:agent-context-skill-filesystem:jvmTest`、`:agent-context-prefix-impl:jvmTest` 和
  `:agent-context-skill-filesystem:linuxX64Test` 均通过。
- 本任务范围的 `git diff --check`、尾随空白和已删除 root 检查通过。
