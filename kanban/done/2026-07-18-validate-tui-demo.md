# Task Tree

- [done] 验证TUI演示程序
  - [done] 覆盖内存会话的新建、切换、fork、checkout和分叉后多轮对话测试
  - [done] 覆盖checkout对settings、plan mode、compaction checkpoint和timeline的恢复测试
  - [done] 覆盖Mosaic界面的关键交互与流式展示测试
  - [done] 使用本地Codex Home鉴权执行真实Responses API端到端测试
  - [done] 在临时工作目录中执行真实文件I/O工具测试并清理临时目录
  - [done] 在Linux、macOS和Windows真实TTY中构建、测试并启动演示程序
  - [done] 明确报告无法取得真实运行环境的目标，只将其列为编译验证

# Details

网络端到端测试使用本地真实凭据，不以mock替代。Linux x64、macOS arm64和mingw x64
需要实际启动交互程序；没有可用机器的目标不能宣称已运行验证。Mosaic不支持JS Node，
该目标不属于本模块的运行矩阵。
