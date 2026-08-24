# 模型目录与上下文预算

- `openai:model-catalog`维护模型快照和模型标识解析；`openai:models`维护只依赖`ModelInfo`的上下文预算计算；`openai:client`只负责`/models`传输。
- `OpenAiModelCatalog`以Rust对齐的内置目录同步初始化，并在自有的独立协程中刷新远端`/models`目录；成功响应保留模型顺序，并按推理档位稳定保留首个预设后发布；失败保留上一个快照，不得读取Codex CLI的`models_cache.json`。
- OpenAI API兼容client version由Kodex代码显式维护，不得从Codex缓存或配置读取。
- `OpenAiModelCatalog.close()`只取消目录自有协程，不关闭外部注入的`OpenAiClient`。
- `ModelInfo`保留服务端的`default_reasoning_level`和有序`supported_reasoning_levels`；推理档位必须保留未知字符串，旧输入`ultra`读取为`max`且后续写回`max`，Responses API只发送截至`max`的内置档位。
- Kodex不解析上游模型`visibility`；内置和远端目录中的完整模型列表都可用于选择与模型标识解析。
- 模型解析遵循最长前缀匹配，并只允许一次provider namespace回退；未知模型使用Codex的272000窗口、95%有效窗口回退。
- 普通Responses只根据OpenAI实际报告的`token_count`推进计数；缺失值不新增change point，也不做本地估算。每个成功compaction checkpoint例外地写入synthetic `0`，预算消费者直接使用该值，直到后续普通response报告新计数。
- 自动压缩与`get_context_remaining`必须复用同一套模型窗口和阈值计算，避免两条路径的预算不一致。
- `get_context_remaining`保持普通JSON function tool形态，构造时只接收剩余预算查询；Direct模式按Codex协议返回文本，未知预算明确返回unknown文本。
