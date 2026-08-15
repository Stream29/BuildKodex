# Codex CLI Storage兼容性

- 将Codex只读文件格式的解码边界收敛在`Kodex/openai/codex-cli-storage`。
- 组合根根据`CODEX_HOME`或默认规则选择外部Codex数据源；持久化的`codexHome`只选择认证和显式导入来源，不是Kodex设置目录。
- 将Codex文件视为只读数据源，不修改、同步或写回原始内容。
- 常规运行只允许认证模块在`authSource=codex`时读取所选数据源的`auth.json`。
- 常规设置、模型目录、Agent上下文和Session不得读取Codex `config.toml`、`models_cache.json`、AGENTS、skills或thread数据。
- `openai:codex-cli-storage`只对下游返回类型化认证或导入候选；下游不得处理原始TOML、JSON、字段别名或字符串联合类型。
- 只有用户在Settings中显式执行`Import from Codex`时才读取Codex Hook配置；导入期间完成平台命令选择、环境替换、timeout规范化和matcher编译。
- Hook导入必须经过preview、筛选和确认；选中结果作为Kodex自有配置原子持久化，运行时不得重新读取或同步Codex Hook文件。
- 只有用户在MCP管理中显式执行`Import from Codex`时才读取Codex `mcp_servers`；导入语义遵循[MCP管理](mcp-management.md)。
- MCP导入必须经过preview、筛选和确认；选中结果作为Kodex自有配置原子持久化，运行时不得重新读取或同步Codex MCP配置。
- 不提供Codex Session导入入口，不保留Codex thread转换或Session兼容依赖。
