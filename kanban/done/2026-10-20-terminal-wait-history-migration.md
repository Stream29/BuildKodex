# Task Tree

- `实现冻结的 0.5.2 历史参数迁移与 fixtures`()
- `交接源码与隔离验证结果`()

# Details

- 父任务：[Agent 等待锁体验](../executable/2026-10-02-cross-platform-agent-synchronization.md)，P1 与当前模型迁移集成测试。
- 用户已授权源码实施，迁移目标 `0.5.2`；不操作真实 Home，不提交或发布。
- 写入范围仅 `Kodex/app/migration/`；旧版本冻结文件不修改。DTO、应用版本和业务协议由 P2 负责。
- subagent 不运行 Gradle；主代理统一调度构建和测试，并记录审阅结果。
- 已实现唯一 `0.5.2` registry entry、独立冻结转换/fixtures 与当前 reader 集成；仅补 typed work/unstable 的缺失等待值，不遍历 legacy subagents，保留未知字段与已完整记录的字节。
- 主代理修正未知字段过度拒绝和 fixture 清理 receiver，`app-migration:jvmTest` 128 例通过，其中新转换及当前模型集成 35 例；旧版本冻结源码/fixtures 未变。其他平台限制见父任务。
