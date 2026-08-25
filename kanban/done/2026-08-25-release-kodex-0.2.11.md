# Task Tree

- [done] 发布 Kodex 0.2.11
  - [done] 核对主分支、子模块和版本可用性
  - [done] 创建两层版本提升提交并推送
  - [done] 在 MacBook 构建四个平台 CLI
  - [done] 打包并校验五个发布资产
  - [done] 汇总 completed tasks 发布说明
  - [done] 创建并复核 GitHub Release
  - [done] 保留最终资产并清理临时目录

# Details

- 用户确认发布补丁版本 `0.2.11`，不是仅准备。
- 发布遵循 `release-kodex` skill，MacBook 是 canonical builder、stager、verifier 和 publisher。
- 只允许两个 path-limited signed `chore: bump version` commit。
- 当前版本为 `0.2.10`；最新 GitHub Release 为 `v0.2.10`。
- 本次终端突发输入修复已由用户签名提交到 Mosaic、Kodex 和 BuildKodex。
- 先按 Mosaic → Kodex → BuildKodex 顺序同步用户现有提交，再要求两个 `main` 与远端一致。
- 在三个规定文件中将版本改为 `0.2.11`，先提交 Kodex，再提交 BuildKodex gitlink。
- MacBook 使用 Java 25 和 `--no-configuration-cache` 在 detached checkout 构建四个 CLI target。
- 四个 archive、checksum、发布说明、tag、Release 与下载资产均在 MacBook 校验。
- 发布说明区间以 `v0.2.10` 对应的 BuildKodex version-bump commit 为 exclusive start，以本次 version-bump commit 为 inclusive end。
- 最终仅在 MacBook `Kodex/out/` 保留五个发布资产，删除其余临时 checkout、staging、日志和下载校验目录。
- `v0.2.11` 的本地 tag、远端 tag 和 GitHub Release 均不存在。
- 用户现有提交已按 Mosaic `3b402d5a`、Kodex `08490559`、BuildKodex `d0d7c8f` 顺序推送并与远端一致。
- 递归子模块工作树干净；根仓库仅保留本发布的未跟踪 kanban 文件。
- IntelliJ IDEA 未运行，发布流程使用终端与 Gradle。
- Kodex 版本提升提交：`e1d78639c0845ef5506b11189f7d97c7fc26371c`。
- BuildKodex gitlink 提升提交：`387d4c1b821a1151d7bf02834d957553134dd552`。
- 两个提交均为有效 GPG 签名，且远端 `main` 已逐字匹配。
- MacBook detached checkout 在 Kodex `e1d78639` 使用 GraalVM Java 25 完成四目标构建：1,311 个任务，耗时 13 分 5 秒。
- 四个 archive 均只有根目录 `kodex` 或 `kodex.exe`；格式、架构、Unix mode 与 SHA-256 校验通过。
- 发布说明以 BuildKodex `c6f3cb49` 为 exclusive start、`387d4c1b` 为 inclusive end，唯一新增 completed task 已由一个 Highlights 条目覆盖。
- GitHub Release：`https://github.com/Stream29/Kodex/releases/tag/v0.2.11`。
- Remote tag 精确指向 Kodex `e1d78639`；Release 非 draft、非 prerelease，最终说明包含 Highlights 和 Full Changelog。
- 五个远端下载资产的名称、大小和内容逐字节匹配 MacBook staging，下载后的 SHA-256 校验通过。
- MacBook Gradle daemon 已停止，本次 detached checkout、staging、构建日志和下载校验目录均已删除。
- 最终五个资产保留在 MacBook `~/ACodeSpace/local/Kodex/out/`。
