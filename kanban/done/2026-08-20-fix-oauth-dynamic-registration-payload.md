# Task Tree

- [done] 修复 OAuth 动态注册请求兼容性
  - [done] 复现实际注册拒绝
  - [done] 确认注册字段不兼容
  - [done] 精简动态注册请求字段
  - [done] 增加注册兼容性回归测试
  - [done] 运行 OAuth 测试与构建

# Details

- 实际授权服务器元数据只支持 `authorization_code`。
- Kodex 注册请求额外声明 `refresh_token`，服务端明确拒绝。
- Kodex 注册请求还发送可选 `scope`，该服务端严格解码并返回 500。
- 修复限于动态客户端注册 payload；授权请求仍携带发现到的 scope。
- 注册请求保留 `client_name`、`redirect_uris` 和显式的公共客户端认证方式 `none`。
- 省略具有标准默认值的 `grant_types`、`response_types`，并省略非必要的 `scope`。
- 实际授权服务器已接受最小注册请求并返回 HTTP 201。
- `:mcp-impl:jvmTest`、IDE 增量构建和 Linux Native release 链接均通过。
- 当前 Kodex 进程仍运行重建前的旧 inode；需要重启后才能使用修复后的注册请求。
