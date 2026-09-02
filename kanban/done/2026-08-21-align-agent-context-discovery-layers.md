# Task Tree

- [done] 对齐 AGENTS.md 与 Skill 四层发现
  - [done] 扩展 Agent context 根目录契约
    - [done] 将应用实际 `dataDirectory` 作为 Kodex Home 传入 context settings
    - [done] 保持 Agents Home、Kodex Home、shell 与 Agent `cwd` 的单次请求快照一致
  - [done] 重构 AGENTS.md 发现
    - [done] 按 Agents Home、Kodex Home、最近 Git 根、`cwd` 构造有序根目录
    - [done] canonicalize 并按物理路径去重；无 Git 根时省略 Git 层
    - [done] 每层只读取 `AGENTS.md`，移除 override 与 fallback 候选
    - [done] Agents Home 与 Kodex Home 全量读取
    - [done] 最近 Git 根与 `cwd` 共享 32 KiB 项目文档预算
    - [done] 保留每份文档的 source provenance 与可恢复 warning
  - [done] 调整 AGENTS.md contract 与渲染
    - [done] 表达有序的全局文档和项目文档集合
    - [done] 按四层顺序组合非空内容
    - [done] 保持 AGENTS.md 与 environment context 的 User role 临时注入
  - [done] 扩展 Skill roots
    - [done] 发现 Agents Home 与 Kodex Home 的 `skills/`
    - [done] 发现最近 Git 根与 `cwd` 的 `skills/` 和 `.agents/skills/`
    - [done] canonicalize roots，并继续按 canonical `SKILL.md` path 去重
    - [done] 保留同名不同路径条目与现有 scope/name/path 排序
    - [done] 保留现有递归扫描、frontmatter、fingerprint cache 与 scope/name/path 排序
  - [done] 保持请求生命周期
    - [done] 每次普通 Responses 请求重新解析 AGENTS.md 与 Skill catalog
    - [done] 不把 transient prefix 写入 history 或 remote compaction
    - [done] 继续由 prefix resolver 丢弃发现 warning
  - [done] 补充发现与投影测试
    - [done] 覆盖完整四层顺序和自定义 `dataDirectory`
    - [done] 覆盖 Git 根等于 `cwd`、无 Git 根和 canonical 重复根
    - [done] 覆盖只读取端点、不读取 Git 根与 `cwd` 之间的中间目录
    - [done] 覆盖忽略 `AGENTS.override.md` 与项目共享字节预算
    - [done] 覆盖两种项目 Skill path、同名 Skill 和 catalog 热更新
    - [done] 覆盖同一 turn 的后续 Responses 请求重新读取上下文
  - [done] 检查格式化配置、运行相关跨平台测试和 diff 校验

# Details

- 状态：`done`。四层发现、请求级重新解析、测试和记录均已完成。
- 本任务只调整固定四层的请求级上下文发现，不提供来源设置。
- 基于请求 `cwd` 的可配置来源工作保留在
  [`kanban/done/2026-07-26-plan-path-scoped-agent-context.md`](../done/2026-07-26-plan-path-scoped-agent-context.md)。
- 已将当前请求生命周期与四层发现约定写入
  [`checklist/agent-state-and-runtime.md`](../../checklist/agent-state-and-runtime.md)。

## 已批准的四层模型

按以下顺序构造上下文：

1. Agents Home，默认 `~/.agents`。
2. Kodex Home，即应用实际 `dataDirectory`，默认 `~/.kodex`。
3. 从 Agent `cwd` 向上找到的最近 Git 根。
4. Agent `cwd`。

- 所有层先 canonicalize，再按物理目录去重。
- `cwd` 等于最近 Git 根时，该目录只贡献一次。
- 找不到 Git 根时省略 Git 层，不以 `cwd` 代替。
- Git marker 继续接受现有文件或目录形式的 `.git`。
- 只处理四个端点，不扫描 Git 根与 `cwd` 之间的其他祖先目录。
- Kodex Home 不是全局设置中的外部 Codex `codexHome`；后者继续只服务认证和显式 MCP 导入。

## AGENTS.md 语义

每个去重后的层只尝试读取直属 `AGENTS.md`：

- `<agents-home>/AGENTS.md`
- `<kodex-home>/AGENTS.md`
- `<nearest-git>/AGENTS.md`
- `<cwd>/AGENTS.md`

- 不再识别 `AGENTS.override.md`。
- 不配置其他 fallback 文件名。
- 缺失、不可读或空白文件不贡献模型可见内容。
- Agents Home 与 Kodex Home 不受项目文档预算限制。
- Git 根与 `cwd` 按该顺序共享现有 32 KiB 项目预算；截断行为与 warning 类型保持不变。
- 全局文档按 Agents Home、Kodex Home 排列，项目文档按 Git 根、`cwd` 排列。
- 渲染时继续使用现有 project-doc 边界区分全局与项目内容，并保留每份文档内部文本。
- AGENTS.md 每次普通 Responses 请求重新读取；同一 turn 的工具续跑不冻结旧快照。

## Skill catalog 语义

发现以下 roots：

- `<agents-home>/skills`
- `<kodex-home>/skills`
- `<nearest-git>/skills`
- `<nearest-git>/.agents/skills`
- `<cwd>/skills`
- `<cwd>/.agents/skills`

- 无 Git 根时不构造对应的两个 roots。
- 相同物理 root 只扫描一次。
- Agents Home 与 Kodex Home 的 `skills/` 条目使用 User scope。
- Git 根与 `cwd` 的两类 roots 使用 Repo scope。
- Catalog 继续只包含 frontmatter 的 `name`、`description` 和 canonical `SKILL.md` path。
- 不自动注入所有 Skill 正文，不新增应用级显式选择流程。
- 模型需要正文时继续按 catalog path 按需读取。
- 同名不同路径 Skill 全部保留；只按 canonical `SKILL.md` path 去重。
- 保持现有 Repo、User、System、Admin，再按 name、path 的 catalog 排序。
- 保持最大扫描深度、每 root 目录数限制、frontmatter 限制、fingerprint 失效和有界 metadata cache。

## 请求与 warning 边界

- `AgentContextPrefixResolver`在每次普通 Responses 请求捕获同一份 context settings，再解析 AGENTS.md 和 Skill catalog。
- Skill catalog 保持 Developer role；AGENTS.md 与 environment context 保持 User role。
- transient prefix 不持久化，也不进入 remote compaction。
- AGENTS.md 与 Skill resolver 可继续收集结构化 warning，但 prefix resolver 不向模型、history 或 UI 发布这些 warning。

## 验证

- 六个相关模块的 JVM 测试均通过。
- 六个相关模块的 Linux X64 测试均通过；首次并行构建遇到 Mosaic Native 已生成 cinterop 输出缺失，停止 Gradle daemon 后以单 worker 重试通过，未修改构建配置或源码规避该问题。
- 目标模块没有配置 Kotlin formatter/lint Gradle task，且环境中没有 `ktlint`、`ktfmt` 或 `spotless` 可执行文件，因此未运行格式化命令。
- 已通过本任务修改范围的 `git diff --check` 与未跟踪文件尾随空白检查。
