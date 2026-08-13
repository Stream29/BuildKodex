# Task Tree

- [done] 提取 OpenAI 只读认证端口
  - [done] 建立 OpenAI 认证 contract
    - [done] 新增 `OpenAiAuthState`
    - [done] 新增只读 `OpenAiAuthStore`
  - [done] 迁移应用侧认证实现
    - [done] 让 `KodexAuthStore` 扩展只读端口
    - [done] 迁移 filesystem store 状态类型
    - [done] 注册应用与 OpenAI 两侧接口
  - [done] 迁移 OpenAI 消费者
    - [done] 移除对 app auth contract 的反向依赖
    - [done] 迁移 client 与 account usage
    - [done] 提供 OpenAI client 测试 fixture
  - [done] 收紧 Application contract
    - [done] 定义不含凭据的认证摘要
    - [done] 移除对 app auth contract 的依赖
  - [done] 更新边界文档并验证
    - [done] 记录认证端口与实现归属
    - [done] 编译 JVM 非 Mosaic 与完整 Linux X64 目标
    - [done] 运行相关无网络 Linux X64 测试
    - [done] 记录既有 Mosaic JVM 编译阻塞

# Details

- 用户已确认将 OpenAI 下游所需的只读认证状态提取为 OpenAI contract，由应用 auth 模块实现。
- 当前 `openai/client` 与 `openai/account-usage/impl` 反向依赖 `app-shared-auth-contract`。
- `OpenAiSubscriptionAuthState` 含 bearer access token；Application contract 必须改为去敏认证摘要，不能把该凭据对象暴露给 frontend。
- `OpenAiAuthStore` 不拥有实现生命周期，不继承 `AutoCloseable`，也不暴露 reload 或登录命令。
- `KodexAuthStore` 继承 `OpenAiAuthStore`，继续承载应用侧 reload、登录与 close。
- 本任务不接入其余 ViewModel contracts，不引入 Koin annotations。
- 验证使用 OpenJDK 26.0.2。
- 相关 JVM production 与 test source 编译通过；`app-cli-view-application` 的 JVM 编译仍被既有 Mosaic `PlatformKt` 重复类名问题阻塞，与本次认证变更无关。
- 相关 Linux X64 production 与 test source 编译通过。
- `openai-client-test`、`openai-account-usage-impl`、`app-shared-auth-filesystem`、`app-viewmodel-settings-login`、`app-viewmodel-application` 与 `app-cli-view-application` 的 Linux X64 测试通过。
