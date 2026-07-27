# Task Tree

- 实现 Codex Lite 私有认证存储与自动续期
  - [done] 审计 Codex 认证文件、续期协议和当前应用装配
  - 实现 `auth.yml` 模型、读取、原子写入和 Codex 首次导入
  - 实现认证状态发布、显式重载和自动续期
  - 接入应用生命周期与 OpenAI Client
  - 补齐真实文件 IO、导入和续期测试
  - 运行相关模块构建与测试并复核修改面

# Details

- `auth.yml` 位于 Codex Lite 私有数据目录，与 `settings.yml` 并列。
- 默认 `type: codex` 直接使用当前 Codex Home 的 `auth.json`，由 Codex 管理续期。
- `type: codex-lite` 将完整凭据保存到 `auth.yml`，由 Codex Lite 管理续期。
- 认证信息不进入 `CodexGlobalSettings`，只有 `codexHome` 继续作为首次导入来源。
