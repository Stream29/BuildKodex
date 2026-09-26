# Task Tree

- `Compare the updated personal skills with repository conventions`()
- `Install workflow skill submodules and update guidance and templates`()
- `Validate references and preserve unrelated work`()

# Details

- 用户要求仓库采用独立的上下文管理 workflow skills，不再通过维护 checklist 承载这些流程。
- 按 `context-template` 接入三个 Git 子模块：
  - `.agents/skills/checklist-workflow/`：`f32f66b`。
  - `.agents/skills/kanban-workflow/`：`dbee6fe`。
  - `.agents/skills/shared-context-workflow/`：`42879d4`。
- Change SOP 迁至仓库本地 `buildkodex-change` skill，保留授权、草稿写作边界和既有目录约定；删除三份重复的 maintenance checklist。
- AGENTS 入口及四阶段模板改用独立 skills 和 Kotlin 脚本伪代码；仅修复一处历史任务的入链，不批量重写旧任务。
- 检查：skill frontmatter、子模块状态、旧路径引用、相对链接和 Git 空白错误。
- 不修改产品代码、领域设计 checklist、用户 Draft 或个人 skills 源仓库；不运行产品构建。
- 验证通过：三个 workflow 子模块已接入，四个本地 skill 文件存在，旧 maintenance checklist 路径无剩余引用，`git diff --check` 通过。
