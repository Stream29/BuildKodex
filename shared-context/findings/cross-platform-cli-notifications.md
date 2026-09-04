# 跨平台 CLI 通知

## 结论

- Kodex 不必把系统通知完全交给用户 Hook。
- 终端桌面通知协议不能作为跨平台主后端；实测的四个常用终端中只有 foot 能产生系统通知。
- OSC 99、OSC 9 和 OSC 777 适合作为已知支持终端及 SSH 场景的可选后端。
- BEL 适合作为最后一级注意力提示，但不等同于带文本的系统通知。
- 本地运行时若要求可靠的系统通知，需要 Linux、macOS、Windows 平台后端或随发布物提供的平台 helper。
- Hook 继续承担飞书、Slack、手机推送和用户自建通知服务等扩展场景。

## 现有基础

- Kodex 发布物是 Kotlin/Native CLI，不是 JVM Compose Desktop 应用。
- Mosaic 已探测 Kitty notification capability，并暴露终端窗口 focused 状态：
  - [`Terminal.kt:15-21`](../../Kodex/Mosaic/mosaic-terminal/src/commonMain/kotlin/com/jakewharton/mosaic/terminal/Terminal.kt#L15-L21)
  - [`Terminal.kt:86-93`](../../Kodex/Mosaic/mosaic-terminal/src/commonMain/kotlin/com/jakewharton/mosaic/terminal/Terminal.kt#L86-L93)
- CLI 已使用 OSC 0 更新终端标题：
  - [`TerminalTitle.kt:30-37`](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/TerminalTitle.kt#L30-L37)
- `Stop` Hook 会在自然完成和唯一待处理 `request_user_input` 时运行：
  - [`TurnHookRuntime.kt:76-103`](../../Kodex/agent-runtime/decorator/turn-hook/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/turnhook/TurnHookRuntime.kt#L76-L103)
- `Stop` Hook 可以要求 Agent 继续执行，因此它不是内建“真正完成”通知的可靠发射边界。

## 终端实测

- 测试日期：2026-09-04。
- Linux 桌面通知以 `dbus-monitor` 观察 `org.freedesktop.Notifications.Notify`；测试前已用直接 D-Bus 通知验证观测链路。
- macOS 以 Notification Center 数据库记录数做前后对照；测试前已用 AppleScript 通知验证观测链路。
- 每个终端均实际执行 OSC 99、OSC 9、OSC 777 和 BEL，并在发送之间留出观察间隔。

### foot 1.25.0

- OSC 99、OSC 9、OSC 777 都产生桌面通知。
- OSC 99 在关闭 `desktop-notifications.inhibit-when-focused` 后确认可用。
- 默认配置会在窗口聚焦时抑制桌面通知。
- 默认 BEL 只触发 bell，不产生桌面通知。

### GNOME Terminal 3.58.0 / VTE 0.84.0

- OSC 99、OSC 9、OSC 777 均未产生 D-Bus 桌面通知。
- BEL 未产生 D-Bus 桌面通知。

### Ptyxis 50.1 / VTE 0.84.0

- OSC 99、OSC 9、OSC 777 均未产生 D-Bus 桌面通知。
- BEL 未产生 D-Bus 桌面通知。
- 独立状态文件确认终端中的测试命令实际执行完成。

### macOS 26.5.1 Terminal 2.15

- OSC 99、OSC 9、OSC 777 均未产生 Notification Center 记录。
- BEL 未产生 Notification Center 记录。
- 独立状态文件确认四种序列均已实际发送。
- Terminal 的 BEL 可由用户配置为声音、闪烁、Dock badge 或后台 Dock bounce，但不是 Notification Center 通知。
- 当前 Basic profile 已关闭 audible bell。

## 终端通知边界

- Kitty 的 OSC 99 支持标题、正文、点击聚焦、更新和能力查询。
- Kitty、WezTerm 等终端也支持较简单的 OSC 9。
- 不支持桌面通知协议时，BEL 仍可交给终端决定声音、闪烁或任务栏提示。
- 终端代发能把远程 SSH 中运行的 Kodex 通知显示在本地桌面。
- 该路线在 foot 可行，但不能覆盖 GNOME Terminal、Ptyxis 和 macOS 系统 Terminal。
- OSC 输出需要处理控制字符、tmux passthrough，并与 Mosaic 渲染输出串行化。

## 建议语义

- 首批事件只包含 `input_required` 和 `turn_complete`。
- 默认在通知来源不可见时发送：终端窗口未聚焦，或事件来自当前未选中的会话或 Agent；并允许用户配置为始终通知。
- 通知事件应在一次 runtime operation 最终结束后产生。
- 取消、失败、Hook 要求继续、排队输入立即触发下一轮时，不应误报完成。
- 事件应是一次性信号，不能只观察可重放的 `StateFlow` 推断，否则重建 UI 时可能重复通知。
- 通知发送属于 CLI 展示层；Agent runtime 只提供准确的生命周期事件。
- 多会话通知应包含会话或 Agent 标识，并做同一等待点去重。

## 原生后端边界

- Linux 桌面通知依赖 session D-Bus 上的 `org.freedesktop.Notifications`，服务不保证存在。
- macOS 通知涉及授权和应用身份；即使 JVM Compose Desktop 路线也要求打包后才能可靠显示。
- Windows 对 unpackaged Win32 推荐 Windows App SDK 的 `AppNotificationManager`，同时引入 Windows App SDK runtime 部署。
- Kotlin/Native 可以通过平台库或 C interop 接入原生 API，但三端仍需独立实现、打包和测试。
- 实测否定了“仅靠终端协议覆盖常用终端”的前提；若产品目标是带文本的系统通知，原生后端不能再只视为可选增强。

## 参考

- [kitty desktop notification protocol](https://sw.kovidgoyal.net/kitty/desktop-notifications/)
- [WezTerm supported OSC notifications](https://wezterm.org/escape-sequences.html#operating-system-command-sequences)
- [Desktop Notifications Specification](https://specifications.freedesktop.org/notification/latest-single/)
- [Apple notification authorization](https://developer.apple.com/documentation/usernotifications/asking-permission-to-use-notifications)
- [Windows notifications overview](https://learn.microsoft.com/en-us/windows/apps/develop/notifications/)
- [Windows App SDK deployment for unpackaged apps](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/deploy-unpackaged-apps)
- [Kotlin/Native C interoperability](https://kotlinlang.org/docs/native-c-interop.html)
- [Compose Desktop tray notifications](https://kotlinlang.org/docs/multiplatform/compose-desktop-tray.html)
- [Apple Terminal bell settings](https://support.apple.com/guide/terminal/change-profiles-advanced-settings-trmladvn/mac)
