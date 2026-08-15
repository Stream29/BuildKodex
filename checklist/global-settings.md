# 全局设置

- `Kodex/app/shared/settings/*`只承载跨会话的应用设置，不得混入按会话版本化的`KodexAgentSettings`。
- `KodexGlobalSettings.codexHome`必须为非空的外部Codex数据源路径，只用于选择`auth.json`和显式Hooks/MCP导入来源；`newLineKey`默认使用`ShiftEnter`。
- `KodexGlobalSettings.authSource` 是OpenAI账号认证来源的唯一持久化真源：`codex`只读当前`codexHome/auth.json`，`kodex`只读并续期私有`auth.yml`；OpenAI账号凭据不得进入`settings.yml`，MCP凭据遵循[MCP管理](mcp-management.md)。
- 应用侧`KodexAuthStore`实现OpenAI侧只读`OpenAiAuthStore`；继承的`state`是当前认证可用性与请求凭据的唯一真源。`authSource`或`codexHome`更新后store必须重新加载并发布；Application contract只能发布不含access token的认证摘要，并以类型化子状态携带不可用原因；设置页只能将该子状态映射为展示文案，显示邮箱（缺失时账号ID）、套餐和不可用原因；`Sign in`只在认证不可用时显示。
- `Settings > Global`中的Codex Usage、Rate Limit和Usage Limit Reset只能读取独立的非持久化账号快照，不得写入`KodexGlobalSettings`或`OpenAiAuthState`；账号切换时必须立即移除旧快照，兑换Reset前必须二次确认，传输失败重试必须复用同一幂等键。
- 输入键位只允许 `ShiftEnter -> Enter` 与 `Enter -> CtrlEnter` 两组配对；提交键由换行键唯一确定，不得独立配置出冲突组合。
- 未引入持久化后端前，`InMemoryKodexGlobalSettings` 只能原子发布完整 `StateFlow` 快照，不得读写设置文件。
- CLI设置对话框的选项为即时提交：每次选择都必须通过对应持久化store更新；Close和Escape只关闭对话框，不回滚已提交的设置。
- `KodexGlobalSettings.newSession`是新Session默认配置的唯一持久化真源，Agent模式默认使用`Single`，提问模式默认使用`AskUser`；`Settings > New session`无论当前标签为何，都必须显示和更新这份全局默认值。虚拟`NewSession`按标签保留名称、工作目录、模型、推理等级、服务等级、Agent模式和提问模式的内存草稿；`Settings > Session`必须读写当前标签草稿，且不得持久化或覆盖全局默认值。
- `KodexGlobalSettings`是模型、推理等级、service tier、输入键位、Hooks和MCP等应用设置的完整持久化真源；缺失或空配置不得回退到Codex配置。
- Settings必须管理`KodexGlobalSettings.hooks`并提供显式`Import from Codex`操作；导入结果作为Kodex自有配置原子持久化和发布，不与Codex保持同步。
- 从`Settings > New session`修改默认值时必须立即持久化，并继续使用后端返回的effective snapshot。
- `KodexGlobalSettings.sessionTitle`是自动 root Session 标题的唯一全局真源；`Settings > Global`必须即时持久化其开关、模型和推理等级，并在 root Agent 接受首个文本时读取。
- `Settings > Global`的MCP区域必须通过`McpManager`管理添加、编辑、删除、启停、OAuth登录/注销和Codex导入，同时保留`McpService`提供的生命周期状态、healthy工具数和reconnect；前端只能接收脱敏状态与配置摘要，具体语义遵循[MCP管理](mcp-management.md)。
- 进入虚拟`NewSession`不得创建repository entry；只有首个有效content通过隐藏initializer完整写入后，才发布真实Session。
- CLI多行输入框必须按当前`newLineKey`插入换行，并按其配对的`submitKey`提交。
- Kodex私有设置使用无版本的宽松YAML模型和原子写入；读取忽略未知字段、以当前应用默认值补齐缺失字段且不得重写文件，正常设置更新写出当前规范快照，已知字段的非法值仍必须报错；Codex认证及Hooks/MCP显式导入边界遵循[Codex CLI Storage兼容性](codex-cli-storage.md)。
