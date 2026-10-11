# Task Tree

- `Trace the error outlets and historical regression`()
- `Restore Agent and Composer error Hook routing`()
- `Validate isolated sources on remote MacBook`()
- `Review scoped changes and report evidence`()

# Details

- 用户授权直接修复，并在 `remote-macbook` 构建和验证；本机另有 History
  展示任务，保留其工作区修改，不运行本机构建，不提交代码。
- 输入框上方不再显示 Agent 操作错误；非错误提示（含 Ultrafast）保留。
- Application 的现有本地报告器接收前端错误；取消不通知。后端已发布
  `Stop.UnhandledError` 的运行失败不能经 RPC 回执再次触发本地 Hook。
- 用明确的远端失败类型区分已发布 Stop 与未通知的拒绝/传输错误，
  不根据消息文本、前端运行状态或事件到达顺序去重。发布仍为 best-effort。
- 回归覆盖真实 Composer 键盘、草稿保留、Session 可用性、取消、
  本地错误 Hook 和后端失败仅一次 Hook，非错误提示仍可见。
- 已发现本机 IDEA 打开项目；IDE MCP 工具未暴露，不控制共享 IDE。
- 远端初查无运行中的 Gradle Daemon；历史 Daemon 使用
  `~/Library/Java/JavaVirtualMachines/graalvm-jdk-25/Contents/Home`。
  验证前再次检测，使用明确 JVM，不创建并行的第二个可复用 Daemon。
- 隔离验证目录：`remote-macbook:~/ACodeSpace/local/kodex-error-hook-748/`。
  源码从 `a8b51e8a` 加本任务白名单文件生成，不包含并行 History 工作区改动；
  `source/SOURCE-748.sha256` 记录本任务文件哈希，日志放在 `evidence/`。
- 首轮编译通过，Agent UI 测试有一个等待条件错误：持久化消息仍出现在
  History，不能等待整屏文字消失来证明 Composer 已清空。改用提交状态退出，
  并保留独立的草稿为空断言。
- 将错误出口设为必填后，第二轮编译检出了 BackendTitle 测试的旧调用；
  已补齐全部构造调用。第三轮构建通过，六组 JVM 共 330 项测试通过，
  macOS ARM64 Release 链接通过（3m54s）。
- 第四轮验证最新 KDoc 与重复同文错误用例，执行 Application JVM 回归、
  integration-test 测试源码编译、五组 macOS ARM64 Native 测试及最终 Release 链接。

## 验证结果

| 模块 | JVM | macOS ARM64 |
| --- | ---: | ---: |
| RPC exception | 18 | 18 |
| RPC server | 139 | 139 |
| Frontend RPC | 91 | 91 |
| Application | 46 | 46 |
| Session | 13 | 未运行 |
| Agent UI | 23 | 23 |
| 合计 | 330 | 317 |

- 上述测试均零失败、零跳过；包含真实 Hook 子进程和 Native 键盘/渲染回归。
- `integration-test:compileTestKotlinJvm` 通过；不将测试源码编译计为测试执行。
- 已验证相同文案的不同失败分别通知，同一次后端失败不附加本地通知；
  取消不通知，普通失败保留草稿/Session，非错误提示继续渲染。
- 本机 19 个修改源码/测试文件与远端 `SOURCE-748.sha256` 一致；
  scoped `git diff --check` 通过。没有运行用户 Hook、访问真实模型或替换安装。

## 完成与交付

- 第四轮完整构建通过（7m6s），包括最新源码的 macOS ARM64 Release 链接。
  未执行人工交互式 CLI smoke；Native 验证为上表自动化测试。
- [验证日志与报告](file:///home/stream/ACodeSpace/local/kodex-error-hook-748/evidence/tests.tsv#L1)
  已复制回本机；同目录保留四轮日志、源码哈希、二进制类型和清理记录。
- MacBook [可执行文件](ssh://stream@macbook/Users/stream/ACodeSpace/local/kodex-error-hook-748/artifacts/kodex)
  为 Mach-O ARM64，55,038,592 bytes，SHA-256：
  `ca8f50fb0716a52defbd9bcac7cb3e01481bba90c9929871bf489d2ae952df0b`。
- 同一 artifacts 目录保留 `source.tar.gz`、`kodex.sha256` 和 `kodex.dSYM.tar.gz`。
  不覆盖远端已有工作区、安装文件或用户 Home。
- 本机临时源码快照和远端临时构建树已删除；本任务启动的 Gradle Daemon
  76019 与 Kotlin compiler daemon 76072 已释放，测试/CLI 子进程均已结束。
- 实现与规范更新未提交、未推送；并行 History 任务文件保持原样，不纳入本次验证快照。

## 后续提交授权

- 用户随后授权由同一执行者负责本任务与 History 任务的分批提交，统一发布
  v0.4.13；本任务验证时“未提交、未推送”为历史状态，不再是当前执行限制。
- 提交前重新核对 19 个文件，均与 `SOURCE-748.sha256` 中的已验证源码一致。
- 合并后的发布门禁见
  [v0.4.13 发布任务](2026-10-12-release-kodex-0-4-13.md)。
