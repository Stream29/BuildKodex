# Task Tree

- [done] 更新产品管理的 kodex-home skill
  - [done] 采用用户确认的精简内容，目标版本为 0.4.3
  - [done] 注册 future migration，保留历史 migration
    - [done] 将批准文本内嵌到独立的 v0_4_3 源码
    - [done] 原子替换 skill，仅清理本版本 temporary
  - [done] 验证安装、保留数据、中断重跑和跨版本升级
    - [done] 使用隔离 Home 覆盖 siblings、未知文件及原有 Session
    - [done] 注入原子替换前后中断，验证版本保持与重跑
    - [done] 验证 0.4.2 不激活、直接升级与跨历史版本升级
  - [done] 清理候选稿并记录验证结果
    - [done] 核对内嵌文本与批准稿逐字节相同，删除候选稿
    - [done] 记录 MacBook 测试结果并清理本地、远端临时检出和报告

# Details

- 用户批准稿已原样内嵌至 `Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/v0_4_3/KodexHomeSkill.kt`。
- 当前应用版本为 0.4.2；0.4.3 migration 由后续独立版本 bump 激活。
- 仅替换 `skills/kodex-home/SKILL.md`，保留同目录其他文件及其他 Home 数据。
- 不修改已发布 migration，不操作真实 Home，不创建普通产品提交。
- 用户单独批准：仅将 `v0_3_3/MigrateToV0_3_3Test.kt` 的全注册表断言限制到历史前缀；保留其余测试与 fixture 原文。
- [验证记录](../../shared-context/findings/kodex-home-skill-0.4.3-validation.md)：MacBook Java 25，44 项 JVM 和 42 项 macOS ARM64 测试通过；新增 14 个迁移场景在两端均通过。
- 未执行 CLI 启动或发布包验证；未 bump 应用版本、创建提交、推送或发布。
