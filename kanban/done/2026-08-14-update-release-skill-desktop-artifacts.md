# Task Tree

- [done] 更新 Kodex 发版 Skill
  - [done] 核对现有发版与 Desktop 任务
  - [done] 明确 Desktop release 资产矩阵
  - [done] 设定 MacBook 优先构建策略
  - [done] 更新并校验 skill

# Details

- 未来 GitHub Release 增加 Desktop shadow JAR、macOS DMG、Windows MSI 与 Linux DEB。
- 远程 MacBook 作为首选 release 构建机和总控机；其他主机仅承担无法在 macOS 完成的平台绑定步骤。
- 当前 Compose Desktop 已提供 release uber JAR 与三个平台安装包任务，但 uber JAR 随构建主机解析 `currentOs` runtime。
- 用户已明确选择单个跨平台 shadow JAR；发版前必须存在 `:app-desktop:shadowJar`，且产物包含 Linux x64、macOS arm64、Windows x64 runtime。
- Desktop 安装包矩阵为 Linux x64 DEB、macOS arm64 DMG、Windows x64 MSI。
- MacBook 负责首选构建、统一暂存、校验和发布；Linux/Windows builder 仅运行无法在 macOS 生成的 DEB/MSI 平台绑定步骤。
- GitHub Release 固定包含四个 CLI 归档、四个 Desktop 产物和一个校验和文件，共九个资产。
- 当前项目尚无 `:app-desktop:shadowJar`；skill 明确禁止用仅含 `currentOs` runtime 的 Compose uber JAR 替代，并在任务缺失时阻止发版。
- `quick_validate.py` 与 `git diff --check` 验证通过。
