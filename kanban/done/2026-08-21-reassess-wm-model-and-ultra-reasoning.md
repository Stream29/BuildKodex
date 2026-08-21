# Task Tree

- [done] 退役 Ultra 推理档位
  - [done] 完成模型隐藏与 Ultra 语义讨论
    - [done] 核查当前模型目录的隐藏逻辑
    - [done] 核查 Ultra 的剩余协议与产品语义
    - [done] 比较 Single/Multi Agent 与 Ultra 的职责
    - [done] 确定保留、移除或迁移路线
  - [done] 制定具体修改计划
    - [done] 盘点 Ultra 的完整代码与存储边界
    - [done] 逐项确认阻塞的不确定决策
      - [done] 确定 catalog 预设去重语义
      - [done] 确定兼容性回归测试边界
      - [done] 确定 Settings 并发改动顺序
      - [done] 确定多平台验证范围
    - [done] 定义最小修改文件与顺序
    - [done] 定义自动化与本机验证
  - [done] 实施推理协议规范化
    - [done] 删除 `ReasoningEffort.Ultra`
    - [done] 将旧 wire value `ultra` 读取为 `Max`
    - [done] 更新 serializer 规范化回归
  - [done] 实施 catalog 规范化
    - [done] 移除内置 Ultra 预设
    - [done] 对远端推理预设稳定首项去重
    - [done] 覆盖启动与显式刷新
  - [done] 清理产品层 Ultra 分支
    - [done] 删除 Agent 请求特殊投影
    - [done] 删除 Session 状态显示分支
    - [done] 删除 Global Settings 固定选项
    - [done] 更新 Agent mode 独立性用例
  - [done] 覆盖旧 Settings YAML
    - [done] 验证旧 Ultra 读取为 Max
    - [done] 验证后续写回使用 `max`
  - [done] 同步模型目录决策文档
  - [done] 执行全目标验证
    - [done] 运行受影响模块 `allTests`
    - [done] 运行两个 Mosaic Linux 测试
      - [done] 运行 Settings Linux 测试
      - [done] 确认 Application 既有 hover 断言处理方式
    - [done] 链接 Linux release executable
    - [done] 执行符号与 whitespace 审计

# Details

- 已完成 planning；用户于 2026-08-21 明确授权进入实现。
- 上游模型元数据用 `visibility=list|hide|none` 控制 picker；2026-08-21 的实时响应中，
  `gpt-reserve` 和 `codex-auto-review` 标记为 `hide`，其余 7 个模型标记为 `list`。
- Kodex 的 `ModelInfo` 未保留 `visibility`，JSON 解码会忽略该字段；内置和远端
  catalog 均被完整投影到各模型选项。因此 Kodex 当前不会显式隐藏已知 hidden 模型。
- 完整 catalog 保留 hidden 模型可供已有配置解析上下文窗口；picker 过滤方案未被采用。
- `AgentMode.Single/Multi` 已独立决定 Multi-agent 工具可见性和 developer policy。
- 普通 Agent 请求把 `ultra` 映射为 API `max`；在 Single 和 Multi 下，
  `ultra` 分别与同模式的 `max` 完全等价。
- 内置模型仍将 `ultra` 描述为自动委派，已与显式 Agent mode 冲突。
- 自动标题请求不经过 Agent 请求投影，选择 `ultra` 时仍会直接发送 `ultra`。
- 2026-08-21 使用 Kodex 当前凭据实际请求
  `GET /backend-api/codex/models?client_version=0.147.0`：
  - 返回 `HTTP 200` 和 9 个模型。
  - Sol 与 Terra 均明确按 `max`、`ultra` 的顺序返回两个预设。
  - `ultra` 的远端描述为 `Maximum reasoning with automatic task delegation`。
- 2026-08-21 使用无工具、无并行和最小提示词实际请求
  `POST /backend-api/codex/responses`，直接发送
  `"reasoning":{"effort":"ultra"}`：
  - 返回 `HTTP 400 invalid_request_error`。
  - `param` 为 `reasoning.effort`，`code` 为 `invalid_value`。
  - backend 声明支持值截至 `max`，不支持 `ultra`。
  - 请求在参数校验阶段失败，没有开始推理或产生模型输出。
- 上游 Codex 将 Ultra 作为客户端组合模式：请求 effort 映射为 `max`，同时将
  Multi-agent policy 切换为 proactive；backend 不存在独立 Ultra 推理档位。
- 本机现有 Kodex Session 和全局设置中没有持久化 `ultra`。
- 用户决定继续显示完整模型目录，不实现 `visibility` 过滤。
- 用户决定退役 `ultra`：
  - 直接删除 `ReasoningEffort.Ultra` 类型。
  - serializer 只兼容读取 `ultra`，并将其规范化为 `max`。
  - 远端同时提供的 `max` 与 `ultra` 预设需要在 catalog 投影中去重。
- 用户已明确要求进入 planning，并逐项确认所有阻塞不确定项。

## 代码与存储边界盘点

- `openai:models`
  - 删除公开的 `ReasoningEffort.Ultra`。
  - 保留 serializer 对旧 wire value `ultra` 的读取，并返回 `ReasoningEffort.Max`。
  - 现有 serializer 同时服务远端 JSON、Session JSON 和 Settings YAML。
- `openai:model-catalog:impl`
  - 内置 Sol/Terra 从 `max + ultra` 收敛为 `max`。
  - 远端 catalog 需要消除反序列化后重复的 `max` 预设。
- `agent-state:impl`
  - 删除请求阶段的 `Ultra -> Max` 特殊投影。
- TUI
  - 删除 Session 状态显示和 Global Settings 固定选项中的 `ultra`。
  - Session/New Session 的动态推理选项依赖规范化后的 catalog。
- 测试
  - 更新 models serializer、model catalog 和 agent-mode 工具可见性用例。
  - Settings 文件存储已有适合加入旧 YAML 兼容性用例的测试边界。
- 本机数据
  - 当前 `~/.kodex` Session 和全局设置均未持久化 `ultra`，无需一次性迁移。
- 文档
  - `checklist/model-catalog.md` 的现行规则仍描述 `ultra` 请求投影，实施时必须同步更新。
  - 已完成的历史 kanban 记录保持不变。

## 已由用户决定的边界

- 不实现或解析模型 `visibility`，模型选择器继续显示完整 catalog。
- 不保留弃用期、别名类型或独立迁移层。
- `ultra` 只作为旧输入兼容值；正常解码后统一为 `max`，后续写回自然输出 `max`。
- `ReasoningEffort.Custom("ultra")` 的显式手工构造不增加特殊写入规则。
- 远端 catalog 在启动刷新和显式刷新发布前，按 reasoning effort 稳定去重并保留首项。
  当前真实响应把正常 `max` 放在 `ultra` 前，因此保留 provider 的正常 Max 描述，
  不重写单独 Max 或其他推理预设的文案。
- 兼容性回归覆盖共享 JSON serializer、远端 catalog 去重和旧 Settings YAML。
  Session settings JSON 复用同一个已覆盖的 JSON serializer，不再增加 filesystem 重开用例。
- `SettingsPopup.kt` 的现有 Settings 改动已包含在用户侧 `6f1b6203` 快照提交中；
  `Kodex/` 工作树在实施开始时干净，不再构成实施阻塞。
- 验证采用受影响全目标方案，不降级为仅 JVM 或仅 JVM/Linux。

## 最小修改文件与顺序

- 协议类型与兼容读取
  - `Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ResponsesApiModels.kt:408-457`
    删除公开 `Ultra` 对象，并保持其他 known/custom effort 不变。
  - `Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/OpenAiJson.kt:171-190`
    使用私有 legacy wire 常量把输入 `ultra` 解码为 `Max`；通用写出仍使用对象的
    `wireName`。
  - `Kodex/openai/models/src/commonTest/kotlin/io/github/stream29/kodex/openai/ModelCatalogModelsSerializationTest.kt:12-89`
    断言旧默认值和预设都解码为 `Max`、重新写出为 `max`，未知值继续保留为 `Custom`。
- 内置与远端 catalog
  - `Kodex/openai/model-catalog/impl/src/commonMain/kotlin/io/github/stream29/kodex/openai/modelcatalog/BuiltInModelCatalog.kt:10-71`
    删除 `ultraReasoningLevels`，让 Sol/Terra 直接复用 `maxReasoningLevels`。
  - `Kodex/openai/model-catalog/impl/src/commonMain/kotlin/io/github/stream29/kodex/openai/modelcatalog/OpenAiModelCatalogImpl.kt:18-69`
    在启动和显式刷新发布前规范化每个模型的预设；按 effort 稳定去重并保留首项，
    不改变模型顺序、非重复预设或单独 Max 的描述。
  - `Kodex/openai/model-catalog/impl/src/commonTest/kotlin/io/github/stream29/kodex/openai/modelcatalog/OpenAiModelCatalogTest.kt:81-190`
    更新内置断言，并验证启动刷新和显式刷新都保留首个 Max 描述且只发布一个 Max。
- Agent 与 TUI 消费方
  - `Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexRequestProjection.kt:14-37`
    删除已失去输入类型的 `Ultra -> Max` 分支，直接投影规范化后的 reasoning。
  - `Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeUiPrimitives.kt:103-107`
    删除 Ultra 显示分支。
  - `Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt:1035-1072`
    在保留现有未提交 Settings 改动的前提下，删除 Ultra 显示分支和固定选项。
  - `Kodex/agent-state/tool/src/commonTest/kotlin/io/github/stream29/kodex/agentstate/tool/KodexVisibleToolSpecsTest.kt:42-54`
    用 `Max` 替换测试中的 Ultra，继续证明工具可见性只由 `AgentMode` 决定。
- 持久化兼容
  - `Kodex/app/shared/settings/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/cli/settings/KodexSettingsStoreTest.kt:262-324`
    加入旧 YAML 的 `ultra` 读取与规范化写回用例，不修改 settings schema 或增加迁移层。
- 持久决策
  - `checklist/model-catalog.md:3-8`
    将“远端原样发布”和“运行时 Ultra 投影”更新为 legacy 输入规范化、首项去重和
    Responses API 只发送截至 `max` 的规则。
- 不修改 `OpenAiSessionTitleGenerator`；旧设置经 serializer 读取后已成为 `Max`，
  UI 和内置目录也不再能产生 `Ultra`。
- 不修改历史 `kanban/done/` 记录。

## 自动化与本机验证

- 显式使用当前 Gradle daemon 的 JDK：
  `/home/stream/.jdks/openjdk-26.0.2`。
- 对以下模块运行 `allTests`：
  - `:openai-models`
  - `:openai-model-catalog-impl`
  - `:agent-state-impl`
  - `:agent-state-tool`
  - `:app-shared-settings-filesystem`
- 运行：
  - `:app-view-settings:linuxX64Test`
  - `:app-view-application:linuxX64Test`
  - `:app-cli:linkReleaseExecutableLinuxX64`
- model catalog 的现有真实 endpoint 用例使用临时 `CODEX_HOME` 凭据夹具；只复制测试所需
  字段，并在命令退出时删除，不输出 token。
- 审计 `ReasoningEffort.Ultra` 和 `ultraReasoningLevels` 已完全消失。
- 审计 Kodex 生产和测试源码中剩余的 `ultra`，只允许 serializer legacy 常量及对应
  JSON/YAML 兼容和 catalog 去重用例。
- 对本任务文件运行 `git diff --check`；全工作树检查若受无关未提交改动影响，单独报告。
- 不做数据迁移或模型 picker 手工复核。实施后的验证不再重复真实 Responses 请求；
  backend 不接受 `ultra` 已由上述 HTTP 400 探针确认，其余行为边界由 serializer、
  catalog、Settings 和最终 release 链接覆盖。

## 完成状态

- 所有阻塞不确定项均已确认，具体修改计划已完成。
- Ultra 退役实现、兼容测试和持久决策文档均已完成。
- 验证过程中发现 `:app-view-application:linuxX64Test` 的 51 个用例中有 1 个既有断言稳定失败：
  `historyComposerSeparatorTest.scroll-to-latest button changes from supporting text to bold on hover`。
  - 本任务在同一生产文件中只删除了 `ReasoningEffort.Ultra` 显示分支。
  - 未修改的测试正则要求按钮前 ANSI 序列以 `1m` 结束。
  - 当前已提交组件输出 `22;1;2m`，同时包含 Bold 与继承的 Dim，行为与测试名称一致。
  - 用户授权顺手修复；正则现按独立 SGR 参数识别 Bold，并兼容组合样式。
- 最终聚合验证通过：
  - 五个受影响模块的 `allTests`。
  - Settings 与 Application 的 `linuxX64Test`。
  - `:app-cli:linkReleaseExecutableLinuxX64`。
- `ReasoningEffort.Ultra`、`ultraReasoningLevels`、whitespace 和临时凭据目录审计通过。
