# Task Tree

- [done] 引入CLI的`NewSession`状态
  - [done] 完成规划
    - [done] 盘点CLI启动、Session选择、设置编辑与首条消息提交链路
    - [done] 确认new-session defaults已有持久化真源
    - [done] 定义`NewSession`与真实Session的状态边界
    - [done] 定义首条prompt的隐藏初始化与发布时序
    - [done] 明确失败、取消、并发和恢复语义
    - [done] 对齐Session surface、top bar、自动标题和Multi-agent V2任务
  - [done] 建立显式UI目标状态
    - [done] 用`NewSession`与`PersistedSession`替代nullable active Session的UI语义
    - [done] 让`NewSession`持有defaults draft、revision、dirty和materializing状态
    - [done] 保持Session overlay状态与目标状态正交
    - [done] 不为`NewSession`伪造Session index、Agent地址、runtime id或`SessionSnapshot`
  - [done] 统一new-session defaults边界
    - [done] 以`CodexGlobalSettings.newSession`作为唯一持久化真源
    - [done] 让status selectors与New session设置页编辑同一份defaults draft
    - [done] 在Apply、首prompt、离开NewSession和shutdown边界提交defaults
    - [done] 持久化成功后使用返回的effective snapshot，不让manager保留第二真源
    - [done] materialize时构造新的`CodexAgentSettings`并只复制四个可配置字段
  - [done] 重构Session选择生命周期
    - [done] 冷启动完成repository list后进入`NewSession`
    - [done] 让New、Ctrl+N和`/new`只进入`NewSession`且不创建repository entry
    - [done] 进入`NewSession`前提交当前Agent的settings draft
    - [done] 保留其他已打开Session及其后台运行资源
    - [done] 让`/close`释放当前真实Session后回到`NewSession`
    - [done] 让选择、fork和import真实Session时切换为`PersistedSession`
  - [done] 实现首prompt materialization
    - [done] 捕获不可变的content payload、composer revision和defaults revision
    - [done] 在创建前持久化dirty defaults并完成输入预校验
    - [done] 通过repository hidden initializer创建root并提交首条用户内容
    - [done] 发布前只执行AgentState输入持久化，不启动Responses、tools或自动标题任务
    - [done] 发布后从最终storage位置打开root、切换UI并以已持久化消息启动response
    - [done] [将首条文本的自动标题触发交给最终root runtime](../done/2026-07-22-plan-automatic-session-title.md)
    - [done] 防止重复提交、重复append和重复创建Session
  - [done] 接入`NewSession`交互
    - [done] 显示`New session`标签及默认model、reasoning、tier和mode
    - [done] 不显示Session index、token count、running标记或rename输入
    - [done] 在`NewSession`中禁用New、Fork和Rename，保留Sessions与Settings
    - [done] 让Session菜单Settings直接打开New session设置页
    - [done] 让全局Settings快捷键继续打开Global页
    - [done] 保持composer焦点并只在首条内容被接受后清理对应revision
    - [done] 保持Session browser只列举真实`SessionEntry`
  - [done] 保留原子初始化contract
    - [done] 继续让repository只理解settings、Agent tree和隐藏initializer
    - [done] 不把prompt或new-session defaults放入agent-session contract
    - [done] 在Multi-agent V2重构中保留等价的hidden root/tree initializer
    - [done] 保证Codex import继续复用同一原子发布边界
  - [done] 完成验证与文档收敛
    - [done] 覆盖冷启动、New、close、选择、fork和import状态转换
    - [done] 覆盖defaults持久化、重启恢复、snapshot隔离与写入失败
    - [done] 覆盖首条文本的单次创建、单次append和response启动
    - [done] 覆盖隐藏初始化失败、取消和发布提交点
    - [done] 覆盖发布后最终open失败时保留真实Session
    - [done] 覆盖重复submit、后台Session运行和多frontend串行化
    - [done] 覆盖New session控件、菜单、settings route、焦点和窄终端布局
    - [done] 更新相关checklist与冲突任务记录
    - [done] 运行CLI Linux测试、AgentState与agent-session相关JVM/Linux测试及Linux Native链接
    - [done] 删除临时文件，不创建Git commit

# Details

- 状态：已完成实现、最终open失败覆盖、自动标题接入与跨平台验证。
- IntelliJ IDEA 2026.2当前正在打开本项目。
- 本任务将冷启动和离开真实Session后的空选择统一解释为显式`NewSession`，但不会提前创建持久Session。
- 本任务修订现有Session surface计划中“New立即创建Session”和“无选中Session显示空状态”的部分；repository布局、Session ordinal和Agent tree设计保持不变。

## 实施前调研结论

- production启动只调用repository list，不会创建或选择Session：`CodexLite/cli/app/src/commonMain/kotlin/io/github/stream29/codex/lite/cli/app/Application.kt:178`、`CodexLite/cli/app/src/commonMain/kotlin/io/github/stream29/codex/lite/cli/app/model/SessionManager.kt:281`。
- 实施前，无选择UI只显示`[Session]`并禁用model、tier和mode：`CodexLite/cli/app/src/mosaicMain/kotlin/io/github/stream29/codex/lite/cli/app/MosaicView.kt:1440`。
- 实施前，首条普通文本会先清空composer，再因没有selected Agent而失败，因此用户输入丢失：`CodexLite/cli/app/src/commonMain/kotlin/io/github/stream29/codex/lite/cli/app/CodexCliViewModel.kt:434`、`CodexLite/cli/app/src/commonMain/kotlin/io/github/stream29/codex/lite/cli/app/model/SessionManager.kt:535`、`:696`。
- 实施前，New、Ctrl+N和`/new`直接调用`SessionManager.newSession()`并立即发布空Session：`CodexLite/cli/app/src/commonMain/kotlin/io/github/stream29/codex/lite/cli/app/CodexCliViewModel.kt:450`、`:726`、`CodexLite/cli/app/src/commonMain/kotlin/io/github/stream29/codex/lite/cli/app/model/SessionManager.kt:217`。
- new-session defaults、设置页和持久化override均已存在，不需要新模型或新文件：`CodexLite/cli/settings/src/commonMain/kotlin/io/github/stream29/codex/lite/cli/settings/CodexGlobalSettings.kt:22`、`:29`、`CodexLite/openai/codex-cli-storage/src/commonMain/kotlin/io/github/stream29/codex/lite/openai/codexclistorage/CodexCliSettings.kt:97`。
- 实施前，`SessionManager.newSessionConfiguration`复制了global settings并形成第二份authority；Settings Apply又先改manager再写文件，写入失败会分叉：`CodexLite/cli/app/src/commonMain/kotlin/io/github/stream29/codex/lite/cli/app/model/SessionManager.kt:200`、`CodexLite/cli/app/src/commonMain/kotlin/io/github/stream29/codex/lite/cli/app/CodexCliViewModel.kt:618`。
- repository已有失败不发布的`createTree(initialSettings, initialize)`隐藏初始化边界，filesystem通过隐藏stage和atomic move发布：`CodexLite/agent-session/contract/src/commonMain/kotlin/io/github/stream29/codex/lite/agentsession/contract/CodexSession.kt:45`、`CodexLite/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/codex/lite/agentsession/filesystem/FileSystemCodexSessionRepository.kt:82`。
- `CodexAgentState.appendUserMessage`已经以补偿式事务提交settings、history和timestamp，可作为stage内唯一写入口：`CodexLite/agent-state/impl/src/commonMain/kotlin/io/github/stream29/codex/lite/agentstate/impl/CodexAgentStateImpl.kt:241`。

## Canonical状态模型

```text
SessionTargetState
  NewSession(defaultsDraft, materialization)
  PersistedSession(sessionIndex, selectedAgent?)

Materialization
  Idle
  PersistingDefaults
  InitializingSession
  Published(sessionIndex)
```

- `NewSession`是CLI UI状态，不是`CodexSession`、`CodexSessionEntry`或特殊Agent节点。
- `NewSession`没有index、地址、title authority、history、token count、lease或runtime资源，也不会预留下一个ordinal。
- `PersistedSession`可以在root runtime尚未打开或打开失败时仍由真实Session index表达，避免publish成功后再次materialize同一prompt。
- 现有`SessionSurfaceState = Closed | Menu | Renaming | Browser`只表达overlay，应与target state保持正交；实现时可重命名为`SessionOverlayState`降低歧义：`CodexLite/cli/app/src/commonMain/kotlin/io/github/stream29/codex/lite/cli/app/CodexCliViewModel.kt:146`。
- Session browser继续只显示repository返回的真实`SessionEntry`；`NewSession`只由顶栏、状态控件和空历史提示表达。

状态转换：

```text
startup ------------------------------------------> NewSession
PersistedSession -- New / Ctrl+N / /new ----------> NewSession
PersistedSession -- /close -----------------------> NewSession
NewSession -- select/import ----------------------> PersistedSession
PersistedSession -- fork -------------------------> PersistedSession(new index)
NewSession -- first accepted content -> staging --> PersistedSession(new index)
```

- New是幂等选择动作，不创建Session、不清空composer、不重置defaults。
- 进入New时只清除当前选择；其他已打开Session和后台Agent继续按既有ownership运行。
- 空白输入不创建Session。`/help`、`/session`和`/quit`等本地命令先按命令处理，不materialize。
- New状态中的配置命令应写入同一defaults draft；fork、rename、checkout和cancel等真实Session动作保持不可用。
- 首条输入接口使用有序`List<ContentItem>`而不是把materialization固化为`String`；当前文本composer不扩展附件UI，但未来有效的纯图片输入也可遵循同一创建边界。

## Defaults真源与提交边界

- 唯一持久化真源是`CodexGlobalSettings.newSession`；它继续通过Codex配置继承和`$CODEX_HOME/GlobalSettings.yml`稀疏override解析。
- `NewSession`持有四字段`SessionConfiguration` draft，不长期持有完整`CodexAgentSettings`；materialize时创建fresh settings以获得新的turn identity和其余默认字段。
- status selectors和Settings的New session页修改同一draft。Settings Cancel不改变draft，Apply持久化后用返回的effective snapshot刷新draft。
- inline修改遵循既有update-commit语义，在首prompt、选择已有Session、退出NewSession和application shutdown时合并提交。
- 首prompt命令必须同时捕获defaults revision和composer revision；排队后的UI编辑不得改变本次创建使用的snapshot。
- global settings持久化成功后，Session创建失败不回滚defaults；这正是“修改NewSession等于修改default settings”的语义。
- 真实Session只复制materialize提交点的defaults。之后修改defaults不会反向更新已有Session，已有Session设置也不会更新defaults。
- Session browser切换也必须携带pending settings commit；当前直接OpenSession会绕过离开边界：`CodexLite/cli/app/src/commonMain/kotlin/io/github/stream29/codex/lite/cli/app/CodexCliViewModel.kt:487`、`:691`。

## 首prompt发布契约

1. ViewModel串行捕获content payload、composer revision和defaults draft revision，并进入materializing状态以阻止重复submit。
2. 完成附件/内容预校验；无有效content时回到Idle且不触碰repository。
3. 如果defaults dirty，先写入global settings，并以写入返回的effective new-session settings构造fresh `CodexAgentSettings`。
4. coordinator调用repository hidden initializer；repository只负责分配index、建立隐藏root、执行caller initializer并在成功后发布。
5. initializer在临时root上创建最小AgentState输入链，刷新context source，并通过CLI用户消息路径恰好一次append首条用户内容与显式skill正文。
6. stage内禁止Responses请求、tool执行、自动标题请求和长期runtime资源；依赖最终storage path的identity也不得逃逸stage。
7. initializer成功后原子发布真实Session；这是不可回滚的materialization提交点。
8. coordinator从最终storage位置打开root，发布`PersistedSession`选择，并以`message = null`恢复已存在的`UserMessage`开始response，避免二次append。
9. 若首条内容包含非空文本，在最终root runtime显式触发一次自动标题资格检查；stage内不启动标题任务。
10. 只有内容已经发布后才按捕获的composer revision清除本次输入；如果用户已继续编辑，保留较新的composer内容。

- global settings与Session repository属于两个存储，不能伪装成单一事务；严格顺序是先持久化defaults，再隐藏初始化Session。
- defaults写入失败：保持`NewSession`和原输入，不创建stage。
- initializer失败或publish前取消：丢弃stage，保持`NewSession`和原输入，ordinal可以复用。
- publish后最终open、配置或response失败：保留包含首prompt的真实Session并刷新列表，不自动删除或创建第二个Session。
- response取消只取消真实Session中的turn；不会退回`NewSession`或删除已发布history。
- 多frontend和重复Enter由ViewModel command channel、materializing token与manager mutation mutex共同串行化；同一token最多publish一次。

## UI与交互

- New状态显示`New session`以及defaults的model、reasoning、tier和mode；不显示`s<index>`、token、running `*`或rename输入。
- status selectors在New状态保持可用，并编辑defaults draft；在真实Session运行中继续遵守现有禁用规则。
- Session菜单在New状态禁用New、Fork和Rename，保留Sessions与Settings；处理器本身仍保持幂等和校验。
- 从Session菜单打开Settings时，New状态直接选择`SettingsRoute.NewSession`，真实Session选择`SettingsRoute.Session`；全局快捷键继续选择Global。
- 冷启动自动聚焦composer。从Session菜单进入New后将焦点交回composer；materialize过程不替换composer节点。
- 空历史提示应说明首条prompt会创建Session，不再使用无法区分New状态的`No committed conversation items`。
- 顶栏位置、颜色、popup anchor和窄终端布局仍由top bar任务负责；本任务只提供它消费的New/Persisted chrome模型。

## 与其他任务的关系

- 保留Session surface任务的冷启动轻量list、两级懒加载和后台Agent语义，但以本任务取代“New立即创建”和“空状态无选择”的定义：`kanban/done/2026-07-22-redesign-session-surface.md`。
- top bar任务的布局与配色不变，但无Session标签从`[Session]`改为`[New session]`，New行为改为选择虚拟状态：`kanban/done/2026-07-22-move-session-control-to-top-bar.md:15`、`:19`、`:49`。
- 自动标题仍以真实root的`Session <index>`为初始名；stage提交的首条文本只在publish后交给最终runtime触发生成：`kanban/done/2026-07-22-plan-automatic-session-title.md:50`、`:58`。
- Multi-agent V2目标contract必须保留当前`createTree`的等价hidden initializer，否则本任务和Codex Session import都会失去失败不发布保证：`kanban/done/2026-07-21-implement-multi-agent-v2.md:67`、`CodexLite/openai/codex-cli-storage/src/commonMain/kotlin/io/github/stream29/codex/lite/openai/codexclistorage/CodexSessionImport.kt:66`。
- storage id恢复任务要求filesystem identity由最终路径派生，因此stage只能写timeline，不能发起依赖identity的外部请求：`kanban/ongoing/2026-07-22-restore-agent-storage-id.md:14`、`:25`。
- 本任务不增加存储schema、不迁移既有Session、不删除现有空Session，也不改变repository的SessionEntry投影。

## 主要实现位置

- `CodexLite/cli/app/src/commonMain/kotlin/io/github/stream29/codex/lite/cli/app/CodexCliViewModel.kt`：target state、defaults draft、提交token、命令结果、settings route和composer恢复。
- `CodexLite/cli/app/src/commonMain/kotlin/io/github/stream29/codex/lite/cli/app/model/SessionManager.kt`：清除选择、hidden initializer、最终open、response resume和结构化materialization结果。
- `CodexLite/cli/app/src/mosaicMain/kotlin/io/github/stream29/codex/lite/cli/app/MosaicView.kt`：New session chrome、selectors、菜单能力、空历史和焦点。
- `CodexLite/cli/app/src/commonTest/kotlin/io/github/stream29/codex/lite/cli/app/TestCodexLiteApplication.kt`：让依赖真实Session的测试显式创建，避免fixture掩盖production启动行为。
- `CodexLite/cli/app/src/commonTest/kotlin/io/github/stream29/codex/lite/cli/app/SessionManagerTest.kt`、`CodexCliViewModelTest.kt`和`CodexLite/cli/app/src/mosaicTest/kotlin/io/github/stream29/codex/lite/cli/app/MosaicViewTest.kt`：状态、事务、恢复和UI覆盖。
- `CodexLite/agent-session/in-memory/src/commonTest/kotlin/io/github/stream29/codex/lite/agentsession/inmemory/InMemoryCodexSessionRepositoryTest.kt`与`CodexLite/agent-session/filesystem/src/commonTest/kotlin/io/github/stream29/codex/lite/agentsession/filesystem/FileSystemCodexSessionRepositoryTest.kt`：initializer失败、取消和publish提交点。

## 验证重点

- 冷启动即使存在真实Session也保持`NewSession`，只list而不open；不产生新目录、lease、AgentState或runtime。
- 修改defaults不会创建Session；Apply与shutdown可持久化并在重启后恢复。
- New、Ctrl+N、`/new`和New状态下重复操作均不增加SessionEntry。
- 首个有效content只创建一个Session、只append一次、选中真实root并从已持久化`UserMessage`启动一次response。
- defaults与首prompt在publish前对list不可见；initializer异常或取消后无SessionEntry、无残留stage且ordinal可复用。
- publish后response失败仍能list、open并恢复同一Session，不能再次创建或丢失prompt。
- materializing期间重复submit不重复创建；较新的composer编辑不会被旧完成事件清除。
- 进入New不取消其他Session的后台Agent；重新选择后其history、stream和request-user-input状态仍正确。
- New状态的model、reasoning、tier和mode选择器、Settings Apply/Cancel、Session菜单enablement及composer焦点同时覆盖键盘与鼠标。
- 更新当前依赖`/new`立即创建的测试，并让多数启动测试使用`createSession = false`的production基线。
