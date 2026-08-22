# Task Tree

- [done] 将 live duration 与 history duration 分离展示
  - [done] 确认两种 duration 的状态归属和生命周期
  - [done] 在 composer/status bar 读取 active turn duration
  - [done] 将 live duration 移到输入框分割线左侧
  - [done] 将 live duration 显示精度改为秒
  - [done] 保持 history duration 作为历史虚拟 item
  - [done] 补充 view 与 duration 格式化测试
  - [done] 构建 release binary 并验证实际 session

# Details

- 用户已明确：live duration 与 history duration 不是同一个展示项。
- live duration 属于 active turn，显示在输入框分割线最左侧，舍入到秒。
- history duration 属于已结束的 history turn，继续显示在最后一个 stable item 后。
- 不改变 HistoryViewModel 对 active turn 的持有和 ticker；只改变 live duration 的
  presentation boundary。
- 预期涉及：
  - `app/view/history/.../AgentHistoryView.kt`
  - `app/view/application/...` 或 composer status bar 的对应布局组件
  - 现有 duration 格式化和 view 测试。
- 验收：
  - active turn 只在输入框分割线左侧显示秒级 live duration。
  - history turn 只在 history 虚拟 item 显示 history duration。
  - active turn 结束后 live duration 消失，history duration 接管。
- 验证：
  - `:app-viewmodel-history:jvmTest :app-view-history:jvmTest :app-view-application:jvmTest`：通过。
  - `:app-viewmodel-history:linuxX64Test :app-view-history:linuxX64Test :app-view-application:linuxX64Test`：通过。
  - `:app-cli:linkReleaseExecutableLinuxX64`：通过。
  - release binary：`Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`。
  - release binary SHA-256：`4165ff5925e5b5534ef821f8c59d4af14aeccc9315f0892adb8597ec1eff1139`。
  - 在 120×40 PTY 中实际创建 Session 128；运行中显示左侧 `Worked for 5s`，结束后 live duration 消失并显示 `---Worked for 6.424s---`。
