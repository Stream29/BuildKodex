# Task Tree

- [done] 将 Hooks Settings 管理交互对齐 MCP
  - [done] 确认 Kodex 自有配置与 Codex 导入边界
  - [done] 收敛 Hooks 主列表
    - [done] 每个 source 只渲染名称与简短状态按钮
    - [done] 保留全局启停、添加和一次性导入入口
  - [done] 新增 Hooks 详情弹窗
    - [done] 显示脱敏 source 摘要
    - [done] 承载启停、编辑和删除操作
  - [done] 对齐 Codex 导入选择流程
    - [done] 打开弹窗时立即加载 preview
    - [done] 默认选中新增与冲突 source
    - [done] 为导入列表增加有界滚动
  - [done] 覆盖界面与默认选择回归测试

# Details

- `KodexGlobalSettings.hooks` 已经是 Hooks 持久化和运行时唯一真源；常规加载与运行时不会读取 Codex。
- 用户确认保留显式的一次性 `Import from Codex`，导入后只运行 Kodex 持久化副本，不继承、不同步 Codex 文件。
- Hooks 主层应像 MCP 一样，每个 source 只显示一个紧凑按钮；详情和 source 级启停、编辑、删除放入详情弹窗。
- Hooks 导入应像 MCP 一样，点击入口后立即显示列表，不保留手动 `Preview` 二级入口；全部受支持项默认选中，同源冲突默认整体替换，不支持项不可选。
- Hooks 全局 enable/disable 是 source 之上的现有能力，继续保留在 section 级操作中。
- 实现复用现有 `HookManager`、preview token、原子 apply 和脱敏状态，不改变 Hook runtime 或持久化模型。
- 验证以 Settings Mosaic snapshot、默认 decision 单元测试和 `:app-view-settings:linuxX64Test` 为主。
- `:app-view-settings:compileTestKotlinLinuxX64` 与 `:app-view-settings:linuxX64Test` 均通过。
- `git diff --check` 通过。
- 验证期间 IntelliJ IDEA 已退出，因此 IDE reformat/inspection 接口不可用；项目未提供 `app-view-settings` 格式化任务。
