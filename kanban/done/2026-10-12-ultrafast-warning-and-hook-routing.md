# Task Tree

- `Trace the Ultrafast warning and historical error routing`()
- `Check saved diagnostics and the upstream subscription client`()
- `Review warning classification and the concrete Hook extension`()
- `Implement the approved notification behavior`()
- `Validate routing and normal-response regressions`()
- `Commit and push the scoped Kodex fix`()

# Details

- 用户要求：真正的错误和警告均走通知 Hook；查清正常 Ultrafast 响应为何告警，
  不能仅通过删除提示代替排查。
- [调查证据](../../shared-context/findings/2026-10-12-ultrafast-warning-classification.md)
  已定位字符串比较和 Error-only UI 过滤。最近样本的 Default、Priority、
  Ultrafast 均回报 default；不代表已证明实际路由相同。
- 已确认的通知出口要求替代上次“非错误提示（含 Ultrafast）保留”的边界；
  原任务保留为历史记录，不能继续作为新实现依据。
- 用户最终决定：Warning 走现有 Error Hook，不新增枚举或配置项。
  订阅成功响应的 tier 字符串差异只保留为诊断，不单独产生 Warning；
  真正的本地 Warning 复用 Application 的 error 消息流和唯一消费者，
  stdin 仍为 unhandled_error JSON，不伪造后端 UnhandledError Stop。
- 实施保留 Information 展示；Warning/Error 不占输入框。
  Application 提供明确的本地 Warning 报告入口，共用 Error Hook 选择、
  缓冲和执行器，日志仍区分 warning 与 error。
  删除仅为 tier 字符串比较建立的 cursor/订阅，避免无意义的 timeline 读取。
- 不改 RPC/Hook 序列化、请求、原始诊断、用户配置或安装。
  用户随后授权提交推送本任务代码、文档和子模块指针，不发版。
- 验证要求：Ultrafast/default 成功响应不误警，body/routing hint 不变；
  真正 Error/Warning 不占输入框，共用 Error Hook，日志区分严重性；
  取消不通知、后端已报告失败不重复、历史记录不重放，
  原有 Hook JSON 和旧配置行为不变。
- 已尝试本项目 IDEA MCP，连接 127.0.0.1:64342 失败；
  不控制其他设备或 GUI。共享远端另有活动操作，不在远端竞争构建资源。
- 已删除 tier 比较 cursor 和订阅，正常响应的诊断字段保持不变；
  UI 只显示 Information。Application.reportWarning 共用原 error 消息流，
  日志仍用 warn，stdin/选择项不变。没有新增 RPC 或配置取值。
- 新增/修改回归：六种返回 tier 的正常响应、运行中设置修改后的诊断归属、
  真错误后不追加 tier 假警告、Warning/Error 不渲染、
  真实 Hook 子进程接收重复同文 warning 的原 unhandled_error JSON。
- 本机未发现 Gradle/Kotlin daemon，20 GiB available；
  使用明确 JBR 25.0.4、单 worker、关闭并行、single-use Gradle 验证。
- 第一轮补齐缺失依赖后，RPC 94、Application 46 项全部通过；
  Agent UI 23 项中本次改动测试通过，但原 Steer 用例等待 pendingSteer=2
  超时。第二轮 Hook 14、OpenAI client 76 项全部通过；
  UI 的同一原用例再次超时，不能将整组记为通过。
- 用干净基线 bbbc9525 的私有 detached worktree
  `~/ACodeSpace/local/kodex-warning-baseline-791/` 单独复核该用例；
  两个产品 submodule 以本地固定 HEAD archive 填充，无网络更新，
  不修改主工作区或共享 submodule 输出。

## 验证结果与交付

| JVM 模块 | 测试数 | 失败 | 跳过 |
| --- | ---: | ---: | ---: |
| Frontend RPC | 94 | 0 | 0 |
| Application | 46 | 0 | 0 |
| Hook executor | 14 | 0 | 0 |
| OpenAI client | 76 | 0 | 0 |
| Agent UI | 23 | 1 | 0 |

- 本次新增/修改的回归全部通过。UI 的原 Steer 用例在两轮修改后验证及
  干净 bbbc9525 基线定向验证中均等待 pendingSteer=2 超时；
  基线只有该一项测试，1 failed，不能将完整 UI suite 记为通过。
  本任务不修改这个独立缺陷，也不靠过滤后通过代替全组结果。
- 第一轮 24m35s（含缺失依赖下载），第二轮 3m2s，基线 2m42s。
  证据见 [验证说明](file:///home/stream/ACodeSpace/demo/kodex-warning-hook-791/README.md#L1)，
  保留各轮日志、JUnit XML、源码 patch 与 SHA-256、基线 commit。
- 9 个修改后文件哈希与验证输入一致；另删除 2 个 tier 警告文件，
  scoped diff check 通过。临时 baseline worktree 已删除；
  检查时没有存活的本机 Gradle/Kotlin daemon，不结束其他 Session 的进程。
- 未执行 Native CLI 链接、真实账号速度/路由验证或安装替换；
  未访问用户凭据或运行用户 Hook。验证完成时尚未提交或推送。

## 提交推送授权

- 用户要求“提交推送一下”；仅提交本任务的 11 个产品文件、
  3 个 checklist、调查记录、任务记录和 Kodex 子模块指针。
- 保留其他任务的工作区改动，不顺带提交、不发布版本、不修改安装。
- 提交前 9 个修改后文件与已验证 SHA-256 一致，2 个删除文件仍未恢复；
  两仓库暂存区原为空，scoped diff check 通过。
- 产品已生成签名提交
  [2b16ae50 — fix: avoid false Ultrafast warnings and reuse error hooks](https://github.com/Stream29/Kodex/commit/2b16ae50c5e195908160dd13f1c366c70e1fe80e)，
  并推送到 origin/main；产品工作区干净。
- BuildKodex 本次提交范围固定为 Kodex 指针、本任务和调查记录、
  hooks / rpc-settings / openai-module-boundaries 三个 checklist；
  其他未提交任务保持原状。
