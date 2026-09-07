# Task Tree

- [done] 从版本历史清理已移除的 Python/Java 文件
  - [done] 确认两个仓库、历史路径、tag 与签名处理范围
  - [done] 备份远端引用并创建隔离副本
  - [done] 确认历史孤立 gitlink 的保留方式
  - [done] 在隔离副本重写历史
  - [done] 验证历史文件与子模块指针
  - [done] 带旧 SHA 校验强推并同步本机引用
  - [done] 验证远端与发布附件并记录结果

# Details

- 用户明确同意重写和强推 Kodex、BuildKodex 的 main 与相关全部 tag，接受原签名移除、历史测试缺少 Java fixture，且不重建发布附件。
- Kodex 清理 `experiments/sqlite-session-fuse/`、两个 `scripts/migrate-*.py` 及 Java fixture 在 `codex/lite`、`kodex` 包下的两个历史路径，保留当前 Kotlin 实现。
- BuildKodex 仅映射历史 `CodexLite`、`Kodex` gitlink；不修改其他子模块或历史文档正文。
- 当前两个仓库工作区干净；基线为 Kodex `a4cb748c`、BuildKodex `f4ac1e3`。
- 路线确定：使用 git-filter-repo 保留提交拓扑并映射 gitlink，校验每个历史树仅包含授权变化。
- 实施计划完成，开始隔离镜像备份与重写；若远端引用变化则停止推送。
- 先备份并离线验证，再按 Kodex、BuildKodex 顺序进行带显式旧 SHA lease 的原子推送；不创建普通提交。
- 本机备份位于 `/tmp/kodex-history-cleanup-2MwVoKAs`，清理前需再次确认；其他克隆、GitHub 缓存不在可保证的清理范围内。
- 两个远端镜像及 bundle 备份完成并通过 bundle verify；Kodex 25 个引用、BuildKodex 2 个引用，备份期间与远端及本机 HEAD 一致；发布元数据已保存。
- 重写前完整 gitlink 检查发现六个历史目标不在 Kodex 现有分支/tag 的可达历史中，但本机对象仍在：`9508e12e`、`c4745143`、`ac178474`、`ab931a1e`、`101519eb`、`ca7ab19a`。
- 用户选择不增加 tag，允许把六个孤立 gitlink 替换为相近可达提交；六个原快照另存本机镜像及 bundle，不增加公开引用。
- 按移除目标文件后的差异路径数最少、再按提交时间距离最小选择替代：`101519eb → 210044d0`（3 路径）、`9508e12e → eb6a74f0`（56）、`ab931a1e → 1d148000`（1）、`ac178474 → dd9bd0e5`（56）、`c4745143 → dd9bd0e5`（46）、`ca7ab19a → 444f4d3f`（同树）；最终 gitlink 使用这些替代提交的重写后 SHA。
- 隔离重写完成：Kodex 263 个提交（262 个 SHA 变化）、BuildKodex 227 个远端可达提交（全部变化）；逐一比对文件树、作者/提交者/时间、提交信息与父提交拓扑，仅有授权文件删除、gitlink 映射与签名移除。
- 两个隔离仓库 `git fsck --full --no-reflogs` 通过，引用名称集合不变，当前 Kodex 文件树完全不变，外层最新 gitlink 与新产品 main 一致。
- 两仓库均通过 dry-run，随后依次用逐引用显式 `--force-with-lease=<ref>:<old SHA>` 与 `--atomic` 推送；每次推送后远端引用逐项核验一致，未新增或删除远端引用。
- 本机 main、tag、origin 引用及外层索引中的 Kodex gitlink 已同步，工作区文件未重置。外层既有 `refs/air-checkpoints/*` 是本机恢复引用，保持原样且未推送；本机 reflog 与临时 bundle 备份也保留。
- 新 main：Kodex `ab543490`，BuildKodex `462b2aeb`；v0.4.3 为 `4468142b`。
- GitHub 全新镜像验证通过：所有目标历史路径消失、6 个 Python/Java blob 版本均不可达、226 处产品 gitlink 均可达；24 个 Release 与 120 个附件元数据及发布正文不变。
- 结果已保存至 [共享记录](../../shared-context/findings/python-java-history-cleanup.md)；回滚备份按约定保留至用户确认清理。
