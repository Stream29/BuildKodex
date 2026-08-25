# Task Tree

- [done] 稳定 Session catalog 加载
  - [done] 核对 timeline 指针与根租约语义
  - [done] 将根 Session repository 挂到 Session ViewModel scope
  - [done] 将 Catalog repository 挂到 Catalog ViewModel scope
  - [done] 为 timeline 暴露无扫描指针读取
  - [done] 为根 Session metadata 增加锁感知恢复
  - [done] 覆盖活跃租约只扫描不修复
  - [done] 覆盖空闲 Session 扫描并修复
  - [done] 覆盖关闭 Session 后释放 repository 与租约
  - [done] 覆盖切换 tab 不释放租约
  - [done] 运行 JVM 与 Native 定向测试
  - [done] 运行 IDEA 检查与格式化
  - [done] 记录可复用的一致性决策

# Details

- `latest.json` 悬空可能来自正在发布数字记录的活跃 writer，也可能来自已崩溃 writer。
- Catalog 仅在快速指针失效时尝试取得 Session 根租约。
- 取得租约后重新确认指针，并对仍失效的 metadata timeline 扫描、修复。
- 租约已被持有时只扫描数字记录，不修改 `latest.json`。
- 只处理 catalog 读取的 root `settings` 与 `timestamp` timeline；完整 Session 打开仍负责对账六条 timeline。
- 快速路径不取得租约，也不枚举 timeline。
- 修复期间持有根租约，避免“确认空闲”与覆盖指针之间再次出现 writer。
- Catalog 修复返回前必须确认临时租约的 heartbeat 文件已释放。
- `KodexSessionRepository` 协议不接收 owner scope；每个 Session ViewModel 与 Catalog ViewModel 分别创建自身 scope 的 repository 子级。
- 根 Session、runtime、subagents 与 lease 自然成为 Session repository 的后代。
- Catalog 恢复 lease 自然成为 Catalog repository 的后代；显式释放是正常路径，关闭 ViewModel 是取消后备。
- 切换 tab 只改变选择状态，不销毁 Session ViewModel 或其资源树。

## 完成核对

- 用户于 2026-08-25 确认任务实际已经完成，仅遗漏看板归档。
- 实现与回归测试可追溯到 `Kodex` commit `5eaa0cd0`；本次核对时相关源码和测试文件均无未提交改动。
- 本次使用 OpenJDK 26.0.2 复验 `:app-viewmodel-session:jvmTest` 与
  `:app-viewmodel-session:linuxX64Test`：通过。Gradle 构建成功，但因既有 888 个 configuration-cache 问题丢弃缓存。
- IDEA 检查与格式化按用户确认的原完成状态收尾；本次复验时 IDEA 未运行，MCP 无法连接，未重复执行。
  相关文件的 `git diff --check` 通过。
- 可复用的一致性决策已写入 `checklist/cli-session-view-models.md`。
