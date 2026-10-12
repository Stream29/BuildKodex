# Task Tree

- `Load authorization, baseline, and utility declarations`()
- `Freeze the eighteen-project source ownership manifest`()
- `Implement utility source-set migration without cross-domain edits`()
- `Check source preservation, dependency direction, and static topology`()
- **`Await repository acceptance of the migration checkpoint`()**

# Details

## 协调者整合回填

- settings/全仓消费者已切换，完整模型、生产图和实际 JVM/JS/Linux main 编译
  已验证。测试与干净基线缺口见 [主任务](2026-10-12-discuss-kmp-spec-impl-source-sets.md#L87)。
- 纯 helper 的真实层次已由协调者补齐，函数体和依赖仍在 commonMain。
  下文静态计数及待协调者/待资源清单是交接时记录；实际检查不外推成
  browser/其他宿主/二进制 artifact 兼容验收。本线继续等待整仓人工验收。

## 授权、基线与独占范围

- 用户授权“全仓迁移执行”，本线仅修改 `Kodex/utils/**` 与本独占 executable。
- 基线 `450e3bd7eeef460c7293d6f6fedb55cc1a4242c5`；保留已有 mutex 试点及其他并行改动。
- 依据：[主任务](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-12-discuss-kmp-spec-impl-source-sets.md#L1)、
  [冻结规范](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/kmp-spec-impl-source-sets.md#L3)、
  [静态审查](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-12-audit-kmp-spec-utilities.md#L27)；
  Draft 为空。
- 不修改根 settings、build-logic、其他域或 checklist；注册和跨域消费者 ID 由协调者统一切换。
- 不运行 Gradle、编译、测试、IDE 或远程重型操作；不提交、不新建 Session、不进入 done。
- 所有文件编辑使用 apply_patch。源码完成不代表构建、平台或人工验收。

## 18 个旧 spec 的执行映射

- S = 原 shared 八目标（JVM、JS browser/Node、wasm browser/Node、四桌面 Native、iOS）。
- H = 原 host 六目标（JVM、Node JS、四桌面 Native）；C = 原 CLI 五目标（无 JS）。
- 以下保持旧项目 ID，唯 filesystem-lease 合并为 `:utils-filesystem-lease`；
  mutex 已是 `:utils-read-write-mutex`，本轮不修改。
- 本线交接时仅有真实契约的项目应用 `kodex.kmp-spec-impl`。
  协调者整合时已为四个纯 helper 补齐同一 convention：原实现和依赖仍在
  commonMain，specMain 没有业务源码，不为满足旧目录名制造接口。
  以下表格的“无契约”不表示最终未接入真实源集层次。

| 旧 spec | 项目去向 / targets | specMain 归属 | commonMain / 下游归属 |
| --- | --- | --- | --- |
| coroutines | 保留 `utils/coroutines/spec` / S | 无自有契约，不造接口 | ChildScope、RunCatchingCancellable 完整辅助实现；原测试 |
| external-url | 保留 spec/impl 两项目 / S→C | OpenExternalUrlResult | 原 impl 工厂、internal expect 和 JVM/native actual 不动 |
| filesystem-lease | spec+impl → `utils/filesystem-lease` / H | FileSystemLease 接口与异常 | 四个 lease 实现/支持文件；commonTest 和 jvmTest 原 scope |
| images-codec | 保留两项目 / H→H | ImageCodecException、UnsupportedImageCodecException | PromptImageCodecIo 读写辅助；原 impl codec/expect/actual/平台组不动 |
| images | 保留 `utils/images/spec` / S | MIME/info/dimensions/limits/models、PromptImages 默认限制、Transformer 契约、请求/EncodedImage、异常；EncodedImage 所需两个 toDataUrl 编码辅助 | 字节读取、格式/尺寸探测、resize/patch 规划、data URL 解码、toPromptImage 执行 |
| kodex-home | 保留 `utils/kodex-home/spec` / H | 无自有契约，不造接口 | 启动时路径求值和环境辅助依赖 |
| kotlinx-io-coroutines | 保留两项目 / S→H | FileFingerprint/FileSystem 与默认方法；RawSource/RawSink/Closeable；默认方法必需 readBytes/writeBytes 及内部 segment 常量 | 独立 use/copyTo 辅助下移；原 impl blocking/posix/actual 不动，impl 的独立 module 常量保留 |
| kotlinx-io-serialization | 保留 `utils/kotlinx-io-serialization/spec` / S | 无新抽象，不造接口 | PathAsStringSerializer 是具体 serializer helper；wire 逻辑不改 |
| ktor-client-ext | 保留两项目 / H→H | HttpHeadersExtensions 的三个规范 header 名 | HeadersBuilderExtensions、SseCompatibility/attribute、SseRequests 同层实现；原 engines/Linux curl 不动 |
| logging | 保留两项目 / S→H | 无自有契约，不造接口 | ScopedLogging 包装实现；原 fileLogging 层不动 |
| os-environment | 保留 `utils/os-environment/spec` / H | 三个 public expect 与 KDoc | requireUserHomeDirectory、环境拼装 helper；原三组 actual 不动 |
| patch | 保留两项目 / H→H | PatchModels 全部 wire/models/sealed Hunk/异常 | FindLineSequence、PatchParser、显式 filesystem apply；impl 默认 filesystem 重载不动 |
| process-client | 保留两项目 / S→H | Command/Session/Client/Exception 与 KDoc | 原 impl 工厂、guard、raw IO、actual、posix/interop 不动 |
| read-write-mutex | 已完成合并 `utils/read-write-mutex` / S | ReadWriteMutex、State 契约 | 原试点工厂、SafeRw、两个测试；本轮不回退或重做 |
| rpc-exception | 保留 `utils/rpc-exception/spec` / C | 六个异常类型（含 sealed RemoteException）的完整层次，message wire 编码 | RestoreRemoteException 是独立 public inline 恢复算法；原测试 |
| search-index | 保留两项目 / S→H | SearchDocument/SearchIndex | 原 impl factory/expect、JS fallback/lucene 分支不动 |
| shell-client | 保留两项目 / S→H | Client/Session/Settings/Shell/StdoutBuffer；DTO 验证、Shell serializer 所需类型识别、snapshot 值语义/渲染及 saturatingPlus 均是模型/合同辅助 | 原 impl 同名 StdoutBuffer 门面、processClient 层、actual/PTY/interop 不动，因独立 module 不发生合并碰撞 |
| terminal-text | 保留 `utils/terminal-text/spec` / S | TerminalCellSegment DTO | TerminalTextWidthBoundary 及 Unicode grapheme/width/prefix/suffix 算法，非传递依赖和测试保留 |

## 已实施声明边界与依赖

- models/default 辅助保持原函数体和 KDoc；不使用 expect/actual 替换普通接口。
- images 的 EncodedImage.toDataUrl 不可反向调用 commonMain；两个基础编码函数保留上游。
  Base64 解码、header parsing、resize 与执行逻辑下游；拆分文件使用不同名字，避免 JVM 门面相撞。
- IO 默认方法依赖 readBytes/writeBytes，故二者和原 spec 内部常量留 specMain；
  use/copyTo 不被契约默认方法调用，移到 commonMain。不合并 IO，不删除 impl 内部常量。
- Shell.Serializer 调用 shellTypeOrNull，snapshot 方法依赖 private saturatingPlus；
  都与合同模型留上游，不拆出向下引用。两个 StdoutBuffer.kt 仍属于不同项目。
- rpc-exception sealed 层次在同一 specMain；Json API 依赖上移，恢复算法下游继承。
- 全契约项目原 commonMain API 依赖移到 `named("specMain")`；不改变 api/implementation 意图。
- mixed 项目分配：codec 的 images API 上移，IO/Path API 留 commonMain；
  patch serialization API 上移，IO/filesystem API 留 commonMain；
  IO 两个 API 全部上移；os 的 Path API 上移；
  Ktor client-core API 上移供 header 合同使用，SSE API 留 commonMain。
- filesystem-lease coroutines API 上移；IO impl API、coroutines/os helper implementation、
  heartbeat JSON implementation 留 commonMain，serialization/tests 插件保留。

## 协调者切换清单

- filesystem-lease 两旧 ID `:utils-filesystem-lease-spec` / `:utils-filesystem-lease-impl`
  → `:utils-filesystem-lease`；根 settings 用新目录注册。
- 扫描时两个跨域消费者仍引用旧 impl ID：
  [agent-session filesystem](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/build.gradle.kts#L16)、
  [app migration impl](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/build.gradle.kts#L31)；
  本线不改它们。旧 impl→spec 自边随合并删除，不创建转发项目。
- 并行复核时已观察到协调者把 settings 注册改为 `module("filesystem-lease")`；
  本线未编辑 settings，不把该观察算作消费者/实际模型验收。
- 其他 utility 旧 ID 暂保留；原依赖层/scope 不作跨域重排。

## 实施与静态结果（2026-10-12）

- [x] 18/18 旧 spec 去向与表内声明归属落实；包括原 mutex 试点，不重复实施。
- [x] 13 个本轮项目 opt in；加原 mutex 共 14 个有真实 specMain 的 utility 项目。
  coroutines、kodex-home、kotlinx-io-serialization、logging 四个 helper 的源码和构建不变。
- [x] filesystem-lease 合并，原接口、四个 common 实现、三份 commonTest、
  一份 jvmTest 逐字节移动；子进程入口文件名、package 和 scope 未改。
- [x] 基线 189 个 Kotlin 文件 → 当前 197 个，增量来自八个输入文件的职责拆分：
  images 五个、IO raw 一个、os 一个、terminal 一个。181 个当前文件与基线某文件逐字节相同；
  其余 16 个文件的业务/KDoc 非 header 行均是对应单一原文件的保序子序列。
- [x] 按 package 比较所有 Kotlin 非空且非 package/import 行的多重集合，完整保留；
  结合保序检查，未删除或改写业务函数体/KDoc。split 只重新分配声明和必要 imports。
- [x] 129 个平台/中间层、测试、interop 输入文件保持字节一致
  （含原 mutex 的两份测试和本轮 lease 移动；不包含 commonMain/specMain）。
- [x] 九个保留 impl 的构建脚本逐字节不变；原 hierarchy、actual、compiler flags、
  cinterop `.def`/package/相对路径与 linkerOpts 不变。目标 convention 和其他原插件列表保留。
- [x] 所有 retained build 的 api/implementation 依赖表达式多重集合保留，
  显式 test dependency block 原文保留；按表内计划上移到 named("specMain")。
  lease 依赖为原两侧并集，删除 self edge，并把重复 coroutines API 合为上游一条。
- [x] opt-in 均先于目标 convention；源码真实在 specMain，不设置额外 commonTest 边或 specTest。
  对本项目 common-only 顶层标识符作去注释/字符串后的 spec 扫描，未发现反向引用；
  另人工核对 images.toDataUrl、IO.readBytes/writeBytes、Shell.serializer/snapshot 依赖闭包。
- [x] IO 两个内部 segment 常量仍在独立 module；Shell 两个 StdoutBuffer 文件门面仍在独立项目。
  split 文件使用不同 basename；未发现同项目 specMain/commonMain 同包顶层文件门面撞名条件。
- [x] utility 构建项目基线 29 → 当前 27（原 mutex 已减一，本轮 lease 再减一）；
  main+test 字面量 project 图共 29 条去重边，均指向存在的 utility build，
  无 self edge、missing target 或环。此图不包括 Gradle 动态/variant 求值。
- [x] `git diff --check`（utils）通过；独占文档已做内容/整文复核。
  HEAD 仍为 `450e3bd7`；只写 utils 和本文件，未干预其他并行领域、checklist 或主任务。
- 检查工具为 Git、只读文本/Python stdin，未生成临时文件或后台资源。
  全库严格空白扫描发现原有 PatchParserRustTest 夹具尾空格；
  该测试与基线字节一致，按保留源码规则未修改；diff 检查和新文档空白检查通过。

## 源码落点证据

- [lease build](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/build.gradle.kts#L1)：
  新项目、plugin 顺序、上游 API 与下游 JSON/helper scope。
- [image contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/specMain/kotlin/io/github/stream29/kodex/utils/images/PromptImageContract.kt#L1)、
  [data URL 合同辅助](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/specMain/kotlin/io/github/stream29/kodex/utils/images/ImageDataUrlContract.kt#L24)：
  EncodedImage 不反向调用 commonMain 编码实现。
- [IO default raw helper](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/spec/src/specMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineRawIo.kt#L7)、
  [独立 raw 算法](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineRawIoHelpers.kt#L7)：
  upstream internal 常量和 use/copyTo 的分层。
- [os expect contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/os-environment/spec/src/specMain/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironmentContract.kt#L5)：
  public expect 上游、拼装和 actual 下游。
- [shell snapshot contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/spec/src/specMain/kotlin/io/github/stream29/kodex/utils/shellclient/StdoutBuffer.kt#L35)：
  DTO 行为和模型辅助不拆成 spec→common。

## 待验证与验收

- [ ] 协调者完成根注册与两个真实跨域 lease 消费者 ID 原子切换，再核对全仓静态图。
- [ ] 待授权资源窗口执行实际 Gradle 模型、metadata/serialization/expect-actual、
  spec/common 可见性负例、main/test friend、发布元数据及 IDE Sync/导航。
- [ ] 八个 split 单元的源代码 API/package/KDoc 保留；拆分顶层函数会改变部分 JVM
  文件门面所在类，需核验重新编译的消费者/产物，不把源 API 保留说成已验证二进制 artifact 兼容。
- [ ] 覆盖 shared 的 browser/Node/wasm/iOS 与原 H/C 桌面平台；模型或文件保留不等于平台编译通过。
- [ ] 测试重点：lease 跨进程 classpath/清理、IO default/copy/use、codec 平台转换/IO、
  Ktor SSE 与 Linux curl cleanup、patch parser/IO、Shell/Path/RPC wire 和取消语义、
  process/Shell PTY/owner 生命周期、search 后端、terminal Unicode 与原 mutex 回归。
- [ ] 真实消费者包含 MCP main/test 的 process/IO、clean-models 的 patch API/test、
  OpenAI client、两个 image tool、tool search、前端及 lease 的两消费者。
- [ ] 待用户人工验收；当前是“utils 源码迁移与静态核验完成”，不是全仓或构建验收完成。
  本文件保持 executable，最后一条 Task Tree 挂起。
