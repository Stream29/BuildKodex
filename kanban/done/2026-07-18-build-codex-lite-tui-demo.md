# Task Tree

- [done] 构建基于Mosaic的Kodex CLI演示程序
  - [done] [引入Mosaic与Koin依赖来源](../done/2026-07-18-add-tui-dependencies.md)
  - [done] [实现Agent历史checkout语义](../done/2026-07-18-add-agent-history-checkout.md)
  - [done] [建立tui:demo模块](../done/2026-07-18-create-tui-demo-module.md)
  - [done] [实现演示会话编排](../done/2026-07-18-build-tui-demo-session-manager.md)
  - [done] [实现Mosaic终端界面](../done/2026-07-18-build-tui-demo-interface.md)
  - [done] [验证TUI演示程序](../done/2026-07-18-validate-tui-demo.md)

# Details

演示程序从本地Codex Home读取鉴权和模型缓存，所有会话使用临时的
`InMemoryKodexAgentStorage`。它需要支持新建、切换、fork、历史checkout、
模型与reasoning配置、多轮对话和plan mode。`checkout`截断当前会话，`fork`
保留原会话并创建新分支；二者都只能指向已完成turn的快照。

Mosaic只覆盖终端宿主目标，不为JS Node提供虚假的TUI实现。MCP、权限和skill
runtime不属于本任务。
