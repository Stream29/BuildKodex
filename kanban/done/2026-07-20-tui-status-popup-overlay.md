# Task Tree

- [done] 将TUI状态选择菜单改为覆盖层
  - [done] 确认 Mosaic 的层叠、定位和指针命中语义
  - [done] 修正 demo 对 Codex CLI `client_version` 的复用
  - [done] 以原始状态按钮的实际坐标作为覆盖层锚点
  - [done] 将菜单项实现为可单独获取焦点的按钮组
  - [done] 覆盖锚点位置、方向键和鼠标交互行为测试
  - [done] 修复偏移覆盖层菜单按钮的 hover 命中
    - [done] 修正 Mosaic 布局偏移后的指针命中路径
    - [done] 覆盖偏移弹出菜单的 hover 加粗行为测试

# Details

- 覆盖层不应占用或压缩历史区域。
- Mosaic 的 `Box` 按子项声明顺序绘制，并以逆序命中指针；后声明的菜单可作为覆盖层。
- `client_version=0.1.0` 会使当前 `/models` 返回空列表，导致目录回退为当前模型。
- demo 现在复用 `models_cache.json` 的 client version，并保留正确的 OpenAI base URL。
- 先前实现错误地按终端底边定位菜单；应当以 `model` 或 `reasoning` 按钮的实际终端坐标为基准，向上展开。
- 已通过 JVM、Linux Native 测试及 Linux Native 真实 TTY 验证。
- 当前补充修复：菜单处于偏移覆盖层时，Mosaic 的布局层不能以未偏移坐标拦截 hover 命中。
- 已通过 Mosaic JVM 指针测试，以及 Kodex TUI 的 JVM、Linux Native 测试和 Linux Native demo 链接。
