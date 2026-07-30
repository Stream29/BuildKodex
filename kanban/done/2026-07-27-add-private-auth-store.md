# Task Tree

- [done] 实现 Codex Lite 私有认证存储与自动续期
  - [done] 审计 Codex 认证文件、续期协议和当前应用装配
  - [done] 实现 `auth.yml` 模型、读取、原子写入和全局来源选择
  - [done] 实现认证状态发布、显式重载和自动续期
  - [done] 接入应用生命周期与 OpenAI Client
  - [done] 补齐真实文件 IO、来源切换和续期测试
  - [done] 运行相关模块构建与测试并复核修改面

# Details

- `auth.yml` 位于 Codex Lite 私有数据目录，与 `settings.yml` 并列。
- `settings.yml.auth_source` 是认证来源的唯一选择：默认`codex`只读当前 Codex Home 的`auth.json`，`codex-lite`使用私有`auth.yml`。
- `auth.yml`只保存 Codex Lite 管理的凭据和续期元数据，不再保存来源类型；Codex Lite 不会写入 Codex 的`auth.json`。
- 凭据本身不进入`CodexGlobalSettings`；仅认证来源作为全局设置持久化。

已验证：`cli-settings-contract`、`cli-settings-filesystem`、`cli-auth-filesystem`和`openai-client`的 JVM 测试，以及`cli-app`的 common metadata 编译。
