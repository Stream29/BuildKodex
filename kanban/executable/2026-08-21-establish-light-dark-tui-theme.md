# Task Tree

- 建立可访问的 light/dark TUI 主题基建
  - [done] 盘点现有颜色角色、默认值和主题入口
  - [done] 定义静态语义配色
    - 以 `#1C444A` 为 seed 生成 SchemeTonalSpot light/dark role values
    - 使用默认 contrast level `0.0`
    - 增加 `onSurfaceVariant` 等已有调用需求的角色
    - 从现有 green 语义色生成协调且可访问的 success 配对
    - 为 `Unknown` 终端主题使用 dark fallback
  - [done] 接入终端主题
    - 保留 `DefaultTuiColorScheme` 作为兼容默认值
    - 提供从 `Terminal.Theme` 到 color scheme 的纯映射
    - 在应用根部按当前终端主题提供 `TuiTheme`
    - 支持终端主题状态变化后的重组
  - [done] 保留透明 History
    - 让 light/dark scheme 的 `background` 保持 `Color.Unspecified`
    - 为透明背景分别提供可读的 `onBackground`
    - 验证空白 cell 和终端原生复制语义不变
  - [done] 迁移共享 TUI 颜色调用
    - 让 supporting content 使用 `onSurfaceVariant`
    - 修正所有 foreground/container 错误配对
    - 不在业务视图引入具体 RGB
  - [done] 补充主题回归测试
    - 覆盖 Light、Dark 和 Unknown 映射
    - 覆盖嵌套自定义主题继承
    - 覆盖实际角色配对与 True Color 对比度
    - 覆盖 ANSI 16 下的非颜色视觉区分
  - [done] 更新共享 TUI 主题设计决策
  - 运行 `git diff --check`、相关模块测试和 Linux release 构建
  - 在 light/dark 终端运行 release executable

# Details

- 状态：`executable`。用户已授权开始实现，完成后由父任务消费并交付 release executable。
- 父任务：`kanban/executable/2026-08-10-adjust-settings-layout.md`。
- 父任务只消费本任务提供的语义角色和 scheme，不重复维护 palette、终端主题选择或共享主题测试。
- 用户已批准 `#1C444A` + SchemeTonalSpot + contrast level `0.0` 的确定性生成路线。

## 当前缺口

- `TuiColorScheme` 只有一套固定 dark 默认值，并缺少 `onSurfaceVariant`：`Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TuiTheme.kt:19-74`。
- `DefaultTuiColorScheme.primary` 是适合作为深色条背景的低明度 teal，不能作为深色 surface 上的 action foreground。
- Mosaic 已提供 `Terminal.Theme.Unknown`、`Light` 和 `Dark`：`Kodex/Mosaic/mosaic-terminal/src/commonMain/kotlin/com/jakewharton/mosaic/terminal/Terminal.kt:153-162`。
- 应用根部仍直接调用无参数 `TuiTheme`，没有消费终端主题：`Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt:113`。
- History 依赖 `background = Color.Unspecified` 保持未绘制 cell 和终端原生复制；主题重构不得把根背景改成不透明 surface。

## 已批准配色路线

- 使用 source color ARGB `0xFF1C444A`。
- 使用 Material Color Utilities 的 `SchemeTonalSpot(Hct.fromInt(seed), isDark, 0.0)` 分别生成 light 和 dark role values；这是 MCU 对默认 Material light/dark scheme 的推荐替代路线：[Creating a Color Scheme](https://github.com/material-foundation/material-color-utilities/blob/main/dev_guide/creating_color_scheme.md)。
- 生成时固定并记录 MCU package version 或 commit；本次使用 `@material/material-color-utilities` npm `0.3.0`；将结果作为静态 RGB 写入 Kotlin，运行时不增加 Material 或 MCU 依赖。
- 具体 RGB 只存放在默认主题定义中，业务视图继续只消费语义角色。
- dark scheme 的 primary 必须能在中性 dark surface 上作为强调文字使用；不沿用当前低明度 `primary` 的错误语义。
- `success` 以现有 `Color.Green` 对应的 `0xFF00FF00` 为 source，先向 `0xFF1C444A` harmonize，再使用 light `40/100` 和 dark `80/20` tone pairs 生成 `success/onSuccess`；不直接沿用 `Color.Green/Color.White`。
- 保留 `DefaultTuiColorScheme` 公开入口并让它指向 dark scheme，避免无必要的调用方和测试破坏。
- `Unknown` 回退 dark scheme，以保持现有默认视觉。
- 只增加已被共享组件或已批准 Settings 设计消费的角色；不机械复制完整 Material `ColorScheme`。
- Material 3 的配色依据是成对语义角色、协调 tonal palette 和独立 light/dark scheme：[Android color](https://developer.android.com/design/ui/mobile/guides/styles/color?hl=en)。

## 必须验证的角色组合

- `onBackground/background`，其中 background 保持 unspecified。
- `onPrimary/primary`。
- `onPrimaryContainer/primaryContainer`。
- `onSecondaryContainer/secondaryContainer`。
- `onTertiaryContainer/tertiaryContainer`。
- `onSurface/surface`。
- `onSurface/surfaceContainer*`。
- `onSurfaceVariant/surface` 与 `onSurfaceVariant/surfaceContainer*`。
- `primary/surface*`，用于中性表面上的强调 action。
- `error/surface*` 与 `onError/error`。
- `success/surface*` 与 `onSuccess/success`。
- `outline/surface*`。

## 对比度与终端能力

- True Color 下，普通文字组合至少达到 `4.5:1`，outline 等非文字组合至少达到 `3:1`：[Android Accessibility](https://developer.android.com/design/ui/mobile/guides/foundations/accessibility?hl=en)。
- ANSI 16 颜色由终端 palette 决定，不能承诺固定 RGB 对比度；必须保留方括号、布局、Bold、Dim、Invert 和物理光标等非颜色 affordance。
- 测试应从实际消费角色枚举组合，避免只验证若干手写示例。

## 范围边界

- 本任务负责 `TuiColorScheme`、默认 palette、终端主题映射、应用根部接入和共享主题规则。
- 本任务不负责 Settings 的分组、文字组件、Checkbox、依赖状态或 dialog 布局；这些由父任务负责。
- 本任务不改变 Focus、hover、press、selected 和 disabled 状态矩阵。
- 本任务不引入用户自定义主题设置、动态壁纸颜色或运行时 palette 编辑。
- 本任务不修改 ViewModel、持久化或业务状态。

## 实施记录

- 已完成 `TuiTheme` 的 light/dark scheme、`Terminal.Theme` 映射、透明 History 语义和应用根部接入。
- 已完成 semantic role contrast 测试，以及 ANSI 16 下 checkbox marker/Bold affordance 测试。
- 已通过 `:app-view-components:linuxX64Test`、`:app-view-settings:linuxX64Test`、`:app-view-history:linuxX64Test`、`:app-view-patch:linuxX64Test` 和 `:app-view-path-picker:linuxX64Test`。
- 已通过 `git diff --check` 和 `:app-cli:linkReleaseExecutableLinuxX64`。
- Linux review executable：`Kodex/out/kodex-settings-md3-review-linux-x64`。
- SHA-256：`b2115f1d33cf22172c748b16deb8670292f0c62fe46b962a877f9e50145f6aec`。
- `:app-view-application:linuxX64Test` 尚未完成：构建在工作区已有/并发修改的 `agent-state/test/.../TestAgentStateDependencies.kt` 报告缺少 `kodexHome` 实现；本任务未涉及该文件。
- 仍待用户在 light/dark True Color 与 ANSI 16 终端运行 review executable，完成后再将任务移入 `done`。

## 验证

- 运行 `:app-view-components:linuxX64Test`、`:app-view-history:linuxX64Test`、`:app-view-patch:linuxX64Test`、`:app-view-settings:linuxX64Test`、`:app-view-path-picker:linuxX64Test` 和 `:app-view-application:linuxX64Test`。
- 运行 `:app-cli:linkReleaseExecutableLinuxX64` 和 `git diff --check`。
- 使用 release executable 在 light 和 dark True Color 终端检查 History、状态栏、侧栏、popup menu、Settings 和代表性 dialog。
- 在 ANSI 16 终端确认关键层级和交互状态不依赖颜色才能识别。
