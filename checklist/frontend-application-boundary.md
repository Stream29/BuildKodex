# Frontend 与共享应用层边界

- 将无 UI 框架依赖的 contract、ViewModel、state、effect 和 command 放在 `Kodex/app/contract/*` 与 `Kodex/app/viewmodel/*`。
- 让 contract 通过 `StateFlow`、结构化 state、effect 和 command 暴露 frontend API，不暴露 Mosaic、终端组件或 Compose Desktop UI 类型。
- Frontend 直接消费准确的 child ViewModel；父 ViewModel 只发布自身状态、父级关系和稳定 child handle，不为 renderer 镜像 child mutable state。
- ViewModel 实现通过 constructor injection 或 typed factory 获取 settings、models、authentication、repository 等依赖；frontend 和 Application contract 不充当 service locator。
- 将领域 view 放在 `Kodex/app/view/*` KMP 模块；renderer 差异分别进入 `mosaicMain` 与未来的 `desktopMain`，renderer 无关的展示逻辑才进入 `commonMain`。
- 领域 view 模块统一应用 `kodex.kmp-view`；不得在各模块重复声明 Mosaic target hierarchy 或平台 `dependsOn`。
- 只将 native entrypoint、Mosaic host 与 CLI 生命周期放在 `Kodex/app/cli`；未来 Desktop 对应内容直接放在 `Kodex/app/desktop`。
- CLI/Desktop host 负责 Ctrl+C、renderer 结束和 process disposal，并在 `finally` 调用 Application `shutdown()`；没有产品级 Exit/Quit 操作时不得在 Application contract 预设 `requestExit()` 或 lifecycle state。
- CLI 入口统一应用 `kodex.kmp-cli-executable`，由 convention 固定 Native executable、entrypoint 和稳定产物名。
- 让 view 模块单向依赖 contract；contract 与 ViewModel 模块不得依赖 `Kodex/app/view/*`、CLI 入口模块或未来的 Desktop 入口模块。
- Application 通过 framework-free `ApplicationPopupState` 发布当前独占 popup；每个 open state 直接提供准确 child ViewModel，frontend 不建立第二份 route、content request 或 request-id authority。
- Frontend 直接渲染 popup child 并将 exact open handle 用于 dismiss；popup child 的 draft 与命令由 child ViewModel 持有。
- 将焦点、hover、popup anchor、菜单、布局、滚动视口和 renderer 专用文案保留在对应 renderer source set。
- 将无 UI 框架依赖的实现测试随 ViewModel 维护，将 renderer 与交互测试放在对应 view 模块的 renderer test source set。
- 不为 Mosaic 和 Desktop 复制领域模块树，也不把完整 screen 强制建模为 `expect`/`actual`；两个 renderer 消费同一 contract 并分别提供根 view。
