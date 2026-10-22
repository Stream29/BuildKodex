# Task Tree

- `Migrate exported component declarations with Session1004`()
- `Review subagent delivery within its directory ownership`()
- `Run centralized compilation tests and lint`()

# Details

- 上游：[导出 API 迁移](2026-10-20-audit-exported-api-spec-boundaries.md)。
- 已读取 [Session1004](file:///home/stream/.kodex/sessions/1004) 的最终交付；
  子任务未运行构建，父 Session 已完成本模块范围的串行验收。
- 唯一写范围：`Kodex/app/view/components/**`；其他目录及既有 26 模块改动保留。
- 按[源集规范](../../checklist/kmp-spec-impl-source-sets.md)分离 composable、
  扩展、类型和成员的公开声明，保留签名、默认参数、KDoc、行为和目标覆盖。
- 不提交推送，不改共享配置或任务记录，不执行 Gradle/编译器或操作用户进程；
  父 Session 统一审查并串行验证。跨范围/窄平台阻塞需精确记录，不以 stub 或隐藏 API 绕过。
- 父 Session 将定位公式收回 mosaicMain，spec 保留策略接口、值及默认绑定；
  修正 const 和 Compose getter 注解，并以 expect/actual sealed DSL
  保留实现层私有 builder。首次 metadata 编译揭示了遗漏的 getter 注解目标，修正后通过。
- 本模块 spec/common metadata、JVM 与 Linux X64 klib 编译通过，
  167 项 JVM 测试通过，含编译期常量回归；当前 lint 本模块 0 项。
- 真实消费者 `app-view` 编译通过，其全量 JVM 测试为 68 项、2 项失败。
  干净 `d212c43cf` 源码导出定向运行旧录制 suite 得 6 项、同样 2 项失败：
  名称、异常类型、消息与源码位置一致，且测试源码未改。
  不把消费者全量测试记为通过；本卡结项不包含修复已复现的基线缺口。
- 回执位于 `~/ACodeSpace/demo/kodex-detekt-998/subagent-*.{log,json}`，
  基线逐项对照为 `subagent-baseline-comparison.json`；临时基线目录已删除。
