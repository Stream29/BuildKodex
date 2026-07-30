# Task Tree

- [done] 规划Session自动标题生成
  - [done] 盘点当前Session name的真源、持久化和UI投影路径
  - [done] 调研Rust主线、启发式分支和模型生成分支的行为
  - [done] 对齐更新后的纯五timeline `KodexSession`模型
    - [done] 明确Session标题恒等于主Agent当前标题
    - [done] 删除独立Session title和metadata authority
    - [done] 让标题生命周期完整跟随root settings timeline
  - [done] 确认默认模型生成基线和触发时机
    - [done] 首条文本用户输入持久化后触发
    - [done] 后台异步任务对齐模型生成分支
    - [done] 生成失败后正常resume永不自动重试
  - [done] 确认其余Session生命周期语义
    - [done] Session始终持有非空标题
    - [done] fork标题添加`[fork]`前缀
  - [done] 固化自动生成与用户rename的优先级和原子更新契约
  - [done] 固化new、fork、resume和multi-agent Session的标题生命周期
  - [done] 固化生成请求的输入、模型、失败、重试与取消语义
  - [done] 固化Kotlin模块边界、交互时序和测试计划
  - [done] 实现并验证Session自动标题生成

# Details

- 状态：规划、实现与跨平台验证均已完成。
- 目标是引入Codex App风格的模型生成Session标题，不直接复制历史Rust分支的存储和任务管理实现。
- planning已完成并获用户确认。
- 实现依赖`2026-07-22-redesign-session-surface.md`中的Session repository、root Agent和settings timeline边界。

## 调研结论

- 固定Rust HEAD和`origin/main`均不发起模型标题请求，只从首条文本用户消息派生title。
- `origin/codex-auto-thread-name`是未合并的本地启发式实验：最多四个词、40字符，不调用模型。
- `origin/fcoury/auto-thread-name`是未合并的模型实验，其prompt和长度规则明确与Codex App对齐。
- 模型实验在首条已接受文本持久化后启动独立后台请求，不等待主turn完成。
- 模型实验默认使用`gpt-5.4-mini`、low reasoning、无tools、最多2,000字符输入和严格JSON schema。
- 模型实验每个live Session只尝试一次；失败静默，不做标题级重试；手工rename通过`IfUnset`仲裁获胜。
- 当前Kotlin将首条文本全文同步写入`KodexAgentSettings.threadName`，且该值随settings timeline被fork和revert。
- 新Session模型没有`session.json`或Session metadata；名称真源只能位于AgentStorage的settings timeline。

## Canonical模型

- Session没有独立title字段或authority；Session标题恒等于root Agent latest `KodexAgentSettings.threadName`。
- `SessionEntry.name`只投影root latest settings；child自己的`threadName`只作为Agent树节点名称。
- Session rename就是root Agent rename。对child rename不会改变Session标题。
- root `threadName`在所有可达快照中必须非空；UI不再提供与持久化值分离的untitled fallback。
- 自动生成资格和attempt token只存在于live root Agent内存，不写入`KodexAgentSettings`或其他文件。
- title不持久化generated/manual provenance；存储层只看到普通`threadName` change point。

## New与迁移

- 新Session在staging发布前写入`threadName = "Session <index>"`，因此从未暴露空标题。
- 新child/subagent不启动Session自动标题生成。
- 移除当前Kotlin“空`threadName`同步复制首条文本”的规则，避免与模型生成并存两套默认命名行为。
- 旧root settings中的非空名称原样保留，不增加自动标题迁移字段。
- 旧root settings timeline的每个空名称change point在迁移staging中规范化为`Session <新index>`；只在末尾追加默认名无法保证checkout后仍非空。

## 触发与提交

- 默认开启模型生成；不采用Rust主线的首条文本派生规则或本地启发式分支。
- 配置启用、root可见history尚无已接受非空文本且标题仍是canonical默认名时，首条非空文本成功持久化后，live gate执行一次`NotStarted -> Running(attemptId, expectedThreadName)` CAS；只有成功者启动后台请求。
- 纯图片输入不认领机会；后续首条文本仍可触发。
- 后台任务的输入、模型、prompt、schema、清洗、one-shot和失败行为以`origin/fcoury/auto-thread-name`为功能基线。
- 网络请求在root Agent writer之外异步运行；模型结果只能通过专用条件写操作进入root settings timeline。
- 条件提交同时匹配live `attemptId`和latest `threadName == expectedThreadName`，基于latest settings copy只修改`threadName`，避免覆盖rename或并发settings变化。
- 生成结果若早于主response结束，排队到root AgentState的下一个稳定提交点；主response不等待标题网络请求。
- 标题提交只追加settings change point，不追加timestamp，不进入模型history，也不改变Session列表时间。
- 用户rename经同一writer写入非空`threadName`并先作废live attempt；即使rename为相同字符串，迟到生成结果也不能覆盖。
- checkout和revert先作废live attempt；`threadName`随后完全按目标settings快照恢复，旧分支结果不得落盘。
- 生成成功或失败后live gate都进入`Finished`，不再自动尝试。
- 触发请求的首条用户文本已经持久化，因此正常resume可从history判断机会已经用完，不恢复attempt也不重试。
- 生成失败保留当前非空标题；后续是否改名完全由用户rename决定。
- destructive revert若删除了首条非空文本，也同时删除了唯一durable触发事实并将live gate重置为`NotStarted`；后续输入按该timeline新的首条文本处理，可再次生成。

## Fork与生命周期

- fork读取所选Agent在fork boundary可见的`threadName`，不是源Session root的当前名称。
- repository先复制所选AgentStorage prefix，再在隐藏staging中追加target-only settings change point：`[fork] <所选Agent标题>`，最后原子发布新Session。
- 所选child标题为空时使用`Session <新index>`作为base，得到`[fork] Session <新index>`；从child fork后，该child成为新Session的root。
- fork不记录source metadata；`[fork]`标题不是canonical默认名，因此live和冷打开都不启动自动生成。重复fork自然重复添加`[fork]`。
- fork创建后的名称仍是普通settings change point；checkout或revert到它之前时，名称按复制来的settings timeline恢复。
- 主response取消不取消已启动的标题任务；rename可best-effort取消，但条件提交仍是正确性边界。
- root runtime close先cancel-and-join标题任务再释放node lease；archive/delete随后使用Session topology lease。

## 模块边界

- title-generator contract只定义纯生成操作，不依赖AgentState、AgentStorage、UI或Session repository。
- OpenAI实现负责prompt、schema、SSE收集、JSON/plain-text兼容和结果清洗。
- root Agent host/coordinator拥有内存one-shot gate、live attempt和`SupervisorJob`，但不直接写storage。
- AgentState提供manual rename和compare-and-set generated rename的串行写语义；通用`updateSettings`不能用陈旧整份settings提交标题。
- Session repository只负责create/fork时的初始名称、列表投影和缓存失效，不成为第二个title authority。
- 普通`OpenAiClient.createResponse`足以承载请求；辅助请求不伪造AgentState window或污染主response chain。

## 请求契约

- 全局配置为`enabled`和nullable model override；默认开启，effective model为`gpt-5.4-mini`。
- 输入只包含触发生成的用户文本，不包含history、cwd、context prefix、AGENTS、skills或tools。
- 输入按Unicode字符截断到2,000；请求使用low reasoning、`store=false`、空tools和独立structured output。
- 输出schema为`{"title": string}`，目标长度18到36字符，并要求使用用户语言。
- 清洗只取首个非空行，移除`Title:`、外围引号和尾部标点，折叠空白，拒绝过短结果并截断过长结果。
- 生成失败、incomplete、无completed或非法结果只记录诊断并保留当前标题，不影响主turn和UI通知。
- feature不做生成级重试，Session重开后也不补生成；OpenAI client当前请求自身的传输重试仍生效。

## 验证计划

- generator覆盖prompt、2,000字符边界、structured output、纯文本fallback、Unicode和18到36字符清洗。
- coordinator覆盖默认非空标题、首条文本、图片后首条文本、禁用配置、内存CAS、one-shot、失败后resume不重试和主响应取消。
- writer竞态覆盖manual-before-generated、同名manual rename、generated-before-manual、response in-flight排队、并发settings更新和stale attempt。
- timeline覆盖名称checkout/revert、旧分支attempt失效、settings-only提交不改变timestamp。
- fork覆盖root/child boundary、空child fallback、`[fork]`前缀、重复fork、staging发布和不生成新标题。
- 迁移覆盖旧空/非空root的所有settings change point，并验证每个可达root快照名称非空。
- repository和UI覆盖冷列表只读root latest settings/timestamp、缓存失效及异步标题刷新。
