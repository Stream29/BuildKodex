# Task Tree

- [done] 修正展开侧栏的区块顺序
  - [done] 确认 Shell sessions 当前插入 Agent tree 标题与列表之间
  - [done] 让 Agent tree 标题与列表保持相邻
  - [done] 将 Shell sessions 区块移到 Agent tree 之后
  - [done] 保持既有高度分配与交互
  - [done] 运行 application 测试并重建 release CLI

# Details

- 用户指出 `Shell sessions` 不应夹在 `Agent tree` 标题和对应列表之间。
- 目标顺序为展开按钮、`Agent tree` 标题、Agent tree 列表、`Shell sessions` 标题及列表。
- 只调整 `SessionAgentSidebar` 的组合顺序；复用现有 `agentTreeRows` 与 `shellSessionListRows` 计算。
- `app-cli-application:linuxX64Test`、IDE inspection 与 `git diff --check` 通过。
- 最终 release executable 已重新链接：`Kodex/app/cli/application/build/bin/linuxX64/releaseExecutable/app-cli-application.kexe`。
