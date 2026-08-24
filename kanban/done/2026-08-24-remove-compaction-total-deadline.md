# Task Tree

- [done] 移除 remote compaction 跨重试总预算
  - [done] 删除 480 秒配置、deadline wrapper 与异常
  - [done] 保留每次 attempt 的 SSE idle timeout 与最多两次重试
  - [done] 更新测试、OpenAI checklist 与回归记录

# Details

- Remote compaction 不再使用跨 attempt 的共享 wall-clock deadline。
- `remoteCompactionMaxRetries` 仍限制最多两次重试，即最多三次实际请求。
- 外部取消、协议错误、非重试 HTTP 错误与每次 attempt 的 transport idle timeout 语义保持不变。
- 已移除 `remoteCompactionDeadlineMillis`、`withTimeoutOrNull` 和 `OpenAiRemoteCompactionV2DeadlineExceededException`。
- 已通过：
  - `:openai-client:compileKotlinJvm`
  - `:openai-client:compileTestKotlinJvm`
  - `:openai-client:compileKotlinLinuxX64`
  - `:openai-client:jvmTest --tests '*openAiRemoteCompactionV2Test*' --tests '*openAiClientConfigTest*'`
  - `git diff --check`
