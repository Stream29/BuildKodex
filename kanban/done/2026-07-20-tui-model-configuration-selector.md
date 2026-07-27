# Task Tree

- [done] 将模型和推理强度选择器合并为一个模型配置入口
  - [done] 移除分离的模型与推理强度触发器
  - [done] 以`[<model> <reasoning>]`显示当前配置
  - [done] 保持模型到推理强度的级联选择与原子配置更新
  - [done] 覆盖鼠标、焦点、方向键、Enter和Esc交互测试

# Details

模型和推理强度不是独立正交的配置。状态栏应只呈现当前模型可用的推理强度。

已通过`:tui-demo:jvmTest`、`:tui-demo:linuxX64Test`和`:tui-demo:linkDebugExecutableLinuxX64`验证。
