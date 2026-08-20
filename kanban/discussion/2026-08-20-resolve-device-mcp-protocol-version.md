# Task Tree

- 解决 DeviceAsMcp 协议版本不兼容
  - [done] 确认 OAuth 已完成
  - [done] 定位初始化失败响应
  - [done] 核实两侧实际协议版本
  - 确定协议兼容修改侧
  - 实现选定的兼容路径
  - 验证认证后的 MCP 初始化

# Details

- `device_as_mcp` 已持久化动态 client ID、resource、scope 和 access token。
- 当前唯一失败是服务端返回 `Unsupported legacy protocol version`。
- Kodex 项目基线固定为 MCP `2025-11-25`，现有清单明确不实现 `2026-07-28` 或新旧协议协商。
- DeviceAsMcp 服务端只接受 legacy `2025-06-18` 或其 modern `2026-07-28`。
- `2026-07-28` 已是官方稳定版本，不是 DeviceAsMcp 自定义版本；它与 2025 系列属于不同 wire era。
- Kodex SDK 以 `2025-11-25` 发起 initialize，但其支持列表也包含 `2025-06-18`。
- 线上探测确认 DeviceAsMcp 的 `server/discover` 发布 `2026-07-28`，legacy initialize 接受 `2025-06-18`，但拒绝 `2025-11-25`。
- 双方实际存在 `2025-06-18` 交集；直接故障是 DeviceAsMcp 未按 legacy initialize 版本协商规则回选 `2025-06-18`。
- 继续修改前需要确定由 DeviceAsMcp 接受标准基线，还是修改 Kodex 的既定协议策略。
