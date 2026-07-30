# Codex CLI Storage兼容性

- 将Codex CLI本地设置和Session兼容逻辑收敛在`Kodex/openai/codex-cli-storage`。
- 先根据`CODEX_HOME`或默认规则解析Codex Home，再读取其他设置；不允许持久化override改变自身所在的Codex Home。
- 将Codex原生配置和thread数据视为只读兼容源，不修改、同步或写回原始内容。
- `openai:codex-cli-storage`是Codex本地文件wire格式的唯一解码边界；下游只接收类型化模型，不处理原始TOML、JSON、字段别名或字符串形式的联合类型。
- Hook配置必须在该模块内展开为handler并完成平台命令选择、环境替换、timeout规范化和matcher编译；`cli:settings`只按优先级组合已解码的配置层。
- 解码后的Hook handler必须按受支持事件保存在具名字段中；不得展平后再用事件名称判别和筛选。
- 在`$CODEX_HOME/GlobalSettings.yml`中保存Kodex自有的稀疏全局设置override；缺失字段继续继承Codex值。
- 对`GlobalSettings.yml`使用带schema version的正式YAML模型和原子写入，不使用手写YAML解析器。
- 将全局设置页面的Codex Session导入作为操作入口，不将其存入`GlobalSettings.yml`。
- 将每个Codex thread独立转换为一个AgentStorage，不将subagent历史合并进root AgentStorage。
- 导入用户选择的根thread及其全部可达subagent thread，并根据Codex parent/path信息重建KodexSession树。
- 在隐藏的staging区中构建完整导入结果，只在全部转换成功后向Lite Session repository原子发布。
- 首版不实时挂载Codex Session，不在Codex源上执行resume、fork或archive；导出到Codex由独立未来任务跟踪。
