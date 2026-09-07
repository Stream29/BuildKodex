# Task Tree

- [done] 修复上一批任务暴露的测试失败与清理死锁
  - [done] 复现历史快照竞态与 lease Job 自等待
  - [done] 显式等待 lease Job，修正历史测试等待条件
  - [done] 修正全量暴露的 fork 与导航测试前提
  - [done] 全量验证 history View 与 filesystem Session

# Details

- 最终 JVM 全量：history View 46 项、filesystem Session 24 项，均无失败或跳过。
- 清空两模块测试结果并关闭 build cache 后，再完整执行两轮，均通过；
  1000 项历史往返约 6.2 / 6.9 秒，12 行 viewport 的窗口峰值均为 9 项。
- filesystem Linux x64 编译通过；没有原生运行测试、CLI 重链接或用户 tmux 验收。
- 保留已有工作树改动；无提交、发布或安装替换。

- 用户授权修复已诊断的两类问题；不改变历史展开产品行为。
- `useAndRelease` 在 `NonCancellable` 内隐式取到当前 Job，需显式选择 lease Job。
- 两组历史测试等待目标行展开，而非把任意一次绘制当成点击完成。
- 复用既有 repair 成功、取消释放和 UI 展开断言进行回归；不放宽断言。
- 上一轮诊断：原始两组测试有 9 项失败，隔离等待下一帧后 31 项通过；
  lease 最小实验确认自等待。正式修复后仍须执行两个模块全量测试。
- 不提交、不发布、不操作用户运行中的 Session。
- 修改范围：filesystem Repository 的清理函数，以及两个 history 测试文件的
  点击辅助函数；以有限超时等待被点击行显示展开标记，保留原内容断言。
- 验证：`:app-view-history:jvmTest`、`:agent-session-filesystem:jvmTest`；
  修复通过后编译 filesystem 的 Linux 目标。当前没有项目 IDE 或运行中 Gradle
  Daemon，沿用上轮本机 JVM 路径启动，不切换设备或另选 JDK。
- 首次全量：原 9 项展开断言全部通过，filesystem 不再停滞；
  新暴露 fork 期待 index 0（初始化仅写 settings/timestamp/tokenCount），
  以及 bounded-window 测试未表达退出 follow-latest 的用户滚动意图。
- history 单独全量重跑通过，说明导航失败存在时序性；修正测试的交互前提，
  不降低原窗口容量、导航结果或时限断言。
- 导航实验进一步确认固定循环次数不等于已完成翻页次数（窗口可能被自动加载
  更新）；改为按 hasOlder/hasNewer 边界结束，保留 15 秒期限，新增确实到达
  最旧端的断言。初次全量通过：1000 项往返约 7.2 秒，12 行窗口峰值 9 项。
- fork 测试比较源与目标的稀疏 index 集合，并在断言失败时也关闭 Repository。
