# 本机Kodex桌面通知

- 2026-09-05配置于Linux主机；属于用户本机配置，不是产品内置功能。
- `~/.kodex/settings.yml`注册`kodex stop notify`与`kodex error notify`。
- 两者调用`~/.kodex/hooks/notify`，分别传入`stop`与`unhandled_error`参数；通知脚本不放入PATH，也不保留PATH内的兼容入口，避免干扰`kodex-cli`补全。
- Stop仅接受Kodex JSON，显示最后一条助手消息，成功后输出`{"action":"finish"}`。
- 异常Hook直接读取stdin纯文本，保留换行；空message显示缺省提示，不解析JSON。
- 通知名称为Kodex，正文区分“本轮已结束”与“未处理异常”；正文转义后交给桌面显示。
- 复用GJS Broker：`~/.local/libexec/kodex-notification-broker.js`。
- 用户服务`kodex-notification-broker.service`已启用；D-Bus名称为`io.github.stream29.Kodex.NotificationBroker`，桌面身份为`io.github.stream29.Kodex`。
- Broker不可用时回退`notify-send`，使用相同桌面身份。
- 旧Codex通知脚本、Broker服务、D-Bus入口、desktop文件及Codex通知Hook注册已移除，无兼容入口。
- 两种真实通知均已在桌面D-Bus调用中验证；旧Codex事件被拒绝。未通过屏幕观察确认气泡可见性。
- 0.4.2预备二进制使用本机配置副本在隔离Home启动/退出通过；真实Home未升级。
