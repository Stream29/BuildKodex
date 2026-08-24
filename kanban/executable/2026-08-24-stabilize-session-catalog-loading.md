# Task Tree

- 稳定 Session catalog 加载
  - [done] 核对 timeline 指针与根租约语义
  - 将根 Session repository 挂到 Session ViewModel scope
  - 将 Catalog repository 挂到 Catalog ViewModel scope
  - [done] 为 timeline 暴露无扫描指针读取
  - [done] 为根 Session metadata 增加锁感知恢复
  - [done] 覆盖活跃租约只扫描不修复
  - [done] 覆盖空闲 Session 扫描并修复
  - 覆盖关闭 Session 后释放 repository 与租约
  - [done] 覆盖切换 tab 不释放租约
  - 运行 JVM 与 Native 定向测试
  - 运行 IDEA 检查与格式化
  - 记录可复用的一致性决策

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
