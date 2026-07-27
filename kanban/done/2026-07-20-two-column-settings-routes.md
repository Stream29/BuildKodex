# Task Tree

- [done] 将 TUI Settings 页面改为两列路由布局
  - [done] 建立 Global、Session、New session 一级路由状态
  - [done] 将左侧路由导航与右侧设置内容拆分布局
  - [done] 保留 Global 设置的暂存、Apply 与 Cancel 语义
  - [done] 支持路由的键盘焦点、方向导航与鼠标选择
  - [done] 更新布局和交互测试
  - [done] 运行 TUI Demo 相关验证

# Details

当前只有 Global settings 具备真实设置模型。Session 与 New session 本轮只建立可切换的独立内容面，不虚构尚未定义的设置字段或保存协议。

验证通过：`:tui-demo:jvmTest`、`:tui-demo:linuxX64Test`、`:tui-demo:linkDebugExecutableLinuxX64`。真实 80×20 tmux 终端验证了 Global 到 Session 的方向键切页和两列渲染。
