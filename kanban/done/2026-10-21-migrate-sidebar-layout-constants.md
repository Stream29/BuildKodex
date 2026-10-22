# Task Tree

- `Trace Sidebar geometry consumers and constant semantics`()
- `Inspect existing JVM constant field ownership`()
- `Confirm source const and behavior compatibility without old JVM field ownership`()
- `Freeze the direct spec relocation and scoped validation manifest`()
- `Migrate the two constants with subagent`()
- `Run centralized compile constant and geometry validation`()

# Details

- 用户要求图片收口同样交给 subagent，主 Session 继续下一项；本卡处理 Sidebar 两个常量。
  本卡不写 Session1026/1028 的产品范围，也不运行构建。
- 用户进一步授权 Sidebar 同样交给 subagent：
  [Session1030](file:///home/stream/.kodex/sessions/1030) 返回编辑回执后，父 Session 已完成下述验收。
  唯一写入范围为 app/component/session-sidebar/view；禁构建和提交推送。
  实施前 4 文件快照：`file:///home/stream/ACodeSpace/demo/kodex-detekt-998/sidebar-constants-before/`。
- 父复核确认原 mosaic 实现仅删除两条常量，所有机制逐字节保持；spec 新增同值声明，
  测试添加 const 初始化表达式及 3/1 断言，原几何断言不变。
- SessionSidebarCollapsedButtonColumns = 3、SessionSidebarCollapsedButtonRows = 1
  是前端 Sidebar 展开按钮的列数/行数，不是后端或领域数据。
  Sidebar 自身使用两者，根 app/view 使用 Rows 来安排收起后的按钮行，
  组件测试验证两个方向按钮共 6 列、1 行。
- 派发前两者 public const 位于 mosaicMain。应保留公共 Kotlin 名称、Int 类型、值、
  编译期常量能力及布局行为，将初始化声明移到现有 View 的 specMain；
  不需要 expect/actual 常量、getter、转发层或新模块。
- 单纯搬到 SessionSidebarApi.kt 会改变 JVM 顶层字段的承载类。
  对当前已构建 class 的 javap 只读检查确认，旧字段在
  io.github.stream29.kodex.cli.app.SessionSidebarKt 中，均为 public static final int。
  Kotlin 顶层 const 的静态字段映射见[官方互操作文档](https://kotlinlang.org/docs/java-to-kotlin-interop.html#static-fields)。
- 用户已选择只保留 Kotlin 源码/const/行为兼容，允许旧 JVM 字段承载类变化；
  不额外维护这两个 SessionSidebarKt 字段入口。该豁免仅适用于两个指定常量，
  不是整个项目放弃二进制兼容，也不是依据没找到调用者擅自变更。
- 最小实施范围：把两条初始化 public const 从 mosaicMain 的 SessionSidebar.kt
  移入现有 specMain SessionSidebarApi.kt，不新增 expect/actual、getter 或转发声明，
  不改 Sidebar 机制、其他符号、模块或目标。保留 public、Int、3/1 及现有 Kotlin 包名。
- 验收需覆盖 metadata、Sidebar JVM 几何测试、根 View 消费者编译及代表性 Native 编译；
  补一个 const 初始化表达式使用样例，防止误降为运行时 val。派发不等于通过验收。
- 代码依据：
  - [两个原常量快照](file:///home/stream/ACodeSpace/demo/kodex-detekt-998/sidebar-constants-before/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt#L863)。
  - [根 View 的行高消费](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L754)。
  - [按钮几何测试](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-sidebar/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SessionSidebarTest.kt#L136)。
  - [现有 View spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-sidebar/view/src/specMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebarApi.kt#L1)。
- 验收前以 42 项为旧检查点，未提前扣除预期变化；父 Session 未运行并发构建。

## 验收 — 2026-10-21 / Session998

- Sidebar metadata、JVM 25 项及 Linux klib 编译通过；根 View、CLI JVM/Linux
  和集成测试源码编译通过，3/1 常量值及原几何机制不变。
- javap 确认字段现在由 SessionSidebarApiKt 持有，仍为 public static final int 3/1；
  private const 初始化用例编译通过，没有新增兼容字段或转发。
- 两条 lint 实际消除；根 View 的两项录制失败与已复现干净基线的名称、异常、
  消息和源码帧完全一致，测试源码哈希未变，未修改旧断言。
- 统一证据：`file:///home/stream/ACodeSpace/demo/kodex-detekt-998/parallel-validation-summary.json`。
  未提交推送。
