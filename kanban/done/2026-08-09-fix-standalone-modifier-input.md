# Task Tree

- [done] 修复独立修饰键被输入为文本
  - [done] 复现并识别 Shift 功能键码
  - [done] 确定错误发生在 runtime 键事件投影
  - [done] 忽略独立修饰键功能事件
  - [done] 补充回归测试
  - [done] 记录修饰键投影约束
  - [done] 运行相关验证

# Details

- 用户报告单按 Shift 会输入 ``。
- 该字符是 U+E061（57441），对应 Kitty 键盘协议的 `LEFT_SHIFT`。
- TTY 解析结果正确；Mosaic runtime 将未命名的私用区功能键码误投影成了文本。
- 修复应位于 Mosaic runtime，不在 Kodex `TextInput` 层添加协议特例。
- 在 `KeyboardEvent.toKeyEventOrNull()` 中丢弃没有关联文本的 Kitty 独立修饰键码。
- 覆盖左右 Shift、其他修饰键边界和关联私用区文本优先级。
- Mosaic runtime Spotless 与 Linux X64 测试通过。
- Kodex CLI 组件 Linux X64 全量测试通过。
- Mosaic runtime JVM 测试受既有重复 `PlatformKt` 类名问题阻塞；测试未进入执行阶段。
- 各层 `git diff --check` 通过。
- 未检测到本项目正在运行的 IDE，因此未执行 IDE 检查。
- 修改范围与验证路径已确定。
