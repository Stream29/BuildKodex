# Task Tree

- [done] 实现 User Message 右键 Revert and edit
  - [done] 核实既有回退边界与主编辑框能力
  - [done] 确认用户交互与首条消息语义
  - [done] 确认移除 AgentHistoryTarget 的合同调整方向
  - [done] 核实回退、Fork、确认状态和菜单引用
  - [done] 确定直接索引合同与回退完成语义
  - [done] 完成具体修改与验收计划
  - [done] 移除 Target 并调整回退与 Fork 合同
  - [done] 接入菜单、成功回填和编辑框焦点
  - [done] 补充边界、失败与交互测试
  - [done] 执行模块测试与实际 CLI 验证
  - [done] 更新持久约束和清理测试环境

# Details

- 授权：用户已要求无阻塞即直接实现；不创建 Git commit。
- 用户要求：User Message 右键菜单在 Revert to here、Fork from here 之外增加 Revert and edit；回到该消息之前，并将该消息内容放入当前主编辑框。
- 已按用户决定移除 AgentHistoryTarget，完成必要的应用层合同调整。
- 不改底层 runtime、storage 协议。
- 并行改动：保留相关 view 已有的菜单时间戳与布局改动，不修改其任务记录。

## 已确认交互

- 仅纯文本 User Message 提供动作；含图片消息不提供，不扩展附件编辑能力。
- 点击直接回退并替换主编辑框草稿，不弹确认，不自动发送。
- 对选中消息 index `i` 使用 exclusive boundary `i`，删除该消息及之后的历史。
- 首条消息也支持，回退后历史为空；不是删除 Session 或清除初始化数据。
- 不寻找前驱消息，不使用上一条 User Message、上一轮或可见行位置作为边界。
- 撤回“无前驱时禁用”建议，用户未接受该限制。

## 实现结果

- 移除 AgentHistoryTarget，不以新 wrapper、before/after 枚举或特殊 index 哨兵替代。
- 操作直接接收 `untilExclusive: Int`；菜单决定边界：Revert to here 和 Fork from here 传 `i + 1`，Revert and edit 传 `i`。
- 区分被选中的消息 index 与截断边界；不要求边界或 `boundary - 1` 必须对应已物化 Message。
- 保留 owner、idle、有效存储范围与陈旧历史校验；expectedGeneration 独立传给操作，不再用 target 对象绑定边界。
- 原 Revert 保留确认，新动作直接执行；确认交互不应成为调用底层回退能力的必经路径。
- 执行方案：`revertHistory(untilExclusive, expectedGeneration)` 挂起至成功，失败传播异常；接受后的操作仍由 Agent scope 持有。
- 原确认状态继续由 Agent 持有，但只记录 generation 与 boundary；确认和直接调用共用同一执行入口。
- 前端在 application screen scope 等待结果并更新原 Agent composer；失败不改草稿，切换 Session 不写入另一 Agent，不抢另一页面焦点。
- 回退完成前不能把“已启动”当成“成功”；不通过解析通知文字或猜测 generation 变化关联结果。

## 验证结果

- JVM 测试通过：`:app-viewmodel-session:jvmTest`、`:app-view-application:jvmTest`、`:app-viewmodel-application:jvmTest`。
- 新增测试覆盖文本过滤、三项菜单路由、首条与稀疏边界、非法范围、陈旧 generation、foreign Agent、等待取消后继续回退，
  以及成功后替换原 Agent 草稿、失败不改草稿。
- 编译通过：`:app-cli:linkDebugExecutableLinuxX64`；未运行其他平台构建。
- 实际 Linux CLI：使用独立临时 Home 和两个种子 Session，无用户认证、无模型请求。
  index 8 回退后仅保留 2/4；多行中文、两端空白和草稿替换正确，追加字符验证焦点及末尾光标；
  首条 index 2 回退后历史为空且 settings/0 保留，无确认、无自动提交。
- 实际菜单：含图片 User、Developer、Assistant 仅显示原两项；原 Revert 仍弹确认且可取消；Fork 创建截断副本并保留源历史。
- IDEA 对主菜单与 AgentRuntimeViewModel 的错误检查通过；`git diff --check` 通过。
- 持久约束更新于 [cli-view-model-state.md](../../checklist/cli-view-model-state.md#history窗口失效与重载)
  和 [cli-session-view-models.md](../../checklist/cli-session-view-models.md#per-session-viewmodel)。
- 独立 CLI 已退出，临时 Home 已清理；未修改用户 Session，未创建 Git commit。
