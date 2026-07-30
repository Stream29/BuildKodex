# Task Tree

- 将 CLI 顶栏改为 Session 标签栏，并让 composer 随 Agent Runtime ViewModel 切换
  - [done] 盘点当前顶层布局、Session 选择路径和 ViewModel 生命周期
  - [done] 建立每个 Agent Runtime ViewModel 独立的 composer draft 状态
  - [done] 为 New session 保留独立的首条输入 draft
  - [done] 将顶栏改为已打开 Session 标签、创建入口和 Sessions 溢出入口
  - [done] 将顶栏以下主内容拆为当前 Agent Runtime Screen 与 New session 例外表面
  - [done] 补充 composer/标签行为的适用测试并运行相关验证

# Details

- 用户已明确：真实 Agent 的 composer 属于 `AgentRuntimeViewModel`；顶栏以下的主内容按当前 Agent runtime 切换。
- 首版假设：顶栏展示已打开 Session，`+` 创建新 Session，既有 Sessions 对话框继续用于选择未打开或溢出的持久 Session。
- New session 在物化真实 root Agent 前没有 runtime，因此保留自己的 composer draft；首次提交只路由到新建 root，不依赖当前选中 Session。
- 验证通过：Linux 的 `cli-agent`、`cli-components`、`cli-app` 测试，以及修复后单独的 `cli-app` 测试。
