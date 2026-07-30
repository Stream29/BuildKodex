# Task Tree

- [done] 修复 New session 提交丢失
  - [done] 复现 virtual New tab 提交后的 root history 状态
  - [done] 定位提交在物化、切换或 runtime 启动链中的丢失点
  - [done] 修复输入持久化、runtime resume 与 UI 切换时序
  - [done] 覆盖 New session 端到端提交回归
  - [done] 运行定向验证

# Details

- 原因是 `NewSessionScreen` 的局部 coroutine scope 会在 root 激活后随该 screen 卸载而取消，可能中断后续输入写入和 runtime resume。
- 提交改由持续存在的 `SessionTreeCliScreen` scope 启动；virtual New screen 只同步转发 submit 事件。
- 回归覆盖草稿进入 root history、root runtime request 流启动，以及 New screen 被替换后的提交仍完成。
- 已通过 `:cli-app:linuxX64Test`。
