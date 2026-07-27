# Task Tree

- [done] 对齐 Mosaic modifier node chain 与 Compose
  - [done] 核对 Compose NodeChain、Myers diff 与 Mosaic 现有生命周期
  - [done] 引入适配后的 Myers diff
  - [done] 用平行 element/node 向量重构节点链
  - [done] 接入 MosaicNode 的 modifier layer 构建
  - [done] 补齐结构差分、复用和生命周期测试
  - [done] 运行格式、JVM、Linux Native、ABI 与 IDE 检查

# Details

- 保留 Mosaic 现有 `Modifier` 公共 API。
- 旧式 modifier 在 node 向量中的对应位置以 `null` 表示。
- 差分算法和复用规则基于 AndroidX Compose 源码适配，不引入 Compose UI 运行时依赖。
- 不提交 Git commit。
- Mosaic runtime、CLI components 的 JVM/Linux X64 测试及 ABI 检查通过。
- IDE 文件检查无警告；IDE 构建动作受复合构建中的 `Mosaic` 项目名歧义阻塞。
