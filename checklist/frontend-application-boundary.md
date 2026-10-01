# Frontend 与共享应用层边界

- 以下约束面向现行进程内应用；RPC 迁移采用 [RPC 前后端分离](rpc-architecture.md)。一期仍由单 CLI 宿主清理两侧资源，前端视图关闭与整个宿主退出分别处理。

- 将跨组件共享且无 UI 框架依赖的 contract、ViewModel、state、effect 和 command 放在 `Kodex/app/contract/*` 与 `Kodex/app/viewmodel/*`；大型、可独立交互的垂直组件则放在 `Kodex/app/component/<component>/{spec,impl/<topic>}`。
- 组件 `spec` 必须声明其 ViewModel 所需 dependency ports、全部交互及顺序/取消语义、对外暴露的 state/effect，以及每个状态分支的 renderer 语义；这些 KDoc 与接口是组件事实来源。
- 让 contract 通过 `StateFlow`、结构化 state、effect 和 command 暴露 frontend API，不暴露 Mosaic、终端组件或 Compose Desktop UI 类型。
- 组件 spec 不得依赖 Mosaic、终端组件或具体平台 I/O；组件 impl 的 `viewmodel` 与 `view` topic 负责接入具体依赖和 renderer。
- Frontend 直接消费准确的 child ViewModel；父 ViewModel 只发布自身状态、父级关系和稳定 child handle，不为 renderer 镜像 child mutable state。
- ViewModel 实现通过 constructor injection 或 typed factory 获取 settings、models、authentication、repository 等依赖；frontend 和 Application contract 不充当 service locator。
- 将领域 view 放在 `Kodex/app/view/*` 或组件 `impl/view` KMP 模块；renderer 差异分别进入 `mosaicMain` 与未来的 `desktopMain`，renderer 无关的展示逻辑才进入 `commonMain`。
- 领域 view 模块统一应用 `kodex.kmp-view`；不得在各模块重复声明 Mosaic target hierarchy 或平台 `dependsOn`。
- 只将 native entrypoint、Mosaic host 与 CLI 生命周期放在 `Kodex/app/cli`；未来 Desktop 对应内容直接放在 `Kodex/app/desktop`。
- CLI/Desktop host 负责 Ctrl+C、renderer 结束和 process disposal，并在 `finally` 调用 Application `shutdown()`；没有产品级 Exit/Quit 操作时不得在 Application contract 预设 `requestExit()` 或 lifecycle state。
- CLI 入口统一应用 `kodex.kmp-cli-executable`，由 convention 固定 Native executable、entrypoint 和稳定产物名。
- 让 view 模块单向依赖 contract/spec；contract/spec 与 ViewModel 模块不得依赖领域或组件 view、CLI 入口模块或未来的 Desktop 入口模块。
- Application 通过 framework-free `ApplicationPopupState` 发布当前独占 popup；每个 open state 直接提供准确 child ViewModel，frontend 不建立第二份 route、content request 或 request-id authority。
- Frontend 直接渲染 popup child 并将 exact open handle 用于 dismiss；popup child 的 draft 与命令由 child ViewModel 持有。
- 按交互与生命周期相似性组织组件迁移批次，覆盖同类功能的全部宿主入口；同批迁移不意味着合并不同领域组件或抹平宿主语义。
- 会话重命名与删除分别采用 `app/component/session-rename`、`app/component/session-delete` 的 spec/ViewModel/View 边界。依赖端口绑定打开时的目标；Settings 的 revision 校验、Application 的标签清理、Catalog 的刷新与结果处理留在宿主适配器。
- 将焦点、hover、popup anchor、菜单、布局、滚动视口和 renderer 专用文案保留在对应 renderer source set。
- 将无 UI 框架依赖的实现测试随 ViewModel 维护，将 renderer 与交互测试放在对应 view 模块的 renderer test source set。
- 不为 Mosaic 和 Desktop 复制领域模块树，也不把完整 screen 强制建模为 `expect`/`actual`；两个 renderer 消费同一 contract 并分别提供根 view。
