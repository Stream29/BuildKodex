# Task Tree

- [done] 修复已完成 Session 建议的历史展示
  - [done] 确认独立 Index Item、只读内容与侧栏范围
  - [done] 核对主历史接线、错误日志与验证入口
  - [done] 实现专用历史 item、加载器与展示
  - [done] 补齐分类、定位、条目交互与侧栏文本
  - [done] 验证 accepted、rejected、failure 与真实加载错误
  - [done] 验证分组边界及完成本机构建

# Details

## 执行结果（已完成）

- 专用 item/state/loader 已接入主历史、定位、content type 和右键上下文操作，
  不实现 WorkGroup 子项接口；沿用现有 Index anchor。
- Renderer 复用 completed request-user-input 的行模型和样式；
  accepted 名称下一行显示 URI，拒绝备注复用 Other 文本行，failure 保留原任务。
- 侧栏使用独立 SuggestSubagents 分类及固定 `suggest subagents` 文本，
  没有新增预览 payload 或操作控件。
- 新增旧 JSON → item 加载、前后 work 分组、读取失败、侧栏文本、
  只读行/Mosaic 渲染和右键 storage index 回归，均通过。
- history ViewModel 共 30 项、agent ViewModel 共 11 项 JVM 测试通过；
  application View 共 77 项通过；历史 View 本任务定向回归通过，
  全量中另有 9 项展开交互断言失败，未声称全量通过。
- Linux CLI 编译及 release 链接通过；未在用户正在使用的 tmux Session 上
  重复批准任务或进行操作。不把进程构建成功表述为实机交互验证。
- 已更新历史展示指导；未提交或发布。

- 用户于 2026-09-07 授权执行，已完成定向验收；从 sealed
  任务中拆出展示修复，不重复开启已完成的 Multi-agent 引入任务。
- 主历史独立 Index Item，不折叠，不进入 WorkGroup，保留现有时间线边界。
- 标题 `Suggested Sessions`；任务名称加粗独占一行；accepted 的对应 URI
  紧接名称下一行，再显示完整换行 prompt。按数组顺序关联，不重复列名称。
- 末尾仅显示所选 `[● Accept]` / `[● Reject]`；拒绝备注用 Other 的 `  > `
  只读文本，无备注不占位。无输入框、操作按钮、配置菜单或执行器实时状态。
- failure 保留任务并显示具体原因，不伪造决定或 URI；加载/解码错误显示
  原有 Error 并记录原始异常，CancellationException 继续传播。
- 侧栏仅显示 `suggest subagents`，不新增结构化详情 payload。
- 修改 app/contract/history、app/viewmodel/history 的专用 item/state、加载、
  HistoryItemKind 和普通工具分类；接入 AgentHistoryViewModel 构造和定位，
  AgentHistoryView renderer/content type/条目交互，以及 CleanEventView。
- 侧栏修正 app/viewmodel/agent 的 HistoryIndex 映射；复用现有简单文本。
- 测试从持久化事件进入真实历史读取/item 加载，再验证只读行、分组边界、
  定位和右键恢复/分叉；覆盖成功、拒绝有无反馈、工具 failure、读取异常。
- 与 sealed 化联合编译，禁止临时空分支或运行时错误兜底。验证涉及的
  history/agent/application JVM 测试、CLI JVM 编译与 Linux release 链接。
- 不创建提交，不请求真实模型，不修改运行中的 Session 数据。
