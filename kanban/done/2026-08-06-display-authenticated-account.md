# Task Tree

- [done] 显示当前认证账号
  - [done] 解析并发布账号邮箱
  - [done] 在全局设置订阅认证状态
  - [done] 展示登录可用性、账号与套餐
  - [done] 覆盖认证解析与设置渲染测试
  - [done] 运行定向 Gradle 验证

# Details

- 用户确认采用认证层公开账号状态、前端直接订阅的最小方案。
- 不在 `OpenAiClient` 增加只转发认证状态的 `whoami()`。
- 保留设置界面当前未提交的 New session 改动。
- 认证模型增加可空邮箱；JWT 优先使用顶层邮箱，再回退 profile 邮箱。
- 设置页显示认证可用性、邮箱或账号 ID，以及已知套餐。
- 验证通过：`git -C Kodex diff --check`、`:openai-models:jvmTest`、`:app-shared-auth-filesystem:jvmTest`、`:app-cli-application:linuxX64Test`。
- `:app-cli-application:jvmTest` 在运行测试前被 Mosaic 现有的重复 `PlatformKt` JVM 类阻塞；Linux X64 目标已完成同模块编译和测试。
