# Task Tree

- [done] 为 `request_user_input` 增加可用性选单
  - [done] 盘点工具可见性与设置路径
  - [done] 明确选单范围、默认值与切换语义
  - [done] 建立两值 Agent setting
    - [done] 定义稳定类型、wire value 与默认值
    - [done] 让旧 Agent settings 缺省为 `ask user`
    - [done] 持久化可配置的新 Session 默认值
    - [done] 让新草稿和 subagent 继承初始值
  - [done] 按 setting 投影模型可见工具
    - [done] `ask user` 暴露 `request_user_input`
    - [done] `no question` 不暴露 `request_user_input`
    - [done] 统一普通请求与 compaction 请求
    - [done] 保留已发出请求和 pending 问题
  - [done] 接入 Agent 与 New Session 选单
    - [done] 扩展 settings 和 ViewModel 字段命令
    - [done] 在两类状态栏增加下拉按钮
    - [done] 在 Session 设置显示当前值
    - [done] 在 New session 设置管理默认值
  - [done] 补齐回归验证
    - [done] 验证序列化、缺省值和 YAML round trip
    - [done] 验证工具可见性、继承和 pending 语义
    - [done] 验证 ViewModel 更新与 TUI 选择
    - [done] 更新相关 checklist 和冲突旧决策

# Details

## 已确认语义

- 该选项是 `KodexAgentSettings` 的一部分。
- 使用两值类型 `RequestUserInputMode`：
  - `AskUser`，wire value 为 `ask_user`，界面显示 `ask user`。
  - `NoQuestion`，wire value 为 `no_question`，界面显示 `no question`。
- 未保存该字段的既有 Agent 和新安装默认使用 `AskUser`。
- 当前 Agent 持久化自己的值；新 Session 草稿复制全局默认后可独立修改。
- `Settings > New session`可以修改以后新建草稿的默认值。
- subagent 创建时继承父 Agent 当时的完整 settings，之后可独立修改。
- 切换只影响切换后组装的请求。已发出的请求不撤回。
- 切换到 `NoQuestion`不会取消已有 pending 问题；问题仍可回答或手动清除。
- 本任务只控制 ToolSpec 可见性，不修改工具 schema、pending 投影和答题表单。

## 修改边界

- 在 `Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/`
  `AgentModels.kt:6`与`CompactionModels.kt:50`增加类型和 Agent settings 字段。
- 在 `Kodex/agent-state/tool/src/commonMain/kotlin/io/github/stream29/kodex/`
  `agentstate/tool/KodexVisibleToolSpecs.kt:43`条件化添加
  `RequestUserInputTools.spec`。普通请求和 compaction 已共用该入口。
- 沿现有 Agent mode 路径扩展全局默认、settings store、Agent/New Session
  ViewModel、Session settings adapter 和对应 TUI。
- 状态栏选单对已持久化 Agent 与虚拟 New Session 都可用，运行期间保持可编辑。
- `Kodex/tool/request-user-input/`、pending clean model 和
  `RequestUserInputViewModel`不需要修改。

## 验证重点

- 序列化覆盖稳定 wire value、旧 settings 缺字段默认值和私有
  `settings.yml` round trip。
- 工具投影覆盖两种 question mode 与 Single/Multi agent mode 的组合。
- 请求投影覆盖普通 Responses 与 remote compaction。
- Session 测试覆盖 subagent 继承后独立更新。
- pending 回归覆盖切换 setting 后原调用与答题状态保持不变。
- TUI 测试覆盖按钮标签、键盘/鼠标选择和状态栏最右侧 Settings 布局。

## 协调项

- 本任务取代
  `kanban/planning/2026-07-26-align-codex-prompt-context-injection.md`
  中“`request_user_input`始终可用”的旧约束。
- 实现与回归验证已完成。
