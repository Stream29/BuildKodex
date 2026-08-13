# Task Tree

- [done] 修复历史条目操作
  - [done] 对照既有交互契约
  - [done] 明确失效与任务归属规则
  - [done] 修复菜单目标与文案
  - [done] 修复 revert 任务归属
  - [done] 增加交互与状态测试
  - [done] 修复自动标题生成回归
    - [done] 检查标题配置注入链
    - [done] 检查首条消息触发时序
    - [done] 增加标题生成回归测试
  - [done] 恢复运行中设置控件
    - [done] 保持模型、模式与 cwd 可用
    - [done] 运行中仅隐藏 compact
    - [done] 增加状态栏交互回归测试
  - [done] 恢复 Session 命名交互
    - [done] Enter 直接提交名称
    - [done] 移除 Rename 按钮
    - [done] 增加键盘交互回归测试
  - [done] 运行 Native 与 CLI 验证

# Details

- 将菜单名称修正为 `Revert to here` 与 `Fork from here`。
- 恢复历史目标、所属 Session/Agent 与弹出锚点的有效性约束。
- 避免已接受的 revert 操作随确认弹窗销毁而被取消。
- 菜单只对当前选中的精确 Session/Agent、当前 generation 中仍存在的稳定条目开放。
- 确认后的 revert 由 Agent ViewModel 的 owner scope 持有。
- 验收中发现自动标题生成也发生回归；该问题与当前 ViewModel/DI 重构属于同一条运行链，一并修复。
- Agent 运行期间，设置类控件继续写入并影响后续请求；仅隐藏不可并发执行的 compact。
- Session 命名弹窗沿用键盘优先交互：输入框按 Enter 直接提交，不提供多余的 Rename 按钮。
- 未显式命名的编号草稿只将 `New Session N` 用作标签显示，物化时写入 `Session <index>`，自动标题链路再次满足默认名称门槛。
- JVM、Linux Native ViewModel/View 测试、Linux CLI 链接与 PTY 启动均通过。
