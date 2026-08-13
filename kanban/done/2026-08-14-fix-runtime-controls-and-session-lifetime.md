# Task Tree

- [done] 修复运行时控件与 Agent 命令所有权
  - [done] 复现并定位两类回归
    - [done] 确认模型、模式与 cwd 控件退化为纯文本
    - [done] 确认旧 view 调用协程错误拥有运行中 turn
  - [done] 修复 Agent 命令所有权
    - [done] 将 resume/compact 与 initial turn 交给 Agent ViewModel 作用域
    - [done] 保留显式停止与关闭 Session 的取消语义
    - [done] 验证前端调用域取消不停止已接受的 turn
  - [done] 恢复底栏直接交互
    - [done] 将模型目录注入准确 Agent/New Session settings child
    - [done] 恢复模型、reasoning 与 service tier 三级选单
    - [done] 恢复 build/plan 模式选单
    - [done] 用 application popup child 恢复 cwd 目录选择按钮
    - [done] 保持控件只依赖子 ViewModel contract
  - [done] 补充回归测试
    - [done] 覆盖调用方取消后的 Agent 运行
    - [done] 覆盖显式 Stop 与 Session close
    - [done] 覆盖模型与模式选单交互
    - [done] 覆盖 cwd 按钮与目录选择
  - [done] 运行相关构建与真实 CLI 验证

# Details

- `RuntimeStatusBar.kt` 在整体重构中把模型、模式和 cwd 三个交互控件替换成了 `Text`，只保留了 Settings 按钮。
- `AgentRuntimeScreen` 通过 `rememberCoroutineScope()` 调用 `submitComposer()`；`KodexAgentRuntimeComposition.resume()` 把当前协程 `Job` 注册为 `runningTurn`。标签切换只更新 `selectedIndex`，原 Session/Agent ViewModel 始终存活，但旧 view 离开组合后会取消它的调用协程，因此错误地连带停止后台 turn。
- 日志已记录：Session 87 于 `17:50:35` 启动 turn，`17:50:42` 打开 Session 52 后，Session 87 的 response request 和 turn 随即被取消。
- 修复不通过继续组合隐藏页面实现。已接受的长时运行命令必须归始终存活的 Agent ViewModel scope 所有，只有显式 Stop、关闭 Agent 或关闭 Session 才能取消。
- 模型、模式和 cwd 继续作为当前子 ViewModel 的直接操作，不把模型、认证或 settings 数据重新挂回 Application 根 ViewModel。
- `AgentSettingsViewModel` 增加只读模型目录 Flow；它是模型选单所需的准确 child 依赖，不是 Application 级聚合。
- cwd 使用一个捕获 exact `AgentSettingsViewModel` target 的 Application popup handle，并直接携带短生命周期 `DirectoryPickerViewModel`；选择后由该 popup child 命令更新捕获目标。
- frontend-local dropdown、focus 与 popup anchor 继续留在 view；业务目标、模型目录和目录选择结果不由 frontend 迟绑定当前标签。
- JDK 26 验证通过：
  - 四个受影响 ViewModel 模块的 JVM tests。
  - `:app-view-application:linuxX64Test`。
  - `:app-cli:linkDebugExecutableLinuxX64`。
  - 真实 Linux CLI 在 PTY 中完成启动和首屏渲染，超时退出前无异常。
