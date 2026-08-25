# Task Tree

- 修复突发终端文本前缀丢失
  - [done] 确认本机语音输入链路
  - [done] 定位终端事件丢弃根因
  - [done] 确定保序无损队列方案
  - [done] 增加突发文本回归测试
  - [done] 替换有损终端事件队列
  - [done] 验证 Mosaic 与 CLI

# Details

- `fcitx5-vinput` 一次提交整段识别结果，Foot 将 UTF-8 文本连续写入 PTY。
- `Mosaic/mosaic-tty-terminal/.../TtyTerminal.kt` 使用容量 64 的 `DROP_OLDEST` 队列。
- 超过队列容量的突发文本会丢弃前缀，只保留末尾。
- 修复必须保持所有终端事件的原始顺序；固定增大容量不能消除问题。
- 在 `TtyTerminalKeyboardTest` 写入超过 64 个不同字符，并用后续 resize 状态确认 parser 已处理完整突发输入。
- 修复前回归测试首个期望码点为 19968，实际为 20001，确认容量 64 的队列丢失前 33 个事件。
- 将终端事件 channel 改为 `UNLIMITED`；不能使用有界 suspending channel，因为 capability bootstrap 返回前还没有消费者。
- 更新 TUI 交互约束，要求终端事件队列不得丢弃键盘、粘贴或其他有序事件。
- `:mosaic-tty-terminal:jvmTest` 与 `:mosaic-tty-terminal:linuxX64Test` 通过。
- `:app-view-components:jvmTest`、`:app-view-components:linuxX64Test` 与 `:app-cli:linkReleaseExecutableLinuxX64` 通过。
- 三层仓库的 `git diff --check` 通过。
- `:mosaic-tty-terminal:spotlessKotlinCheck` 仅被未修改的 `EventParserCsiMouseEventTest.kt:102,110` 既有格式问题阻断；本次修改文件未被报告。
