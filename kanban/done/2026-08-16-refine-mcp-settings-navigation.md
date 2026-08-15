# Task Tree

- [done] 改进 MCP Settings 导航与滚动
  - [done] 让 Codex 导入一次点击直接显示服务器选择列表
  - [done] 让全部可导入项默认选中，同名冲突默认替换
  - [done] 将 Global Settings 中每个 MCP 收敛为一个详情入口按钮
  - [done] 将服务器详情和全部管理操作移入弹窗
  - [done] 为 Settings 内容区增加有界垂直滚动
  - [done] 覆盖主列表、导入默认选择和滚动回归测试

# Details

- 路线已确定：只调整 Settings 前端结构和调用时机，复用现有 `McpManager`、import preview、生命周期命令及脱敏状态。
- 主列表由 `McpSettingsContent` 渲染紧凑按钮，新增详情弹窗承载现有状态、详情和命令。
- 导入弹窗打开时立即调用现有 preview API，并以 preview item kind 构造默认决策。
- Settings 主弹窗固定终端内高度，以右侧 content column 作为滚动 viewport。
- 用户确认不保留手动 `Preview` 二级入口。
- `Import from Codex` 打开后立即加载列表；新增项默认 `Import`，同名冲突默认 `Replace`，不支持项保持不可选。
- Global Settings 主层只显示 section 级 Add/Import 和每个服务器的紧凑按钮；URL、command、headers 摘要以及启停、编辑、删除、OAuth、reconnect 都放入服务器详情弹窗。
- Settings 标题、左侧页面导航和底部 Close 保持固定，右侧页面内容独立滚动。
- IDEA 增量构建与错误级 inspections 通过。
- `:app-view-settings:linuxX64Test` 通过；Gradle 仍报告仓库已有的 configuration-cache 序列化问题并丢弃缓存。
- `:app-view-settings:jvmTest` 受 vendored Mosaic 的 `com.jakewharton.mosaic.PlatformKt` 重复 JVM class 阻断，失败发生在应用模块测试之前。
