# Task Tree

- [done] 修复虚拟 New session 的当前标签设置
  - [done] 扩展每个 New session 标签的工作目录草稿
  - [done] 让 Session 页编辑当前 New session 草稿
  - [done] 保持 New session 页编辑全局默认值
  - [done] 让 New 标签的 Settings 默认打开 Session 页
  - [done] 覆盖 ViewModel 与终端界面回归
  - [done] 更新约束并运行定向验证

# Details

- 用户确认：虚拟 New session 不提前创建真实 Session。
- `Settings > Session` 在虚拟标签下编辑名称、工作目录、模型、推理等级、服务等级和模式。
- `Settings > New session` 继续编辑未来标签使用的全局默认值。
- 在 `NewSessionViewState` 保存每个标签独立的工作目录，并在物化时复制到 root Agent settings。
- Session 页根据当前目标选择真实 Agent settings 或虚拟标签草稿，不改变全局 defaults 的真源。
- 定向验证覆盖草稿隔离、物化工作目录、设置页内容、默认路由及全局 defaults 边界。
- 修改范围限于 New session 共享 ViewModel、CLI application 设置界面、对应测试和全局设置约束。
- 未检测到打开本项目的 IDE 进程；仅检测到 JetBrains Toolbox。
- `:app-shared-new-session:linuxX64Test` 与 `:app-cli-application:linuxX64Test` 通过。
- 本次 Kodex 文件的 `git diff --check` 通过。
