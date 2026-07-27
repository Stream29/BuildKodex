# 全局设置

- `cli:settings`只承载跨会话的应用设置，不得混入按会话版本化的`CodexAgentSettings`。
- `CodexGlobalSettings.codexHome` 必须为非空的本地 Codex CLI 根目录，`newLineKey` 默认使用 `ShiftEnter`。
- 输入键位只允许 `ShiftEnter -> Enter` 与 `Enter -> CtrlEnter` 两组配对；提交键由换行键唯一确定，不得独立配置出冲突组合。
- 未引入持久化后端前，`InMemoryCodexGlobalSettings` 只能原子发布完整 `StateFlow` 快照，不得读写设置文件。
- CLI设置对话框中的编辑必须暂存；只有Apply可以发布新快照，Cancel和Escape不得修改当前设置。
- `CodexGlobalSettings.newSession`是新Session默认配置的唯一持久化真源；虚拟`NewSession`与设置页必须编辑同一份draft，不得在Session manager中保留第二份配置。
- `CodexGlobalSettings`必须包含从当前Codex Home读取并投影出的有效Codex设置；`hooks`未被Codex Lite覆盖时合并当前项目`.codex`来源，覆盖后则持久化并完整替代Codex配置，两种结果都随`refresh/update`原子发布。
- New session defaults必须在Apply、首条prompt、离开虚拟`NewSession`和应用退出边界提交；持久化后继续使用后端返回的effective snapshot。
- 进入虚拟`NewSession`不得创建repository entry；只有首个有效content通过隐藏initializer完整写入后，才发布真实Session。
- CLI多行输入框必须按当前`newLineKey`插入换行，并按其配对的`submitKey`提交。
- Codex设置继承、`GlobalSettings.yml`持久化和Session导入入口必须遵循[Codex CLI Storage兼容性](codex-cli-storage.md)。
