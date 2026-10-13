# Task Tree

- `Freeze the reviewed candidates and explicit adoption authorization`()
- `Apply canonical versions, build-logic, consumer-local kRPC and no fallback`()
- `Verify the combined full model, routing and representative functions on Xiaoxin`()
- `Commit and push coherent product batches before the context gitlink`()
- `Read back remote commits and release owned resources`()

# Details

- 用户授权「去应用吧，然后提交推送」：采用
  [逐项验证报告](2026-10-11-validate-remaining-gradle-candidates.md)的 A/B/C/D。
- 固定产品基线 `1b6ad5d3922fdfec63a472e01304ecf58d160577`；
  候选来自 Xiaoxin `~/ACodeSpace/demo/kodex-gradle-remaining-20261011/overlays`，
  1513 文件的 `SEALED.sha256` 已重新回读通过。
- 官方 KGP2.4.0、Gradle9.5.1、Koin compiler1.0.1、完整 targets/source sets
  和原最大堆不变。没有 Kotlin fork、实验插件、目标 profile 或发版。
- 组合时 kRPC convention 放到 `build-logic`，根保留三个 Kotlin
  `apply false` 共享类路径声明。其余候选内容保持冻结版本。
- 构建与测试仅在 Xiaoxin 的新私有目录，使用原共享重型锁和资源阈值。
  IDEA MCP 已断开，不启动本地 IDE。完整 GUI 的26GiB启动空间不足，
  原导入/OOM/Json导航边界见已阶段结项的[父任务](../done/2026-10-02-rescue-gradle-development-experience.md)。
- 仅提交本批产品、开发说明、发版路径规范和相关看板；版本不变。
  先推送内层，再提交外层 gitlink。其他任务和参考子模块不动。

## Validation and delivery

- Xiaoxin 新私有根 `~/ACodeSpace/demo/kodex-gradle-adoption-20261011`；
  1313 文件的代码输入从已应用工作区封存。仅开发说明随后补充，不改测量代码。
- `combined-model-routing-r03` 26.736s通过：216项目、1344 targets、
  5026 source sets和全部项目记录等价；仅新增预期 `build-logic` included build。
  KMP实现类计数为1，17实际消费者与306组解析图和原始基线相等。
- 实际加载官方 `kotlin-gradle-plugin-2.4.0-gradle813.jar`；
  校验原始缓存摘要，实验 parser字段和CInterop类均不存在。
- 首两次组合模型运行因验证脚本漏传 routing input/output系统属性失败，
  均在初始化脚本的 `File(null)` 处中止。修正脚本参数后复跑通过；
  不是产品/插件故障，失败结果原样保留。
- `combined-jvm` 121.452s：Migration、Application、CLI、RPC client/contract、
  PathPicker共280项测试强制执行，零失败/错误/跳过；完整CLI JVM编译通过。
- `combined-platforms` 83.246s：8项stdio Node和1项utils patch Linux测试
  强制执行通过；contract、in-memory和Migration的实际Linux Native编译通过。
- 清理每阶段生成物前，将测试XML/reports归档并逐文件校验，只释放本轮私有
  工程的build目录，不删除共享缓存。原资源阈值和共享重型锁保持不变。
- `combined-root-rendering` 121.409s：限定现有root渲染suite的8项集成测试
  强制执行通过，没有运行付费/live模型用例。本轮共297次执行，零失败/错误/跳过。
- 产品五批签名提交并已推送 `main`：
  `fa59711b` 原版本单一来源，`38add004` build-logic迁移，
  `8feef43d` kRPC规则收口，`b82a2168` 明确禁止回退，
  `58be3be4` 开发说明。远端回读准确tip
  `58be3be4e358ee1d556758b9d32b5dfd26529e4f`，内层干净。
- 外层本批分为验证/采用记录、产品指针/规范/父看板两组提交；
  只纳入关联文件，不更改参考仓库、fork pins或发布版本。
- 清理前1313代码输入逐文件回读通过；46证据文件的 `SEALED.sha256`
  全部核对。私有checkout、复制凭据和自建JVM已释放，未删除共享缓存或旧堆。
- 完整GUI本轮未运行，原26GiB磁盘门槛仍阻塞；没有采用私有daemon退出或GC参数。
  Mac/Windows本机执行、四平台release链接及完整导航不算本轮通过。
  父任务继续开放；本卡仅结束已授权A/B/C/D落地，不宣称OOM/导航修复。
