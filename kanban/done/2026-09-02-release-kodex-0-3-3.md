# Task Tree

- [done] 发布 Kodex 0.3.3
  - [done] 准备本地迁移测试候选包
    - [done] 核对版本、提交与 migration gate
    - [done] 在 MacBook 构建四平台 CLI
    - [done] 打包并校验候选归档
    - [done] 复制候选包供本地测试
  - [done] 确认真实 Home migration
  - [done] 补充 migration 运行提示
    - [done] 只在实际 migration 时提示
    - [done] 显示起止版本和等待说明
    - [done] 覆盖 migration 回调顺序
  - [done] 正式发布 0.3.3
    - [done] 创建版本提交并推送
    - [done] 生成完整 release notes
    - [done] 发布并验证 GitHub Release

# Details

- 上一正式版本是 `0.3.2`，目标 patch 版本是 `0.3.3`。
- `0.3.3` 会首次激活 `app/migration/impl` 中已登记的 patch event cleanup migration。
- 本阶段只构建迁移测试候选包，不创建版本提交、不推送、不打 tag、不发布 GitHub Release。
- 候选构建在 MacBook 的临时干净 checkout 中暂时把三个版本位置改为 `0.3.3`。
- 用户使用候选 binary 完成真实 Home migration 后，再明确授权正式发布阶段。
- 候选基于当前 `origin/main`：BuildKodex `1a9cb91e`、Kodex `63b7d273`。
- MacBook 保存四个标准命名归档和 `SHA256SUMS`；Linux 本机接收同一组文件。
- MacBook fresh recursive clone 当前被 `Kodex/LuceneKmp` gitlink 阻塞：
  - Kodex `63b7d273` 引用 LuceneKmp `549f8afc`。
  - 该提交只存在于本机 `fix/host-logging-configuration` 分支。
  - `origin` 没有任何包含该提交的 ref，fresh clone 报 `not our ref`。
- 已删除失败的 MacBook 临时 checkout；继续前需要先让该 submodule commit 可从远端获取。
- [done] 经用户确认，已将 `fix/host-logging-configuration` 从 `8af2c20e` fast-forward
  推送到 `549f8afc`；未创建提交，也未修改 LuceneKmp `master`。
- 候选 checkout 的三个版本位置临时更新为 `0.3.3`；测试中的版本断言也临时同步为
  `0.3.3`，因为该断言当前写死为 `0.3.2`。
- Migration、lease 和 filesystem layout 的四组 `check` 已通过。
- 四个 CLI release link task 已在 MacBook 使用 Java 25 和
  `--no-configuration-cache` 一次性构建通过。
- 四个归档均已校验单一根目录文件、Unix executable mode、二进制格式、目标架构和
  SHA-256。
- 候选资产位于：
  - MacBook：`Kodex/out/0.3.3-candidate/`
  - Linux 本机：`Kodex/out/0.3.3-candidate/`
- Linux x64 归档 SHA-256：
  `75c184d07110826807bd0528337134ae9acc64ba2c493609f0b5626152f3b1bc`。
- `app-viewmodel-application:check` 额外触发的 Mosaic JVM JNI Zig build 在当前
  macOS SDK 上链接失败；四个 Native CLI release link task 不依赖该失败路径。
- 用户已使用 Linux x64 候选 binary 完成真实 Home migration。
- `prepareKodexHome` 现在只在执行实际 migration 前回调起止版本；普通同版本启动不提示。
- CLI 提示示例：
  `Migrating Kodex Home from 0.3.2 to 0.3.3. Please wait...`
- 使用 Java 25 运行 `:app-migration-impl:check` 和
  `:app-cli:compileKotlinLinuxX64`，均通过。
- 正式版本提交：
  - Kodex：`3d7b0cc02f9588c007e0fd26346a0025b4fa17ef`。
  - BuildKodex：`29b52eabccb9af53d7c890401ec4c2d4b286577d`。
- 正式构建从精确 Kodex 提交在 MacBook 干净 checkout 中完成：
  - Java 25。
  - Migration、lease 和 filesystem layout 门禁通过。
  - 四个平台 Native CLI release link task 通过。
- 正式资产在 MacBook 和 Linux 两端通过 checksum、单文件结构、执行权限、格式与架构校验。
- 正式归档 SHA-256：
  - Linux x64：`172ff2fb65c6c9535242c8212efcf7ebb3ddeda286f431c9cf158cc916c63502`。
  - Linux ARM64：`406d79e140f77efb0f45feac360f63b8ff938da3e0f2f567437f477ae47ebd81`。
  - macOS ARM64：`dbefdd51f494a667027acf3d970845d379d364df9d3154656316396a64fbfb3b`。
  - Windows x64：`1561e802bce0585a90872746cc877d36f7219583fb8c8f597bb034df26e513bc`。
- GitHub Release 已发布并核验：
  `https://github.com/Stream29/Kodex/releases/tag/v0.3.3`。
- 已删除临时 checkout、候选归档、传输副本和临时日志；正式资产只保留在 MacBook
  `Kodex/out/0.3.3/`。
