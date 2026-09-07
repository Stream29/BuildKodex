# Kodex Home skill 0.4.3 验证

- 用户确认的精简稿已逐字节核对并内嵌到 [版本源码](../../Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/v0_4_3/KodexHomeSkill.kt#L8)。
- [迁移](../../Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/v0_4_3/MigrateToV0_4_3.kt#L8)只原子替换 `skills/kodex-home/SKILL.md`，只清理本版本 temporary，不扫描 Session 历史。
- 2026-09-07 在 MacBook 独立检出验证，基于 `cb97b98e` 加本次改动；5 个被测源码文件的 SHA-256 与本地一致。
- 使用 GraalVM Java 25、`--no-configuration-cache`；每个测试任务显式 `--rerun`，不是复用缓存测试结果。

| 模块 | JVM | macOS ARM64 |
| --- | --- | --- |
| `app-migration-impl` | 31 | 30 |
| `app-migration-contract` | 2 | 2 |
| `agent-storage-filesystem-layout` | 4 | 4 |
| `utils-filesystem-lease-impl` | 7 | 6 |

- 共 86 次测试执行，零失败、零跳过；[新迁移的 14 个场景](../../Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/v0_4_3/MigrateToV0_4_3Test.kt#L25)在两端均通过。
- 覆盖全新 Home、跨版本升级、future entry 不激活、原有文件保留、重复执行，以及原子发布前后的 I/O 失败和真实协程取消后重跑。
- 相对 v0.4.2，历史迁移实现和 fixture 原文未改；用户单独批准仅将 [历史注册表断言](../../Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/v0_3_3/MigrateToV0_3_3Test.kt#L24)限制到 `<= 0.3.5`。
- 被测构建生成的当前版本仍为 0.4.2；升级测试显式传入 0.4.3。上述是隔离 Home 实文件测试，不是 0.4.3 CLI 启动或发布包验证。

## 版本 bump 后复验

- 2026-09-07 在 MacBook 独立检出 `b443742b`，仅应用三处 0.4.2 → 0.4.3 版本修改；三处被测文件与本地 SHA-256 一致。
- 继续使用 GraalVM Java 25 和 `--no-configuration-cache`；显式重跑 `:app-migration-impl:jvmTest`、`:app-migration-impl:macosArm64Test`，共 31 + 30 次测试，零失败、零错误、零跳过。
- `:mcp-impl:compileKotlinJvm` 通过；构建生成的 `GeneratedKodexApplicationVersion` 为 `0.4.3`。
- 仅完成发布准备：未构建四平台发布包，未启动 0.4.3 CLI，未写入真实 Home。
