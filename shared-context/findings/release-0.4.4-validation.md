# Kodex 0.4.4 发布验证

- [GitHub Release v0.4.4](https://github.com/Stream29/Kodex/releases/tag/v0.4.4) 于 `2026-09-08T16:56:59Z` 正式发布，为 Latest、非草稿、非预发布。
- Kodex/tag：`92b572d4db24d46feca333e82f3376f71bab27cd`；BuildKodex：`fa021e9b80f3f8e511df0bf632d525dd9bd3873f`。
- 两个 `chore: bump version` 提交签名均验证通过，仅包含三处应用版本文件及对应 gitlink；按 Kodex、BuildKodex 顺序推送并核验远端。

## 迁移与测试

- 上一版本为 0.4.3；0.4.4 不激活新 migration。已发布实现、codec、fixture 和 registry 均未改变；历史性能探针只有 TestBalloon 入口调整。
- `Kodex/app/migration/impl/build.gradle.kts:11` 从 `project.version` 生成当前版本，发布检出实际生成值为 0.4.4。
- MacBook 新建精确 detached 递归检出，三个子模块匹配；所有 Gradle 命令使用 GraalVM Java 25 和 `--no-configuration-cache`。
- migration contract/impl、filesystem layout、filesystem lease 的 JVM/macOS ARM64 八个任务：44 + 42 次测试执行全部通过。
- 十模块 JVM 强制重跑：Application/History View、Agent/Application/History/Session ViewModel、filesystem/in-memory Session repository、OpenAI client、unified-exec，共 353 项全部通过。
- 最终 18 个任务共 439 次执行，零失败、错误或跳过；手动性能探针未启用测量，未运行全项目测试或真实账号模型请求。

## 构建环境与产物

- MacBook 初次 SSH 超时由用户恢复；递归克隆的网络断开在同一主机重试成功。
- Zig 下载两次截断；仅下载命令使用已有 `127.0.0.1:7897` HTTPS 代理后成功，未更改系统代理。
- Mosaic JVM JNI 的 Zig build runner 在默认 Xcode SDK 下出现系统符号链接错误；命令级 `DEVELOPER_DIR=/Library/Developer/CommandLineTools` 后该任务及 JVM 回归成功。未改源码、Zig 版本、SDK 安装或系统选择。
- 四个 Native CLI Release 链接任务在 MacBook 默认 Xcode 环境全部成功，耗时 12 分 55 秒；未回退其他 builder。
- 每个归档均提取检查根目录单入口、Unix 可执行权限、二进制格式、架构及与原始产物字节一致；全部 SHA-256 校验通过。
- macOS `codesign --verify` 通过；不等同于 Apple 公证。

## 运行与发布核验

- macOS ARM64 在全新隔离 Home 显示真实 TUI 并正常退出；Home 版本为 0.4.4，安装的 skill 与冻结 0.4.3 Kotlin 文本逐字节一致。
- Linux x64 在隔离 Home 完成 0.4.3 → 0.4.4 升级，显示真实 TUI 并正常退出；保留设置、已有产品 skill、相邻文件、自定义 skill 和未知文件。
- 两个运行验证均确认 Home 租约释放；未操作用户 Home。Windows、Linux ARM64 仅验证构建和归档，未做实际运行。
- 发布说明范围 `(177a59c, fa021e9]`：23 个新增 done 任务由 13 条英文短列表覆盖；已读取各任务正文，排除旧 done 文件的纯修改。
- 旧发布记录中的 SHA 已被此前授权的历史清理映射；本次通过当前 v0.4.3 tag `4468142b` 与 BuildKodex `177a59c` gitlink 确认基线。
- 发布前重查两个远端 main、tag/Release 不存在及附件校验和；由 MacBook 使用精确 target、标题、摘要和生成说明发布。
- 发布后回读确认标题等于 tag、13 条英文摘要及 Full Changelog 的 GitHub Markdown 渲染正确；远端 tag、Latest 状态、五个附件名称、大小和服务端 SHA-256 与 MacBook 暂存文件一致。

## 已发布附件

| 附件 | 字节数 | SHA-256 |
| --- | ---: | --- |
| `kodex-0.4.4-linux-x64.tar.gz` | 22684086 | `050704cd35e026845a454ae5e46c331f006d92dc1928e6f197fbd911a526fbb6` |
| `kodex-0.4.4-linux-arm64.tar.gz` | 20774130 | `43dfb2926417ecf5692a1b3d7c9b0123ef9d31539b2b2bd8a949c9fda0a9aeb0` |
| `kodex-0.4.4-macos-arm64.tar.gz` | 15912905 | `26a4bf0632a91abfda5c94df05449de59aef69edb2720d59ccb26bc90cc3b758` |
| `kodex-0.4.4-windows-x64.zip` | 11456430 | `c2624a07b32336a8dc3bdb8d5a8d95e9fc736182fcbdeb104a057a58d265fdf1` |
| `kodex-0.4.4-SHA256SUMS.txt` | 383 | `d28f21e4b053aa0fd1c724f6809ba6a2ce546e537cf6ef34e0c09b8327ac0172` |

## 本机安装与保留

- 从正式 Release 重新下载 Linux x64 归档和校验文件，验证 SHA-256，并与发布前烟测归档、二进制逐字节比较。
- 原子替换 `Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`，权限为 `755`；`~/.local/bin/kodex-cli` 仍准确指向该文件。
- 安装的二进制 SHA-256：`45d40b110e04b439927524499ec864f433f160db4409ff031a41121de9c5619a`。
- 未重启运行中的 Kodex；既有进程保留原可执行文件，重启后才使用新版。
- 五个最终附件保存在 MacBook `~/ACodeSpace/local/Kodex/out/0.4.4/`。
- MacBook 与本机临时检出、脚本、日志、下载、归档提取、隔离 Home，以及本轮 Zig 下载/解压临时文件均已清理；清理后再次核验最终归档和本机二进制哈希。
- 构建期间外部修改的 kRPC 讨论文档保持不动；本次任务和验证记录留待用户提交，不创建第三个提交，不部署文档站点。
