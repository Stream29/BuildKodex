# Task Tree

- [done] 修复Mosaic TUI的Unicode输入崩溃
  - [done] 建立可向上游提交的`Stream29/mosaic` fork与submodule remote
  - [done] 确认Mosaic键盘事件转换的上游边界与可复现接入方式
  - [done] 补丁化合法Unicode code point到文本键的转换
  - [done] 为Native测试桥接与本地runtime一致的Mosaic测试辅助源码
  - [done] 增加非ASCII与补充平面字符的回归测试
  - [done] 在Linux原生TUI目标验证输入路径

# Details

Mosaic 0.18.0在将非ASCII `KeyboardEvent`转换为布局键事件时抛出
`UnsupportedOperationException`。修复必须保留控制键和私有功能键映射，并让合法的
Unicode标量值作为普通文本输入传递给TUI。

验证：`tui-demo:jvmTest`、`tui-demo:linuxX64Test`和Linux原生可执行文件的PTY输入路径均通过。
