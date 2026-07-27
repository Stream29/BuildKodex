# Task Tree

- [done] 建立`TUI:demo`模块
  - [done] 让设置脚本递归识别`TUI`模块并映射为`:tui-demo`
  - [done] 使用现有`kmp-host`约定建立适合Mosaic的source set层级
  - [done] 为JVM、Linux、macOS和Windows建立可直接在真实TTY启动的入口
  - [done] 建立最小Koin composition root与关闭生命周期
  - [done] 用最小Mosaic界面验证运行入口和依赖解析

# Details

模块只面向Mosaic实际支持的终端宿主目标。共享代码应位于共同的中间source set，
不得通过重复平台实现或新增第三个convention plugin解决目标差异。

入口必须作为可执行程序在终端启动；IDE和Gradle控制台不是交互TUI的验收环境。
Koin只承担对象组合，不能用阻塞方式构造需要suspend初始化的AgentState。
