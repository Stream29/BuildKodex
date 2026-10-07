# Task Tree

- `Confirm scoped commit, evidence sealing and binary-build authorization`()
- `Commit reviewed repairs in coherent, path-limited batches`()
- **`Seal code, review provenance and baseline validation results`()**
- `Build the complete Linux x64 release CLI from the detached baseline`()
- `Verify executable, checksums and isolated launch instructions`()
- `Hand off the fixed binary for full end-to-end acceptance`()

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
