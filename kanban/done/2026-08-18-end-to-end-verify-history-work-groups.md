# Task Tree

- [done] 端到端验证 History WorkGroup release 二进制
  - [done] 同步当前源码到隔离 MacBook 工作区
  - [done] 使用 Java 25 强制重建 Linux release
  - [done] 校验并传回实际构建产物
  - [done] 使用隔离真实历史验证折叠交互
  - [done] 检查日志并清理临时环境

# Details

- 状态：`done`。实际构建、交互验收、日志检查和清理均已完成。
- 用户要求重新实际构建二进制并执行端到端测试，不复用先前产物作为测试对象。
- 构建遵循 MacBook-first release policy；测试使用隔离 Kodex home，避免修改现有 session。
- MacBook 同步时排除 `.git`、`build` 和既有 `out`，随后删除目标 executable 并使用 `--rerun-tasks` 构建。
- Linux 产物传回后重新计算 SHA-256，并从新隔离目录启动，验证折叠 summary、breaker、展开、收起和旧端滚动。
- MacBook 使用 GraalVM Java 25 执行 Linux x64 release cross-build；491 个任务全部执行，构建耗时 4 分 49 秒。
- 新产物是 67,980,672 bytes 的 Linux x86-64 ELF；MacBook 与 Linux 本机 SHA-256 均为
  `3c8146430d8758d234c9fa33ea9b29ecae2747b554ff0eba29195d8e5339b1bb`。
- 隔离验收复制真实 session 80 为唯一 session；fixture 含 13,388 个 stable event、60,574 个文件，占用约 294 MiB。
- 实际界面显示 `Take 8 actions`、正常 `Updated Plan`、`Take 5 actions`，确认 plan update 打断两侧分组。
- 展开 `Take 8 actions` 后先显示固定单行占位，再在约 98 ms 内显示 8 个原始 child renderer；再次点击可收起。
- 展开旧区间 `Take 17 actions` 后，滚动离开并返回仍保持展开状态；scroll-to-end 按钮可返回最新历史。
- 30 次连续旧端滚动均产生新画面且无超时；含 tmux 轮询开销的 smoke latency 为
  24 ms minimum、39 ms p50、51 ms p95、63 ms maximum。该结果不是独立性能基准。
- 大历史打开后的进程 RSS 观测值为 181,276 KiB；测试期间未出现冻结、崩溃或越界。
- 最终 clean-settings run 日志只有完整的 application/session open-close 生命周期，无 error、exception、history error 或残留 lock。
- 测试进程正常退出；本机临时 HOME、临时二进制、tmux socket 和 MacBook 隔离工作区均已删除。
