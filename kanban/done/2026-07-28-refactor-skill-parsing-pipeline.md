# Task Tree

- [done] 重构 skill 的解析链路
  - [done] 分离 available catalog 与已选 skill 正文和 resource
  - [done] 以按 `cwd` 解析的不可变结果替换共享解析状态
  - [done] 实现逐请求目录枚举与文件指纹 metadata cache
  - [done] 将完整正文和 resource 读取绑定到本次 source authority
  - [done] 对齐四层 roots、canonical 去重与稳定排序
  - [done] 覆盖解析、热更新、权限和发现测试

# Details

- 状态：`done`。原任务没有独立验收项，其合理范围已由后续 Skill 架构任务完整吸收。
- catalog、完整正文、resource 和 source authority 的边界见
  [`重新设计 skills 上下文来源`](../done/2026-07-22-redesign-skills-context-source.md)。
- 按 `cwd` 解析、逐轮重新枚举、fingerprint cache 和正文按需读取见
  [`简化 skill context 的按 cwd 解析`](../done/2026-07-22-simplify-skill-context-resolution.md)。
- 四层 roots、canonical 去重、同名保留及发现测试见
  [`对齐 AGENTS.md 与 Skill 四层发现`](../done/2026-08-21-align-agent-context-discovery-layers.md)。
- 2026-08-27 复核通过 `agent-context-skill-filesystem`、`agent-context-skill-render` 和
  `agent-context-prefix-impl` 的 JVM tests。
