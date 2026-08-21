# Task Tree

- 让顶栏 Session tabs 响应横向滚动输入
  - [done] 确定输入语义与修改边界
    - [done] 保留纵向滚轮驱动横向容器的兼容行为
    - [done] 只让横向容器消费原生横向滚轮
    - [done] 在 Mosaic 和通用滚动组件修复，不增加 Session tab 特判
  - [done] 扩展 Mosaic 横向滚轮事件
    - [done] 在 `MouseEvent.Button` 末尾增加 `WheelLeft` 与 `WheelRight`
    - [done] 将终端 mouse button code `66` 与 `67` 解析为横向滚轮
    - [done] 将四种滚轮方向统一排除在 pointer capture 之外
    - [done] 覆盖传统与 SGR 编码、modifier 和 capture 回归测试
  - [done] 接入 Kodex 通用滚动组件
    - [done] 让横向 `scrollable` 消费 `WheelLeft` 与 `WheelRight`
    - [done] 保持 `WheelUp` 与 `WheelDown` 对横向容器的兼容映射
    - [done] 让纵向容器拒绝横向滚轮并保持边界零消费冒泡
    - [done] 覆盖方向、interaction 与嵌套冒泡测试
  - 验证 Session tab 集成
    - [done] 用原生横向滚轮访问溢出标签
    - [done] 保持 `Sessions`、`+`、分页键和选中标签可见性行为
    - 在支持 button `66`/`67` 的终端手工验证横向与纵向输入
  - [done] 完成质量检查与决策沉淀
    - [done] 对全部修改文件运行 IntelliJ 错误检查
    - [done] 运行 Mosaic 与 Kodex 受影响模块的定向测试
    - [done] 验证 JVM 与 Linux Native 受影响目标
    - [done] 更新 TUI 交互组件约束并运行 diff 检查

# Details

- 当前 Session tab viewport 使用横向 `ScrollState`，并支持纵向滚轮映射、`PageUp`/`PageDown` 和选中标签自动进入 viewport。
- 通用 `scrollable` 只识别 `WheelUp` 与 `WheelDown`，再把该 delta 应用于调用方声明的逻辑轴；因此顶栏实际由纵向滚轮驱动横向位移。
- Mosaic 的 `MouseEvent.Button` 和终端事件解析目前都没有独立的横向滚轮表示；横向滚动输入不会到达 Session tab 的 `ScrollState`。
- 终端 mouse button code `66` 与 `67` 分别按 `WheelLeft` 与 `WheelRight` 接入；传统编码与 SGR 编码共用该 button 映射。
- 横向容器同时接受原生横向滚轮和既有纵向滚轮映射，避免改变当前鼠标滚轮体验；纵向容器不消费原生横向滚轮。
- 所有滚轮事件在 `ScrollableState` 实际消费非零 delta 后才停止冒泡；边界事件继续交给祖先处理。
- 横向滚轮和纵向滚轮一样没有可靠的 release 配对，因此不能建立 pointer capture。
- `SessionTabBar` 已使用通用 `horizontalScroll`，生产实现不需要业务特判；只补充集成验证。
- 最小生产修改范围是 Mosaic 的 terminal event、TTY parser、pointer owner，以及 Kodex 的通用 `scrollable`。
- 用户已明确授权实施；保持现有 `SessionTreeCliScreen.kt` 用户改动不变。
- 已通过 JVM 与 Linux Native 的 Mosaic parser/runtime、Kodex lazy-list，以及 `SessionTabBarTest` 定向测试；IntelliJ 对全部 Kotlin 修改文件未报错误。
- IntelliJ 临时与既有测试运行配置均返回空输出且 exit code 1，已使用同一 Gradle daemon JDK 的定向 Gradle 测试完成验证。
- 仍需在可上报 terminal mouse button `66`/`67` 的实际终端中手工验证；任务因此保留在 `kanban/executable/`。
