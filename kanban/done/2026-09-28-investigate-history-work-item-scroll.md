# Task Tree

- `核对既往滚动结论与当前实现`
- `比较 Index 与 Work item 的滚动链路`
- `构造可重复的差异测量`
- `记录根因、证据及修复建议`

# Details

- 用户报告主历史页滚动时，Index item 正常，Work item 显著变慢；本轮要求从看板开始调查原因。
- 本任务限于诊断，不修改滚动行为或生产实现；调查已完成。
- 需区分每个滚轮事件的实际行位移、输入处理延迟、同步测量/绘制和历史窗口加载；不能仅凭视觉速度推断滚动量错误。
- 参考既往调查：`../done/2026-09-01-align-lazy-column-scroll-behavior.md` 与 `../done/2026-09-01-investigate-residual-history-scroll-speed.md`。

## 诊断

- 当前 `AgentHistoryView` 的 LazyColumn 使用反向布局。旧端分页只有边缘 item 已进入可见视口时才发出请求，不因预组合或 overscan 自动提前请求。
- `readHistoryChunk` 对尚未由下一 Index anchor 封口的 Work 后缀，每次只返回一个 Work item；ViewModel `loadOlder()` 每次需求也只调用一次 `readHistoryChunk`，加载结束发布 `Ready` 后才可能继续触发下一次需求。这使一行折叠 Work item 对应一次异步加载回合。
- 每回合还需读取 Work payload 以判断 item 类型；在 RPC 前端上，`work.valuesIn` 无条件发起远程调用，即使此前已经缓存该 item 的值。Index anchor 的 `getExact` 可以命中前端 cache。
- 临时诊断测试在 `app-viewmodel-history` 的 100-item InMemory fixture 上连续读 20 个结构块：纯 Index 共 120 次 timeline 接口调用、0 个 Work payload；未封口 Work 共 140 次接口调用、20 个 Work payload。测试通过，临时代码已撤销。此数据是调用次数，不是实际 RPC 往返耗时或 UI 帧耗时。
- 折叠 Tool/Patch Work 通常只有一行；若对照的 Index 是多行消息，相同的屏幕滚动距离会在 Work 段遇到更多分页边界。边界等待期间当前 materialized provider 不含后续 item，滚轮无法越过该边界；已发生的滚动请求不会在异步加载后自动补足。故体感既可能表现为停顿，也可能表现为位移不足。Index item 本身不保证多行。
- RPC 改造并未引入新的滚轮距离算法；它把原本就串行、粒度为单个 Work item 的存储访问转为多次远程往返及 payload 序列化，足以放大旧分页设计的延迟。现有证据不足以证明这是唯一瓶颈，也不足以给出 RPC、payload 解码、UI 重测三者的实际耗时占比。
- 已有 2026-09-01 调查处理过 LazyColumn item 粒度相关的同步重测延迟；其「远离 History 数据边界」测量不能排除本次 RPC 后频繁触碰 History 边界的问题。

## 建议方向

- 优先在 History 分页层按有限的视觉行或 item 预算批量物化多个 Work 结构块，使单次需求跨过一个滚轮 burst 可遇到的边界，并保持已有有界窗口和稳定锚点语义。
- 另行测量真实 RPC 下的 timeline 方法次数与耗时、分页等待、每轮滚轮实际消费行数，确认是否还需优化 `work.valuesIn` 的 payload 往返或 Mosaic 重测。
- 本轮只调查，尚未实施修复，也未运行真实 CLI（避免触碰用户当前运行中的 CLI）。
- 后续 planning 与修复已登记为 [待启动任务](../discussion/2026-09-28-plan-fix-history-work-scroll-after-rpc.md)。
