# Task Tree

- [done] 统一 Settings 条目配色并恢复认证分组
  - [done] 复现普通字段配色不一致
  - [done] 定位认证区块被标题设置分隔
  - [done] 为普通设置字段建立单一中性背景
  - [done] 保留标题、菜单和操作栏颜色层级
  - [done] 连续排列认证来源、账号状态和用量
  - [done] 将标题生成设置移到认证区块之后
  - [done] 补充渲染与顺序测试
  - [done] 更新 TUI 交互决策
  - [done] 运行格式化和回归验证

# Details

- 状态：`done`。
- 用户已明确要求直接落地。
- 修改前普通设置字段分别使用 `surface`、`surfaceContainerHighest`、`surfaceContainerHigh`、`secondaryContainer` 和 `tertiaryContainer`，没有统一的字段颜色角色。
- 修改前 Global Settings 把标题生成字段放在认证来源与账号状态、Codex usage 之间，破坏了认证信息的连续分组。
- `SettingsDropdownField` 现在自行消费唯一的字段背景角色，调用方不能再按字段传入任意容器色。
- Global Settings 现在先完整显示 Authentication、OpenAI account 与 Codex usage，再显示三项标题生成字段。
- ANSI 渲染测试覆盖普通字段只使用中性 surface；分组测试覆盖标题生成区位于完整认证区之后。
- IDEA 格式化、定向构建和检查通过；只报告既有的弱重复代码提示。
- `:app-view-settings:linuxX64Test` 通过。

## 1. 普通设置条目配色

- 所有普通设置选择字段使用同一个中性背景角色。
- 章节标题、下拉菜单和主要操作栏继续使用各自独立的颜色角色。

### 用户审批

- Settings 菜单条目应使用统一的配色方案。

## 2. 认证与标题生成分组

- 认证来源、OpenAI 账号状态和 Codex usage 连续排列。
- 标题生成设置整体移动到认证区块之后。

### 用户审批

- Authentication 不应被 title generation 设置截断。
