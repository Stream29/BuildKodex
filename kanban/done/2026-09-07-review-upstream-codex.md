# Task Tree

- [done] 更新官方 Codex 并审查与 Kodex 的差异
  - [done] 读取当前约束、既有设计与工作区状态
  - [done] 更新官方子模块并确认比较区间
  - [done] 审查近期功能、请求缓存和模型可见上下文
  - [done] 保存有源码依据的结论并向用户汇报

# Details

- 用户授权更新 `shared-context/codex` 并开展审查；不授权修改 Kodex 实现或创建提交。
- 明确排除上下文换窗；既有设计差异不自动视为缺陷。
- 保留现有 Kodex UI 修改和其他任务文档修改。
- 路线：比较旧子模块提交与最新 `origin/main`，再沿 Kodex 请求、历史、工具与压缩路径核对；不以静态差异冒充性能实测。
- 更新前确认子模块干净；更新后验证 HEAD 与远端分支一致、祖先关系及 diff 检查。审查成果写入 `shared-context/findings/`，无需构建未修改的应用。
- 修改范围限于官方子模块指针、本任务记录与审查报告。
- 官方比较区间：`343074d4207d572809bd8cea15f4be1d09d98e0b` → `8d7cc24a87f4aa66aa434eb4f25f4f4bafc0e0a9`，共 650 个提交。
- 已核对请求投影、推理项、缓存键和 usage、动态前缀、工具目录/MCP 输出、模型能力与压缩保留；追溯到 8 月 31 日保留区重构的边界截断和文本预算变化。
- 只读检查当前 Session 的设置与 reasoning 字段存在性，不输出正文或凭据；没有发起额外模型采样。
- 结果：[模型调用差异审查](../../shared-context/findings/2026-09-07-codex-kodex-model-path-review.md)。
- 复查实际内层 `item.encrypted_content` 后排除“include 为空导致丢推理”的初步误判；不能据此建议修复。
- 用户后续确认压缩原文整条保留是有意简化设计；撤回“回归、应恢复截断”的结论，并同步修正审查报告与当前 checklist。
- 按用户要求建立三个独立讨论任务：[稳定缓存键](../done/2026-09-07-add-stable-prompt-cache-key.md)、[显式 Medium effort](../done/2026-09-07-send-explicit-medium-reasoning-effort.md)、MCP 文本去包装。不将讨论视为实现授权；MCP 文本去包装于 2026-09-07 经用户评估收益不大后丢弃，任务文件已删除，未实施。
- 子模块干净、祖先关系及远端 HEAD 一致性检查通过；报告源码链接检查通过。没有修改 Kodex 实现、创建提交或运行额外模型 A/B。
