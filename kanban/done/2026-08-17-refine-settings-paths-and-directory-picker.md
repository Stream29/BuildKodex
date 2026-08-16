# Task Tree

- [done] 完善 Settings 路径与目录选择交互
  - [done] 删除 New Session 空状态提示
  - [done] 验证 Session Working directory 入口
  - [done] 为 Global Codex home 增加 Path Picker
  - [done] 区分管理区块标题与内容背景
  - [done] 恢复 Path Picker 顶部默认 Select
  - [done] 修复 Scroll to end 的 hover 加粗
  - [done] 补充 ViewModel 与渲染测试
  - [done] 更新对应交互决策
  - [done] 运行格式化和回归验证

# Details

- 状态：`done`。
- 用户已明确要求直接落地以下调整。
- 路线已确定：复用 Settings 已注入的 `DirectoryPickerViewModel` 工厂，由 Global 与 Session 子 ViewModel 分别持有自己的短生命周期 Picker；前端继续只负责渲染和回调。
- Session 入口与 Picker 状态链已存在，本轮补充回归覆盖，不建立第二套实现。
- Path Picker 恢复 Select 在列表上方、Cancel 在底部的专用浏览器布局。
- Global Picker 使用当前子 ViewModel 身份拒绝陈旧回调；确认时只对最新全局快照变换 `codexHome` 字段，并在选择、取消或 Settings 关闭时释放子 ViewModel。
- 验证至少覆盖 Settings ViewModel、Settings Mosaic、Path Picker Mosaic 与 Application Mosaic 模块，并构建 Linux release executable。
- IDEA 定向构建通过；检查未发现 error，剩余提示为既有未使用语义色与重复映射 warning。
- `:app-view-path-picker:linuxX64Test :app-viewmodel-settings:linuxX64Test :app-view-settings:linuxX64Test :app-view-application:linuxX64Test` 通过。
- `:app-cli:linkReleaseExecutableLinuxX64` 通过。
- 已在隔离配置的 100×30 终端中验证空白 New Session、两个 Settings 路径入口、管理标题背景层级及 Path Picker 的顶部默认 Select；Scroll to end hover 由 ANSI 渲染测试验证为 Bold。

## 1. New Session 空状态提示

- 删除 `Enter a prompt to create a session`，保留原有 History 空白区域。

### 用户审批

- 可以删除。

## 2. Settings 路径选择

- Session 的 Working directory 继续统一复用 Path Picker。
- Global 的 Codex home 接入同一 Path Picker，并在确认后持久化。

### 用户审批

- Working directory 与 Codex home 都应走 Path Picker。

## 3. Settings 区块标题层级

- MCP servers、Hooks 等管理区块的标题操作行使用区别于条目内容的背景角色。

### 用户审批

- 标题栏背景色应与设置内容不同。

## 4. Path Picker Select 语义回归

- Select 恢复到目录列表上方并成为弹窗默认焦点。
- Select 不与 Cancel、Delete 等底部对话框操作共用确认操作语义。
- 保持“输入过滤词 → Enter 进入首个匹配目录 → Enter 选择当前目录”的连续键盘流程。

### 用户审批

- Select 应恢复为默认选择并回到上方，保证过滤、进入、选择的连续 Enter 体验。

## 5. Scroll to end hover 状态

- Scroll to end 按钮应遵守统一的按钮状态规则，在 hover 时使用 Bold。

### 用户审批

- 修复当前 hover 不加粗的问题。
