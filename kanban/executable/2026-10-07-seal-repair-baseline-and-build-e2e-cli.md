# Task Tree

- `Confirm scoped commit, evidence sealing and binary-build authorization`()
- `Commit reviewed repairs in coherent, path-limited batches`()
- `Seal code, review provenance and baseline validation results`()
- `Build the complete Linux x64 release CLI from the detached baseline`()
- `Verify executable, checksums and isolated launch instructions`()
- `Hand off the fixed binary for full end-to-end acceptance`()
- **`Execute the full E2E matrix on this unchanged binary`()**

# Details

## Scope and authorization

- 用户要求先分批提交、封存现场，再构建完整二进制以开展端到端验收。
  本轮授权本地提交与验收产物准备；不包含推送、版本升级、tag 或 GitHub Release。
- 构建当前主机 Linux x64 的完整 Native CLI，不是测试 harness 或替代实现；
  不把单个架构产物称为四平台发布。默认 `.local/bin/kodex-cli` 和用户运行中
  进程保持不动，提供独立产物与隔离启动入口。
- [修复与独立复审](../done/2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)
  的已确认修复范围现获封存提交授权，不等同所有 B2/U 或完整 E2E 已验收。

## Fixed code and submission slices

- 外层记录提交 `08f7839`、gitlink 提交 `1fd584b`，均本地签名提交，
  未推送。构建/封存回执单列为第三批，不改动代码基线；
  最终外层 SHA 由包内 manifest 记录，避免文档提交自引用。

| 内层提交 | 责任 |
| --- | --- |
| `13288e08` | 平台/Home 真实清理、等待屏障、Path/Patch 归属与相关依赖退役 |
| `f3d1f833` | State 最新 settings、持久化失败后的缓存/CAS 权威 |
| `93e304ce` | 按需后端、MCP 唯一准入/身份发布、认证和 RPC 失败边界 |
| `0f17fb6e` | 前端父 owner、错误出口、准确 History intent 和保留的 UI 回归 |
| `7b23c481` | 收紧未支持能力宣称，保留 wire 与人工回答路径 |

- 最终代码基线为 `7b23c481`，内层须干净；完整 SHA 与三个干净子模块 pins
  写入封存 manifest。外层验收/审查记录与 gitlink 分开提交，只纳入本批文件。
  用户其他改动不 stash、不提交、不复制内容进封存包。
- 原修复基线 `6b7129fa`、固定快照及逐轮 red/green 结论保留；
  1152 项 JVM 结果包含 up-to-date，不升级成全部 forced rerun。

## Sealed acceptance package

- 持久位置：`~/ACodeSpace/local/kodex-e2e-20261007-7b23c481/`。
- 保存内外层 Git bundle、三个子模块的准确源码归档、复审快照和本轮日志、
  最终计数及 SHA-256 manifest；不保存用户真实 Home、凭据或无关工作文件。
- Git bundle 校验可读、当前 refs/commit/tree 与 manifest 一致；
  封存前后真实源码、16 个冻结 Home 路径和 bytes、版本/外部 pins 再核对。
- 构建目录是此代码提交的 detached 本地 clone；三个子模块从本机准确 Git
  对象独立 checkout。仅共享 Git 对象读取，不共享源码、build 输出或 `.gradle`
  项目缓存。构建结束删除该临时 checkout，保留最终产物与证据。
- 两个 bundle 已 `verify` 通过且包含完整历史；五个复审目录和 48 份原日志/
  JSON 已迁入持久 `evidence/`。报告中的旧 `/tmp` 路径按原目录名称映射到
  `evidence/reviews/` 或 `evidence/logs/`，原 manifest 哈希不变。
- detached checkout 干净，三个子模块 HEAD 与 gitlink 相同，1292 个当前
  审查文件哈希全部匹配；构建不使用未经提交的用户文件。

## Build receipt and acceptance entry

- 完整新构建通过，**6m39s**；840 actionable tasks，700 executed /
  140 from-cache，configuration cache 关闭。不是复制上一轮已构建产物。
- 代码 SHA `7b23c48139b38c4eb75e2f0914dcd6daf3e32720`；
  原应用版本 `0.4.8` 未变，没有新 tag/release。
- [完整 ELF](file:///home/stream/ACodeSpace/local/kodex-e2e-20261007-7b23c481/artifacts/kodex)
  是 Linux x86-64，73,386,368 bytes，约 70 MiB，mode `0555`。
  SHA-256 `5d9276ecd5781d028a2b71bcc170f4fb5b3924f5e111aaaae08baf34e0d84b87`。
  `ldd` 当前主机依赖全部可用；不宣称所有 Linux 发行版或其他架构兼容。
- 单入口 tar.gz 解包校验与 ELF 相同；完整 Git bundle 已在不依赖原仓库对象
  的临时仓库恢复、核对 commit/tree 并 `fsck` 通过。
- 本轮真实 PTY smoke 通过：初始草稿→Settings→OpenAI→Close→草稿编辑→
  Ctrl+C，退出 0、无 lease 文件、这些路径不创建 sessions。
  smoke 使用临时 Home 并已删除；不是有账号或完整 E2E 验收。
- 专用 launcher 也用相同脚本字节、同一 ELF 和 checksum，在临时包根执行
  上述 PTY 链路通过；实际验证 Home/workspace 重定向，正式验收 Home 未被
  smoke 初始化或写入，避免把 harness 的外部 HOME 当 launcher 结果。
- [启动脚本](file:///home/stream/ACodeSpace/local/kodex-e2e-20261007-7b23c481/run-e2e.sh#L1)
  和[包说明](file:///home/stream/ACodeSpace/local/kodex-e2e-20261007-7b23c481/README.md#L1)
  已准备，启动前核验 ELF checksum；E2E 数据保存到包内独立、可写 Home/
  workspace/results，不向真实 Home 复制凭据。
- `~/.local/bin/kodex-cli` 未替换，仍是上一轮二进制；
  **本批验收必须使用新脚本/新 ELF**。旧产物也封存到 `evidence/`，仅供历史
  现场核对，不作为 E2E 入口。
- 后续在此固定产物执行下表完整矩阵；故障结果另存
  `acceptance-results/`，不修改被 checksum 覆盖的基线、证据或产物。
- 临时 build checkout、smoke Home/PTY 和本轮 idle Daemon 均已清理。
  历史 `/tmp` 证据引用保留为指向持久封存副本的临时符号入口，没有重复数据；
  正式入口是包内 `README.md` / `BASELINE.json`。E2E 完成后删除这些临时入口，
  不删除用户验收 Home 或原工作区。

## Build and acceptance handoff

- 先探测 Gradle Daemon；存在时明确复用其 JVM，没有时使用已验证
  `/home/stream/.jdks/openjdk-26.0.2`，单 worker、8GB heap。
- 完整构建任务 `:app-cli:linkReleaseExecutableLinuxX64`，
  使用 `--no-configuration-cache`；记录命令、JVM/OS、退出码与完整日志。
  这是验收构建，不触发 release-kodex 的版本/发布流程。
- 产物独立保存为 `artifacts/kodex`；验证 ELF x86-64、动态链接依赖、
  可执行权限、SHA-256，并做真实隔离 PTY 启动/Settings/退出检查。
  上一轮二进制不作为本轮新提交构建的替代证明。
- 提供 `run-e2e.sh`、独立 `acceptance-home/` 和 `acceptance-workspace/`，
  默认不读写用户真实 `.kodex/.codex`；真实 OAuth/API key/MCP 外网场景的
  授权和配置在 E2E 阶段单列，不为 smoke 自动复制账户。

## Full E2E matrix to execute on this binary

| 链路 | 验收闭包 |
| --- | --- |
| 启动/Home | 新 Home、升级 fixture、双进程租约、关闭/重开与未知文件保留 |
| 草稿/Settings | 多草稿、准确 tab、字段持久化、Popup/Login 往返和取消 |
| Agent/History | 真实 append/resume/stop、流式/错误出口、分页/seek/tab 恢复 |
| State/存储 | 压缩期间更新、失败后拒绝陈旧写、关闭重开和合法 fork/delete |
| Shell/工具 | pipe/PTy、stdin/stdout、取消/退出与输出保留 |
| MCP | 正常路由、歧义拒绝、auth/reconnect、旧工具不调用新 owner |
| 后端能力 | 登录/refresh/logout、异步标题、前端关闭后观察与晚到结果 |
| 故障收尾 | 用户取消、错误报告一次、主异常/清理异常、退出后 lease/进程检查 |

- 上表是下一阶段完整 E2E 范围，本轮 smoke 不宣称已覆盖。
  原审查保留的 History admission、历史 timeout 唯一根因和其他平台 U 继续
  跟踪；验收发现新问题以固定二进制和具体复现另立修复，不修改封存基线。
