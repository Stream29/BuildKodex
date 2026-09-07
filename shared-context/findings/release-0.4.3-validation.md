# Kodex 0.4.3 发布验证

- [GitHub Release v0.4.3](https://github.com/Stream29/Kodex/releases/tag/v0.4.3) 于 `2026-09-07T14:05:17Z` 发布，为 Latest、非草稿、非预发布。
- 发布源码及 tag：`fb09a6714c60f75e5b4017c1788d32735c9fa9fc`；BuildKodex 发布基线：`5205ea9c3f1f6f9c629c465dfb81a6d5799f4153`。
- 构建期间新增的 `a4cb748c` / `cd9d465` 不在此次发布范围，本轮未推送这些后续提交。

## 验证

- MacBook 全新 detached checkout，三个递归子模块精确匹配；复用 GraalVM Java 25 Daemon，使用 `--no-configuration-cache`。
- 8 个迁移、版本选择、布局和租约测试任务显式重跑：44 项 JVM + 42 项 macOS ARM64，零失败、零错误、零跳过；生成版本为 0.4.3。
- 四个 Native CLI Release 任务均在 MacBook 成功，耗时 12 分 34 秒；未回退其他构建主机。
- 四个归档逐一提取，核验根目录单入口、Unix 可执行权限、格式、架构及原始二进制字节；macOS 本地 codesign 校验通过，不等同于 Apple 公证。
- macOS ARM64 首次启动、Linux x64 从 0.4.2 升级均使用隔离 Home，验证真实 TUI 和正常退出；版本为 0.4.3，新 skill 与冻结源码逐字节一致，Home 租约释放。
- Linux 升级保留设置及同目录、同级未知文件。只在这两个验证宿主运行启动冒烟；Windows、Linux ARM64 仅验证构建和归档，未重跑全项目测试或请求真实模型。
- 发布说明范围为 `(cc2e7e8, 5205ea9c]`，10 个新增 done 任务由 8 条短列表覆盖；GitHub Markdown 渲染确认列表及 Full Changelog。
- SSH 大文件传输低速时，使用 GitHub 草稿暂存，经本机下载验证后公开；MacBook 发布命令复用现有代理，未修改系统代理、服务或凭证。

## 已发布附件

- 以下五个附件的名称、字节数及服务端 SHA-256 均与 MacBook 产物一致；公开后再次核验。

| 附件 | 字节数 | SHA-256 |
| --- | ---: | --- |
| `kodex-0.4.3-linux-x64.tar.gz` | 22827435 | `6cae4e96f0e320112f8d6f0c95b5ba6e6285ffc463a3fe0901c08824788cb311` |
| `kodex-0.4.3-linux-arm64.tar.gz` | 20854339 | `b19ec7682a2b5e1e855ca4f5e14ba25f2684e6e0a9fa09ba8af5a50b120df34f` |
| `kodex-0.4.3-macos-arm64.tar.gz` | 15968260 | `2653b114334deb49ef6a2a4dba83aa3918aa190be2f9825217ffa9c308d16326` |
| `kodex-0.4.3-windows-x64.zip` | 11593187 | `f39e313ae1b5491b8c94a913849707311cea6ab748849658107517c54048d0f9` |
| `kodex-0.4.3-SHA256SUMS.txt` | 383 | `a78eccd55d7264caebbeca90be29c42d7cca597723087c9d1a0fe5be51ff4a09` |

## 本机安装

- 从正式 Release 重新下载 Linux x64 包及校验和；校验后与烟测二进制逐字节比较，再原子替换 `Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`。
- `~/.local/bin/kodex-cli` 仍指向该文件，权限为 `755`；二进制 SHA-256 为 `92539baaa88f4cb78a58ad73c5a23844ef49f65babb4ee968a692c80f26e6720`。
- 没有重启运行中的 Kodex，也没有操作真实 Home；运行中的进程保留原可执行文件，需重启才使用新版。

## 保留与清理

- 五个最终附件保存在 MacBook `~/ACodeSpace/local/Kodex/out/0.4.3/`。
- MacBook 和本机的临时检出、脚本、日志、下载、提取目录及隔离 Home 已清理；保留最终附件和本机安装。
