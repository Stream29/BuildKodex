# Task Tree

- [done] 修复 tmux 下 Home/End 按键解析
  - [done] 在 Mosaic 中补充失败回归测试
    - [done] 覆盖 `CSI 1 ~` 到 Home 的解析
    - [done] 覆盖 `CSI 4 ~` 到 End 的解析
  - [done] 扩展 CSI 波浪号按键映射
    - [done] 将参数 `1` 兼容为 Home
    - [done] 将参数 `4` 兼容为 End
    - [done] 保留现有 `H`/`F` 与 `7 ~`/`8 ~` 变体
  - [done] 验证 Mosaic 解析器
    - [done] 运行目标文件格式检查并记录既有阻断
    - [done] 检查两个变更文件的 IDE 问题
    - [done] 运行 JVM 解析器测试
    - [done] 运行 Linux X64 Native 解析器测试
  - [done] 验证 Kodex 实际输入链路
    - [done] 重建 Linux X64 release CLI
    - [done] 在隔离 tmux 会话中验证 Home
    - [done] 在隔离 tmux 会话中验证 End
    - [done] 清理临时会话和数据目录

# Details

## 已确认事实

- 已完成源码实施与验证。
- 已在 `foot → tmux 3.6 → Kodex` 环境复现。
- tmux 向应用发送 Home=`CSI 1 ~`、End=`CSI 4 ~`。
- 修复前 Mosaic 将两者解析为 `UnknownEvent`。
- Shift/Ctrl 组合由 tmux 编码为 `CSI 1;<modifier> H/F`，现有解析器已支持。
- `TextInput` 已正确处理成功投影后的 Home/End 事件。
- Mosaic 上游 `trunk` 同样没有参数 `1` 和 `4` 的映射，不能通过同步现有上游修复解决。

## 实施结果

- `EventParser` 现在将 CSI 波浪号参数 `1`、`4` 分别投影为 Home、End。
- 新增 `homeTildeOne` 与 `endTildeFour` 回归测试。
- 修复前新增测试分别收到 `UnknownEvent(1b5b317e)` 与 `UnknownEvent(1b5b347e)`；修复后通过。

## 修改范围

- 修改 `Kodex/Mosaic/mosaic-tty-terminal/src/commonMain/kotlin/com/jakewharton/mosaic/tty/terminal/EventParser.kt`。
- 修改 `Kodex/Mosaic/mosaic-tty-terminal/src/commonTest/kotlin/com/jakewharton/mosaic/tty/terminal/EventParserCsiKeyboardEventTest.kt`。
- 不修改 Kodex `TextInput`、按键语义或 tmux 配置。
- 不扩展未观测到的其他终端序列。

## 测试设计

- 回归测试直接使用已捕获的 `CSI 1 ~` 和 `CSI 4 ~` 字节。
- 修复前确认新增测试因返回 `UnknownEvent` 而失败，修复后确认返回对应的 `KeyboardEvent`。
- 现有 `CSI H/F` 测试继续保护原有路径。
- 现有映射中的 `CSI 7 ~/8 ~` 保持不变。

## 端到端验收

- 使用当前 Gradle daemon 的 JVM 运行 Gradle。
- 通过 Mosaic 的 `:mosaic-tty-terminal:jvmTest` 与 `:mosaic-tty-terminal:linuxX64Test`。
- 通过 Kodex 的 `:app-cli:linkReleaseExecutableLinuxX64`。
- 在临时 HOME 和临时 tmux 会话中运行新 CLI：
  - `abcdef` 中间位置按 Home 后输入 `X`，结果为 `Xabcdef`。
  - `abcdef` 中间位置按 End 后输入 `X`，结果为 `abcdefX`。
- 不复用或操作用户当前运行中的 Kodex 会话。
- 现有并行改动不在本任务范围内；若其阻断完整 CLI 构建，仅记录阻断，不修改相关文件。

## 实际验证记录

- 在 `Kodex/Mosaic` 运行 `:mosaic-tty-terminal:jvmTest`：通过。
- 在 `Kodex/Mosaic` 运行 `:mosaic-tty-terminal:linuxX64Test`：通过。
- 在 `Kodex/Mosaic` 运行 `:mosaic-tty-terminal:spotlessKotlinCheck`：被既有格式问题阻断，问题在 `mosaic-tty-terminal/src/commonTest/kotlin/com/jakewharton/mosaic/tty/terminal/EventParserCsiMouseEventTest.kt:102-110`，未修改无关文件。
- IDEA 检查：测试文件无问题；`EventParser.kt` 仅报告已有警告，未新增本次变更相关问题。
- 在 `Kodex` 运行 `:app-cli:linkReleaseExecutableLinuxX64`：通过；配置缓存因已知 888 个问题被丢弃，但任务成功完成。
- 隔离 tmux smoke test：Home 后得到 `> Xabcdef`，End 后得到 `> abcdefX`。

## 提交边界

- `Mosaic/` 是可直接修改的嵌套 fork。
- 不创建 Git commit；提交与父仓库 submodule 指针更新由用户完成。
