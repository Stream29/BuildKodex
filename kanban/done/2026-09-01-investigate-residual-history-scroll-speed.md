# Task Tree

- [done] 调查最新版对话历史滚动速度差异
  - [done] 复现真实 History 内容与滚轮节奏
  - [done] 对比通用 LazyColumn 回归场景
  - [done] 区分布局消费、历史分页与渲染延迟
  - [done] 汇总证据和后续修正范围
  - [done] 确定残余性能修正设计
  - [done] 实现 Mosaic 脏节点测量缓存
  - [done] 实现目标 subtree 同步重测
  - [done] 验证并否决单帧滚轮预算
  - [done] 验证 History 分页放大因素
  - [done] 完成性能与二进制验收
  - [done] 撤销 Mosaic 通用帧率和滚轮预算
  - [done] 实现按实际行数的预测量窗口
  - [done] 验证普通和反向布局复用预测量
  - [done] 重新完成性能与二进制验收

# Details

- 用户在最新 Linux x64 release binary 的对话历史中仍感觉细碎 item 滚动较慢。
- 调查阶段先只测量；用户随后授权按调查结论修正并验证。
- 测试必须使用 History 的 reverse layout、真实消息 item 结构和真实终端滚轮事件链路。
- 记录每个输入帧的滚轮事件数、请求行数、实际消费行数、anchor 位移、重测次数和
  materialized history 边界。
- 将输入消费不足、同步重测耗时、消息 composition/measurement、终端 draw 和异步历史
  分页分别判断，不能只用最终截图推断。

- 实测结果
  - 使用最新版代码、`reverseLayout`、真实 Mosaic `MouseEvent`分发和三十行视口。
  - 两组内容均为一万行可滚动内容：一万项单行文本，对比一百项百行文本。
  - 同帧注入一百个向上滚轮事件；两组均准确移动三百行。
  - 单行项耗时约`92.7 ms`，发生九次同步重测。
  - 百行项耗时约`3.8 ms`，发生一次同步重测。
  - 临时计数与计时测试已删除，没有保留调查用生产接口或测试。

- 结论
  - 上一轮已经修复滚动距离不足；当前残余现象是明显的处理延迟差异，不是消费行数差异。
  - 细碎 item 更频繁跨出按行缓存，每次跨出都会调用`forceRemeasure()`。
  - Mosaic 当前的 node remeasurement 实际执行完整 root measure、focus reconcile 和 pointer
    reconcile，并非只重测 LazyColumn。
  - 百行 item 不能被部分测量；一个原子 item 会让实际缓存自然超过目标行数，因此同样
    三百行滚动只需要一次同步重测。
  - 当前单 item prefetch只复用下一项的composition和measurement，不会扩展活动测量窗口，
    也不会避免完整 root remeasurement。
  - History 还可能放大该差异：`readHistoryChunk`每次只读取一个结构 chunk，没有按视觉行
    批量读取；细碎历史在 materialized 边界会更频繁等待异步加载。通用 LazyColumn 在远离
    数据边界时已经复现延迟，因此分页不是本次实测差异成立的必要条件。

- 修正设计
  - Mosaic 节点记录最后一次 constraints 和 measurement dirty 状态；constraints 未变且
    节点及后代未请求 relayout 时复用 MeasureResult。
  - relayout、measure policy、modifier 和树结构变化必须沿祖先链标记 dirty，不能复用陈旧
    几何。
  - `Remeasurement.forceRemeasure()`先重测请求节点并按原位置重新 placement；节点尺寸变化
    时退回完整 root layout。
  - 成功局部重测后仍重建全局 focus 和 pointer 投影，但不重新测量无关 sibling。
  - `premeasure`返回缓存后的实际尺寸；LazyColumn沿滚动方向逐项预测量，累计到两个
    viewport行数后停止。
  - 预测量在正式measure前完成新item的composition和measurement，不改变Mosaic时间循环、
    输入顺序或滚动消费量。
  - 只有真实History边界仍产生独立停顿时，才增加有界的多chunk预加载。

- 验收条件
  - 同一帧十个滚轮事件仍完整移动三十行，不受 item 粒度影响。
  - 普通和reverse layout跨缓存边界时复用预测量结果，不重新measure预测量item。
  - 大item达到目标行数后停止继续预组合，不按细碎item数量过量预取。
  - 同 constraints 的 clean sibling 不因 LazyColumn 同步重测而重新 measure。
  - 子节点 relayout、内容变化、constraints 变化和目标尺寸变化均不会错误复用缓存。
  - JVM、Linux Native、Mosaic API check 和 Linux x64 release build 通过。

- 修正结果
  - `forceRemeasure()`在目标尺寸稳定时只重测目标subtree，并按原坐标重新placement；尺寸
    变化时回退完整root layout。
  - 同constraints的clean descendants复用已有测量结果；普通帧布局在测量前保守失效整棵
    树，避免composition更新、viewport变化或slot重用后出现陈旧几何。
  - 曾验证全局滚轮预算、六十帧节拍和仅重新placement的快路；它们分别改变通用输入/时间
    语义或产生陈旧绘制，均已完整撤回。
  - History的旧端和新端demand marker位于LazyColumn数据中并参与overscan；marker被测量后
    会在滚动到边界前异步连续补充结构chunk，现有长History回归还约束窗口峰值不超过四倍
    viewport。没有发现需要另改storage分页的独立证据，本次不扩大History读取批次。

- 验证结果
  - `:Mosaic:mosaic-runtime:jvmTest`、`:app-view-components:jvmTest`和
    `:app-viewmodel-history:jvmTest`通过。
  - `:Mosaic:mosaic-runtime:linuxX64Test`、`:app-view-components:linuxX64Test`和
    `:Mosaic:mosaic-runtime:apiCheck`通过。
  - 普通和reverse layout均预测量缓存外两个viewport；跨窗口时预测量item的measure计数
    不增加。首个预测量item高于目标行数时，不继续组合后续item。
  - Linux x64 release binary构建并在固定`120x30` PTY中完成启动、渲染和Ctrl-C退出冒烟。
  - 二进制为`Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`，大小
    `68,543,064`字节，SHA-256为
    `8e91bed7a8327e83dad4c30ec2bd8635322c8fe698a230fef57323ff8082db0f`。
  - 验证和构建复用IDEA配置的Gradle 9.5.1、Temurin 25环境；全程保持单一daemon
    `93859`。
  - `:app-view-history:jvmTest`仍被用户已有测试fake缺少
    `requestScrollToStorageIndex`阻塞；该接口不属于本次修改，未越界调整。

- 返工决定
  - 用户确认不应通过修改 Mosaic 通用时间循环或全局滚轮预算修正 LazyColumn 性能。
  - 撤销全局限速和输入预算，不再聚合或节流滚轮事件。
  - 根因收敛为预测量不足：原实现只准备缓存外一个item，细碎内容首次跨窗口仍需同步测量约
    一个viewport的新item。
  - `premeasure`现在返回缓存后的实际尺寸；LazyColumn沿滚动方向逐项准备，累计到两个
    viewport行数后停止，大item会自然提前停止。
  - 普通和reverse layout回归均证明跨已测量窗口时不会再次measure已预测量item。
  - 保留通用局部重测和 clean descendant 测量缓存。
