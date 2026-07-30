# Task Tree

- [done] 添加自管 OpenAI 凭据登录 popup
  - [done] 定义认证登录 flow contract 与一次性尝试生命周期
  - [done] 实现 localhost PKCE callback、令牌交换和私有凭据原子写入
  - [done] 新建 `cli:settings:login` 的 ViewModel 与 Mosaic popup
  - [done] 在全局设置中暴露认证来源和登录入口
  - [done] 覆盖 flow、ViewModel、popup 与应用组合的相关测试
  - [done] 运行相关 Gradle 验证

# Details

- 登录 URL 仅作为一次性 effect 传递给系统 URL 打开器，不进入可持久化 UI state 或日志。
- 成功登录只写 Codex Lite 自有 `auth.yml`，随后将全局 `authSource` 切换为 `codex-lite`；不写 Codex 的 `auth.json`。
- 已验证 RFC 7636 PKCE、真实 loopback callback、取消收尾、ViewModel effect 与 popup 渲染；Linux x64 编译覆盖 `cli-app`。
