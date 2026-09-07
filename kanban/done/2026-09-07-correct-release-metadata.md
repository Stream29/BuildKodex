# Task Tree

- [done] 修正 Release 文案并补充发版技能
  - [done] 核对最新说明与全部 Release 标题
  - [done] 将 v0.4.3 说明改为英文
  - [done] 将四个异常标题改为对应 tag
  - [done] 补充技能并验证远端元数据

# Details

- 用户要求最新 Release 说明使用英文，历史 Release 标题直接使用 `v<版本号>`，并将要求写入发版技能。
- 已核对 24 个 Release；v0.4.3 的 8 条说明为中文，标题异常仅有 v0.3.4、v0.4.0、v0.4.1、v0.4.2。
- 仅翻译 v0.4.3 现有摘要并保留 Full Changelog，其他版本正文不变；只修正四个异常 name，不改 tag、提交、附件或 latest 归属。
- 按发版技能复用 MacBook 执行 GitHub 元数据修正，不构建、不发布新版本、不创建 Git 提交。
- 路线已确定：按 Release ID 做限定字段更新，前后对比全部 Release、附件及远端 tag，保留 latest 为 v0.4.3。
- 计划完整，进入执行；技能补充英文说明、显式 `--title vX.Y.Z` 及发布前后标题/语言检查。
- 已在 MacBook 通过限定字段 PATCH 完成五个 Release 修正；回读验证全部 24 个标题等于 tag，latest 仍为 v0.4.3，8 条英文摘要及 Full Changelog 完整。
- 前后对比确认其他版本正文、所有分支/tag SHA、发布状态及 120 个附件的 ID、名称、大小、digest、创建/更新时间等元数据均未变。
- 已更新 `.agents/skills/release-kodex/SKILL.md:55`：说明统一英文，标题精确匹配 `vX.Y.Z`，创建时显式传 `--title`，发布前检查并在发布后回读核验。
- GitHub 公开页面复核 v0.4.3 的英文摘要、Full Changelog 与 Latest 标识，以及 v0.4.2 的规范标题；API 已覆盖全部 24 个 Release。`git diff --check` 通过。
- 本次仅改发布元数据与技能文档，没有代码或构建产物变化，未运行编译/运行测试；不创建提交。
