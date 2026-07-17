# Task Tree

- [done] 让 `Shell` 保留模型指定的可执行文件路径
  - [done] 以 `ShellType` 和 executable 路径重建 `Shell` 模型
  - [done] 将模型字符串解析收口到 `Shell` serializer
  - [done] 让 `exec_command` DTO 直接使用 `Shell?`
  - [done] 覆盖默认值、名称、路径和无效输入的测试
  - [done] 验证受影响的平台 target

# Details

`shell` 对模型仍是字符串；反序列化后的 `Shell` 同时携带已识别的类型和实际将启动的路径。未提供 `shell` 时仍使用宿主默认 shell。

已通过 JVM、Node.js、Linux x64、macOS ARM64 测试，并编译 Linux ARM64 与 Mingw x64 源集。
