# Task Tree

- [done] 讨论 Kodex 跨平台系统通知方案
  - [done] 调查现有 Hook 与应用生命周期边界
  - [done] 调查 Kotlin/JVM 与 Native 平台能力
  - [done] 比较内建通知与用户 Hook 方案
  - [done] 形成推荐路线与明确边界
  - [done] 实测终端原生通知控制序列
    - [done] 测试本机 foot
    - [done] 测试本机 GNOME Terminal
    - [done] 测试本机 Ptyxis
    - [done] 测试 MacBook 系统 Terminal
    - [done] 根据实测修订推荐路线
  - [done] 用户决定放弃方向并归档

# Details

- 目标：判断 Kodex 能否直接提供跨平台系统通知，还是只能依赖用户自行配置 Hook。
- 当前阶段只做讨论与调研，不实施代码。
- 用户要求实际测试 foot、GNOME Terminal、Ptyxis 和 macOS 系统 Terminal 的终端通知能力。
- 当前未发现 IntelliJ IDEA、VS Code 或 Fleet 正在打开项目；仅 JetBrains Toolbox 在运行。
- 初始建议是内建 best-effort 终端通知，自动按 Kitty OSC 99、已知终端 OSC 9、BEL 降级。
- 建议首批事件为 `input_required` 和 `turn_complete`；默认在终端未聚焦或来源不是当前选中会话或 Agent 时发送。
- Hook 保留为飞书、Slack、手机推送和用户自建通知服务等扩展机制。
- 暂不建议维护三套原生系统通知后端；Linux D-Bus、macOS 授权与应用身份、Windows App SDK 部署的差异和成本较高。
- 不应从可重放状态直接推断一次性通知，也不应在 `Stop` Hook 运行时发送完成通知；最终发射点需要位于 runtime operation 确认结束之后。
- 实测环境：foot 1.25.0、GNOME Terminal 3.58.0 / VTE 0.84.0、Ptyxis 50.1 / VTE 0.84.0、macOS 26.5.1 Terminal 2.15。
- foot 的 OSC 99、OSC 9、OSC 777 均成功产生桌面通知；默认聚焦抑制生效，BEL 默认不产生桌面通知。
- GNOME Terminal 和 Ptyxis 的 OSC 99、OSC 9、OSC 777、BEL 均未产生桌面通知。
- macOS 系统 Terminal 的 OSC 99、OSC 9、OSC 777、BEL 均未产生 Notification Center 记录；BEL 只能按 profile 配置为声音、闪烁、Dock badge 或 bounce。
- 修订结论：终端协议只能作为已知终端或 SSH 场景的可选后端，不能作为跨平台系统通知主方案。
- 若目标是本地带文本的可靠系统通知，需要平台后端或随发布物提供的平台 helper；Hook 仍作为自定义渠道。
- 最终状态：用户决定放弃该方向，不进入 planning 或 implementation。
- 持久化调研结论：[`shared-context/findings/cross-platform-cli-notifications.md`](../../shared-context/findings/cross-platform-cli-notifications.md)。
