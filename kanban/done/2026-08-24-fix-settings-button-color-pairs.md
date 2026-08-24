# Task Tree

- [done] 修复 Settings 按钮语义配色
  - [done] 审计按钮前景、容器和交互状态
  - [done] 为 Settings 建立统一按钮角色
  - [done] 配对 filled action 的 container/on-color
  - [done] 区分普通、危险和内容操作
  - [done] 为选中导航使用显式 tonal 配对
  - [done] 保持 hover、press 和 disabled 可读
  - [done] 迁移 Settings 主页面与子弹窗
  - [done] 补充 light/dark 与状态测试
  - [done] 更新 TUI 交互决策
  - [done] 运行格式化、测试和 release 构建

# Details

- 状态：`done`。
- 用户已明确要求直接修正 Settings 按钮。
- 修改前 enabled 静态色的 True Color 对比度达标，但约四十处 Settings 按钮仍由调用方直接组合文字色与外围背景，缺少统一组件约束。
- 修改前 `TuiButton` 的 pressed/selected `Invert` 会交换前景和背景，不再保持 Material 3 的 container/on-color 语义配对。
- 修改前 disabled action 只是对 `primary` 或 `error` 执行终端 `Dim`，没有回退到中性 disabled 角色。
- 本任务只改变 Settings 及其子弹窗的按钮样式；其他页面继续保留现有终端交互矩阵。
- Material 3 的按钮 API 分别建模 enabled/disabled container 与 content color；状态规范要求交互状态继续保持清晰对比，而不是任意交换语义角色：[ButtonDefaults](https://developer.android.com/reference/kotlin/androidx/compose/material3/ButtonDefaults)、[Interaction states](https://m3.material.io/foundations/interaction/states/overview)。
- Settings 已集中使用 action、primary、danger、content 和 navigation 五类按钮入口；只有主要确认和最终危险确认绘制自身 container。
- 新增保色交互样式后，Settings 的 hover、press 和 selected 只增加 Bold；默认 TUI 按钮仍保留原有 Invert 矩阵。
- Disabled filled action 使用 `surfaceContainerHighest/onSurface` 后再应用 Dim，不保留 enabled accent container。
- True Color 测试验证五类按钮和 disabled 状态消费准确的前景/容器配对，并确认 Settings selected 不输出 inverse video。
- IDEA 格式化、定向构建和检查通过；仅剩 MCP/Hook 与 reasoning 显示函数的既有弱重复提示。
- `:app-view-components:linuxX64Test`、`:app-view-settings:linuxX64Test` 与 `:app-view-application:linuxX64Test` 通过。
- `:app-cli:linkReleaseExecutableLinuxX64` 通过。
- Review executable：`Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`。
- SHA-256：`2ea81785c32714775be64a303da7625ec882a678ab04033cc3add13a2ff1fea9`。

## 1. 按钮颜色角色

- 普通文字操作使用中性表面上的 `primary`。
- 主要确认操作使用 `primary/onPrimary`。
- 最终危险确认使用 `error/onError`。
- 内容按钮使用 `surface/onSurface`。
- 选中导航使用 `secondaryContainer/onSecondaryContainer`。

### 用户审批

- Settings 中的按钮风格和文字/背景关系应遵守 Material 3 的语义配色。

## 2. 交互状态可读性

- Settings 按钮不通过 Invert 交换语义色。
- Hover、press 和 selected 使用不破坏颜色配对的强调样式。
- Disabled 使用中性颜色后再降低强调度。

### 用户审批

- Settings 按钮在默认和交互状态下都必须保持可读。
