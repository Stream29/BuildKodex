# Task Tree

- [done] 以clean event实现Remote Compaction V2保留前缀
  - [done] 确认保留类型
    - [done] 保留UserMessage
    - [done] 保留PlanUpdate
    - [done] 保留RequestUserInput
  - [done] 定义clean compaction模型
    - [done] 定义RemoteCompactionV2RetainedItem
    - [done] 将encrypted compaction建模为Stable ContextCompaction
    - [done] 将checkpoint prefix改为retained item列表
  - [done] 从原始StableCleanEvent构建前缀
    - [done] 直接筛选三类retained item
    - [done] 复用现有窗口预算与淘汰语义
    - [done] 仅允许边界UserMessage截断
  - [done] 简化model input投影
    - [done] 按prefix加stable events投影
    - [done] 将compaction边界纳入stable范围
    - [done] 排除全部unstable clean event
  - [done] 更新模型、存储与行为测试
  - [done] 运行JVM、Linux与JS验证
  - [done] 交付本机历史数据迁移
    - [done] 只读预检全部checkpoint
    - [done] 提供严格的一次性迁移脚本
    - [done] 验证fixture与运行进程保护
    - [done] 打包包含新模型的Linux二进制

# Details

- 不调整远端压缩方法、触发条件或token计数语义。
- 存储层只保存clean event模型，不保存原始ResponseItem。
- `prefix`类型为`List<RemoteCompactionV2RetainedItem>`。
- checkpoint只保存retained prefix与窗口谱系；encrypted compaction由同索引的`StableCleanEvent.ContextCompaction`保存。
- model input严格等于checkpoint prefix加`[historyBaseIndex, snapshotIndex]`内的stable events。
- `historyBaseIndex`包含同索引的ContextCompaction；unstable clean event不进入model input。
- 构建前缀时直接处理原始`StableCleanEvent`，不从投影后的`ResponseItem`反向识别。
- PlanUpdate与RequestUserInput均为不可拆分的完整clean event。
- 不增加旧checkpoint schema的运行时兼容或兜底分支。
- `scripts/migrate-compaction-stable-events.py`严格预检、逐文件原子替换并验证迁移结果；检测到运行中的`kodex-cli`时拒绝写入。
- 打包后的只读预检发现764个checkpoint，其中593个需要迁移，结构错误为0；最终数量由脚本在停机后重新扫描。
- 交付包为`Kodex/out/kodex-clean-compaction-linux-x64.tar.gz`，包含迁移脚本与新版`kodex-cli`。
- 用户将在所有旧`kodex-cli`停止后执行迁移，再以包内新版二进制重启。
