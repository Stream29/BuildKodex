# Task Tree

- [done] 调查 filesystem index 查询重复目录扫描
  - [done] 核对 `storedIndexes` 实现与调用路径
  - [done] 核对查询级扫描次数与对象分配
  - [done] 复测会话 132 的目录规模与耗时
  - [done] 判断其与 GC CPU 的因果强度

# Details

- 未修改实现。IntelliJ IDEA 正在打开根项目。
- Direct filesystem 实现每次调用 `storedIndexes()` 都会枚举、解析并排序目录。
- CLI 打开 Session 时会先为六条 timeline 各扫描一次，再由
  `CachedIndexVersioned` 在内存中维护有序索引。普通 `floorToIndex()`、
  `ceilToIndex()`、`indexesIn()` 和 `valuesIn()` 不再扫描目录。
- Session 132 的 `work`、`timestamp` 和 `unstable` 分别约有 52k、103k 和
  49k 条记录。`uv` Python 等价扫描代理的中位耗时为 95.7 ms、192.6 ms 和
  92.3 ms，峰值追踪分配约为 7.0 MiB、14.4 MiB 和 7.0 MiB。
- 六条 timeline 的扫描代理中位耗时合计约 422 ms。这是 Session
  打开或重新打开时的真实优化机会，但不是已打开 Session 的逐查询持续成本。
- Catalog 在有效 `latest.json` 存在时不枚举 timeline；Session 132
  的六条 pointer 均有效。
- Cached range 查询仍会复制命中的 index 列表，`valuesIn()` 还会创建 Pair
  并读取或解码未缓存值，因此仍可能产生范围大小相关的 CPU/GC，但不产生报告所述的
  `Path`、文件名解析和排序成本。
- 现有持久说明见
  `shared-context/findings/agent-storage-compensation-semantics.md:26-35`。
