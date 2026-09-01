# Task Tree

- [done] 简化 migration contract
  - [done] 确认三整数 MigrationVersion 语义
  - [done] 定义 toVersion 与 action
  - [done] 设计 migration contract 与 impl 边界
  - [done] 拆分 contract 与 impl 模块
    - [done] 实现 MigrationVersion
    - [done] 实现 migration entry contract
    - [done] 移动 Home coordinator 至 impl
  - [done] 迁移 Home 启动与测试
    - [done] 更新 registry 选择与版本推进
    - [done] 更新 Application 与 CLI 依赖
    - [done] 覆盖版本解析和 migration 顺序
  - [done] 更新持久规则与完成验证

# Details

- `MigrationVersion` 只包含 major、minor、patch 三个整数，并提供字符串构造入口和比较。
- 每个 migration 只包含 `toVersion` 和普通
  `suspend (home: Path, fileSystem: CoroutineFileSystem) -> Unit` action。
- Stored version 小于 `toVersion` 且 `toVersion` 不高于当前应用版本时执行该 migration。
- Migration 模块拆分为 `app/migration/contract` 与 `app/migration/impl`。
- Contract 只保存 `MigrationVersion` 和 migration entry 形状。
- Impl 保存 Home coordinator、全局 registry、基线验证、版本文件和 Gradle 生成的当前应用版本。
- 删除旧 `app/migration` 单模块入口，不保留 `KodexVersion` 或 `KodexMigration` 兼容层。
- `MigrationVersion(String)` 只接受 canonical `major.minor.patch`，三个分量必须是非负 `Int`。
- Registry 按 `toVersion` 唯一且严格递增；future entry 继续由当前应用版本过滤。
- 验证 contract tests、impl tests、Application tests，以及 JVM 和 Linux x64 编译。
- `app-migration-contract:allTests`、`app-migration-impl:allTests`、
  `agent-storage-filesystem-layout:allTests` 与 Application JVM tests 通过。
- Application 和 CLI 的 Linux x64 编译通过。
