# Task Tree

- [done] 读取当前 Codex 参考源、Kodex 模型目录和相关约束
- [done] 更新官方 Codex 参考子模块到最新 `origin/main`
- [done] 同步 Kodex 内置模型目录与当前官方 bundled catalog
- [done] 运行模型目录测试并记录升级边界

# Details

- 用户请求及时跟进 Codex 最新版本。
- 官方参考子模块从 `a86631502d49274cb47208925c7d3dcece032029` 快进到 `d42056091aded7feb1d88ac7e83972108b2aa478`；同一时点最新稳定标签为 `rust-v0.159.2`。
- Kodex 的内置模型目录已同步 GPT-6 系列、Daybreak 条目、上下文窗口、comp hash、reasoning presets 和服务档位描述。
- 上游大范围的 WebSocket、MCP 授权、app-server 协议、工具元数据预算与重试策略变更留作后续兼容性任务，不在本次模型目录跟进中直接移植。
- 验证记录见 [Codex upstream refresh](../../shared-context/findings/2026-09-30-codex-upstream-refresh.md)。
