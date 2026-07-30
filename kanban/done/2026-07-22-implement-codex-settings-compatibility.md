# Task Tree

- [done] 实现Codex设置管理兼容性
  - [done] 在`openai:codex-cli-storage`中建立Codex Home和设置管理API
  - [done] 只读解析Codex原生配置的受支持层级和来源诊断
  - [done] 实现`$CODEX_HOME/GlobalSettings.yml`稀疏override模型与原子持久化
  - [done] 合并Codex继承值与Kodex override并发布有效设置快照
  - [done] 将应用启动和全局设置UI接入新设置管理API
  - [done] 删除被新边界取代的Codex设置适配器和重复文件I/O
  - [done] 覆盖缺失文件、无效配置、未知字段、重启和原子写入测试
  - [done] 运行相关格式化、类型检查和测试

# Details

- 状态：实现完成，Linux与macOS相关测试通过。
- 实现必须遵循[Codex CLI Storage兼容性](../../checklist/codex-cli-storage.md)。
- Codex原生配置保持只读；Kodex只写入自有`GlobalSettings.yml`。
- Codex Session导入入口由Session导入实现任务接入全局设置页面。
