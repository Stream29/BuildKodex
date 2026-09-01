# Task Tree

- [done] 修复 patch view Mosaic 交互测试
  - [done] 复现四个失败并收集实际渲染
  - [done] 验证动态 pressable 跨帧 capture
  - [done] 修复测试交互时序
  - [done] 复验 patch view JVM 与 Native

# Details

- `app-view-patch` 的 JVM 与 Linux Native 测试均有相同 4 个失败。
- 失败已在 clean HEAD 独立复现，不是 slim patch persistence 改动引入。
- 用户授权在修复简单时随本轮一起处理。
- 顶层 header 点击成功；第二个 `Changes` pressable 在 press 后保持折叠。
- 正文缺失导致结构化 diff、窄屏换行、ANSI 颜色和大 diff 分页四组断言连锁失败。
- 共享 `TuiPressable` 的静态非零纵向位置 press/release 测试通过。
- 新增动态插入按钮的跨帧 press/release 回归测试，JVM 与 Linux Native 均通过。
- 产品 pointer capture 正常；失败来自测试在新控件首帧后立即注入 Press，早于 hover composition 与 pointer tree 收敛。
- patch click helper 先发送 Motion 并等待，再保留跨帧 press/release，避免把测试框架过渡帧当成稳定 UI。
- `:app-view-components:jvmTest --no-build-cache` 通过。
- `:app-view-patch:jvmTest --no-build-cache` 通过，共 16 项。
- 两个模块的 `linuxX64Test --no-build-cache` 均通过。
- 两个模块的 `check` 均通过；macOS 与 MinGW 测试因当前 Linux host 跳过。
