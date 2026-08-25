# Task Tree

- [done] 修复 MCP 服务器名称展示回归
  - [done] 定位名称丢失的渲染路径
  - [done] 恢复紧凑按钮的名称与状态
  - [done] 修正服务器列表回归断言
  - [done] 运行 Settings 测试与 release 构建

# Details

- 用户发现 Settings 的 MCP 列表只显示 `Healthy` 等状态，不再显示服务器名称。
- 原因是按钮样式迁移时将 label 从 `名称 · 状态` 错改为只有状态，并同步削弱了测试断言。
- 按 `checklist/mcp-management.md` 恢复每个紧凑按钮同时显示服务器名称和简短状态。
- `:app-view-settings:linuxX64Test` 通过。
- `:app-cli:linkReleaseExecutableLinuxX64` 通过。
- `git diff --check` 通过。
- Linux release executable：`Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`。
- SHA-256：`1432667481e641acd9ffe62526c628ea6ee5f5dd302e08ae28b1427304d02d3c`。

## 1. MCP 条目缺少服务器名称

- 恢复 `服务器名称 · 简短状态`。
- 保留当前紧凑按钮、点击详情和按钮配色。
- 不修改 MCP 状态、详情弹窗或管理行为。

### 用户审批

- 用户要求恢复 MCP 设置页中的服务器名称；状态不能单独作为条目标识。
