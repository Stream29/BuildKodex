# Task Tree

- [done] 修复大写锁定键被输入为字符
  - [done] 复现并定位功能键投影
  - [done] 确定大写锁定键语义
  - [done] 修复 Mosaic 键事件投影
    - [done] 将 `57358` 投影为 `CapsLock`
    - [done] 保留关联文本优先级
  - [done] 补充运行时与输入框回归测试
    - [done] 验证运行时键名与关联文本
    - [done] 验证输入框不插入功能键码
  - [done] 记录功能键投影约束
  - [done] 运行相关验证
    - [done] 运行 Mosaic runtime 测试与格式检查
    - [done] 运行 Kodex CLI components 测试
    - [done] 运行 IDE 文件检查

# Details

- Kitty 将 `Caps Lock` 报告为私用区功能键码 `57358`。
- TTY 已将序列正确解析为 `KeyboardEvent`。
- Mosaic runtime 当前把未命名的 `57358` 回退为私用区文本 `U+E00E`，随后被 `TextInput` 当成单个 Unicode 标量插入。
- 修复应位于 Mosaic runtime，将该功能键投影为非文本语义键名；输入框不增加协议特例。
- 不扩展 `KeyEvent` 的修饰符模型；Caps Lock 状态位已由 `KeyboardEvent` 保存，但本问题仅涉及 Caps Lock 按键自身不能退化为文本。
- 修改范围限于 Mosaic runtime 投影测试、Kodex `TextInput` 集成测试和既有 TUI 决策说明。
- Mosaic runtime Spotless 与 Linux X64 测试通过，其中新增的 `CompatTest` 11 项全部通过。
- Kodex CLI components Linux X64 全量测试通过；新增的输入框测试随 `textInputTest` 8 项全部通过。
- Kodex CLI components JVM 测试受既有重复 `PlatformKt` 类名问题阻塞，测试未进入执行阶段。
- 三个变更 Kotlin 文件的 IntelliJ IDEA 检查均无问题。
- 根仓库、Kodex 与 Mosaic 的 `git diff --check` 均通过。
