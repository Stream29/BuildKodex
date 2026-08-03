# Task Tree

- 将现有应用与 ViewModel 提取为共享层
  - [done] 确定本轮分层与范围
    - [done] 保留现有 DI、application lifecycle 与 ViewModel 行为
    - [done] 让 CLI 与未来 Desktop 分别实现各自的 render view
    - [done] 先完成现有代码拆分，再单独规划 Desktop
    - [done] 本任务不创建 Desktop 模块、依赖或界面
  - 盘点现有文件与依赖边界
    - 按 application runtime、共享 frontend state 与 CLI renderer 分类现有代码
    - 识别需要随实现迁移的 common tests 与 Mosaic tests
    - 确认完成单向依赖所需的最小 visibility、package 与构建调整
  - 建立 `app/shared` 共享层
    - 抽取 application factory、DI 装配、资源所有权与 shutdown 生命周期
    - 抽取 application、Session、Agent、NewSession 与 history ViewModel
    - 抽取 composer、request-user-input、settings、auth 与 title 等无 UI 状态和服务
    - 只公开 UI-neutral state、effect 与 command contract
    - 保持 runtime、storage、tool、MCP、hook 与配置语义不变
  - 将现有 TUI 收敛到 `app/cli`
    - 保留 native CLI entrypoint 与 Mosaic application host
    - 保留 Mosaic screen、terminal component 与终端专用 renderer
    - 保持焦点、滚动、hover、展开、popup anchor 与终端尺寸为 CLI-local state
    - 将终端文案、terminal-cell 计算与 Mosaic 类型排除在共享层之外
  - 清理 Gradle 依赖方向
    - 让 `app/cli` 单向依赖 `app/shared`
    - 移除共享 JVM 产物对 `mosaicMain` 与终端组件的传递依赖
    - 保持共享层同时支持现有 JVM 与 Native targets
    - 将 Compose Desktop 构建配置留给后续独立任务
  - 验证拆分不改变现有行为
    - 迁移并运行共享 ViewModel 与 application tests
    - 运行 Mosaic component、renderer 与 CLI interaction tests
    - 验证 JVM 编译与适用 Native executable 链接
    - 检查共享模块不导入 Mosaic、终端 UI 或 Compose Desktop UI 类型
    - 手动验证 CLI 启动、Session、Agent、history、输入、设置与退出生命周期

# Details

- 状态：`planning`；路线已经确定，具体文件映射、构建调整与验证范围尚待收口，本文件不构成自动开工授权。
- `app/shared` 表示无 UI 框架依赖的共享模块族，不要求把全部职责合并进一个大模块；物理粒度按现有职责与单向依赖做最小拆分。
- `app/cli` 是现有 native CLI 与 Mosaic renderer 的所有者。
- `app/desktop` 是后续方向，不在本任务中创建；共享层和 CLI 拆分完成后，等待用户另行启动 Desktop 规划。
- 第一阶段以结构迁移为主，不借拆分重写 Agent/runtime、storage schema、tool、MCP、hook、auth 或配置行为。
- ViewModel 继续通过 `StateFlow`、结构化 state/effect 与命令方法服务 frontend，不接受 Mosaic 或 Compose Desktop 的控件、布局和本地交互状态。
- 现有 [多 Session 与 ViewModel 规划](../executable/2026-07-26-plan-multi-session-view-models.md)继续约束长期 application、Session、Agent 与 lazy history 边界；本任务先建立跨 frontend 的物理所有权，不自行推进未获授权的行为重构。
- 实施前需重新盘点 live worktree，并避开当前用户未提交修改；不创建 Git commit。
