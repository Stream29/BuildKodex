# Task Tree

- `Freeze the unchanged CLI entrypoint and Native probe result`()
- `Implement the expected entrypoint with the assigned subagent`()
- `Review unchanged startup and exit behavior`()
- `Link the real Native CLI and smoke test isolated commands`()
- `Record scoped acceptance without publishing artifacts`()

# Details

- 用户已授权剩余17项由subagent实施、父Session验收；本组仅1项。
  [Session1054](file:///home/stream/.kodex/sessions/1054)已交付，父Session已验收。
  唯一写范围app/cli/**，原则上只新增specMain Main.kt同FQN
  public expect fun main(args:Array<String>)，原mosaicMain main加actual。
  原文件/函数体、退出码、参数路由、TUI/Daemon与日志生命周期不变。
- 同版Native隔离小样已链接并运行，但不是正式CLI验收。
  保持convention现有entryPoint，不改根convention/build脚本、包名或创建转发入口，
  不internal化、加lint豁免或移动启动机制进spec。
- 子任务不执行构建/测试或提交推送；RPC server和kRPC补丁并行编辑。
  父Session在xiaoxin-ubuntu集中串行metadata/Native编译、真实debug链接，
  在临时HOME/CWD/XDG目录检查help和未知命令的输出/退出码，
  不启动真实TUI/后端、升级用户Home或安装/发布release。
- 基线源码：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/final17-before/`。
  [入口讨论与小样证据](2026-10-22-review-final-spec-api-gaps.md#cli-main--1项)。
- 原Main.kt仅加actual，其余整文件逐字一致；新spec仅同包expected main，
  convention的entryPoint未改。metadata与真实Linux debug链接通过。
- 隔离HOME、USERPROFILE、CWD及XDG目录，实际执行--help/help返回0，
  未知参数返回2，stdout逐字符合旧入口，stderr为空、隔离目录无新增项。
  Native ELF动态依赖无JVM。未启动TUI/后端、迁移用户Home或替换release。
- 一项入口发现已消除，三组严格/报告17→0，未提交推送。汇总：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/final17-validation-summary.json`。
