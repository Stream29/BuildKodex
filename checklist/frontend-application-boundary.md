# Frontend 与共享应用层边界

- 当前宿主所有权按 [Daemon 设计](backend-daemon.md)拆开：
  `withFrontendApplication` 只装配前端；独立后端持有业务图。
  TUI 退出不清理后端，网络重绑定不关闭页签/Composer owner。
  旧组合根与启动测试检查点只在 test-support；生产应用不依赖后端服务、
  Home migration 或 in-memory transport，不按具体连接实现做类型分支。
  下文单宿主清理及旧 binding 恢复规则不要求在短暂断线时销毁这些本地状态。
- 以下保留进程内 RPC 阶段的业务与组件边界；其单 CLI 宿主清理两侧资源的接线已被 Daemon 替代，前端视图关闭与后端宿主退出分别处理。

- Application 的 framework-free 契约位于 `Kodex/app/application/src/specMain`，Session 根契约在独立的 `Kodex/app/session/contract/src/specMain`；真实所有者及宿主装配放在 `Kodex/app/{application,session,rpc}` 的实现源集。常规组件在 `viewmodel` 项目内按 [KMP 源集设计](kmp-spec-impl-source-sets.md) 使用 specMain/commonMain；View 保持独立。不为旧层保留转发坐标或平行 ViewModel。
- Agent 共享契约保留独立 `app/component/agent/contract` 项目；Sidebar 契约与 Application 同项目，Tab-bar 契约位于其 `view/src/specMain`。Settings persistence 契约位于 `app/settings/filesystem/src/specMain`；Home migration 契约位于 `app/migration/src/specMain`，CLI 实现位于同项目 cliHostMain。这些职责差异不按目录名硬造配对。
- 组件契约必须声明其 ViewModel 所需 dependency ports、全部交互及顺序/取消语义、对外暴露的 state/effect，以及每个状态分支的 renderer 语义；这些 KDoc 与接口是组件事实来源。
- 让 contract 通过 `StateFlow`、结构化 state、effect 和 command 暴露 frontend API，不暴露 Mosaic、终端组件或 Compose Desktop UI 类型。
- 组件 ViewModel 的 specMain 不得依赖 Mosaic、终端组件或具体平台 I/O；ViewModel 的 commonMain 与独立 View 的 renderer source set 负责接入具体依赖和 renderer。既有 renderer 自有的公开 API 仍遵循 specMain 导出规则，不因此成为可移植业务契约。
- Frontend 直接消费准确的 child ViewModel；父 ViewModel 只发布自身状态、父级关系和稳定 child handle，不为 renderer 镜像 child mutable state。
- ViewModel 实现通过 constructor injection 或 typed factory 获取 settings、models、authentication、repository 等依赖；frontend 和 Application contract 不充当 service locator。
- 根 renderer 放在 `Kodex/app/view`，组件 renderer 放在对应 `view`；多消费者共享的纯 visual helper 保留在 `Kodex/app/view/components` 等现有 UI 基础设施。renderer 差异分别进入 `mosaicMain` 与未来的 `desktopMain`，renderer 无关的展示逻辑才进入 `commonMain`。
- 领域 view 模块统一应用 `kodex.kmp-view`；不得在各模块重复声明 Mosaic target hierarchy 或平台 `dependsOn`。
- 只将 native entrypoint、Mosaic host 与 CLI 生命周期放在 `Kodex/app/cli`；未来 Desktop 对应内容直接放在 `Kodex/app/desktop`。
- CLI/Desktop host 负责 Ctrl+C、renderer 结束和 process disposal，并在 `finally` 调用 Application `shutdown()`；没有产品级 Exit/Quit 操作时不得在 Application contract 预设 `requestExit()` 或 lifecycle state。
- CLI 入口统一应用 `kodex.kmp-cli-executable`，由 convention 固定 Native executable、entrypoint 和稳定产物名。
- 当前 CLI/TUI 产品目标遵循[四个 Native 目标](kmp-spec-impl-source-sets.md#current-native-product-targets--2026-10-22)，不再支持 JVM 产品；既有 JVM 构建/测试记录不代表仍支持 JVM Desktop，Web/Wasm 规划另计。
- 打开外链属于客户端宿主能力，API/实现跟随[客户端目标边界](kmp-spec-impl-source-sets.md#client-owned-external-urls--2026-10-21)，不通过 RPC 要求后端机器打开链接；浏览器实现随 Web 客户端接入，不补伪宿主能力。
- 前端业务代码直接使用普通 KotlinLogging，不调用后端业务 scope 扩展或依赖两个后端日志库。TUI 文件诊断由 CLI 内部启动接线维护，不复用后端初始化；[日志库边界](kmp-spec-impl-source-sets.md#independent-backend-logging-libraries--2026-10-21)要求同时检查解析后的生产依赖图。双模式 CLI 的 backend 子命令及测试接线不属于前端业务图。
- 让 view 模块单向消费 contract；可依赖承载契约的合并 ViewModel 项目。契约源码与 ViewModel 模块不得依赖领域或组件 view、CLI 入口模块或未来的 Desktop 入口模块。
- Application 通过 framework-free `ApplicationPopupState` 发布当前独占 popup；每个 open state 直接提供准确 child ViewModel，frontend 不建立第二份 route、content request 或 request-id authority。
- Frontend 直接渲染 popup child 并将 exact open handle 用于 dismiss；popup child 的 draft 与命令由 child ViewModel 持有。
- 原 Agent、Settings、Sidebar 的 spec 由对应组件接管。Agent/Global Settings 的真实 RPC 实现在 `app/rpc`，Sidebar 实现在 `app/application`；不为物理配对新增无职责的 provider 或实现项目。
- 一个 binding 发布唯一 Agent ViewModel，Session 的 rootAgent 直接借用其发布源。恢复撤回并关闭旧 Agent，再发布新 binding 的 child；本地关闭不 Stop backend，也不重放已接受写入。settings/name 的最后可见值缓存可以保留，不构成另一份可写事实来源。
- Application 的 persisted select/close 命令在原 mutex 内验证捕获的 child 实例；过期目标返回 false，不先换成异步等待前的 tab index。
  NewSession 是 Application 稳定单例，提交以准确 composer revision 寻址，拒绝进行中重复事件，成功追加并选择真实 tab；真实失败与取消仍传播。
- 普通创建只传 cwd，后端读取自己的新建默认值；NewSession 只编辑共享四字段与本地 cwd/composer，不再伪装为持久 Session。
  Settings 的目标为 nullable persisted handle；null 使用恒为 Unavailable 的 source，不回查当前 tab。
- Shell 模型仍属于 Agent spec，列表、摘要、菜单和 hover 随 Sidebar View；方向与 popup 几何只属于 renderer。Settings 根拥有切页/清理/refresh，Application 拥有 popup 的准确关闭及 Login returnTo。
- 按交互与生命周期相似性组织组件迁移批次，覆盖同类功能的全部宿主入口；同批迁移不意味着合并不同领域组件或抹平宿主语义。
- 会话重命名与删除分别采用 `app/component/session-rename`、`app/component/session-delete` 的 spec/ViewModel/View 边界。依赖端口绑定打开时的目标；Session Settings 拥有 revision-bound Rename child，Catalog 拥有准确 Delete child 及删除后刷新，Application 适配器仍负责根会话操作与标签清理。
- 会话删除等常规组件的消费者使用原 ViewModel 项目坐标；此处 spec/ViewModel/View 指职责，不是要求恢复已移除的 spec 项目。
- 工作目录选择采用 `app/component/working-directory`，拥有一个 Path Picker child 与绑定目标的 selection port；不镜像 browser state。Application 保留目标与 suggestion callId 校验，Settings 保留 exact-handle 消费和 revision 队列规则；renderer 借用 browser，由外层组件统一关闭。
- 待处理工具分别采用 `app/component/request-user-input` 和 `app/component/suggest-subagent-task`。组件拥有答案/配置/拒绝草稿及提交状态；Agent 适配器提供准确 pending、完成事件、resume 和 Session 创建端口。工具面板卸载不关闭 Agent 持有的 child，也不撤销 owner scope 已接受的提交。
- MCP 与 Hook 设置分别采用 `app/component/mcp-settings`、`app/component/hook-settings`。组件拥有完整列表交互、业务草稿、对话框和导入决策；RPC 适配器隐藏原始配置/凭据及存储。两个 child 与其他全局编辑共享原应用级队列，接受不代表持久化成功，已接受的 baseline/payload 不因 child 关闭而失效。
- MCP 前端仅依赖[共享 contract](kmp-spec-impl-source-sets.md#mcp-contractbackend-boundary--2026-10-21)，不持有或运行 MCP SDK client、连接或 Manager；执行与管理由 `mcp/backend` 拥有。旧混合 `:mcp` 已撤除，不留兼容转发项目。
- ToolSearch 前端只消费[共享事件数据契约](kmp-spec-impl-source-sets.md#toolsearch-contractbackend-boundary--2026-10-21)，不持有搜索引擎；目录构建、索引查询和 Lucene 属于 backend。已完成拆分，最新12个共享/前端项目的 Native 编译依赖图均无搜索 backend、索引或 Lucene；此前 JVM 编译/运行图是历史证据，测试接线与双模式 CLI 的后端子命令另计。
- MCP/Hook 编辑回调只提交字段更新，由 ViewModel 在 exact token 的最新草稿上执行；不使用 composition 捕获的整份 draft 覆盖其他同帧输入。
- MCP URL effect consumer 挂在稳定 Settings owner 上，Settings → Login → return 保留同一 mount；整棵 renderer 卸载仍取消其捕获的 in-flight effect，真实 owner close 清理其他 VM-owned 等待。
- Global Settings 的配置编辑分别采用 `app/component/context-source-settings`、`app/component/session-title-settings`、`app/component/application-preferences`。父级发布稳定 child，不镜像其投影或保留第二组命令。标题默认值与环境路径策略经 typed dependency 注入；持久化写入共用应用队列，侧栏宽度保持应用作用域的临时状态。
- OpenAI 设置分别采用 `app/component/authentication-settings`、`app/component/account-usage`、`app/component/usage-reset`。认证/用量 wire 摘要保留原 FQCN 与序列化形状；Account Usage 只发出重置意图，由宿主连接独立 Reset child。Reset renderer 提交准确 confirmation，不以最新选择替换旧回调，也不自动选券或重发消费。
- Settings 切页清理未接受的编辑/导入，关闭才结束组件短期观察与认证。MCP URL effect 由整个 Settings 生命周期内的单一 handler 消费；打开失败只取消 effect 捕获的准确操作。共享失败由宿主或组件渲染一次，不建立另一份 failure authority。
- 当前会话配置与新会话默认值分别采用 `app/component/session-settings`、`app/component/new-session-defaults`。前者绑定准确 source/revision，关闭取消本地 CAS 等待并释放 source；后者冻结字段 baseline，使用独立应用级队列，关闭后排空已接受写入。不能为统一 API 混同两种关闭策略，也不能把 Title 配置并成默认值的第五字段。
- Agent 与 New Session draft 各公开稳定的 `app/component/runtime-configuration` child；组件只观察四项配置和模型能力，tier leaf 一次原子提交 model/effort/tier，questions 单独更新。命令等待属于调用者，不改成应用级队列；popup 菜单由宿主直接挂载并与 triggers 共用 renderer handles。
- `app/component/session-catalog` 包含完整目录 renderer；`app/component/history-index` 拥有稀疏扫描和 row/detail/timestamp 的独立读 handle。Index 的 Check out 只通过绑定 Agent 的滚动端口定位，不回退历史；nonce/revision、目标变化和 release 后的迟到结果不能重新发布。宿主只保留左右栏布局、Shell/Index popup 仲裁和 hover grace，不镜像业务加载状态。
- 将焦点、hover、popup anchor、菜单、布局、滚动视口和 renderer 专用文案保留在对应 renderer source set。
- History spec/ViewModel 只发布真实有界窗口、portable follow intent 与准确目标的 pending scroll effect；renderer 上报当前窗口的实际 visible child，保留原可见 chunk 淘汰规则。LazyList、测量和输入分类归 View，不在 VM 中监听 Compose snapshot。
- 根 View 按准确 History VM 实例保留 renderer-only 滚动状态，tab 卸载不重置，binding 替换/关闭清理；不保留第二份业务窗口或注册表。滚动 effect 执行后准确 acknowledgment，迟到 acknowledgment 不清除新请求。
- LazyList 与通用滚动组件复用 `app/view/components`，不设独立领域 VM/spec。Patch 呈现随其实际消费者位于 History View，保留原 `cli.patch` 类型与完整数据，不为目录配对增加 Patch Controller 或依赖转发层。
- 将无 UI 框架依赖的实现测试随 ViewModel 维护，将 renderer 与交互测试放在对应 view 模块的 renderer test source set。
- 同时装配 Application、真实 RPC/backend 与根 renderer 的跨模块测试归现有 `integration-test` 的适用 Native 测试层；历史 `jvmTest` 目录已非输入，其覆盖仍需按需求迁移，不能视作已运行 Native 回归。测试通过返回既有 contract 的真实创建工厂装配，不公开内部实现类，也不为这些测试向 Application 的生产 source set 注入 Compose/Mosaic。
- 不为 Mosaic 和 Desktop 复制领域模块树，也不把完整 screen 强制建模为 `expect`/`actual`；两个 renderer 消费同一 contract 并分别提供根 view。
