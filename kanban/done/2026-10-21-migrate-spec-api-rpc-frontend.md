# Task Tree

- `Migrate RPC frontend then NewSession with Session1006`()
- `Review subagent delivery within its directory ownership`()
- `Run centralized compilation tests and lint`()

# Details

- 上游：[导出 API 迁移](2026-10-20-audit-exported-api-spec-boundaries.md)。
- 已读取 [Session1006](file:///home/stream/.kodex/sessions/1006) 的最终交付；
  子任务未运行构建，父 Session 已完成本范围的串行验收。
- 唯一写范围：`Kodex/app/rpc/**`、`Kodex/app/component/new-session/viewmodel/**`；
  不包括 `Kodex/rpc/**`，其他目录及既有 26 模块改动保留。
- 按[源集规范](../../checklist/kmp-spec-impl-source-sets.md)先分离 RPC 前端适配 API，
  再处理 NewSession 公开类、构造入口和成员；保留签名、KDoc、行为与平台覆盖。
  不承接 RPC/前端架构重构，不改变 wire/persisted 形状或生命周期。
- 不提交推送，不改共享配置或任务记录，不执行 Gradle/编译器或操作用户进程；
  父 Session 统一审查并串行验证。跨范围/窄平台阻塞需精确记录，不以 stub 或隐藏 API 绕过。
- 父 Session 从 spec 删除 RpcGlobalSettings/RpcSessionBinding 的私有构造参数，
  只保留公开属性和操作；补充 expect class 的窄 beta-warning 确认，不压制匹配诊断。
  修正错误的 SessionTitleSettings 类型 import，将 expect 构造属性改为普通参数加
  独立属性；metadata 编译还要求显式列出 RpcNewSessionSettings 继承的公开成员，
  补齐后保留原来的实际委托与 private 主构造，不增加人工转发实现。
- 两模块 spec/common metadata、JVM、Linux X64 klib 编译通过；
  JVM 测试分别 107/5 项通过，lint 两模块均为 0 项。
  `app-view`、`app-application`、`app-session`、`agent-runtime-composition`
  及 integration-test 测试源码编译通过；未运行真实模型集成测试。
- 首次 metadata 离线解析缺 compiler-plugin 的 stdlib 2.0.0 缓存，
  联网补齐后继续验证；未改产品 Kotlin pin 或压制匹配错误。
  回执为 `subagent-metadata-fixed`、`subagent-native-js`、`subagent-jvm-final`。
