# Task Tree

- 完成 Settings 的 MD3 语义层级与主题接入
  - [done] 盘点现有布局、主题和渲染缺口
  - [done] 以 MD3 Settings 模式复查现有方案
  - [done] 完成用户审批并收敛实施边界
    - [done] 审批 Settings 文字与组件语义
    - [done] 审批 light/dark TUI 配色范围
    - [done] 审批页面分组与设置项结构
    - [done] 审批布尔控件与依赖状态
    - [done] 审批弹窗表面与操作样式
  - [建立 light/dark TUI 主题基建](../executable/2026-08-21-establish-light-dark-tui-theme.md)
  - [done] 根据审批修订 TUI 设计决策
    - 更新 Settings 文字、分组和表面规则
    - 将布尔设置规定为 Checkbox，其他互斥值继续使用 dropdown
    - 更新 Automatic session title 的依赖禁用规则
    - 保留获批的终端交互和窄终端偏差
  - [done] 建立 Settings 语义组件边界
    - 区分 dialog headline、section heading 和 item label
    - 抽取使用中性色标题行的 `SettingsSection`
    - 抽取带 headline、supporting、trailing 和 enabled 槽位的 `SettingsItem`
    - 统一正文、状态、控件标签和错误信息入口
  - [done] 迁移 Settings 主页面
    - 为 Global、Session 和 New session 建立获批分组
    - 将路径、Session name 和选择字段迁移为设置项
    - 将 MCP 与 Hook 名称和状态拆入独立槽位
    - 按获批方案拆分 New line key 与 Submit key
  - [done] 实现获批的 Checkbox 与依赖状态
    - 新增直接基于 Pressable 的共享 `TuiCheckbox`
    - 使用 `[ ]`、`[x]` 和整行可触发语义
    - 不预建未被当前需求使用的 Toggle/Switch 抽象
    - 迁移 Automatic session title 与 MCP OAuth
    - 让其他非布尔选择继续使用 dropdown
    - 禁用并解释不可用的标题模型和推理设置
  - [done] 迁移 Settings 子弹窗
    - 使用中性 dialog container 和统一 headline
    - 区分正文、supporting、error 和 action label
    - 覆盖登录、重命名、MCP、Hooks 和 Codex usage 弹窗
  - [done] 补充语义样式回归测试
    - 验证自定义 typography 和颜色到达每个语义槽位
    - 验证 Settings 在 light/dark scheme 下消费准确角色
    - 验证分组、设置项槽位、Checkbox 和 disabled 依赖
    - 验证 True Color 与 ANSI 16 渲染仍可区分层级
  - [done] 运行 `git diff --check`、模块测试和 Linux release 构建
  - [done] 打包 release executable 供用户运行复核
  - [done] 修正用户复核发现的 popup 表面边界
    - [done] 保留普通 Settings 内容和应用 shell 的原生背景
    - [done] 让 Settings、dialog、目录选择器和 popup menu 使用 `surfaceContainer`
    - [done] 让 popup menu 未显式传色时回退到 `surfaceContainer`
    - [done] 按用户意见不调整已确认可接受的文字前景色
    - [done] 重新打包 release executable 供用户复核

# Details

- 状态：`executable`。用户已授权开始实现，完成 release executable 后交由用户运行复核。
- Light/dark 配色属于共享主题基建，已拆为独立 executable 子任务；父任务在实现 Settings 主题接入前依赖该子任务完成。
- 应用根部按终端主题接入 light/dark scheme；普通 Settings 内容、应用 shell 和 History 大面积背景使用 `Color.Unspecified`，交给终端原生背景，不查询背景色；Settings、sidebar、目录选择器、dialog 和 popup menu 的容器使用 `surfaceContainer`，保留与原生背景的边界。
- 任务创建后的独立工作已经完成有界页面滚动、同行下拉字段和管理操作标题行；用户也已决定不增加极窄终端 compact 单栏。
- 本轮 MD3 复查确认，剩余问题不只是给现有文字套用 `title` 和重新接线前景色；当前任务的文字分类、表面方案、布尔控件和部分设置项结构需要先修正。

## MD3 依据

- Settings 应使用 list 或 list-detail 结构，以 primary label 表示设置名称，以 supporting text 表示状态或简要说明；相关设置应分组，而不是给每个单独设置项建立独立 container：[Settings 指南](https://developer.android.com/design/ui/mobile/guides/patterns/settings?hl=en)。
- Material 3 `ListItem` 的 item label 使用 `BodyLarge`，supporting text 使用 `BodyMedium` 和 `OnSurfaceVariant`；设置项标签不等同于 section title：[AndroidX ListTokens](https://android.googlesource.com/platform/frameworks/support/+/2fd71467c295a57331c2c23e66e3574230d209d3/compose/material3/material3/src/commonMain/kotlin/androidx/compose/material3/tokens/ListTokens.kt)。
- 明确的开关值应使用 switch 或 checkbox；父设置关闭时，其依赖项应紧随其后并显示 disabled 原因：[Settings 选择与依赖模式](https://developer.android.com/design/ui/mobile/guides/patterns/settings?hl=en#patterns-and-components)。
- Material 3 使用成对语义色、light/dark scheme 和中性 tonal surface；AlertDialog container 使用 `SurfaceContainerHigh`：[Android color](https://developer.android.com/design/ui/mobile/guides/styles/color?hl=en)、[AndroidX AlertDialog token 变更](https://android.googlesource.com/platform/frameworks/support/+/efd4966fd5d2a6c8acd162c60a8a1871545a7508)。
- TUI 只转译 MD3 的层级、语义、状态和 containment；不机械复制 dp、圆角、阴影、触摸尺寸、动画或终端无法表达的字号级别。

## 已确认缺口

- `TuiTypography` 已声明五个角色，但没有规定各角色的完整用途；`body` 和 `label` 仍为 `TextStyle.Unspecified`：`Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TuiTheme.kt:42-50`、`:76-83`。
- Settings 只给顶栏使用 `headline`。`Session name` 和 `Model` 仍是普通 `Text`，但它们属于 item label，而不是 section title：`Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt:583-594`、`:727-741`。
- 当前没有真正的页面分组标题；`Codex home`、`Working directory`、`MCP servers` 和 `Hooks` 使用 section 背景，实际混合了单个设置项和设置组两种语义。
- Settings 子弹窗仍直接使用 `TextStyle.Bold` 和 `TextStyle.Dim`，绕过主题角色；主题替换无法完整影响 Settings。
- Settings 全部表面共用 `SettingsForeground = onSurface`。header 和 action 使用 `primary` 背景时没有消费对应的 `onPrimary`：`Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt:1079-1112`。
- 默认 `onSurface` 和 `onPrimary` 都是白色，当前视觉结果掩盖了上述错误；自定义主题会暴露配对不正确的问题。
- 当前 scheme 缺少 `onSurfaceVariant`，只有固定 dark palette；Mosaic 已发布 `Light`、`Dark` 和 `Unknown` 终端主题，但应用根部没有据此选择 TUI scheme：`Kodex/Mosaic/mosaic-terminal/src/commonMain/kotlin/com/jakewharton/mosaic/terminal/Terminal.kt:153-162`、`Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt:113`。
- Automatic session title 和 MCP OAuth 是布尔值，但仍使用 dropdown：`Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt:496-509`、`Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/McpSettingsDialogs.kt:189-249`。
- Automatic session title 关闭后，Title model 和 Title reasoning 仍保持可操作，也没有说明不可用原因。
- New line key 和 Submit key 是两个设置项，但被压进同一行：`Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt:453-466`。
- MCP 与 Hook 把名称和状态拼成同一个按钮字符串，没有独立 headline、supporting 或 trailing 槽位：`Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/McpSettingsContent.kt:52-59`、`Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/HookSettingsContent.kt:38-44`。
- 主 Settings 和子弹窗使用整行 `primary` header/action；部分错误信息仍使用 `onSurface + Bold`，没有消费 error 角色：`Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/McpSettingsDialogs.kt:220-230`。
- 现有 Settings 测试只验证字段背景和文字排列，没有验证 title、body、supporting 或前景色角色：`Kodex/app/view/settings/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/SettingsDropdownFieldTest.kt:21-82`。

## 当前配色核对

- 以下结果使用当前默认 RGB 按 WCAG relative luminance 计算，适用于 True Color 输出：
  - white / primary：`10.63:1`。
  - primary / surface：`1.17:1`。
  - primary / surfaceContainerHigh：`1.06:1`。
  - outline / surface：`1.98:1`。
  - error red / surface：`3.10:1`。
- 普通文字目标至少为 `4.5:1`，表面与非文字元素至少为 `3:1`：[Android Accessibility](https://developer.android.com/design/ui/mobile/guides/foundations/accessibility?hl=en)。
- 当前 primary 适合作为深色条背景，但不能直接作为深色 surface 上的 MD3 action foreground；实施前必须先重建 palette，不能只替换调用方角色。

## 已批准文字与组件语义

- `dialogHeadline`
  - 用于 Settings 和独立子弹窗标题。
  - 映射 `TuiTypography.headline`。
- `sectionHeading`
  - 只用于页面内真实设置组标题。
  - 映射 `TuiTypography.title`。
- `itemLabel`
  - 用于 `Model`、`Session name`、`Working directory` 和 `Authentication` 等设置名称。
  - 作为 Settings 组件语义，默认使用 `body + Bold`，不占用 section `title`。
- `itemValue`
  - 用于模型值、路径、账号身份和普通详情。
  - 映射 `body/onSurface`。
- `supporting`
  - 用于状态、解释、元数据、空状态和进度。
  - 映射 `supporting/onSurfaceVariant`。
- `controlLabel`
  - 用于按钮、导航项、dropdown trigger 和 dialog action。
  - 映射 `label`，交互组件继续叠加 hover、press、selected 和 disabled 样式。
- `SettingsSection`
  - 负责分组标题和组间 containment。
  - 标题整行使用 `surfaceContainerHigh/onSurface` 和 `title`；组内设置项继续使用 `surface`。
  - 不把每个单独设置项绘制为 section header。
- `SettingsItem`
  - 提供 headline、supporting、trailing 和 enabled 槽位。
  - 路径、选择字段、Session name、账号状态、MCP 和 Hook 条目复用该结构。
- `TuiCheckbox`
  - unchecked 使用 `[ ]`，checked 使用 `[x]`，只依靠 marker 表达 checked，不复用导航 selected 的持续反显。
  - marker 和 label 组成同一个 Pressable；Enter、Space 和主鼠标点击切换值。
  - Focus、hover、press 和 disabled 继续使用现有物理光标、Bold、Invert + Bold 和 Dim 规则。
- 不增加 `displayLarge`、`titleMedium` 等无法在终端表达差异的字号型 token。

## 已批准 Settings 颜色与表面

- Light/dark palette、`onSurfaceVariant`、终端主题选择和透明 History 由[独立主题子任务](../executable/2026-08-21-establish-light-dark-tui-theme.md)负责。
- Settings 只消费子任务提供的语义角色，不自行维护 RGB、scheme 选择或 theme fallback。
- Settings dialog 使用中性 tonal surface；header 和 action row 不再使用整行 primary 填充。
- 真实 section heading 使用 `surfaceContainerHigh/onSurface`，不将该表面扩散到单个 item label。
- 普通内容与设置项使用原生背景 `Color.Unspecified`，文字继续使用对应的 `onSurface` 角色。
- Supporting content 使用 `onSurfaceVariant`，并可叠加 `Dim`。
- 页面导航和 popup menu 使用中性 container role。
- 普通 action 在中性表面上使用可读的 primary；危险 action 和错误信息使用可读的 error。
- 所有实际使用的 foreground/container 配对必须同时通过角色测试和对比度测试。

## 已批准页面分组

- Global
  - `General`：Codex home。
  - `Integrations`：MCP servers、Hooks。
  - `Account`：Authentication、OpenAI account、Codex usage。
  - `Session titles`：Automatic session title、Title model、Title reasoning。
  - `Input`：New line key、Submit key。
- Session
  - `Identity and location`：Session name、Working directory。
  - `Model and behavior`：Model、Reasoning、Service tier、Agent mode、Questions。
- New session
  - `Model and behavior`：Model、Reasoning、Service tier、Agent mode、Questions。
- 保留现有设置的相对顺序；允许插入分组标题，并将 New line key 与 Submit key 拆为各自一行。

## 已批准 Supporting 文案

- 本轮只增加以下最小必要集合，不为 Service tier 或其他自解释字段增加说明。
- Session 与 New session 的 `Questions`：`Controls whether the agent may pause to ask for input.`
- Session 与 New session 的 `Agent mode`：`Controls whether the agent may delegate work to sub-agents.`
- Global 的 `Title reasoning`：`Reasoning effort used to generate automatic session titles.`
- Automatic session title 关闭时，Title model 与 Title reasoning 使用：`Available when automatic session titles are enabled.`
- Title reasoning disabled 时以依赖说明替换普通 supporting text，不同时显示两条说明。

## 保留边界

- 保留 Settings 最大宽高、双栏导航、固定标题与操作区、右侧有界滚动和当前 popup 模式。
- 保留用户已批准的 Focus、hover、press、selected 和 disabled 终端状态矩阵。
- 不新增极窄终端 compact 单栏；继续接受内容区在不合理终端尺寸下退化。
- 不修改 Settings ViewModel、持久化真源或字段提交语义。
- 保留 MCP 和 Hooks 在 Global 主层的 section 操作与紧凑条目，不在本任务改为 subscreen：`checklist/mcp-management.md:49-50`。
- 保留 dialog action 的尾端排列、dismissive/confirming 顺序和危险操作安全默认焦点。

## 审批结论

- A 已批准：Settings 使用独立 `itemLabel`；真实分组标题才使用 `title`。完成重构并打包后由用户运行复核。
- B 已批准拆分：Light/dark TUI 配色作为[独立主题子任务](../executable/2026-08-21-establish-light-dark-tui-theme.md)单独 executable；后续明确使用 `#1C444A`、SchemeTonalSpot 和 contrast level `0.0`。
- C 已批准：增加真实页面分组、拆分快捷键设置，并拆分 MCP 与 Hook 的名称和状态槽位；后续明确使用 `surfaceContainerHigh` 中性色标题行。
- D 按意见修订后批准：所有 true/false 设置使用真实 `TuiCheckbox`；其他选择继续使用 dropdown；不提前建立未使用的 Toggle/Switch 抽象。Automatic session title 关闭后仍需禁用并解释其依赖项。
- E 已批准：Settings 和子弹窗使用中性 dialog surface，普通 action 使用 primary，错误与危险动作使用 error。
- 后续文案澄清已批准：只增加依赖说明、Questions、Agent mode 和 Title reasoning 的最小 supporting text 集合。

## 审批记录

### A. 文字角色与 SettingsItem

- 推荐：`Model`、`Session name` 等使用独立 `itemLabel`，默认 `body + Bold`；只有真实分组标题使用 `title`。
- 影响：替换原计划中“所有字段标题使用 title”的分类，并新增 `SettingsSection`、`SettingsItem` 组件边界。

> 用户审批意见：
>
看着没什么问题。你做好重构以后打包我运行复核一下就可以。

### B. Light/dark TUI 配色

- 推荐：在本任务内重建可访问的 light/dark 静态 scheme，增加 `onSurfaceVariant`，并由终端主题选择；History 继续透明。
- 影响：范围会覆盖 TUI theme 和应用根部，不再是 Settings view 内部的局部换色。

> 用户审批意见：
>
这个是theme层面的基建，需要做成独立子任务并单独plan

### C. 页面分组与设置项结构

- 推荐：加入 General、Integrations、Account、Session titles、Input 等真实分组；拆分两个快捷键设置；拆分 MCP 与 Hook 的名称和状态槽位。
- 影响：保留字段相对顺序，但会修改当前“只做视觉接入、不改变控件位置”的边界。

> 用户审批意见：
>
可以，更合理。

### D. Toggle 与依赖状态

- 推荐：实现 Toggle/Switch，将 Automatic session title 和 MCP OAuth 从 dropdown 迁移为开关；父项关闭后禁用并解释 Title model 与 Title reasoning。
- 影响：需要修改 `checklist/tui-interaction-components.md:64` 中 OAuth 使用 dropdown 的既有决定。

> 用户审批意见：
>
对于true/false的开关，我们实现真正的checkbox作为component。其他情况仍保留dropdown，降低实现成本。（未来如果有正式需要可以再提）

### E. Dialog 表面与操作样式

- 推荐：主 Settings 和子弹窗使用中性 dialog surface；取消整行 primary header/action，普通 action 使用 primary 文字，错误和危险动作使用 error。
- 影响：保留 action 位置和交互，只改变表面层级、文字角色和配色。

> 用户审批意见：
>
> 可以。

## 实施记录

- 已完成 `SettingsSection`、`SettingsItem`、`SettingsCheckboxItem`、`SettingsErrorText` 和共享 `TuiCheckbox`。
- 已完成 Global、Session、New session 的 MD3 分组、supporting copy、dependent disabled 状态和独立槽位迁移。
- 已完成 MCP OAuth、Automatic session title 的 Checkbox 迁移；其他互斥选择仍使用 dropdown。
- 已完成登录、重命名、MCP、Hook、Codex usage 弹窗的中性 surface、primary action 和 error/danger role 接入。
- 已修正 Settings 之外发现的颜色角色误配：彩色背景上的按钮使用对应 `on*` 角色，Settings dialog header/action 和 Session Catalog 使用中性 surface。
- 已将普通 Settings 内容、应用 shell 和 History 的大面积背景保持为 `Color.Unspecified`；Settings、sidebar、目录选择器、dialog 和 popup menu 的容器改用 `surfaceContainer`，bounded header、action 和按钮仍保留成对的语义 container roles。
- 已通过 `:app-view-settings:linuxX64Test`，并通过共享组件、History、Patch、Path Picker 的 Linux 测试。
- 已通过 `git diff --check` 和 `:app-cli:linkReleaseExecutableLinuxX64`。
- 已按复核意见区分普通内容与 popup 容器：前者继续使用 `Color.Unspecified`，后者使用 `surfaceContainer` 形成边界；未调整用户确认可接受的文字前景色。
- Linux review executable：`Kodex/out/kodex-settings-md3-review-linux-x64`。
- SHA-256：`112469d2271645943e78c71e38fe64407921ac3038a8c5bf15c5411105c573be`。
- `:app-view-application:linuxX64Test` 被工作区已有/并发修改的 `agent-state/test/.../TestAgentStateDependencies.kt` 编译错误阻塞：缺少 `kodexHome` 实现；本任务未涉及该文件。
- 用户复核前保持 `executable`，不移入 `done`；待用户重点检查三个 Settings 页面和代表性弹窗。

## 验证

- 使用自定义且可区分的 typography 和颜色渲染测试，证明每个 Settings 槽位消费准确语义角色。
- 覆盖 dialog headline、section heading、item label、item value、supporting、control label、error 和危险动作。
- 使用可区分的 light/dark 测试 scheme 验证 Settings 的 foreground/container 角色接入；共享 palette 对比度由主题子任务验证。
- 覆盖中性色 section heading、MCP/Hook 槽位、Checkbox、最小 supporting 文案和 dependent disabled 状态。
- 验证 True Color 与 ANSI 16 中仍可通过文字样式、控件边界和布局区分层级，不把颜色作为唯一 affordance。
- 运行 `:app-view-components:linuxX64Test`、`:app-view-settings:linuxX64Test`、`:app-view-path-picker:linuxX64Test` 和 `:app-view-application:linuxX64Test`。
- 运行 `:app-cli:linkReleaseExecutableLinuxX64` 和 `git diff --check`。
- 使用 release executable 在 `100×30` light/dark True Color 终端检查三个 Settings 页面及代表性的登录、MCP、Hook、usage 和 rename 弹窗。
- 在 `60×20` 终端确认已批准的窄终端退化行为没有新增裁切或不可达控件。
- 将构建产物路径交给用户运行复核；用户复核完成前不将任务移入 `done`。
