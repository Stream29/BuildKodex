# Task Tree

- 重构 Settings 页面导航
  - [done] 盘点现有页面、分组与导航状态
  - [done] 确认页面划分、命名与设置归属
  - [done] 更新 Settings 页面契约与入口
    - 将页面定义为 General、OpenAI、MCP、Hooks、Current session、New session
    - 将现有 Session 初始入口迁移到 Current session
    - 保持三个设置 ViewModel 的所有权边界不变
  - [done] 按页面拆分 Global 内容
    - General 显示 Codex home、Sidebars 和 Input
    - OpenAI 显示 Authentication、OpenAI account 和 Codex usage
    - MCP 独立显示服务器管理入口
    - Hooks 独立显示 Hook 管理入口
  - [done] 补齐 OpenAI 账号管理
    - 为当前认证来源提供手动 Reload
    - 为 Kodex 私有来源提供 Sign in、Sign in again 和 Log out
    - 为 Codex 来源保留只读边界并说明由 Codex CLI 管理
    - 退出前确认并只删除 Kodex 私有凭据
    - 串行化账号操作并展示进行中与失败状态
    - 登录完成或取消后恢复原 OpenAI 设置页
  - [done] 将 Title generation 移入 New session
    - 保留 `KodexGlobalSettings.sessionTitle` 持久化与运行语义
    - 在 Model behavior 后显示 Title generation
    - 为标题模型与推理等级使用独立 dropdown state
    - 同时挂载新会话默认值与标题生成 dropdown menu
  - [done] 收紧页面级弹窗与生命周期
    - OpenAI 页面进入时刷新 usage，离开时关闭 reset
    - 只在 OpenAI 页面挂载 usage reset dialog
    - 离开 MCP 页面时关闭 Codex MCP import
    - 切页时继续关闭页面局部编辑与详情弹窗
  - [done] 更新测试与持久决策文档
    - 覆盖页面顺序、文案和设置项归属
    - 覆盖来源感知的账号操作与私有凭据删除
    - 覆盖 OpenAI usage 页面生命周期
    - 覆盖 New session 两组 dropdown 独立交互
    - 更新 Settings 路径与分组 checklist
  - 完成整体验收并收口前置任务
    - 使用新 release executable 一并复核现有 MD3 与 light/dark 结果
    - 完成被本任务承接的 Settings 与主题任务
  - [done] 运行相关检查与构建
    - 检查 IDEA 问题与 Git diff
    - 运行 Settings contract、ViewModel 和 View 测试
    - 运行 Application 测试与 Linux release 链接

# Details

- 状态：executable。实现、相关测试和Linux试用二进制已完成，等待用户试用复核。
- 用户确认的左侧顺序：
  - `General`
  - `OpenAI`
  - `MCP`
  - `Hooks`
  - `Current session`
  - `New session`
- Title generation 只移动 UI 入口，不进入 `KodexGlobalSettings.newSession`，也不改变新建或现有 Session 的继承语义。
- OpenAI 账号管理采用来源感知语义：
  - `kodex` 来源支持重新登录、Reload 和 Log out。
  - `codex` 来源只支持 Reload；登录和退出继续由 Codex CLI 管理。
  - 不调用 Codex CLI，不修改或删除外部 `auth.json`。
- Log out 需要危险操作确认，只删除 Kodex 私有 `auth.yml`；`authSource` 保持 `kodex`，成功后认证状态变为 credentials unavailable。
- 本任务承接现有 Settings MD3 与 light/dark 任务尚未完成的人工复核；不再单独运行旧 review executable。
- Sign in 或 Sign in again 暂时切换到登录 popup；完成或取消后必须恢复同一个 Settings owner 及其 OpenAI 页面。

## 修改设计

- 在 `Kodex/app/contract/settings/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/SettingsViewModel.kt:7` 将 `SettingsPage` 改为 `General`、`OpenAi`、`Mcp`、`Hooks`、`CurrentSession`、`NewSession`。
- 保留 `SettingsViewModel.global`、`session`、`newSession` 三个 child；页面数量与状态所有者数量不绑定。
- 在 `Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt:163` 继续按 enum 顺序渲染导航，并映射为用户确认的六个标签。
- 将 `SettingsPopup.kt:401` 的 Global 聚合内容拆为四个页面内容：
  - General：现有 General、Sidebars、Input。
  - OpenAI：现有 Account。
  - MCP：现有 `McpSettingsContent`。
  - Hooks：现有 `HookSettingsContent`。
- 将 `SettingsPopup.kt:503` 的 Session titles 内容拆为独立 Title generation section，并在 `SettingsPopup.kt:705` 的 New session 页面中放到 Model behavior 之后。
- New session 同时消费 `NewSessionSettingsViewModel` 和 `GlobalSettingsViewModel`；标题生成更新仍调用 global child。
- 扩展 `SettingsDropdownStates`，为 `Title model` 和 `Title reasoning` 增加独立 state，避免它们与 New session 的 `Model`、`Reasoning` 共用同一个 trigger state。
- 按页面拆分 `SettingsPopup.kt:769` 的 dropdown host：
  - General 只挂载输入键位菜单。
  - OpenAI 只挂载认证来源菜单。
  - Current session 挂载当前会话配置菜单。
  - New session 同时挂载默认配置菜单和标题生成菜单。
- 将 `Kodex/app/viewmodel/settings/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/SettingsViewModel.kt:32` 的 usage 可见性边界从 Global 改为 OpenAI。
- 将 `SettingsPopup.kt:133` 的 MCP import 清理边界改为 MCP，并将 `SettingsPopup.kt:213` 的 usage reset host 改为 OpenAI。
- 将应用内 `SettingsPage.Session` 初始入口更新为 `SettingsPage.CurrentSession`；不改变打开 Settings 时绑定当前目标的行为。

## OpenAI 账号管理

- 在 `Kodex/app/shared/auth/contract/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/KodexAuthStore.kt` 增加删除 Kodex 私有登录的 suspend command；OpenAI API 消费方继续只依赖只读 `OpenAiAuthStore`。
- 在 `Kodex/app/shared/auth/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/FileSystemKodexAuthStore.kt` 实现私有退出：
  - 与登录完成、reload 和 token refresh 串行化。
  - 取消尚未完成的私有登录 attempt，防止退出后旧 attempt 再次写回。
  - 删除私有 `auth.yml`，不访问 Codex `auth.json`。
  - 当前来源为 `kodex` 时发布 `CredentialsNotFound` 并触发 usage 快照清理。
  - 删除失败时保留原凭据与已认证状态，并向设置层返回失败。
- 让 Settings 的 global child 依赖应用侧 `KodexAuthStore`，继续只向 frontend 投影不含 token 的认证摘要。
- 在 `GlobalSettingsViewModel` 增加串行账号操作状态和 command：
  - `Reloading`：调用 auth store 的现有 `reload()`。
  - `SigningOut`：调用私有退出 command。
  - `Failed`：只携带稳定操作类型，由 view 映射固定文案；原始异常只写日志。
  - 操作进行时禁用其他账号操作，避免 reload、退出和重复请求竞态。
- 放宽现有 `requestLogin()` 的已认证限制，但只在 `authSource=kodex` 时发送 `OpenLogin`：
  - 私有来源不可用时显示 `Sign in`。
  - 私有来源已认证时显示 `Sign in again`。
  - 重新登录取消或失败时保留原私有凭据；只有登录成功才原子替换 `auth.yml`。
- OpenAI 页面的账号条目按来源和状态渲染：
  - `codex`：显示安全账号摘要或不可用原因、`Reload`，以及 `Managed by Codex CLI. Update credentials there, then reload, or select Kodex to sign in here.`。
  - `kodex` 未认证：显示不可用原因、`Sign in` 和 `Reload`。
  - `kodex` 已认证：显示安全账号摘要、`Sign in again`、`Reload` 和 `Log out`。
  - 操作状态使用 supporting/error 语义展示，不把原始异常或凭据带入 frontend。
- 不把三个账号操作压在同一个 `SettingsItem` trailing row：
  - 账号摘要与操作使用独立设置项或纵向操作行。
  - 在 60 列终端产生的约 38 列内容区内，每个操作必须保持可见、可聚焦和可触发。
  - 继续依赖右侧页面的有界垂直滚动处理增加的行数，不增加水平滚动。
- Reload 或 Log out 进行时同时禁用认证来源 dropdown；确认 Log out 时重新检查当前来源仍为 `kodex`。
- `Log out` 打开确认弹窗；使用 Cancel 作为安全默认操作，最终确认使用 danger 样式，并明确只删除 Kodex 私有凭据、不影响 Codex CLI。

## 登录返回与 popup 所有权

- 调整 application popup contract，使一个 Login handle 可以持有发起登录的确切 Settings handle 作为返回目标。
- `openLoginPopup` 必须接收并校验当前 Settings handle：
  - 只接受仍为当前 popup 的确切实例。
  - 安装 Login 时不关闭 Settings child。
  - Settings 的 selected page 和 global/session/new-session child 保持存活。
- 让 application popup handle 成为 Settings child 的唯一关闭 owner；Settings composable 暂时离开 composition 时不得自行关闭仍被 Login parent 保留的 ViewModel。
- 关闭 Login 时：
  - 只关闭 Login child。
  - 恢复原 Settings handle，而不是创建新的 Settings ViewModel。
  - OpenAI 页面立即消费登录完成后 auth store 发布的最新状态。
- Login 被其他 popup 替换、应用关闭或底层 Session target 被删除时，必须同时关闭 Login 与保留的 Settings child，避免 owner 泄漏。
- stale Login 或 Settings handle 的 dismiss/open 请求继续按引用身份拒绝，不能恢复已失效的 parent。
- 账号 reload 和退出使用 application owner scope 完成；Settings 关闭后不得留下仍会更新已关闭 presentation state 的任务。

## 文档

- 更新 `checklist/global-settings.md:4-17`：
  - Codex home、Sidebars 改为 General 路径。
  - 账号和 usage 改为 OpenAI 路径。
  - MCP 改为 MCP 路径。
  - Session 改为 Current session。
  - Title generation 改为 New session 路径，并明确其仍是全局真源。
  - 将原有“仅认证不可用时显示 Sign in”改为来源感知的 Sign in、Sign in again、Reload 和 Log out 规则。
  - 明确私有退出只删除 `auth.yml`，Codex `auth.json` 继续只读。
- 更新 `checklist/tui-interaction-components.md:71` 的 Settings 页面结构决定。
- 不改写历史 done task。
- 更新以下前置任务的状态说明，明确其剩余人工复核由本任务承接：
  - `kanban/executable/2026-08-10-adjust-settings-layout.md`
  - `kanban/executable/2026-08-21-establish-light-dark-tui-theme.md`

## 测试与验证

- 更新所有 `SettingsPage.Global`、`SettingsPage.Session` 引用，并按测试意图选择对应新页面。
- ViewModel 测试覆盖：
  - OpenAI 作为初始页时刷新 usage。
  - 切入 OpenAI 时刷新 usage。
  - 离开 OpenAI 时关闭 usage reset。
  - 其他 global-backed 页面不触发账号页面生命周期。
  - 已认证的 Kodex 来源允许请求重新登录，Codex 来源拒绝应用内登录。
  - Reload 与 Log out 不并发，失败状态不暴露敏感信息。
  - 登录 popup 保留并恢复同一个 Settings handle 及 OpenAI selected page。
  - popup 替换、应用关闭和 target 删除会完整关闭 Login 与保留的 Settings owner。
  - stale handle 不能恢复已经失效的 Settings。
- Auth store 测试覆盖：
  - 私有退出删除 `auth.yml` 并发布 `CredentialsNotFound`。
  - 私有退出不读取、修改或删除 Codex `auth.json`。
  - 删除失败时保留当前已认证状态。
  - 退出与未完成或刚完成的登录竞态不会重新发布旧凭据。
  - 重新登录取消或失败保留旧凭据，成功后原子替换。
- Mosaic 测试覆盖：
  - 六个导航标签与顺序。
  - General、OpenAI、MCP、Hooks 的内容归属。
  - Current session 文案。
  - New session 同时显示 Model behavior 与 Title generation。
  - 两组 model/reasoning dropdown 可独立打开和提交。
  - Codex 来源只显示 Reload 和外部管理说明。
  - Kodex 来源按认证状态显示 Sign in 或 Sign in again、Reload、Log out。
  - Log out 确认弹窗使用安全默认焦点和危险确认样式。
  - 约 38 列内容宽度下所有账号操作仍可见、可聚焦和可触发。
- 使用 IDEA 检查受影响 Kotlin 文件。
- 使用运行中 Gradle daemon 的 JVM 执行：
  - `:app-shared-auth-contract:linuxX64Test`
  - `:app-shared-auth-filesystem:linuxX64Test`
  - `:app-shared-auth-filesystem:jvmTest`
  - `:app-viewmodel-settings:linuxX64Test`
  - `:app-view-settings:linuxX64Test`
  - `:app-view-application:linuxX64Test`
  - `:app-cli:linkReleaseExecutableLinuxX64`
- 执行 `git diff --check`。
- 实施后由用户使用 release executable 复核六个页面的导航、内容归属和弹窗行为。

## 实施结果

- JVM测试通过：auth filesystem、Settings ViewModel、Application ViewModel、Settings view。
- Linux x64测试通过：auth filesystem、Settings ViewModel、Application ViewModel、Settings view。
- Linux release链接通过。
- IDEA MCP在验收时连接丢失，未取得IDE问题列表；Gradle JVM与Linux编译均通过。
- 试用产物：`Kodex/out/kodex-settings-navigation-review-linux-x64`。
