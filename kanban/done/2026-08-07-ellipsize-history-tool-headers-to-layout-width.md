# Task Tree

- [done] 按实际布局宽度裁剪 History 工具标题
  - [done] 实现通用动态省略文本组件
    - [done] 从有限布局约束读取实际可用宽度
    - [done] 按终端 cell 安全添加省略号
    - [done] 保留文本颜色与样式参数
  - [done] 接入 History 工具标题
    - [done] 让语义摘要只负责单行规范化
    - [done] 对包含展开符与语义前缀的完整标题裁剪
    - [done] 保持普通 History 内容按宽度换行
  - [done] 补充定向回归测试
    - [done] 覆盖宽字符、窄宽度和充足宽度
    - [done] 覆盖布局宽度变化后的重新裁剪
    - [done] 运行组件与 History 本机测试

# Details

- 调查、修改路线与验证范围已经确定，用户已确认进入 execution。
- 复用现有 terminal cell 宽度与字素安全截取能力，不修改 Mosaic。
- 动态宽度必须来自当前容器布局约束，不能直接使用全局终端列数，因为侧栏会缩小 History 实际宽度。
- 新组件仅用于要求保持单行的标题；消息和详情继续使用 `WrappedHistoryText` 换行。
- 删除工具语义摘要固定 96 列的提前截断，避免动态裁剪前已经丢失内容。
- 已新增通用 `EllipsizedText`，并让工具标题将展开符、语义前缀和完整摘要一起按实际布局宽度裁剪。
- 已更新 `checklist/tui-interaction-components.md` 保存组件与 History 使用约束。
- `:app-cli-components:linuxX64Test` 与 `:app-cli-history:linuxX64Test` 通过；IDEA 定向构建通过，新增代码无 inspection 警告。
