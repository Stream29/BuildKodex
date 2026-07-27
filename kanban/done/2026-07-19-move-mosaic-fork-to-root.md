# Task Tree

- [done] 将Mosaic fork作为根目录工作对象
  - [done] 将Git submodule从`shared-context/mosaic`移动到`Mosaic`
  - [done] 更新CodexLite源码桥接的fork路径
  - [done] 验证submodule远端和Gradle配置

# Details

`Mosaic`是可直接修改并向上游提交补丁的fork，不再视为只读共享上下文。

验证：`Mosaic`保留`origin`与`upstream`远端，`tui-demo:jvmTest`通过且configuration cache已写入。
