# Task Tree

- [done] 精简history与runtime配置交互
  - [done] 移除history row的hover背景
    - [done] 删除history renderer的hover颜色参数
    - [done] 保留完整row的focus与secondary action
  - [done] 将模式标签精简为build/plan
    - [done] 移除模式trigger的`mode`后缀
    - [done] 移除模式菜单项的`mode`后缀
  - [done] 合并model、reasoning与service tier选择器
    - [done] 删除独立tier trigger与dropdown state
    - [done] 建立model、reasoning、tier三级菜单
    - [done] 按model catalog限制可选tier
    - [done] 原子更新model、reasoning与tier
    - [done] default tier不追加到按钮标签
  - [done] 补充定向测试并构建release CLI
    - [done] 覆盖合并标签与三级选择
    - [done] 运行history与application Linux测试
    - [done] 重新链接Linux X64 release CLI

# Details

- History row继续保留focus背景，不显示hover背景。
- 模式按钮和菜单项只显示`build`或`plan`，不追加`mode`。
- Runtime模型菜单调整为model、reasoning、service tier三级菜单。
- Runtime状态栏只保留一个模型配置按钮；非default tier追加到标签，default不显示。
- Settings对话框中的持久化配置入口不在本次调整范围内。
- IntelliJ IDEA 2026.2正在打开本项目。
- 修改路径与验证方式已明确，任务可直接执行。
- `app-cli-history:linuxX64Test`与`app-cli-application:linuxX64Test`通过。
- `app-cli-application:linkReleaseExecutableLinuxX64`通过。
- IntelliJ inspection未发现本轮runtime selector问题；`git diff --check`通过。
