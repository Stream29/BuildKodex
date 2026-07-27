# Task Tree

- [done] 重新设计Session列表与交互
  - [done] 盘点当前Session名称、时间戳、列表和打开路径
  - [done] 审计Codex及其他Agent runtime的会话、fork和multi-agent模型
  - [done] 建立AgentStorage之上的递归CodexSession模型
  - [done] 建立CodexSession存储契约
  - [done] 实现in-memory和filesystem CodexSession
  - [done] 实现无持久化index的Session list投影
  - [done] 将CLI改为Session、Agent tree和Agent runtime三级懒加载
  - [done] 建立虚拟NewSession状态和首次提交物化流程
  - [done] 建立Session菜单、rename和Session browser交互
  - [done] 接入自动Session标题与root Agent标题投影
  - [done] 验证Session存储、迁移、懒加载和主要UI路径

# Details

- 状态：已完成并归档。
- CodexSession以root AgentStorage为根，通过`subagents/`递归表达Agent tree；Session标题来自root Agent当前settings。
- filesystem和in-memory repository提供Session创建、打开、fork、删除、child管理及轻量SessionEntry列表投影。
- CLI冷启动只读取SessionEntry；进入Session后按需展开Agent tree，选中Agent后才打开storage并创建runtime。
- 空repository和关闭Session后进入虚拟`NewSession`；首个有效用户内容才发布真实Session。
- Session UI已经包含菜单、内联rename、Session browser和Agent tree。

以下内容是独立后续重构，不影响本任务完成：

- [Multi-agent V2](2026-07-21-implement-multi-agent-v2.md)已替换第一版公开address/handle、per-node lease和显式release生命周期。
- [恢复AgentStorage id](2026-07-22-restore-agent-storage-id.md)已移除临时runtime请求身份方案并恢复稳定的storage identity投影。
