# Task Tree

- [done] 移除持久化请求工具视图并简化工具运行时
  - [done] 移除`KodexAgentSettings.tools`及它的存储真源语义
    - [done] 让旧session中的`tools`字段仍能安全读取或迁移
    - [done] 删除Tool Runtime向settings写回派生tool generation的路径
  - [done] 在每次请求构造时生成`List<ToolSpec>`
    - [done] 合成固定Direct工具定义
    - [done] 按持久化collaboration mode决定`update_plan`
    - [done] 从Tool Search动态取得当前`ToolSearchSpec`
  - [done] 将Tool Runtime收敛为工具调度与执行层
    - [done] 移除`ToolRuntimePlan`、exposure投影和generation持久化责任
    - [done] 调度固定本地工具和当前MCP工具
    - [done] 对已加载但当前不可执行的工具返回明确失败
  - [done] 将Tool Search收敛为动态目录与索引拥有者
    - [done] 合并固定Deferred工具和当前MCP catalog
    - [done] 根据当前searchable sources动态生成`ToolSearchSpec`
    - [done] 每次搜索时检查工具集generation或fingerprint
    - [done] 仅在可搜索工具集变化时重建索引
    - [done] 继续通过`tool_search_output`将命中schema写入history
  - [done] 更新组装、兼容性与回归验证
    - [done] 覆盖固定Direct工具、mode条件和动态Tool Search spec
    - [done] 覆盖MCP catalog变化、索引复用和失效工具调用
    - [done] 更新与新authority边界冲突的checklist决策

# Details

- 状态：已完成。
- 完整`request.tools`是由固定工具定义、持久化mode和当前Tool Search状态推导出的请求级数据，不再进入agent settings时间线。
- 请求构造层只消费`List<ToolSpec>`；不引入`ToolCatalogSnapshot`等额外聚合模型。
- `ToolSearchSpec`的参数结构固定，source description根据当前Deferred工具来源动态生成。
- MCP catalog变化只影响Tool Search目录、索引和执行路由，不将MCP工具schema展开到持久化settings。
- 本任务会取代`kanban/done/2026-07-22-redesign-tool-runtime-settings-source.md`中“完整tool generation写回settings”的已完成设计结论；旧done文件保留为历史记录。
- Linux本地回归、旧session文件兼容读取和CLI mode投影测试通过；完整integration-test中的真实OpenAI探针因统一请求超时未通过，和本任务的本地行为无关。
