# Task Tree

- `Plan downstream adaptation and compiler error fixes`()
- `Implement main module type adaptations and UUID opt-ins`()
  - `Adapt AgentRuntimeViewModel token count projection`()
  - `Update CachedAgentStorage to use TokenCountSnapshot`()
  - `Add ExperimentalUuidApi opt-in to SessionViewModels`()
- `Update downstream tests to match TokenCountSnapshot contracts`()
  - `Apply kotlin serialization plugin to agent-storage:contract`()
  - `Adapt KodexAgentCompactionRuntimeTest token count assertions and helpers`()
  - `Adapt FileSystemKodexSessionRepositoryTest and InMemoryKodexSessionRepositoryTest`()
  - `Adapt IndexHistoryReadTest mock storage contract`()
  - `Adapt AgentHistoryActionTest and HooksIntegrationTest token count assertions`()
  - `Adapt GetContextRemainingToolBehaviorTest token count setup`()
  - `Adapt integration-test OpenAiClient recording delegates to new createResponse signatures`()
- `Run comprehensive verification across all affected targets`()
  - `Verify local JVM compilation and test suites`()
  - `Verify Linux and native compilation`()

# Details

## Background & Root Cause

- 在前置任务 [kanban/planning/2026-09-20-subscription-usage-cache-and-tool-output.md](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/planning/2026-09-20-subscription-usage-cache-and-tool-output.md#L152) 中，设计要求将 `KodexAgentStorage.tokenCount` 由 `Long` 升级为结构化快照 `TokenCountSnapshot`，并明确提出需同步改动消费端（`CachedAgentStorage`、`AgentRuntimeViewModel` 等）及相关测试。
- 提交 `46b96543` 仅对底层存储、状态机、上下文窗口和迁移模块实施了修改，因验证时仅运行了局部模块测试，遗漏了下游业务模块与测试的类型同步。
- 提交 `4de3591e` 引入 `Uuid.generateV7()`，在开启 `-Werror` 的编译器配置下缺少 `@OptIn(ExperimentalUuidApi::class)` 注解。

## Modification Inventory

### 1. Main Modules

- `file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt#L474`:
  - 将 `mutableTokenCount.value = session.storage.tokenCount[index]` 修改为提取快照 totalTokens：`mutableTokenCount.value = session.storage.tokenCount[index]?.totalTokens`，保持 ViewModel 暴露的 `StateFlow<Long?>` 语义不变。
- `file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt#L40`, `#L54`, `#L82`:
  - 将 `cachedTokenCount` 与 `tokenCount` 的类型参数由 `Long` 改为 `TokenCountSnapshot`，正确实现 `MutableKodexAgentStorage` 契约。
- `file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt#L350`, `#L393`:
  - 在文件头部或包含 `Uuid.generateV7()` 的函数/类上添加 `@OptIn(ExperimentalUuidApi::class)`。

### 2. Test Suites

- `file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/decorator/compact/src/commonTest/kotlin/io/github/stream29/kodex/agentruntime/decorator/compact/KodexAgentCompactionRuntimeTest.kt#L193`, `#L441`, `#L617`:
  - 将直接读取比较的 `storage.tokenCount[5]` 改为 `storage.tokenCount[5].totalTokens`。
  - 将写入辅助方法 `appendUserMessage` 中的赋值修改为构造 `TokenCountSnapshot(TokenCountKind.Response, tokenCount)`。
- `file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepositoryTest.kt#L134`:
  - 将断言改为 `assertEquals(0L, session.storage.tokenCount[0].totalTokens)`。
- `file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/in-memory/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/inmemory/InMemoryKodexSessionRepositoryTest.kt#L88`:
  - 将断言改为 `assertEquals(0L, session.storage.tokenCount[0].totalTokens)`。
- `file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/history/src/commonTest/kotlin/io/github/stream29/kodex/cli/history/IndexHistoryReadTest.kt#L246`:
  - 将桩存储属性改为 `override val tokenCount: IndexVersioned<TokenCountSnapshot> = delegate.tokenCount`。
- `file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/session/src/commonTest/kotlin/io/github/stream29/kodex/cli/session/AgentHistoryActionTest.kt#L72`:
  - 将断言改为 `assertEquals(0L, root.storage.tokenCount[0].totalTokens)`。
- `file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/commonTest/kotlin/io/github/stream29/kodex/integrationtest/HooksIntegrationTest.kt#L110`:
  - 将断言改为 `assertEquals(0L, session.storage.tokenCount[0].totalTokens)`。
- `file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/get-context-remaining/src/commonTest/kotlin/io/github/stream29/kodex/tool/getcontextremaining/GetContextRemainingToolBehaviorTest.kt#L42`:
  - 将写入值包装为 `storage.tokenCount[0] = TokenCountSnapshot(TokenCountKind.Response, 760L)`。

## Verification Plan

1. **本地全量编译**：
   - `./gradlew compileKotlinJvm compileTestKotlinJvm` 确保无任何类型不匹配或警告转错误。
2. **受影响模块回归测试**：
   - `./gradlew :app-viewmodel-agent:jvmTest :agent-session-filesystem:jvmTest :agent-session-in-memory:jvmTest :app-viewmodel-session:jvmTest :agent-runtime-decorator-compact:jvmTest :app-viewmodel-history:jvmTest :tool-get-context-remaining:jvmTest :integration-test:jvmTest`
3. **MacBook 原生编译验证**：
   - 在 MacBook 检出运行 `:app-cli:linkReleaseExecutableMacosArm64`，确保原生 CLI 链接完全畅通。
