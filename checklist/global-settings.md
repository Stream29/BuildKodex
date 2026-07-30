# 全局设置

- `cli:settings`只承载跨会话的应用设置，不得混入按会话版本化的`CodexAgentSettings`。
- `CodexGlobalSettings.codexHome` 必须为非空的本地 Codex CLI 根目录，`newLineKey` 默认使用 `ShiftEnter`。
- `CodexGlobalSettings.authSource` 是认证来源的唯一持久化真源：`codex`只读当前`codexHome/auth.json`，`codex-lite`只读并续期私有`auth.yml`；凭据本身不得进入`settings.yml`。
- 输入键位只允许 `ShiftEnter -> Enter` 与 `Enter -> CtrlEnter` 两组配对；提交键由换行键唯一确定，不得独立配置出冲突组合。
- 未引入持久化后端前，`InMemoryCodexGlobalSettings` 只能原子发布完整 `StateFlow` 快照，不得读写设置文件。
- CLI设置对话框的选项为即时提交：每次选择都必须通过对应持久化store更新；Close和Escape只关闭对话框，不回滚已提交的设置。
- `CodexGlobalSettings.newSession`是新Session默认配置的唯一持久化真源；`Settings > New session`无论当前标签为何，都必须显示和更新这份全局默认值。虚拟`NewSession`可保留其创建时的内存草稿，但该草稿不得成为设置页的读写目标。
- `CodexGlobalSettings`必须包含从当前Codex Home读取并投影出的有效Codex设置；`hooks`未被Codex Lite覆盖时合并当前项目`.codex`来源，覆盖后则持久化并完整替代Codex配置，两种结果都随`refresh/update`原子发布。
- 从`Settings > New session`修改默认值时必须立即持久化，并继续使用后端返回的effective snapshot。
- `CodexGlobalSettings.sessionTitle`是自动 root Session 标题的唯一全局真源；`Settings > Global`必须即时持久化其开关、模型和推理等级，并在 root Agent 接受首个文本时读取。
- 进入虚拟`NewSession`不得创建repository entry；只有首个有效content通过隐藏initializer完整写入后，才发布真实Session。
- CLI多行输入框必须按当前`newLineKey`插入换行，并按其配对的`submitKey`提交。
- Codex设置继承、`GlobalSettings.yml`持久化和Session导入入口必须遵循[Codex CLI Storage兼容性](codex-cli-storage.md)。
