# Task Tree

- [done] 统一完成Codex Home配置独立化
  - [done] 统一升级Kodex settings schema与持久化边界
    - [done] 让Hooks与MCP配置始终由Kodex完整持久化
    - [done] 删除`config.toml`缺失字段继承和空值回退语义
  - [done] 停止常规设置加载自动读取Codex `config.toml`
    - [done] 让模型、推理等级、service tier和输入键位只使用Kodex设置
    - [done] 将`CodexCliStorage`限制为认证和显式导入解码边界
  - [done] [通过自有Settings管理Hooks并提供显式导入](../done/2026-08-15-manage-hooks-in-kodex-settings.md)
  - [done] [通过MCP管理提供显式Codex导入](../done/2026-07-27-plan-complete-mcp-capabilities.md)
  - [done] 将用户级Agent上下文迁移到`~/.agents`
    - [done] 从`~/.agents`读取用户级`AGENTS.override.md`或`AGENTS.md`
    - [done] 只从`~/.agents/skills`读取用户级skills
    - [done] 停止扫描Codex Home下的AGENTS和skills
  - [done] 移除Codex `models_cache.json`运行时依赖
    - [done] 由Kodex维护兼容的API client version
    - [done] 使用内置目录和远端`/models`维护模型目录
  - [done] 清理已废弃的Codex Session导入残留
  - [done] 收窄`codexHome`语义并同步契约与清单
  - [done] 完成统一迁移和回归验证
    - [done] 覆盖旧settings schema升级与空配置
    - [done] 覆盖Hooks和MCP导入preview、筛选及原子提交
    - [done] 覆盖`~/.agents`上下文发现和无Codex缓存启动

# Details

- 状态：`done`。
- 本任务是统一实施入口；Hooks与MCP保留各自任务树，本文件只维护共享边界、执行顺序和最终验收。
- 实施顺序固定为settings所有权与schema迁移、Hooks/MCP管理及导入、Agent上下文与模型目录解耦、历史残留清理、统一回归验证。
- 常规运行不得自动读取或继承Codex本地设置。
- `codexHome`只作为外部Codex数据源路径，用于选择Codex认证时读取`auth.json`，以及用户显式触发Hooks或MCP导入。
- Hooks和MCP导入只读取一次源配置，经过preview、筛选和用户确认后持久化为Kodex自有配置，不建立持续同步。
- 用户级AGENTS和skills统一迁移到`~/.agents`；项目级`AGENTS.md`和`.agents/skills`发现语义保持不变。
- 模型目录不得依赖Codex `models_cache.json`；兼容的API client version由Kodex自身维护。
- Codex Session导入已经废弃，不保留入口、实现依赖或当前设计文档中的有效要求。
- Kodex自身设置、sessions、日志和产物继续存放在`~/.kodex`。
