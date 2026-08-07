# Task Tree

- [done] 已认证时隐藏重复登录入口
  - [done] 确认入口当前无条件渲染
  - [done] 明确只在认证不可用时显示入口
  - [done] 调整认证设置 UI 条件
  - [done] 补充已认证与不可用状态测试
  - [done] 运行应用模块定向验证
  - [done] 记录认证入口显示条件

# Details

- 状态：`done`。
- 修改限定在 `AuthenticationSettingsContent` 及其现有渲染测试。
- 用户确认这是 UI 缺少条件判断，不增加切换账号或重新登录交互。
- `KodexAuthState.Authenticated` 继续显示账号与套餐，不显示 `Sign in`。
- `KodexAuthState.Unavailable` 继续显示不可用原因，并提供 `Sign in`。
- 定向验证通过：`JAVA_HOME=/home/stream/.jdks/openjdk-26.0.2 ./gradlew --no-configuration-cache --console=plain :app-cli-application:linuxX64Test`。
