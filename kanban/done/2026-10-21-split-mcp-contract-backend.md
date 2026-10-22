# Task Tree

- `Freeze declaration ownership and preserve existing target coverage`()
- `Split projects and switch frontend and backend consumers`()
- `Verify resolved frontend isolation and serial platform regressions`()
- `Record acceptance and preserve remaining narrow API gaps`()

# Details

- 用户授权拆分 `mcp/contract`、`mcp/backend`；父目录仅分组，不留原项目或转发别名。
- contract 保留共享状态、配置、草稿、导入模型、纯校验及 Agent 签名所需的轻量
  `McpService` / `McpClient` / `McpTool` 接口；不带 SDK、transport、process 实现或后端日志。
  管理、OAuth、持久化端口与运行时归 backend。保留包名、类型身份、wire 形状和 KDoc。
- contract 保持原 Host 六目标（JVM、JS、Linux x64/arm64、macOS arm64、Windows x64）；
  backend 保留同样 Host 传输覆盖及 JVM/四 Native 的 cliHost 管理实现，不删 JS stdio 能力。
  本次不解决原有 8 个 cliHost 专属公开声明的 specMain 缺口，不加 stub 或豁免。
- 现有全部脏改动保留；实施前 1513 份源文件校验清单及 MCP 33 份源码快照：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/mcp-before/`。
- 验收：两个项目无反向依赖；前端/RPC 模型的解析后生产图无 MCP backend、SDK 或后端日志；
  metadata、JVM MCP/消费者测试、实际 Node stdio 与代表性 Native 编译；
  检查拆分前后源码、序列化测试及 lint 发现等价。集中串行复用已有 daemon，不提交推送。
- 两模块 metadata/JVM 与原 MCP 43 项 JVM 测试通过；8 个共享/前端项目的生产
  编译/运行图无 backend、SDK 或后端日志，contract 也无 Ktor/进程系统实现。
  纯 ProcessClient/Shell 端口仍经 Tool 签名传递，不能误写成共享层没有相关领域概念。
- 定向 JVM 最新 492 项为 491 通过、1 项旧失败：RPC models 字段清单断言遗漏
  hooks/daemonTtlSeconds，已在干净 d212c43cf 基线精确复现；测试及 BackendSettings
  源码与基线逐字节一致。不删字段或改断言冒充协议兼容修复。
  CLI JVM 和集成测试源码编译通过；未运行真实模型集成测试。
- 两模块 Node 18 项、两模块 JS/Linux、cliHost metadata 和 CLI Linux 编译通过。
  首次离线运行缺 Node 24.16.0，随后联网取得工具并成功运行；未更改全局 Node 选择。
- 拆分暴露聚合 lint 的目录迁移缓存问题：源码内容不变而换模块时，任务仍显示 UP-TO-DATE，
  SARIF 保留旧路径。根任务增加仓库相对源码路径输入，以触发迁移后的正确分析；
  已验证相同内容跨模块移动时重新分析、恢复时命中缓存、无变化时 UP-TO-DATE。
  路径 provider 仅捕获本地 File，不捕获 Gradle script；configuration cache 存储/复用通过。
  首次实现捕获 script 导致缓存存储失败，已修正后重验，未关闭缓存或添加豁免。
- 最终严格/报告均 42 项/13 模块，逐条一致，除 MCP 路径迁移外与拆分前完全相同。
  contract 清零，backend 的原 8 项保留；严格入口仍正确失败，未宣称整仓通过。
  53 个规则样例通过。共享机器样本：跨模块重扫 2.333 秒、恢复缓存 0.740 秒、
  无变化 0.772 秒；不是隔离 A/B 或性能上限保证。
- 初次依赖探针触发 Gradle 跨项目解析锁限制，改为各项目自己的验证任务后通过；
  纯进程端口不等于系统实现，纠正探针过宽断言后重新验收。该探针不写入产品构建。
- 辅助无文件访问审查已返回；其中“适配实现放入 specMain”是对简述的误读，
  实际 spec 只有签名、common 才有实现。采纳依赖闭包/领域端口与 CLI 分发范围的措辞澄清。
- 保护检查：32 份原源码逐字节搬迁，McpManager 按原声明块精确拆分；
  新增 2 项共享序列化回归；13 份消费者脚本仅替换 MCP 项目依赖坐标。
  1462 份范围外文件保持不变；必要的 root lint 缓存输入及其 README 修正单独列入范围。
- 内存低于 1.5 GiB 时脚本取消自身验证，临时源码探针撤除；等待资源回升后重跑通过。
  共享 daemon 忙时未启动竞争构建，未停止 IDE 或用户进程。
- 未提交推送；迁移与前序脏工作树保留。验收、原始日志、模型、基线对照及保护记录：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/mcp-validation-summary.json`。
  临时干净基线和源码探针已清理；模型 init 探针内容随验收记录留存。
