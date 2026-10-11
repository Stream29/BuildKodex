# Task Tree

- `Inspect local command resolution and release installation rules`()
- `Restore kodex as the sole local command and update release rules`()
- `Verify command resolution without starting the application`()

# Details

- 用户确认本机仅保留 `kodex`，移除旧 `kodex-cli` 入口，并同步更新发布规则。
- PATH 已包含 `~/.local/bin`，但只有 `kodex-cli` 软链接；它指向
  `Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`。
- 发布包内文件名为 `kodex`，旧发布规则却继续验证本机 `kodex-cli` 入口。
- 仅重命名本机软链接；保留构建产物路径、二进制内容和运行中的进程。
- 修改 `.agents/skills/release-kodex/SKILL.md`，将本机正式入口统一为 `kodex`。
- 验证：PATH 解析到新入口，目标可执行且与原链接一致，旧入口不存在。
- 并行的 v0.4.13 发布任务不在本次范围内；不修改其任务记录或构建资源。

## 验证结果

- 已将原软链接重命名为 `~/.local/bin/kodex`；链接目标保持不变。
- 当前工具 shell 与无初始化文件的新 Bash 均解析到 `~/.local/bin/kodex`，
  且无法解析 `kodex-cli`。
- 目标仍可执行，重命名前后的设备号、inode、大小、修改时间和权限均一致。
- 未启动应用、重编译或修改 Kodex Home；运行中的实例不受影响。
- 2026-10-12 用户明确授权把剩余本地改动分批提交、推送；本任务的发布入口规范与记录独立成批，不再次修改本机软链接或启动应用。
