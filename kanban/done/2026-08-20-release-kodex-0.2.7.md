# Task Tree

- [done] 发布 Kodex 0.2.7
  - [done] 核对主分支、子模块和版本可用性
  - [done] 创建两层版本提升提交并推送
  - [done] 在 MacBook 构建四个平台 CLI
  - [done] 打包并校验五个发布资产
  - [done] 汇总 completed tasks 发布说明
  - [done] 创建并复核 GitHub Release
  - [done] 保留最终资产并清理临时目录

# Details

- 状态：`done`。构建、发布、远端复核与临时环境清理均已完成。
- 用户明确要求直接发布并返回链接。
- 发布遵循 `release-kodex` skill：只允许两个 path-limited signed version-bump commit，MacBook 是 canonical builder、stager、verifier 和 publisher。
- 当前 IntelliJ IDEA 正在打开本项目；发布流程不操作 IDE。
- 现有未跟踪 discussion task 与本发布无关，不修改也不纳入发布提交。
- Kodex 版本提升提交：`3be86ed03061028ca9067866d303a6f85d8e5798`。
- BuildKodex gitlink 提升提交：`feef37961fdc36385e7fcfd2e897363ee18fc986`。
- MacBook Java 25 四目标构建成功：1,311 个任务，耗时 13 分 1 秒。
- 四个 archive 均只有根目录 `kodex` 或 `kodex.exe`；格式、架构、Unix mode 与 SHA-256 校验通过。
- 发布说明以 `cf131233` 为 exclusive start、`feef379` 为 inclusive end，覆盖该区间新增的三个 completed task。
- GitHub Release：`https://github.com/Stream29/Kodex/releases/tag/v0.2.7`。
- Remote tag 精确指向 Kodex `3be86ed0`；Release 非 draft、非 prerelease，五个下载资产逐字节匹配 staging。
- MacBook Gradle daemon 已停止，detached checkout、staging、下载校验目录和构建中间产物均已删除。
- 最终五个资产保留在 MacBook `Kodex/out/`。
