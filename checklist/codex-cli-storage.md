# Codex CLI Storage兼容性

- 将Codex只读文件格式的解码边界收敛在`Kodex/openai/codex-cli-storage`。
- 组合根严格使用当前用户的`~/.codex/`作为外部Codex数据源，不读取`CODEX_HOME`；该路径不是Kodex设置目录，也不进入持久化设置。
- 将Codex文件视为只读数据源，不修改、同步或写回原始内容。
- 常规运行只允许认证模块在`authSource=codex`时读取固定数据源的`auth.json`。
- 常规设置、模型目录和Session不得读取Codex `config.toml`、`models_cache.json`或thread数据；启用 `Context sources > Codex home` 后，Agent context 可按其独立来源配置读取固定 `~/.codex/` 下的 `AGENTS.md` 与 skills。
- `openai:codex-cli-storage`只对下游返回类型化认证或MCP导入候选；下游不得处理原始TOML、JSON、字段别名或字符串联合类型。
- `openai:codex-cli-storage`不得读取、解码或导入Codex Hook配置。
- 只有用户在MCP管理中显式执行`Import from Codex`时才读取Codex `mcp_servers`；导入语义遵循[MCP管理](mcp-management.md)。
- MCP导入必须经过preview、筛选和确认；选中结果作为Kodex自有配置原子持久化，运行时不得重新读取或同步Codex MCP配置。
- 不提供Codex Session导入入口，不保留Codex thread转换或Session兼容依赖。
