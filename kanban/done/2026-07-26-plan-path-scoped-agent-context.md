# Task Tree

- [done] 为基于请求工作目录的 Agent context 提供可配置来源
  - [done] 确认 `Context sources` 一级设置入口
  - [done] 确认内置来源及固定顺序
  - [done] 确认自定义来源管理交互
  - [done] 确认来源读取与生效语义
  - [done] 复审现有注入链路与运行时阻塞点
  - [done] 将目标路径语义收敛为请求 `cwd`
  - [done] 确认同名 skill 按来源顺序后项覆盖前项
  - [done] 确认目录重叠时项目作用域优先
  - [done] 固定外部 Codex 数据源
    - [done] 从全局设置模型和 YAML 写入模型移除 `codexHome`
    - [done] 从 General 页面和 ViewModel 移除 Codex Home Path Picker
    - [done] 将认证、MCP 导入和 context source 接到固定 `~/.codex`
    - [done] 移除 `CODEX_HOME` 解析和公开 `codexDirectory` 参数
    - [done] 保留内部测试依赖注入入口
  - [done] 持久化 context source 配置
    - [done] 增加五个内置来源的启用状态
    - [done] 增加保留原始路径的自定义来源列表
    - [done] 增加自定义路径解析、规范化和重复检测
    - [done] 兼容并自然清理旧 `codex_home`
  - [done] 统一构造请求级来源计划
    - [done] 按设置快照和请求 `cwd` 解析有序来源
    - [done] 统一供 `AGENTS.md` 与 skills 发现消费
    - [done] 保留现有作用域、预算、警告和最终 skill 排序语义
    - [done] 按规范化目录身份处理跨来源重复
  - [done] 实现 `Context sources` 设置页
    - [done] 在 General 后增加一级导航项
    - [done] 渲染只可启停的内置来源
    - [done] 渲染可启停和删除的自定义来源
    - [done] 增加手填路径弹窗及行内校验
    - [done] 保持设置即时持久化
  - [done] 更新规范与验证
    - [done] 更新全局设置、Codex 数据源和 Agent context 文档
    - [done] 覆盖设置默认值、持久化和旧字段迁移
    - [done] 覆盖固定 Codex 路径及内部测试注入
    - [done] 覆盖来源顺序、禁用、缺失和重复目录
    - [done] 覆盖 Settings ViewModel、弹窗和页面交互
    - [done] 运行格式化、相关模块测试和项目检查

# Details

- 状态：Done；已按确认路线完成实现和验证。
- “目标路径”正式定义为每次模型请求使用的 `KodexAgentSettings.cwd`。
- 每次请求在调用模型前读取一份全局设置快照，并据此重新解析 context。
- 不解析工具参数中的文件路径，不在工具副作用前插入新的模型请求，也不改造 `Tool.handle` 协议。
- Settings 左侧一级导航在 `General` 后增加 `Context sources`，后续依次保留 `OpenAI`、`MCP`、`Hooks`、`Current session` 和 `New session`。
- 页面将内置来源与自定义来源分组；一个来源统一控制 `AGENTS.md` 与 skills，不拆分开关。

## 配置模型

- `KodexGlobalSettings` 增加有默认值的 `contextSources`，并移除 `codexHome`。
- `contextSources` 保存五个内置来源的布尔启用状态和有序自定义来源列表。
- 自定义来源仅保存用户输入的路径字符串和启用状态；列表顺序就是添加顺序。
- `settings.yml` 使用 `context_sources` 节点；所有内置来源默认启用，自定义列表默认为空。
- YAML 继续宽松读取。旧 `codex_home` 被忽略，并在下一次正常设置写入时从规范文件中移除。

## 固定 Codex 数据源

- 外部 Codex 数据源严格解析为当前用户目录下的 `~/.codex/`，不读取 `CODEX_HOME`。
- General 页面不再显示 Codex Home，也不再为它维护设置状态或 Path Picker。
- `KodexApplication.open` 不再公开 `codexDirectory` 参数。
- 应用组合根只解析一次固定 Codex Home，并直接注入以下只读消费者：
  - 选择 Codex 认证源时读取 `auth.json`。
  - 用户明确执行 MCP 导入时读取 `config.toml`。
  - 启用 Codex home context source 时读取上下文。
- 认证状态只监听 `authSource`，不再监听可变 Codex Home。
- 测试通过模块内部依赖注入传入临时目录，不通过公开应用设置或环境变量重定位。

## 内置来源

- 内置来源默认全部启用，只能启用或禁用，不得添加、删除、改路径或调整顺序。
- 固定顺序为：
  1. `Agents home`：`~/.agents/`。
  2. `Kodex home`：`~/.kodex/`，实际使用应用 `dataDirectory`。
  3. `Codex home`：固定 `~/.codex/`。
  4. `Git root`：动态 `<git-root>/`。
  5. `Working directory`：动态 `<cwd>/`。
- 允许禁用全部内置来源。
- `<git-root>` 和 `<cwd>` 只持久化动态来源身份，不保存某次 Session 解析出的绝对路径。

## 自定义来源

- 自定义来源单独显示在三个 Home 来源之后、Git root 与 cwd 之前。
- 自定义来源始终作为全局来源生效；新增项按添加顺序追加，不提供排序。
- 页面标题行提供 `Add source`；添加时打开单行路径输入弹窗，不使用 Path Picker。
- 输入接受绝对路径、`~` 和 `~/...`，不接受相对路径，也不展开 `$HOME` 等环境变量。
- 输入目录可以暂时不存在；缺失或不可读时保留配置并在本次解析中跳过。
- 空白、非法或重复路径在弹窗内显示错误且不关闭弹窗；若重复来源仅被禁用，则重新启用已有项。
- 自定义来源行提供统一启停 Checkbox 和 `Remove`；不提供路径编辑。修改路径时先移除再重新添加。
- `Remove` 只更新设置，不修改来源目录，不要求二次确认。
- 路径保留用户输入形式用于展示；运行时展开并规范化后解析和去重。

## 路径校验与去重

- 空字符串、相对路径和 `$HOME` 等环境变量表达式无效。
- `~` 与 `~/...` 使用当前用户目录展开；其他输入必须是当前平台的绝对路径。
- 已存在路径使用文件系统解析后的目录身份；缺失路径使用展开后的词法规范路径。
- 新增路径与任一静态内置来源或自定义来源重复时，不创建新项；若已有项被禁用，则重新启用该项。
- Git root 与 cwd 属于动态来源，只能在请求解析时参与去重。
- 同一目录同时命中全局来源和项目来源时保留项目来源；同一作用域内保留顺序更靠前的来源。
- 缺失路径只能按词法规范结果去重；目录出现后，每次请求会按文件系统解析结果重新去重。

## 请求级来源计划

- 完整来源顺序为三个 Home、自定义来源、Git root、cwd；项目来源保持最后且更具体。
- Home 与自定义来源读取直属 `AGENTS.md` 和 `skills/`。
- Git root 与 cwd 读取直属 `AGENTS.md`、`skills/` 和 `.agents/skills/`。
- Git root 不存在时不报设置错误，该来源只在本次解析中不贡献内容。
- `AgentContextPrefixResolver` 为一次模型请求构造一份共享的已解析来源计划，避免 `AGENTS.md` 与 skills 各自推导出不同目录集合。
- `AGENTS.md` 保留全局内容不计项目预算、项目内容共享现有预算的规则。
- skills 保留 User/Repo scope、扫描限制、警告和最终排序规则。
- 不同来源出现同名 skill 时，来源顺序后项覆盖前项；项目来源位于全局来源之后，最终只向模型暴露一个同名项。
- 设置变更即时持久化，并从下一次模型请求起重新解析；这也包括运行中 turn 的后续模型请求。
- 页面不显示实时文件或 skill 数量，因为动态项目来源会使该数字随请求变化。

## 阻塞点复审

- 原阻塞点是工具调用已经由模型生成后，运行时无法在副作用前让模型读取新路径规则并重新决策。
- 将目标路径定义为请求 `cwd` 后，现有模型请求前缀边界已经满足生命周期要求，因此不存在架构级硬阻塞，也不需要工具目标提取或重试协议。
- 固定 Codex Home、配置持久化、设置 UI 和文件系统发现均有现成扩展点，不构成阻塞。
- 来源冲突语义已经确定：同名 skill 按来源顺序后项覆盖前项，同一目录跨全局和项目来源重叠时采用项目作用域。
- 主要实施风险是两个发现器的来源漂移、跨平台路径规范化，以及缺失路径暂时无法识别文件系统等价关系；统一来源计划和对应单元测试负责约束这些风险。

## 验证范围

- 设置存储：
  - 验证默认五项启用、自定义项顺序和启停状态可往返。
  - 验证旧 `codex_home` 可读取且下一次写入后消失。
  - 验证无效路径、等价路径和重新启用已有项。
- Codex 数据源：
  - 验证认证、MCP 导入和 context source 使用同一固定目录。
  - 验证公开应用入口不再接受 Codex Home 覆盖，内部测试仍可隔离文件系统。
- context 发现：
  - 验证各内置开关、自定义来源顺序、缺失目录跳过和动态 Git root。
  - 验证全局与项目目录冲突时项目作用域生效。
  - 验证同名 skill 只保留来源更具体的一项。
  - 验证同一请求中的 `AGENTS.md` 与 skills 使用相同来源计划。
- Settings：
  - 验证导航顺序、内置项限制、自定义项增删启停和弹窗错误。
  - 验证 General 不再显示或响应 Codex Home。
- 完成实现后运行相关模块测试、格式化检查和项目级 `check`；执行 Gradle 时复用当前 Daemon JVM。

## 完成记录

- JVM 相关模块测试通过，包含 Agent context、Settings、认证、应用和状态模块。
- Linux x64 相关模块测试编译通过。
- 相关模块 `check` 通过，包含 JVM、Linux x64、JS 和可用的跨平台目标检查。
- `git diff --check` 通过；未创建或保留临时文件。
