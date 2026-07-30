# Task Tree

- [done] 新增外部 URL 打开工具模块
  - [done] 定义公共全局 API 与结果类型
  - [done] 实现 JVM 与 Native 平台启动逻辑
  - [done] 添加确定性输入校验测试
  - [done] 记录模块边界
  - [done] 运行相关 Gradle 验证

# Details

- 模块只提供打开系统 URL 处理器的能力，不接入登录 UI、ViewModel 或认证流程。
- `Started` 只表示系统 URL 启动器接受了请求，不表示浏览器页面已加载。
- 已验证：`TESTBALLOON_INCLUDE_PATTERNS='*externalUrlTest*' :utils-external-url:jvmTest :utils-external-url:compileKotlinLinuxX64`。
