# Task Tree

- [done] 将现有应用与 ViewModel 提取为共享层
  - [done] 确定本轮分层与范围
    - [done] 保留现有 DI、application lifecycle 与 ViewModel 行为
    - [done] 让 CLI 与未来 Desktop 分别实现各自的 render view
    - [done] 先完成现有代码拆分，再单独规划 Desktop
    - [done] 本任务不创建 Desktop 模块、依赖或界面
  - [done] [纯拆分现有共享应用层与 CLI renderer](../done/2026-08-03-separate-shared-application-and-cli.md)

# Details

- 状态：`done`；共享层与 CLI renderer 已完成纯拆分，Desktop 仍等待后续单独规划。
- `app/shared` 表示无 UI 框架依赖的共享模块族，不要求把全部职责合并进一个大模块；物理粒度按现有职责与单向依赖做最小拆分。
- `app/cli` 是现有 native CLI 与 Mosaic renderer 的所有者。
- `app/desktop` 是后续方向，不在本任务中创建；共享层和 CLI 拆分完成后，等待用户另行启动 Desktop 规划。
- 第一阶段以结构迁移为主，不借拆分重写 Agent/runtime、storage schema、tool、MCP、hook、auth 或配置行为。
- ViewModel 继续通过 `StateFlow`、结构化 state/effect 与命令方法服务 frontend，不接受 Mosaic 或 Compose Desktop 的控件、布局和本地交互状态。
- 现有 [CLI 全层级 contract/ViewModel 迁移](../done/2026-08-07-refactor-session-tree-cli-screen.md)继续约束长期 application、Session、Agent 与 lazy history 边界；本任务先建立跨 frontend 的物理所有权，不自行推进未获授权的行为重构。
- 实施前需重新盘点 live worktree，并避开当前用户未提交修改；不创建 Git commit。
