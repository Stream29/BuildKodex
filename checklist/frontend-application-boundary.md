# Frontend 与共享应用层边界

- 将 application factory、DI 装配、资源生命周期和无 UI 框架依赖的 ViewModel、state、effect、command 放在 `Kodex/app/shared/*`。
- 让共享层通过 `StateFlow`、结构化 state、effect 和 command 暴露 frontend API，不暴露 Mosaic、终端组件或 Compose Desktop UI 类型。
- 将 native entrypoint、Mosaic host、screen、terminal component、renderer 专用转换和 frontend-local 交互状态放在 `Kodex/app/cli/*`。
- 让 frontend 模块单向依赖共享模块；共享模块不得依赖 `Kodex/app/cli/*` 或未来的 `Kodex/app/desktop/*`。
- 将焦点、hover、popup anchor、菜单、布局、滚动视口和 renderer 专用文案保留在对应 frontend。
- 将无 UI 框架依赖的实现测试随共享模块维护，将 renderer 与交互测试随对应 frontend 维护。
- 后续 Desktop frontend 使用 `Kodex/app/desktop/*`，不得把 Desktop 界面实现放入共享层。
