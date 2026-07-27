# Task Tree

- [done] 将模型和推理强度改为级联菜单
  - [done] 为弹层菜单补充子菜单锚点、右键展开和左键返回能力
  - [done] 为子菜单添加贴靠父项的定位策略，并验证叠层命中顺序
  - [done] 将模型与推理强度作为一个原子配置提交
  - [done] 将demo模型选择改为候选模型和推理子菜单流程
  - [done] 覆盖键盘、鼠标、取消和模型支持范围的测试
  - [done] 运行TUI组件与demo的相关测试及Linux链接验证

# Details

模型菜单先选择候选模型；只有在推理强度子菜单完成选择后才更新会话配置。只有一个可用推理强度时直接提交。子菜单使用父模型菜单项作为锚点，右方向键或 Enter 进入，左方向键或 Esc 返回父菜单，菜单外点击关闭整组。

验证通过：`:tui-components:jvmTest`、`:tui-demo:jvmTest`、`:tui-components:linuxX64Test`、`:tui-demo:linuxX64Test`、`:tui-demo:linkDebugExecutableLinuxX64`。
