# Task Tree

- [done] 统一Mosaic实现与构建的事实来源
  - [done] 修正Mosaic现有Unicode键盘实现和已知测试回归
  - [done] 让CodexLite通过Gradle composite build依赖Mosaic模块
  - [done] 删除`tui-mosaic-runtime`与`tui-mosaic-testing`源码拼装模块
  - [done] 删除已在Mosaic覆盖的重复底层测试
  - [done] 验证Mosaic自身测试和CodexLite TUI构建测试

# Details

Mosaic源码、底层测试和模块构建只在根目录`Mosaic/`维护。CodexLite只保留焦点、组件和业务TUI，通过Gradle composite build消费本地Mosaic fork。

Linux与macOS ARM64上的Mosaic runtime、TTY、CodexLite TUI测试和demo链接均已通过。普通任务图可以存储并复用configuration cache；包含`cklib`原生编译的任务图由任务显式声明不兼容，继续使用Gradle daemon和构建缓存。
