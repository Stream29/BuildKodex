# Task Tree

- `Trace prompt image codec production consumers`()
- `Distinguish portable capability contracts from platform implementation names`()
- `Confirm withdrawal of the five platform implementation exports`()
- `Apply internal visibility with subagent`()
- `Run centralized API and behavior regression validation`()

# Details

- 用户要求 ToolSearch 交给 subagent 拆分，主 Session 同时讨论下一项；本卡仅讨论图片编解码。
  不修改 Session1026 的 ToolSearch、注册树或消费者写入范围，不运行并发构建。
- 用户已选择收回五个具体实现名的公开承诺，并授权交给 subagent：
  [Session1028](file:///home/stream/.kodex/sessions/1028) 返回编辑回执后，父 Session 已完成下述验收。
  唯一写入范围 utils/images-codec，原则上仅四文件五处 object 可见性；
  禁构建/提交推送，父 Session 集中串行验证。
  实施前 15 文件快照：`file:///home/stream/ACodeSpace/demo/kodex-detekt-998/image-exports-before/`。
- 父复核已确认四文件只做五个 public object -> internal object 的精确替换，
  其余 11 份文件与快照逐字节一致；算法及 GDI+/Korim 回退代码未改。
- 最近报告中 images-codec 有 5 项：JvmImageIoPromptImageTransformer、
  SharpPromptImageTransformer、SkikoPromptImageTransformer、GdiPlusPromptImageTransformer、
  WindowsNativePromptImageTransformer。它们各自仅在 JVM、JS Node、三 Native 或 Windows 有实现。
- 当前生产调用来自后端 ViewImage/ImageGeneration 工具：读本地图片、调整尺寸/格式，
  构造交给模型的图片输入；不是前端图片渲染器。前端 app/rpc 无该 codec 的直接业务引用，
  上轮已解析的 app/application、app/view JVM runtime 图也不含 images-codec。
  以上是当前用途与检查点，不是禁止未来其他业务复用的决定。
- 正常能力入口 HostPromptImageTransformer 已在 specMain 声明 expect，各平台提供 actual；
  PromptImageTransformer 支持依赖注入。KorimPromptImageTransformer 也已有独立 spec 契约，
  此外图片类型、计划与接口在 utils/images。它们不在本次收回的五个名字中。
- 五个具体平台对象的仓库引用仅用于模块内部的平台选择/组合，没有发现外部直接点名调用。
  这不是自动降可见性的授权；本次 internal 化以用户明确收回导出的选择为依据。
  保持统一入口、Korim 和注入能力、行为与平台覆盖，不新增模块或公开替身。
- 代码依据：
  - [工具默认使用统一能力](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/view-image/src/specMain/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageToolApi.kt#L13)。
  - [后端读图与变换](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/view-image/src/commonMain/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageToolClient.kt#L32)。
  - [图片编辑输入处理](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/image-generation/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationToolClient.kt#L101)。
  - [统一 spec 入口](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/src/specMain/kotlin/io/github/stream29/kodex/utils/images/codec/HostPromptImageTransformer.kt#L8)。
  - [现有公共能力测试](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/src/commonTest/kotlin/io/github/stream29/kodex/utils/images/codec/PromptImageTransformerContract.kt#L56)。
- 验收前 42 项为旧检查点，未把子任务预期当作结果。

## 验收 — 2026-10-21 / Session998

- codec metadata、JVM 12 项及 Node 12 项测试通过，Host/Korim/JPEG 断言保留；
  ViewImage/ImageGeneration 共 16 项 JVM 测试及真实消费者编译通过。
- Linux、mingwX64 klib 编译通过；没有在 Linux 主机声称执行 Windows/macOS 测试。
- 五个结构发现实际消除，公共 Host/Korim/IO API 保留，无目标、回退或资源逻辑变更。
  合并批次严格/报告均 34 项，余项不在本任务范围。
- 静态精确替换、平台命令和测试证据：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/parallel-validation-summary.json`。
  未提交推送。
